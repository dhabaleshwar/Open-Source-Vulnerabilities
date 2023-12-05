## Teacher Subject Allocation Management System V1.0


**Bug Description:**

An unauthenticated reflected cross-site scripting (XSS) vulnerability in PHPGurukul Teacher Subject Allocation Management System 1.0 allows attackers to execute arbitrary web scripts via a crafted payload injected into the "searchdata" field.

**Steps to Reproduce:** 

```
# Exploit Title: Unauthenticated Reflected cross-site scripting (XSS) vulnerability in PHPGurukul Teacher Subject Allocation Management System
# Date: 05-12-2023
# Exploit Author: dhabaleshwardas
# Vendor Homepage: https://phpgurukul.com/
# Software Link: https://phpgurukul.com/teacher-subject-allocation-system-using-php-and-mysql/
# Version: 1.0
# Tested on: firefox/chrome/brave
# CVE : 
```


To reproduce the attack:

1- Head to the http://localhost/tsas/index.php endpoint 



2- Here you would be asked to give a value in the "searchdata" parameter. We simply put an XSS payload <script>alert(5)</script>.

<img width="889" alt="1" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/59c21c32-4ed1-439a-9f96-43f5d08f8524">



3- As soon as you hit "Go" intercept the request and then check the response, you can see the payload  directly embedded into the HTML content without proper sanitization or encoding, and hence, a pop-up is shown with the number "5".

<img width="685" alt="2" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/7e11692e-348c-4e44-a6fa-7d49f8b40704">


<img width="391" alt="3" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/56c7cd8a-4c70-46bd-bd68-15ee35a75f1a">


<img width="625" alt="4" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/ba780684-255a-4c2f-8a97-0a7ad1ed433c">



- Although Reflected XSS is not as critical as Stored XSS but still it can be used to steal user session cookies, allowing the attacker to impersonate the victim and perform actions on their behalf and can even redirect users to malicious websites. 

**Remediation:** 

1- Validate and sanitize user input on the server side. Ensure that input adheres to expected patterns and formats.

2- Encode user input before displaying it in the HTML output. HTML-encode special characters to prevent them from being interpreted as HTML or JavaScript.
