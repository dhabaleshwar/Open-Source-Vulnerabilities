## Online Notes Sharing System


**Bug Description:**

A Stored cross-site scripting (XSS) vulnerability in Online Notes Sharing System  1.0 allows attackers to execute arbitrary web scripts via a crafted payload injected into the "Name" and "Email" field.

**Steps to Reproduce:** 

```
# Exploit Title: Stored cross-site scripting (XSS) vulnerability in Online Notes Sharing System
# Date: 20-12-2023
# Exploit Author: dhabaleshwardas
# Vendor Homepage: https://phpgurukul.com/
# Software Link: https://phpgurukul.com/online-notes-sharing-system-using-php-and-mysql/
# Version: 1.0
# Tested on: firefox/chrome/brave
# CVE : 
```


To reproduce the attack:

1- First login to the application then head to the http://localhost/onss/user/profile.php endpoint 

2- Then click on "Update" and capture the request in Burp Intercept.

<img width="655" alt="1" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/125689be-10de-4c3f-82a9-c92f675b437c">


3- Here, we will change the value of "Full Name" and "Email" parameter to payloads for XSS. In the "name" parameter we put "><script>alert(2)</script> and in the "email" parameter we put "><script>alert(document.cookie)</script> the rest of
parameter we leave as it is.

<img width="887" alt="2" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/3061a825-767c-48d2-9b4e-6105ae912fef">


4- We then forward the request. We see in the browser that the payloads got executed, first the payload in the  "name" parameter got executed displaying a "2" and then the payload in the "email" parameter got executed displaying the Cookie in the popup.

<img width="724" alt="3" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/1789dec9-fb08-4d9b-9200-eebc0768387c">

<img width="736" alt="4" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/c32b8d37-a6ca-4bfa-9851-664620c8fc0b">


5- This shows us that, the user input is directly embedded into the HTML content without proper sanitization or encoding. The strings "><script>alert(2)</script> and  "><script>alert(document.cookie)</script> is rendered as part of the HTML, making it vulnerable to script injection.



6- Stored XSS is a high severity vulnerability as, Attackers can steal sensitive information, such as login credentials, session tokens, or personal details, from users who unknowingly execute the malicious script. If a user with administrative privileges is affected, attackers can hijack their session, gaining unauthorized access to sensitive areas of a website or application.

**Remediation:** 

1-  Implement strict input validation on both the client and server sides. Validate and sanitize user input to ensure that it does not contain malicious code.

2- Encode user-generated content before rendering it in the browser. This helps to neutralize any malicious scripts and ensures that user input is treated as data, not executable code.
