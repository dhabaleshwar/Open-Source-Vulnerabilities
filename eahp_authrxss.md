## Emergency Ambulance Hiring Portal 

**Bug Description:**

A reflected cross-site scripting (XSS) vulnerability in PHPGurukul Emergency Ambulance Hiring Portal 1.0 allows attackers to execute arbitrary web scripts via a crafted payload injected into the "search request" field.


Steps to Reproduce:

```
# Exploit Title: Reflected XSS in "Search Request" functionality of Emergency Ambulance Hiring Portal 
# Date: 28-03-2024
# Exploit Author: dhabaleshwardas
# Vendor Homepage: https://phpgurukul.com/
# Software Link: https://phpgurukul.com/emergency-ambulance-hiring-portal-using-php-and-mysql/
# Version: 1.0
# Tested on: firefox/chrome/brave
# CVE:
```


To reproduce the attack:

1- Login to the application and then head to the http://localhost/eahp/admin/search.php endpoint .



2- Here you would be asked to give a value in the "search by request number" parameter. We simply put an XSS payload "><script>alert(2)</script>.

<img width="960" alt="rxss" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/ea65ef06-7673-4550-a386-9b37879743e8">




3- As soon as you hit "Search" intercept the request and then check the response, you can see the payload  directly embedded into the HTML content without proper sanitization or encoding, and hence, a pop-up is shown with the number "2".

<img width="716" alt="rxss1" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/42e2f845-b979-4914-a97d-ef826875fb80">



4- Although Reflected XSS is not as critical as Stored XSS but still it can be used to steal user session cookies, allowing the attacker to impersonate the victim and perform actions on their behalf and can even redirect users to malicious websites. 

**Remediation:** 

1- Validate and sanitize user input on the server side. Ensure that input adheres to expected patterns and formats.

2- Encode user input before displaying it in the HTML output. HTML-encode special characters to prevent them from being interpreted as HTML or JavaScript.
