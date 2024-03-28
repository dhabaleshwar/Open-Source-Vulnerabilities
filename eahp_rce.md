## Emergency Ambulance Hiring Portal 

**Bug Description:**

A vulnerability in the Emergency Ambulance Hiring Portal 1.0 allows an unauthenticated attacker to execute code on the server by exploiting SQL injection and escalating it to remote code execution.

Steps to Reproduce:

```
# Exploit Title: Remote Code Execution in "searchdata" parameter of Emergency Ambulance Hiring Portal 
# Date: 28-03-2024
# Exploit Author: dhabaleshwardas
# Vendor Homepage: https://phpgurukul.com/
# Software Link: https://phpgurukul.com/emergency-ambulance-hiring-portal-using-php-and-mysql/
# Version: 1.0
# Tested on: firefox/chrome/brave
# CVE:
```

To exploit the vulnerability:

1- First visit this endpoint http://localhost/eahp/ambulance-tracking.php 

<img width="801" alt="0" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/9a0cb932-710f-42fb-a09b-ebfcc14b8893">


2- Then write any random data in the "searchdata" parameter and intercept the request. Save the request in your local machine, then use the command below for sqlmap.

<img width="947" alt="1" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/a8189ac2-f946-4311-a842-1a0b2909ed29">


3- The screenshot below shows that the parameter is vulnerable to SQLi and thus we opened up a shell to execute system commands causing Remote Code Execution.

<img width="947" alt="2" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/3eca1172-9530-49c9-a40b-20c3bbeb3015">



