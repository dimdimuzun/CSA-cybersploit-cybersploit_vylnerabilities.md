# Mapping of Corrosion2 Vulnerabilities to OWASP Top 10
Below is the correspondence between the discovered vulnerabilities and the OWASP Top 10 (2021) categories.
## OWASP Top 10 Categories
1. **A01: Broken Access Control**
2. **A02: Cryptographic Failures**
3. **A03: Injection**
4. **A04: Insecure Design**
5. **A05: Security Misconfiguration**
6. **A06: Vulnerable and Outdated Components**
7. **A07: Identification and Authentication Failures**
8. **A08: Software and Data Integrity Failures**
9. **A09: Security Logging and Monitoring Failures**
10. **A10: Server-Side Request Forgery (SSRF)**
---
# Mapping Table
| Vulnerability | Description | OWASP Top 10 Category |
|--------------|-------------|------------------------|
| Open ports 22, 80, 8080 | No access restrictions on services | **A05: Security Misconfiguration** |
| Public backup.zip file | Sensitive data exposed in public directory | **A05: Security Misconfiguration** |
| Weak archive password | Unprotected cryptographic mechanisms | **A02: Cryptographic Failures** |
| Plaintext Tomcat credentials | Credentials stored in plaintext | **A07: Identification and Authentication Failures** |
| Ability to upload WAR files | Lack of access control / RCE risk | **A01: Broken Access Control**, **A05: Security Misconfiguration** |
| Deployment of reverse shell (revshell) | Arbitrary code execution | **A03: Injection**, **A05: Security Misconfiguration** |
| Weak password for user randy | Brute-force susceptible | **A07: Identification and Authentication Failures** |
| Dangerous sudo privileges | Misconfigured privilege settings | **A05: Security Misconfiguration** |
| Editable system module base64.py | Integrity violation | **A08: Software and Data Integrity Failures** |
| Privilege escalation via Python module | Root-level malicious code execution | **A01: Broken Access Control**, **A08: Software and Data Integrity Failures** |
| Lack of monitoring & integrity controls | No audit of suspicious activity | **A09: Security Logging and Monitoring Failures** |
---
# Conclusion
The Corrosion2 vulnerabilities most strongly align with:
- **A05: Security Misconfiguration** (misconfiguration issues),
- **A07: Identification and Authentication Failures** (weak/incorrect credentials),
- **A08: Software and Data Integrity Failures** (trust boundary violations),
- **A01: Broken Access Control** (improper access control management).
These weaknesses together enabled a full attack chain from initial access to complete root compromise.
