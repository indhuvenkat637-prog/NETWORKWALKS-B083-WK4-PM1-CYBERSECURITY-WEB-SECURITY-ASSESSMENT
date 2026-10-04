# Penetration Testing Report

## Mediroza General Hospital

**NetworkWalks Cybersecurity Internship — Week 4**
**Batch:** B083
**Assessment Type:** Web Application Penetration Testing
**Target:** `https://medirozahospital.com`

---

## 1. Executive Summary

A controlled penetration testing assessment was performed against the Mediroza General Hospital web application as part of the NetworkWalks Week 4 cybersecurity internship project.

The objective was to identify security weaknesses in the application's authentication mechanism, patient report access, PDF protection, web server configuration, and exposed backup files.

The assessment identified multiple vulnerabilities, including username enumeration, SQL Injection, authentication bypass, weak PDF passwords, sensitive PDF metadata, directory listing, and exposure of a database backup containing sensitive information.

The most critical issues identified were SQL Injection authentication bypass and public exposure of the database backup.

---

# 2. Scope

### Target

```text
https://medirozahospital.com
```

### Areas Tested

* Web application
* Patient login portal
* Authentication mechanism
* Patient reports
* PDF security
* `/old/` directory
* Database backup exposure

---

# 3. Methodology

The assessment followed the following testing process:

```text
Reconnaissance
       ↓
Patient Portal Discovery
       ↓
Username Enumeration
       ↓
SQL Injection Testing
       ↓
Authentication Bypass
       ↓
Patient Report Access
       ↓
PDF Password Testing
       ↓
Deep Reconnaissance
       ↓
Backup Directory Discovery
       ↓
Database Backup Analysis
       ↓
Risk Assessment
       ↓
Remediation Recommendations
```

---

# 4. Findings Summary

| ID   | Vulnerability                           | Location                           | Risk     |
| ---- | --------------------------------------- | ---------------------------------- | -------- |
| V-01 | Username Enumeration                    | `/patient/login.php`               | Medium   |
| V-02 | SQL Injection Authentication Bypass     | `/patient/login.php`               | Critical |
| V-03 | Patient Report Exposure                 | Patient portal                     | High     |
| V-04 | Weak PDF Passwords                      | Patient reports                    | High     |
| V-05 | Sensitive PDF Metadata                  | `patient_report_3.pdf`             | Medium   |
| V-06 | Directory Listing and Backup Exposure   | `/old/`                            | Critical |
| V-07 | Sensitive Database Information Exposure | `/old/mediroza_db_backup_2019.sql` | Critical |

---

# 5. Detailed Findings

## V-01 — Username Enumeration

### Severity

**Medium**

### Location

```text
/patient/login.php
```

### Description

The login application responds differently depending on whether the supplied username exists.

A fake username produced:

```text
Username not found
```

When the username `admin` was tested with an incorrect password, the application responded:

```text
Incorrect password
```

The difference in responses allows an attacker to determine whether a username exists.

### Impact

An attacker can enumerate valid usernames and use them for further attacks.

### Recommendation

The application should return the same generic error message for invalid usernames and incorrect passwords.

For example:

```text
Invalid username or password.
```

---

# V-02 — SQL Injection Authentication Bypass

### Severity

**Critical**

### Location

```text
/patient/login.php
```

### Description

SQL Injection testing was performed against the login form.

A single quote entered into the username field caused a MySQL SQL syntax error:

```text
Warning: mysqli_query(): You have an error in your SQL syntax
```

This indicates that user input was being inserted into the SQL query without proper protection.

The vulnerability allowed the login authentication to be bypassed.

### Impact

Successful exploitation can allow unauthorized access to the patient portal without knowing the legitimate password.

### Recommendation

The application should:

* Use prepared statements.
* Use parameterized queries.
* Validate user input.
* Never concatenate raw user input into SQL queries.
* Apply least-privilege permissions to the database account.

---

# V-03 — Patient Report Exposure

### Severity

**High**

### Location

```text
Patient Portal
```

### Description

After bypassing the login mechanism, the patient portal became accessible.

Three patient reports were available:

```text
patient_report_1.pdf
patient_report_2.pdf
patient_report_3.pdf
```

### Impact

Unauthorized access to patient reports can result in disclosure of confidential information.

### Recommendation

Patient reports should:

* Be stored outside the public web root.
* Require proper authorization.
* Verify that the requesting user has permission to access each report.
* Use server-side access controls.

---

# V-04 — Weak PDF Passwords

### Severity

**High**

### Location

```text
patient_report_1.pdf
patient_report_2.pdf
patient_report_3.pdf
```

### Description

The patient reports were protected using password-based PDF encryption.

Password-cracking testing was performed against the extracted PDF hashes.

The following passwords were recovered:

```text
patient_report_1.pdf → 123456
patient_report_2.pdf → password
patient_report_3.pdf → !@#$%^&
```

The first two passwords were recovered using the built-in wordlist. The third required the larger JTR wordlist provided for the exercise.

### Impact

Weak passwords significantly reduce the protection provided by encrypted PDF files.

An attacker who obtains an encrypted PDF can perform offline password-cracking attempts.

### Recommendation

Use strong, unique passwords for sensitive documents and avoid common or predictable passwords.

---

