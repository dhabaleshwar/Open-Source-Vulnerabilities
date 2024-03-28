## Emergency Ambulance Hiring Portal 

**Bug Description:**

A stored cross-site scripting (XSS) vulnerability in PHPGurukul Emergency Ambulance Hiring Portal 1.0 allows attackers to execute arbitrary web scripts via a crafted payload injected in the "Add Ambulance" functionality.


Steps to Reproduce:

```
# Exploit Title: Stored XSS in "Add Ambulance" functionality of Emergency Ambulance Hiring Portal 
# Date: 28-03-2024
# Exploit Author: dhabaleshwardas
# Vendor Homepage: https://phpgurukul.com/
# Software Link: https://phpgurukul.com/emergency-ambulance-hiring-portal-using-php-and-mysql/
# Version: 1.0
# Tested on: firefox/chrome/brave
# CVE:
```


To reproduce the attack:

1- First, login to the application then, head to the http://localhost/eahp/admin/add-ambulance.php endpoint . 


2- Here you would be asked to fill all the fields. We simply put XSS payloads in "Ambulance Reg No." and "Driver Name" fields and clicked "Add".

<img width="833" alt="1" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/b816e3f4-5100-4333-bd9b-a1edc3385f79">


3- We can see that the payloads are directly embedded into the HTML content without proper sanitization or encoding, and hence, pop-ups are shown.

<img width="667" alt="2" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/76f3b773-9868-423b-b7bf-f3b36178b062">

