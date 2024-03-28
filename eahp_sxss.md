## Emergency Ambulance Hiring Portal 

**Bug Description:**

An unauthenticated stored cross-site scripting (XSS) vulnerability in PHPGurukul Emergency Ambulance Hiring Portal 1.0 allows attackers to execute arbitrary web scripts via a crafted payload injected in the "Hire an Ambulance" functionality.


Steps to Reproduce:

```
# Exploit Title: Stored XSS in "Hire an Ambulance" functionality of Emergency Ambulance Hiring Portal 
# Date: 28-03-2024
# Exploit Author: dhabaleshwardas
# Vendor Homepage: https://phpgurukul.com/
# Software Link: https://phpgurukul.com/emergency-ambulance-hiring-portal-using-php-and-mysql/
# Version: 1.0
# Tested on: firefox/chrome/brave
# CVE:
```


To reproduce the attack:

1- Head to the http://localhost/eahp/index.php endpoint . Then click on "Hire an Ambulance".

<img width="745" alt="1" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/82771142-e11a-45fb-adfe-d4f29fe584b3">


2- Here you would be asked to fill all the fields. We simply put XSS payloads in all the fields and clicked "Submit".

<img width="736" alt="2" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/65ec7d74-0ab0-469b-ab9c-399f7e06b25d">


3- Login to the Admin account and you'll see a new request notification on the top left corner, click on it, and all the payloads would be executed showing pop-ups after pop-ups.

<img width="957" alt="4" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/5c6b9bef-baa7-4b1c-a964-1e5a180775df">

<img width="498" alt="5" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/614f16e9-daf2-4a9b-b746-49da576e3bc9">



3- We can see that the payloads are directly embedded into the HTML content without proper sanitization or encoding, and hence, pop-ups are shown.

<img width="557" alt="6" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/0465274b-9b18-4c8e-a461-f94d294ea67e">

