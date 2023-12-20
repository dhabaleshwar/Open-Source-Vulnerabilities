## Online Notes Sharing System

**Bug Description:**

A security vulnerability in the Online Notes Sharing System 1.0 exposes users to potential risks by allowing registration with default weak passwords. The issue arises due to the absence of proper password strength enforcement during user registration, enabling individuals to set weak and easily guessable passwords.

**Steps to Reproduce:**

```
# Exploit Title: Default Weak Password Enabled in Online Notes Sharing System
# Date: 20-12-2023
# Exploit Author: dhabaleshwardas
# Vendor Homepage: https://phpgurukul.com/
# Software Link: https://phpgurukul.com/online-notes-sharing-system-using-php-and-mysql/
# Version: 1.0
# Tested on: firefox/chrome/brave
# CVE:
```

To exploit the vulnerability:

1- Navigate to the "signup.php" page of the application.


2- Register a new account by providing any valid information.

<img width="721" alt="1" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/7f973687-b43f-4e9f-b574-b637ca79a3e9">


3- Set a weak and easily guessable password, such as "1" during the registration process.


4- Complete the registration process, and the system accepts the weak password without enforcing adequate password strength.

<img width="941" alt="2" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/21c4ebf6-1be1-4701-afa5-4ff227798fad">


5- Log in to the application using the registered account with the weak password.

<img width="935" alt="3" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/80bf14ec-f8df-42b1-bfb0-f11400707110">


## Impact:

This vulnerability allows users to register with default weak passwords, making it easier for malicious actors to perform brute-force attacks, compromise user accounts, and potentially gain unauthorized access to sensitive information within the application.

## Remediation:

Implement password complexity requirements, including a minimum length, a mix of uppercase and lowercase letters, numbers, and special characters.
