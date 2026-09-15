
# Vulnerabilities Identified During the Corrosion2 CTF Walkthrough

Below is a list of key vulnerabilities discovered and exploited during the compromise of the Corrosion2 virtual machine.

## 1. Open and Unprotected Ports
- **SSH (22), HTTP (80), Tomcat (8080)** were accessible without additional protection or access restrictions.
- This allowed:
  - enumeration of web servers,
  - access to the Tomcat Manager,
  - executing further attacks through available interfaces.

## 2. Publicly Accessible `backup.zip` File
- The backup file was stored in an open Tomcat directory, accessible without authentication.
- It allowed downloading Tomcat configuration files containing **admin credentials (admin:melehifokivai)**.

## 3. Weak Password for the `backup.zip` Archive
- The archive password (`@administrator_hi5`) was successfully cracked using a brute‑force attack with the `rockyou.txt` wordlist.
- Lack of strong password protection enabled access to sensitive files.

## 4. Plaintext Credentials Stored in Tomcat Configuration
- The `tomcat-users.xml` file contained administrator credentials in plaintext.
- No encryption or obfuscation was used.
- This enabled authentication into the **Tomcat Manager**.

## 5. Ability to Upload Malicious WAR Files via Tomcat Manager
- The administrator account allowed uploading `.war` files.
- This enabled deployment of a reverse shell application (`revshell.war`).

## 6. Unprotected Reverse Shell Execution
- The server executed the malicious WAR payload and initiated a reverse connection back to the attacker's machine using `netcat`.
- Vulnerability: **lack of restrictions on executing arbitrary web applications**.

## 7. Sensitive Files Found in User Home Directories
- The user `randy` had files `note.txt` and `user.txt`, which contained hints and confidential information.

## 8. Weak System User Passwords
- Password hashes stored in `/etc/shadow` were easily cracked using `john` + `rockyou.txt`.
- User `randy`'s password: `07051986randy` — weak and predictable.

## 9. Dangerous Sudo Privileges
- The user `randy` was allowed to run the Python script `randombase64.py` as root:
```
(root) /usr/bin/python3.8 /home/randy/randombase64.py
```
- The script imported the standard `base64` library, allowing **library hijacking** to execute arbitrary code as root.

## 10. Incorrect Permissions on Python System Libraries
- The file `/usr/lib/python3.8/base64.py` was editable by a non‑privileged user.
- This is a critical **Privilege Escalation** vulnerability.

## 11. Privilege Escalation via Python Module Hijacking
- Modifying `base64.py` allowed inserting:
```python
import os
os.system("/bin/bash")
```
- Running the script via sudo resulted in obtaining **root access**.

## 12. Lack of Monitoring and System Integrity Controls
- No mechanisms existed to detect:
  - modifications of system files,
  - deployment of new web applications,
  - unknown processes or outbound connections.

---

# Conclusion
The system contained numerous vulnerabilities, with the most critical being:
- presence of sensitive files in open directories,
- weak passwords,
- Tomcat configuration weaknesses,
- improper sudo privilege assignments,
- ability to modify system libraries.

These vulnerabilities enabled full system compromise and root access.
