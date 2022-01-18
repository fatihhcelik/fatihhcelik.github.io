---
title: Deserialization of Untrusted Data in pytorch-lightning
author: Fatih Çelik
date: 2021-12-12 11:34:00 +0800
categories: [Vulnerability Research]
tags: [vulnerability research]
math: true
mermaid: true
---

**Software**: [https://github.com/pytorchlightning/pytorch-lightning)

**Vulnerability**: Insecure Yaml Deserialization

**CVE**: CVE-2021-4118

**Description of the product:**

> Lightning disentangles PyTorch code to decouple the science from the engineering.

**Summary:**

There is untrusted YAML Deserialization vulnerability on PyTorchLightning Github repository. PyTorchLightning's saving.py (core.saving.load_hparams_from_yaml) functionality is calling "yaml.UnsafeLoader" from pyyaml Python library which is not secure method. Because of that, maliciously crafted yaml config file can cause code execution on the victim's machine.

**Fix:**

[Github PR](https://github.com/pytorchlightning/pytorch-lightning/commit/62f1e82e032eb16565e676d39e0db0cac7e34ace)
