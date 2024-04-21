## Authentication Bypass Vulnerability in HP Deskjet Ink Advantage Ultra 4800 Printer

### Vulnerability Description:

An authentication bypass vulnerability has been identified in the web interface of the HP Deskjet Ink Advantage Ultra 4800 printer. This vulnerability allows unauthorized users to gain access to administrative functionalities without providing the correct credentials. By manipulating the response of the authentication process, an attacker can bypass the authentication mechanism and gain unauthorized access to sensitive features of the printer.

### Vulnerability Details:

```
Product: HP Deskjet Ink Advantage Ultra 4800 Printer
Affected Version: 4800
Vulnerability Type: Authentication Bypass via Response Manipulation
CVE: [if applicable]
```
Description: The authentication process in the web interface of the HP Deskjet Ink Advantage Ultra 4800 printer requires users to input a PIN to access administrative functionalities.

Vulnerability: However, it has been discovered that by submitting an incorrect PIN, an attacker can manipulate the response from the server to include the parameter "hasAuth" : true, which bypasses the authentication mechanism, granting unauthorized access to the administrative interface.

### Exploit Steps:

1- After setting up your printer, access the web interface of the HP Deskjet Ink Advantage Ultra 4800 printer. Here we need a pin to signin and access the functionalities of the printer.

<img width="751" alt="1" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/1dc29be6-75f3-40c6-888b-ac5e6078c5d1">


2- Attempt to log in with an incorrect PIN.

3- Then intercept the authentication request using a proxy tool such as Burp Suite.

<img width="608" alt="2" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/7811d5fb-ab03-4e0a-9355-7e45919144ad">


4- As we can see the response is 403 forbidden because we used the wrong pin. 

<img width="634" alt="3" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/133fe58c-57db-4054-b3a4-d563bdf223c7">


5- Now we modify the response by changing the "403 Forbidden" to "200 OK" and adding a parameter:value pair in the response body "hasAuth" : true. 

<img width="620" alt="4" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/6992174a-8700-4731-a640-c4f463af833c">


5- Then forward the manipulated response to the server.

6- We are successfully logged in as "admin". Now we can access all the functionalities as administrator.

<img width="771" alt="5" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/379c6155-5edb-4fa4-be85-3afe33ba7f07">
