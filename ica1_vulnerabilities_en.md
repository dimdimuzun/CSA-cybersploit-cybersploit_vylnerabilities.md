# 📚 Vulnerabilities Identified During ICA1 CTF

## 🔎 1. Information Disclosure (Network Discovery)
**Description:**  
The target machine's IP address was easily identified using `netdiscover`.

**Risk:**
- Allows attackers to locate the target in the network
- Facilitates further attacks

**Cause:**
- Lack of network isolation or monitoring (IDS/IPS, VLAN segmentation)

---

## 🌐 2. Exposed Services Without Protection
**Description:**  
Open ports discovered:
- 22 (SSH)
- 80 (HTTP)
- 3306 (MySQL)

**Risk:**
- Expands attack surface
- Enables service enumeration and brute-force attacks

**Cause:**
- Missing firewall rules or access restrictions

---

## 💻 3. Vulnerable Web Application (qdPM 9.2)
**Description:**  
The system uses a version with known vulnerabilities.

**Risk:**
- Remote code execution or data exposure
- System compromise

**Cause:**
- Outdated or unpatched software

---

## 📂 4. Exposure of Configuration File via HTTP
**Description:**  
The file `/core/config/databases.yml` was publicly accessible.

**Risk:**
- Leakage of database credentials
- Full database compromise

**Cause:**
- Misconfigured web server
- Missing access controls

---

## 🛢 5. Unrestricted MySQL Access
**Description:**  
MySQL service was accessible remotely using leaked credentials.

**Risk:**
- Unauthorized data access
- Credential harvesting

**Cause:**
- Missing network restrictions
- Weak configuration

---

## 🔑 6. Weak Password Hashing (MD5)
**Description:**  
Passwords stored using MD5 (encoded with Base64).

**Risk:**
- Easily cracked using brute-force or dictionary attacks

**Cause:**
- Use of outdated hashing algorithm
- No salting

---

## 📚 7. Weak Password Policy
**Description:**  
User passwords were found in common wordlists (rockyou).

**Risk:**
- Easy compromise of accounts

**Cause:**
- Lack of password complexity requirements

---

## 🔐 8. No Protection Against Brute-Force (SSH)
**Description:**  
SSH allowed unlimited login attempts.

**Risk:**
- Successful password guessing attacks

**Cause:**
- Missing rate limiting or fail2ban

---

## 👤 9. Excessive User Privileges
**Description:**  
Users had access to sensitive hints (note.txt).

**Risk:**
- Disclosure of internal information
- Simplified privilege escalation

**Cause:**
- Poor access control implementation

---

## ⚙️ 10. Unsafe SUID Binaries
**Description:**  
The binary `/opt/get_access` had SUID permissions.

**Risk:**
- Execution with root privileges
- Full system compromise

**Cause:**
- Misconfigured file permissions

---

## 🧪 11. Hardcoded Commands in Binary
**Description:**  
Binary contained the command `cat /root/system_info`.

**Risk:**
- Command manipulation
- Exploitable execution flow

**Cause:**
- Hardcoded system calls

---

## 🛤 12. PATH Hijacking
**Description:**  
System relied on PATH variable without absolute paths.

**Risk:**
- Execution of malicious binaries
- Privilege escalation

**Cause:**
- Improper use of environment variables

---

## 👑 13. Full Privilege Escalation (Root)
**Description:**  
Attacker gained root access via binary abuse.

**Risk:**
- Complete system takeover

**Cause:**
- Combination of multiple vulnerabilities

---

# ✅ Conclusion

The compromise was possible due to a chain of vulnerabilities rather than a single flaw:

1. Information disclosure
2. Misconfiguration
3. Credential exposure
4. Weak authentication
5. Privilege escalation

Even minor issues combined can lead to full system compromise.
