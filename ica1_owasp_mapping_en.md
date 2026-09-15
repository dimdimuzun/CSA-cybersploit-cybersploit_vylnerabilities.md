# 🛡️ Mapping ICA1 Vulnerabilities to OWASP Top 10 (2021)

## 📊 Mapping Table

| # | Vulnerability | OWASP Top 10 | Description |
|---|--------------|--------------|-------------|
| 1 | IP Address Disclosure | A05: Security Misconfiguration | Lack of network isolation and access control |
| 2 | Open Ports and Services | A05: Security Misconfiguration | Improper configuration of services and firewall |
| 3 | Vulnerable qdPM Version | A06: Vulnerable and Outdated Components | Use of outdated software |
| 4 | databases.yml Exposure | A02: Cryptographic Failures | Leakage of credentials |
| 5 | MySQL Access | A05: Security Misconfiguration | Unrestricted database access |
| 6 | MD5 Hashing | A02: Cryptographic Failures | Weak cryptographic algorithm |
| 7 | Weak Passwords | A07: Identification and Authentication Failures | Poor password strength |
| 8 | SSH Brute Force | A07: Identification and Authentication Failures | No protection against brute-force attacks |
| 9 | Excessive User Privileges | A01: Broken Access Control | Improper permission management |
| 10 | SUID Files | A05: Security Misconfiguration | Unsafe permission settings |
| 11 | Hardcoded Command | A03: Injection | Potential unsafe command execution |
| 12 | PATH Hijacking | A05: Security Misconfiguration | Improper environment configuration |
| 13 | Privilege Escalation | A01: Broken Access Control | Unauthorized root access |

---

## 🧠 Key Findings

- Most vulnerabilities fall under **Security Misconfiguration (A05)**.
- Authentication issues (**A07**) and access control (**A01**) are also critical.
- The attack path demonstrates a common chain: misconfiguration → credential exposure → privilege escalation.

---

## ✅ Recommendations

- Keep software up to date (A06)
- Use strong cryptographic algorithms (A02)
- Implement brute-force protection (A07)
- Apply least privilege principle (A01)
- Regularly audit system configuration (A05)
