## Summary:
I have discovered a security issue in the Branda plugin that allows an attacker with Administrator privileges to execute arbitrary JavaScript code within the "search module" section. This vulnerability could potentially lead to Cross-Site Scripting (XSS) attacks.

## Vulnerability Details:

Affected Component: Search Module

Vulnerable Parameter: Search Module

Payload Used for Testing: "><script>alert(2)</script>

Date: 20-12-2023

## Additional Information:
WordPress Version: 6.4.2

Plugin Version: 3.4.15

PHP Version: 8.2.4

## Steps to Reproduce:

1- Log in to the WordPress dashboard with admin account.

2- Then Install "Branda" plugin and activate it.

3- Go to the "Dashboard" under "Branda" section and click on "Search Module" 

4- Input the following payload in the "Search Module" field
"><script>alert(2)</script>

<img width="724" alt="request and response" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/f178837c-54c9-4e87-91ce-9f124f0933a0">


5- Observe the behavior on the frontend, and you can see that the payload has been successfully executed.

## Expected Behavior:
The Branda plugin should sanitize and validate user input to prevent the execution of arbitrary scripts.

