
# Mapping of MoneyBox Vulnerabilities to OWASP Top 10

## OWASP Top 10 (2021 Version)

| Vulnerability | Description | Corresponding OWASP Top 10 Category |
|---------------|-------------|-------------------------------------|
| Anonymous FTP Access | Allows unauthenticated file access | **A01:2021 – Broken Access Control** |
| Storage of Sensitive Data in Open FTP | Sensitive data stored without protection, publicly accessible | **A02:2021 – Cryptographic Failures** |
| Hidden Directories and Hints in HTML (blogs, S3ct3r-T3xt) | Sensitive paths exposed in HTML source | **A05:2021 – Security Misconfiguration** |
| Steganographically Hidden Confidential Data | Weak data protection inside image files | **A02:2021 – Cryptographic Failures** |
| Weak User Password (easy to brute force) | Password `987654321` cracked via Hydra | **A07:2021 – Identification and Authentication Failures** |
| Lack of SSH Access Restrictions | No proper SSH hardening, open attack surface | **A01:2021 – Broken Access Control** |
| Public Key of One User Inside Another User’s authorized_keys | Unauthorized lateral movement enabled | **A01:2021 – Broken Access Control** |
| Passwordless sudo perl (Privilege Escalation) | Misconfigured sudo rules allow root execution | **A05:2021 – Security Misconfiguration** |
| Users Accessing Other Users’ Files | Improper filesystem permissions | **A01:2021 – Broken Access Control** |
| Sensitive Files Stored in Readable Directories | Poor protection of sensitive information | **A02:2021 – Cryptographic Failures** |