# V-05 — Sensitive PDF Metadata

### Severity

**Medium**

### Location

```text
patient_report_3.pdf
```

### Description

After unlocking the third PDF, metadata analysis revealed internal information.

Important metadata included:

```text
Author: j.malik
Comments: DB backup moved to /old before site migration, do not delete
```

This information provided a clue about a potentially exposed backup directory.

### Impact

Sensitive metadata can reveal:

* Internal employee information
* Internal file locations
* Operational information
* Infrastructure-related details

This information can assist further reconnaissance.

### Recommendation

Sensitive metadata should be removed from documents before they are distributed.

---

# V-06 — Directory Listing and Backup Exposure

### Severity

**Critical**

### Location

```text
/old/
```

### Description

The `/old/` directory was publicly accessible and directory listing was enabled.

The directory exposed the following database backup:

```text
mediroza_db_backup_2019.sql
```

The backup could be downloaded from the web server.

### Impact

Public exposure of a database backup can allow an attacker to obtain sensitive database information without requiring direct database access.

### Recommendation

* Disable directory listing.
* Remove obsolete files from public web directories.
* Store database backups outside the web root.
* Apply strict access controls.
* Encrypt sensitive backups.
* Regularly review publicly accessible directories.

---

# V-07 — Sensitive Database Information Exposure

### Severity

**Critical**

### Location

```text
/old/mediroza_db_backup_2019.sql
```

### Description

The exposed SQL backup contained database information including staff and shareholder records.

The assessment connected the PDF metadata clue to the exposed backup. The staff information also contained a matching identity for the person referenced in the PDF metadata, completing the reconnaissance chain.

### Impact

Exposure of the database backup can result in:

* Confidential information disclosure
* Employee information exposure
* Business information exposure
* Privacy risks
* Further reconnaissance opportunities

### Recommendation

Database backups should never be stored in publicly accessible web directories.

Recommended controls include:

* Secure backup storage
* Access control
* Encryption
* Regular backup reviews
* Secure deletion of obsolete backups
* Monitoring for accidental exposure

---

# 6. Attack Chain

The assessment demonstrated how multiple vulnerabilities could be connected:

```text
Reconnaissance
      ↓
robots.txt
      ↓
/patient/
      ↓
Login Page
      ↓
Username Enumeration
      ↓
SQL Injection
      ↓
Authentication Bypass
      ↓
Patient Reports
      ↓
PDF Password Testing
      ↓
PDF Metadata
      ↓
/old/
      ↓
Database Backup
      ↓
Sensitive Database Information
```

This demonstrates that individual vulnerabilities can combine to create a much greater overall security risk.

---

# 7. Evidence

The practical evidence collected during the assessment is stored in the repository's `screenshots` directory.

### Screenshot 1 — Patient Login

```text
screenshots/Screenshot1.png
```

### Screenshot 2 — Patient Reports

```text
screenshots/Screenshot2.png
```

### Screenshot 3 — PDF Hash

```text
screenshots/Screenshot3.png
```

### Screenshot 4 — Password Cracking

```text
screenshots/Screenshot4.png
```

### Screenshot 5 — Old Directory

```text
screenshots/Screenshot5.png
```

### Screenshot 6 — Staff Database

```text
screenshots/Screenshot6.png
```

### Screenshot 7 — Shareholder Database

```text
screenshots/Screenshot7.png
```

---

# 8. Risk Assessment

The overall risk of the identified vulnerabilities is considered **Critical**.

The highest-priority vulnerabilities are:

1. SQL Injection authentication bypass
2. Publicly accessible database backup
3. Sensitive database information exposure
4. Patient report exposure
5. Weak PDF passwords

These vulnerabilities demonstrate the importance of secure authentication, input validation, access control, secure document handling, and proper backup management.

---

# 9. Remediation Recommendations

## Immediate Actions

* Fix the SQL Injection vulnerability.
* Remove database backups from public web directories.
* Disable directory listing.
* Restrict access to patient reports.
* Replace weak document passwords.

## Short-Term Actions

* Prevent username enumeration.
* Remove sensitive PDF metadata.
* Review all public directories.
* Review backup storage and access permissions.
* Implement secure authentication practices.

## Long-Term Actions

* Conduct regular vulnerability assessments.
* Perform periodic penetration testing.
* Implement secure software development practices.
* Apply least-privilege access controls.
* Monitor public-facing systems for accidental information exposure.
* Establish secure backup management procedures.

---

# 10. Conclusion

The Mediroza General Hospital penetration testing exercise demonstrated how several security weaknesses can be identified and connected during a structured penetration test.

The assessment began with reconnaissance and progressed through authentication testing, SQL Injection, authentication bypass, patient report access, PDF password testing, metadata analysis, directory discovery, and database backup analysis.

The most serious security issues were the SQL Injection vulnerability and the publicly accessible database backup.

Implementing the recommended remediation measures would significantly reduce the risk of unauthorized access and sensitive information disclosure.

---

# 11. Authorization and Disclaimer

This penetration testing activity was performed as part of an authorized NetworkWalks educational cybersecurity lab.

The target was provided for security testing as part of the internship exercise.

The techniques documented in this report must only be used against systems for which explicit written authorization has been provided.

Unauthorized security testing or access to systems and data is prohibited.
