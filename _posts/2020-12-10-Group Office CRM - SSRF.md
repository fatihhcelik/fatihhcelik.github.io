---
title: Group Office CRM | SSRF
author: Fatih Çelik
date: 2020-12-10 11:33:00 +0800
categories: [Vulnerability Research]
tags: [vulnerability research]
math: true
mermaid: true
---

**Software**: https://sourceforge.net/projects/group-office/

**Version**: 6.4.196

**Vulnerability**: SSRF

**CVE**: CVE-2021-28060

**Description of the product:**

> Group Office is an open source groupware application. It makes your daily office tasks easier. Share projects, calendars, files and e-mail online. It is a complete solution for all your online office needs. From a customer phone call to a project and finally an invoice. The support system helps to keep your customers happy. Group Office is fast, secure and has privacy by design. You can stay in full control of your data by self hosting your cloud and e-mail. Our document editing solution keeps all data on the secured server instead of synchronising it to all user devices. GroupOffice is open source and modular. Which means it’s easy to customise and extend. You can turn off and on features and it enables any developer to create new modules for the platform.

## Description of the vulnerability

A Server-Side Request Forgery (SSRF) vulnerability in the "set image from url" allows a remote attacker to forge GET requests to arbitrary URLs.

![](/photos/crmssrf-1.png)
