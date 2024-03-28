## Emergency Ambulance Hiring Portal 

**Bug Description:**

An unauthenticated reflected cross-site scripting (XSS) vulnerability in PHPGurukul Emergency Ambulance Hiring Portal 1.0 allows attackers to execute arbitrary web scripts via a crafted payload injected into the "searchdata" field.


Steps to Reproduce:

```
# Exploit Title: Reflected XSS in "searchdata" parameter of Emergency Ambulance Hiring Portal 
# Date: 28-03-2024
# Exploit Author: dhabaleshwardas
# Vendor Homepage: https://phpgurukul.com/
# Software Link: https://phpgurukul.com/emergency-ambulance-hiring-portal-using-php-and-mysql/
# Version: 1.0
# Tested on: firefox/chrome/brave
# CVE:
```


To reproduce the attack:

1- Head to the http://localhost/eahp/ambulance-tracking.php endpoint .



2- Here you would be asked to give a value in the "searchdata" parameter. We simply put an XSS payload <script>alert(5)</script>.

<img width="701" alt="1" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/3c2e9f59-1ebd-4e1d-ac24-7247d622c8fb">



3- As soon as you hit "Search" intercept the request and then check the response, you can see the payload  directly embedded into the HTML content without proper sanitization or encoding, and hence, a pop-up is shown with the number "5".

<img width="519" alt="2" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/25f853ad-fd9c-4f68-b3af-ff1c33ffae33">

<img width="469" alt="3" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/5ebb89cc-9152-4b76-b38f-bf08e3f47153">



4- Although Reflected XSS is not as critical as Stored XSS but still it can be used to steal user session cookies, allowing the attacker to impersonate the victim and perform actions on their behalf and can even redirect users to malicious websites. 

**Remediation:** 

1- Validate and sanitize user input on the server side. Ensure that input adheres to expected patterns and formats.

2- Encode user input before displaying it in the HTML output. HTML-encode special characters to prevent them from being interpreted as HTML or JavaScript.
