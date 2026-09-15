
# Mapping of MoneyBox Vulnerabilities to MITRE ATT&CK (Enterprise)

Below is the mapping of vulnerabilities identified during the CTF walkthrough to the tactics and techniques of **MITRE ATT&CK (Enterprise)**. Technique identifiers follow the `TXXXX[.XXX]` format.

> Note: This mapping reflects **how an attacker could leverage each weakness**, correlating them to ATT&CK tactics and techniques.

| Vulnerability / Observation | How an Attacker May Use It | Tactic(s) | Technique (ID & Name) |
|---|---|---|---|
| Anonymous FTP access | Retrieve/upload files without authentication; prepare tools for further attacks | Collection, Command and Control | **T1005 – Data from Local System**; **T1105 – Ingress Tool Transfer** |
| Sensitive data stored in open FTP directory | Collect valuable information aiding further compromise | Collection | **T1005 – Data from Local System** |
| Hidden directories and hints in HTML | Discovery of hidden paths through active scanning | Reconnaissance | **T1595 – Active Scanning**; **T1593 – Search Open Websites/Domains** |
| Steganographically hidden confidential data | Covert transfer/obfuscation of data to evade detection | Defense Evasion | **T1027.003 – Steganography** |
| Weak password (bruteforced) | Password guessing to gain system access | Credential Access | **T1110 – Brute Force**; **T1110.001 – Password Guessing** |
| SSH login using obtained credentials | Legitimate access to host; lateral movement | Initial Access, Lateral Movement | **T1078 – Valid Accounts**; **T1021.004 – Remote Services: SSH** |
| Public key of one user inside another user’s authorized_keys | Persistence and lateral movement using SSH keys | Persistence, Lateral Movement | **T1098.004 – Account Manipulation: SSH Authorized Keys**; **T1078 – Valid Accounts** |
| sudo perl with NOPASSWD (root escalation) | Abuse of privilege escalation mechanisms; script execution | Privilege Escalation, Defense Evasion | **T1548.003 – Abuse Elevation Control Mechanism: Sudo and Sudo Caching**; **T1059.006 – Command and Scripting Interpreter: Perl** |
| Users having access to each other’s files | Collecting sensitive data due to weak file permissions | Collection | **T1005 – Data from Local System**; **T1552 – Unsecured Credentials** |
| Sensitive keys/secrets in webpage source code | Extraction of credentials from exposed files | Credential Access | **T1552.001 – Credentials in Files** |

## Summary by ATT&CK Tactic
- **Reconnaissance:** T1595, T1593
- **Initial Access:** T1078
- **Credential Access:** T1110, T1552.001, T1552
- **Persistence:** T1098.004
- **Privilege Escalation:** T1548.003
- **Lateral Movement:** T1021.004, T1078
- **Defense Evasion:** T1027.003, T1548.003
- **Collection:** T1005
- **Command and Control (via FTP as a transfer channel):** T1105
