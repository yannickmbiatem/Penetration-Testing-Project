# Penetration Testing Report

**MEDIROZA GENERAL HOSPITAL - WEB APPLICATION ASSESSMENT**

**W4-PM-FINAL | CYBERSECURITY | NETWORKWALKS**

| Field | Details |
|-------|---------|
| **Pentester Name** | Yannick Mbiatem |
| **Program/Batch** | B084-Networkwalks |
| **Date** | 26/11/2026 |
| **Modules completed** | Week 4 Penetration Testing Project |
| **Client/Target** | Mediroza General Hospital ([medirozahospital.com](https://medirozahospital.com/)) |
| **Permission secured from client?** | Yes (Authorized by Networkwalks) |
| **Scope** | Full Black-Box Penetration Test |

---

## 1. Liability Disclaimer

I have performed these activities only on the systems & devices where I had secured written permission or the devices/systems that I own myself. All materials are for education and research purposes only. Do not use anything from here to break the law. The instructor, the authors and Networkwalks are not responsible for what you do with this knowledge. Every action you take is your own responsibility. Misuse can lead to criminal charges, heavy fines, loss of your job and a permanent record. In most countries, unauthorized access is a crime even when nothing is damaged.

---

## 2. Executive Summary

Between **[Start Date]** and **[End Date]**, a black-box penetration test was conducted against the Mediroza General Hospital web infrastructure. The objective was to identify vulnerabilities, exploit them to demonstrate real-world impact, and retrieve specific confidential data as outlined by the client.

The assessment revealed critical vulnerabilities that allowed for full authentication bypass and unauthorized access to sensitive patient and staff data. The most severe findings included:

- A **SQL Injection** vulnerability on the Patient Portal login page, which allowed unauthorized access to confidential lab reports.
- A publicly accessible database backup file (discovered via `robots.txt`) that exposed employee salaries and shareholder details.

---

## 3. Scope and Methodology

- **Target:** [https://medirozahospital.com](https://medirozahospital.com/)
- **Approach:** Black-Box (No prior knowledge of the system provided)

### Tools Used
- **Kali Linux Terminal** (Whois, WhatWeb) – Reconnaissance and Fingerprinting
- Web Browser – Manual SQL injection, `robots.txt` analysis, and navigation
- **Networkwalks Hash Calculator** – Extracting PDF password hashes
- **Networkwalks Password Cracker** – Cracking the extracted PDF hashes
- **ExifTool** – File metadata analysis
- Command Line Interface (CLI) tools (`cURL`, `findstr`)

---

## 4. Findings and Proof of Exploitation

### 4.1 Milestone 1: Initial Access (Reconnaissance & SQL Injection)

#### Phase 1: Reconnaissance & Footprinting
Before attempting to attack the web application, I performed initial reconnaissance using Kali Linux tools to understand the target's infrastructure.

- **WHOIS:**  
  `whois medirozahospital.com`  
  Results showed the registrar is **NameCheap, Inc.**, and the name servers are `DNS1.NAMECHEAPHOSTING.COM` and `DNS2.NAMECHEAPHOSTING.COM`.

- **WhatWeb:**  
  `whatweb medirozahospital.com`  
  The scan revealed the site is hosted on a **LiteSpeed** web server, uses HTML5, and the server IP is `199.188.201.16`. It also returned a 301 redirect and 403 Forbidden headers.

**Evidence:**  
*(Insert screenshots of WHOIS and WhatWeb results here)*

#### Phase 2: Authentication Bypass (SQL Injection)
After mapping the site, I navigated to the Patient Portal (`/patient/login.php`) to analyze the authentication mechanism.

- **Vulnerability:** SQL Injection (SQLi) / Authentication Bypass
- **Observation:** By entering a specific SQL payload into the username field (`admin' --`) and a dummy password, I was able to bypass authentication and log in as the admin user without valid credentials.
- **Impact:** Unauthorized access to restricted patient areas. I successfully accessed the portal and retrieved **three (3) confidential patient PDF lab reports**.

**Evidence:**  
*(Insert screenshots of the SQL Injection payload and successful login here)*

---

### 4.2 Milestone 2: Data Extraction (Weak Password Encryption)

- **Vulnerability:** Weak Password Policy / Insufficient Encryption
- **Observation:** The three retrieved PDF reports were password-protected. Using the **Networkwalks Hash Calculator**, I extracted the crackable password hashes. I then used the **Networkwalks Password Cracker** (dictionary/wordlist attack) to successfully crack the encryption. The passwords were weak and easily guessable (e.g., `"password"`).
- **Impact:** Loss of confidentiality regarding patient medical records.

**Evidence:**  
*(Insert screenshots of hash extraction and successful password cracking here)*

---

### 4.3 Milestone 3: Critical Data Exposure (Exposed Backup File via robots.txt)

- **Vulnerability:** Sensitive Data Exposure / Directory Listing / Insecure `robots.txt`
- **Observation:**  
  Examining `https://medirozahospital.com/robots.txt` revealed the following entries:
Disallow: /patient/

Disallow: /staff/

Disallow: /old/Text
The `Disallow: /old/` directive revealed a hidden legacy directory. Navigating to `https://medirozahospital.com/old/` showed that **Directory Listing** was enabled. Inside this folder, I discovered a critical backup file named `mediroza_db_backup_2019.sql`.

- **Impact:** Massive data breach. The `.sql` file opened as plain text in the browser and contained unencrypted plaintext data, including monthly salaries of all hospital employees and personal details of hospital shareholders.

**Evidence:**  
*(Insert screenshots of robots.txt, directory listing, and the exposed .sql file content here)*

---

## 5. Risk Analysis / Impact

| # | Vulnerability                              | Potential Impact                                                                 | Risk Level |
|---|--------------------------------------------|----------------------------------------------------------------------------------|------------|
| 1 | SQL Injection (Authentication Bypass)      | Attackers can gain unauthorized access to patient portals, view/modify medical records, and potentially compromise the entire database. | **Critical** |
| 2 | Exposed Database Backup (.sql)             | Full compromise of sensitive internal data (salaries, PII, financials), leading to severe reputational damage and regulatory fines. | **Critical** |
| 3 | Weak PDF Encryption / Weak Passwords       | Unauthorized individuals can easily crack the passwords and access confidential patient lab reports. | **High** |
| 4 | Insecure `robots.txt` & Directory Listing  | `robots.txt` points attackers directly to sensitive directories, and Directory Listing allows them to browse and download files they should not access. | **High** |

---

## 6. Recommendations and Remediation

1. **Fix the SQL Injection**  
 Implement Prepared Statements (Parameterized Queries) across all login forms and input fields. Never concatenate user input directly into SQL queries.

2. **Remove Public Backups**  
 Immediately delete the `/old/` directory and the `mediroza_db_backup_2019.sql` file from the public web server. Database backups should be stored in a secure, offline location or a restricted internal network.

3. **Disable Directory Listing**  
 Configure the web server (LiteSpeed) to disable directory indexing (Directory Browsing) to prevent users from seeing the contents of folders.

4. **Sanitize robots.txt**  
 Do not put sensitive directory names in `robots.txt`, as this acts as a roadmap for attackers. If a directory must be restricted, use proper authentication (e.g., `.htaccess`) rather than relying on `robots.txt`.

5. **Enforce Strong Passwords**  
 Require complex passwords for both user accounts and PDF encryption. Implement Multi-Factor Authentication (MFA) for all portals.

---

## 7. Conclusion

This penetration test successfully demonstrated the severe consequences of poor input validation and improper server configuration. By starting with basic reconnaissance (`whois`, `whatweb`), chaining together a SQL Injection, cracking weak PDF passwords, and utilizing `robots.txt` to discover an exposed database backup, I was able to access deeply confidential information.

Addressing the recommendations above will significantly improve the security posture of Mediroza General Hospital.

---

**Author:** Yannick Mbiatem  
**Cybersecurity Intern – B084**  
**LinkedIn:** [Your LinkedIn Link]  

**Project Information**  
- Program: Cybersecurity program at Networkwalks  
- Week: 04  
- Repository: GitHub
