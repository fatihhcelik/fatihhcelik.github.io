---
title: Missing IP Address Control in isPublic() Function Leads to SSRF Bypass PoC
discoverer: Fatih Çelik & Emre Durmaz
date: 2024-03-01 11:34:00 +0800
categories: [Vulnerability Research]
tags: [vulnerability research]
math: true
mermaid: true
---

**Software**: [NPM - Ip Package](https://www.npmjs.com/package/ip)

**Vulnerability**: Missing IP Address Control
**CVE**: CVE-2023-42282

**Description of the product:**

> IP address utilities for node.js Get your ip address, compare ip addresses, validate ip addresses, etc.

**Description:**

In the code snippet of library, a security vulnerability arises due to the ip.isPublic function's incorrect identification of the IP address 0x7f.1 as public. This address is actually a hexadecimal representation of the private IP 127.0.0.1.  This misclassification can lead to potential Server-Side Request Forgery (SSRF) attacks, as the code may unintentionally permit HTTP requests to internal network resources, creating a significant security risk. The core issue is the function's failure to accurately distinguish between public and private IP addresses.

**Root Cause:**

The root cause of this vulnerability is the ip.isPublic function's inability to correctly interpret and classify certain IP address representations.  Specifically, the function fails to recognize 0x7f.1 as a private IP address, which is actually a hexadecimal and shorthand notation for 127.0.0.1, a well-known loopback address. This misinterpretation leads to the function erroneously categorizing a private IP as public, thereby opening up the potential for Server-Side Request Forgery (SSRF) attacks. The core issue lies in the inadequate handling of non-standard IP address formats by the IP address validation mechanism.

**Proof of Concept:**

```js
var ip = require('ip'); 
console.log(ip.isPublic("0x7f.1")); // true
 
// Check IP and IF IP is not public do HTTP req ==> SSRF 
var my_ip = "0x7f.1"; // private but ip package doesn't recognize as private!
  
// var my_ip2 = "google.com"; // public --> fetch will be completed     
// var my_ip3 = "127.0.0.1"; // private --> else statement will be run
   
if (ip.isPublic(my_ip)) {         
  fetch("http://" + my_ip).then((response) => {            
  console.log(response);});    
} else {       
  console.log("IP is not public");    
}
```

**Impact:**

The impact of this vulnerability is significant, as it can lead to Server-Side Request Forgery (SSRF) attacks. SSRF attacks occur when an attacker is able to induce a server to make requests to internal resources, which the attacker normally couldn't access directly. In this case, the misclassification of 0x7f.1 as a public IP address allows for unintended internal network requests. This could potentially expose sensitive information, interact with internal services, or exploit vulnerabilities within the network. Additionally, such vulnerabilities can be leveraged to perform port scanning, gain unauthorized access, or as part of a larger attack chain. The risk is heightened in environments where the server has access to critical internal systems.

**Disclosure Timeline**

* 14 December 2022 - First Contact  ([via huntr](https://huntr.com/bounties/bfc3b23f-ddc0-4ee7-afab-223b07115ed3/)):
* 17 January 2023 - Reminder (No Response)
* 28 February 2023 - Reminder (No Response)
* 8 February 2024 - Public Disclosure
