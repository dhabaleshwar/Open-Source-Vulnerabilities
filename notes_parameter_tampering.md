## Online Notes Sharing System

**Bug Description:**

A vulnerability in the Online Notes Sharing System 1.0 allows an attacker to tamper with parameters, specifically the Contact Number, during the user profile update process. While the application restricts the direct modification of the contact number through the browser interface, an attacker can manipulate the request after clicking the "Update" button, leading to unauthorized changes in the user's contact information.

Steps to Reproduce:

```
# Exploit Title: Parameter Tampering in Contact Number in Online Notes Sharing System
# Date: 20-12-2023
# Exploit Author: dhabaleshwardas
# Vendor Homepage: https://phpgurukul.com/
# Software Link: https://phpgurukul.com/online-notes-sharing-system-using-php-and-mysql/
# Version: 1.0
# Tested on: firefox/chrome/brave
# CVE:
```

To exploit the vulnerability:

1- Log in to the application and navigate to the user profile page. View the current contact number displayed in the Contact Number field. Attempt to modify the contact number directly through the browser interface. Note that the application prevents direct modification at this stage.

<img width="960" alt="1" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/d9ebf5e7-85df-4006-8639-85497141a4f5">


2- Click the "Update" button to submit the profile update.

3- Intercept the request using a tool like Burp Suite or the browser's developer tools. Modify the "mobilenumber" parameter in the intercepted request to a new, unauthorized contact number.

<img width="882" alt="2" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/c163461d-2432-4cd8-bab3-d67a39db9ba0">

4- Forward the modified request to the server.

<img width="437" alt="3" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/d93a562c-36a2-493d-a2cc-3722b489036d">


5- The server accepts the tampered request, and the user's mobile number is updated to the unauthorized value.

<img width="960" alt="4" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/4a73220a-382d-45b8-b476-dc6adfeaa052">



**Impact:**

This vulnerability allows an attacker to tamper with parameters during the user profile update process, leading to unauthorized changes in the user's contact information. The impact may include impersonation, social engineering, or unauthorized access to sensitive data associated with the modified contact number.

**Remediation:**

Implement strong server-side validation for all parameters, including the Contact Number, during the update process.
