---
title: Advisories
icon: fas fa-bullhorn
order: 5
---

## Security Advisories

- **CVE-2024-25712**

In versions of the swaggo/http-swagger library below v1.2.5, proper HTTP method validation is not enforced. As a result, the handler 'httpSwagger.WrapHandler' and the 'PUT' request can be used to upload a file to memory through *webdav.memFile. Subsequently, this file can be accessed using the GET method. An attacker could exploit this by uploading an HTML file containing malicious JavaScript to memory, making it accessible to other users.

- **CVE-2023-42282**

In the code snippet of library, a security vulnerability arises due to the ip.isPublic function's incorrect identification of the IP address 0x7f.1 as public. This address is actually a hexadecimal representation of the private IP 127.0.0.1. This misclassification can lead to potential Server-Side Request Forgery (SSRF) attacks, as the code may unintentionally permit HTTP requests to internal network resources, creating a significant security risk. The core issue is the function's failure to accurately distinguish between public and private IP addresses.

- **CVE-2023-1496**

SVG Sanitization Bypass Leads to Reflected Cross-site Scripting (XSS) in GitHub repository imgproxy/imgproxy prior to 3.14.0. Full story [here](https://huntr.com/bounties/de603972-935a-401a-96fb-17ddadd282b2/).

- **CVE-2021-4118**

There is untrusted YAML Deserialization vulnerability on PyTorchLightning Github repository. PyTorchLightning's saving.py (core.saving.load_hparams_from_yaml) functionality is calling "yaml.UnsafeLoader" from pyyaml Python library which is not secure method. Because of that, maliciously crafted yaml config file can cause code execution on the victim's machine.

- **CVE-2021-28855**

In Deark before 1.5.8, a specially crafted input file can cause a NULL pointer dereference in the dbuf_write function (src/deark-dbuf.c).

- **CVE-2021-28856**

In Deark before v1.5.8, a specially crafted input file can cause a division by zero in (src/fmtutil.c) because of the value of pixelsize.

- **CVE-2021-28060**

A Server-Side Request Forgery (SSRF) vulnerability in Group Office 6.4.196 allows a remote attacker to forge GET requests to arbitrary URLs via the url parameter to group/api/upload.php.

- **CVE-2020-35419**

Cross Site Scripting (XSS) in Group Office CRM 6.4.196 via the SET_LANGUAGE parameter.

- **CVE-2020-35418**

Cross Site Scripting (XSS) in the contact page of Group Office CRM 6.4.196 by uploading a crafted svg file.

- **CVE-2020-25538**

An authenticated attacker can inject malicious code into "lang" parameter in /uno/central.php file in CMSuno 1.6.2 and run this PHP code in the web page. In this way, attacker can takeover the control of the server.

- **CVE-2020-25557**

In CMSuno 1.6.2, an attacker can inject malicious PHP code as a "username" while changing his/her username & password. After that, when attacker logs in to the application, attacker's code will be run. As a result of this vulnerability, authenticated user can run command on the server.

- **CVE-2020-26803**

In Sentrifugo 3.2, users can upload an image under "Assets -> Add" tab. This "Upload Images" functionality is suffered from "Unrestricted File Upload" vulnerability so attacker can upload malicious files using this functionality and control the server.

- **CVE-2020-26804**

In Sentrifugo 3.2, users can share an announcement under "Organization -> Announcements" tab. Also, in this page, users can upload attachments with the shared announcements. This "Upload Attachment" functionality is suffered from "Unrestricted File Upload" vulnerability so attacker can upload malicious files using this functionality and control the server.

- **CVE-2020-26805**

In Sentrifugo 3.2, admin can edit employee's informations via this endpoint --> /sentrifugo/index.php/empadditionaldetails/edit/userid/2. In this POST request, "employeeNumId" parameter is affected by SQLi vulnerability. Attacker can inject SQL commands into query, read data from database or write data into the database.

- **CVE-2020-11827**

In GOG Galaxy 1.2.67, there is a service that is vulnerable to weak file/service permissions: GalaxyClientService.exe. An attacker can put malicious code in a Trojan horse GalaxyClientService.exe. After that, the attacker can re-start this service as an unprivileged user to escalate his/her privileges and run commands on the machine with SYSTEM rights.

- **CVE-2020-2909**

Vulnerability in the Oracle VM VirtualBox product of Oracle Virtualization (component: Core). Supported versions that are affected are Prior to 5.2.40, prior to 6.0.20 and prior to 6.1.6. Easily exploitable vulnerability allows low privileged attacker with logon to the infrastructure where Oracle VM VirtualBox executes to compromise Oracle VM VirtualBox. Successful attacks require human interaction from a person other than the attacker. Successful attacks of this vulnerability can result in unauthorized ability to cause a partial denial of service (partial DOS) of Oracle VM VirtualBox.

- **CVE-2020-11819**

In Rukovoditel 2.5.2, an attacker may inject an arbitrary .php file location instead of a language file and thus achieve command execution.

- **CVE-2020-11817**

In Rukovoditel V2.5.2, attackers can upload an arbitrary file to the server just changing the the content-type value. As a result of that, an attacker can execute a command on the server. This specific attack only occurs with the Maintenance Mode setting.

- **CVE-2020-11811**

In qdPM 9.1, an attacker can upload a malicious .php file to the server by exploiting the Add Profile Photo capability with a crafted content-type value. After that, the attacker can execute an arbitrary command on the server using this malicious file.

- **CVE-2020-11815**

In Rukovoditel 2.5.2, attackers can upload arbitrary file to the server by just changing the content-type value. As a result of that, an attacker can execute a command on the server. This specific attack only occurs without the Maintenance Mode setting.

- **CVE-2020-11816**

Rukovoditel 2.5.2 is affected by a SQL injection vulnerability because of improper handling of the reports_id (POST) parameter.

- **CVE-2020-11812**

Rukovoditel 2.5.2 is affected by a SQL injection vulnerability because of improper handling of the filters[0][value] or filters[1][value] parameter.

- **CVE-2020-11820**

Rukovoditel 2.5.2 is affected by a SQL injection vulnerability because of improper handling of the entities_id parameter.

- **CVE-2020-11826**

Users can lock their notes with a password in Memono version 3.8. Thus, users needs to know a password to read notes. However, these notes are stored in a database without encryption and an attacker can read the password-protected notes without having the password. Notes are stored in the ZENTITY table in the memono.sqlite database.

- **CVE-2020-11818**

In Rukovoditel 2.5.2 has a form_session_token value to prevent CSRF attacks. This protection mechanism can be bypassed with another user's valid token. Thus, an attacker can change the Admin password by using a CSRF attack and escalate his/her privileges.

- **CVE-2019-20074**

On Netis DL4323 devices, any user role can view sensitive information, such as a user password or the FTP password, via the form2saveConf.cgi page.