
# Mapping of Corrosion2 Vulnerabilities to MITRE ATT&CK
Below is the correspondence between the vulnerabilities identified in the Corrosion2 CTF and the MITRE ATT&CK Enterprise Matrix techniques.

---
# Mapping Table
| Vulnerability | MITRE ATT&CK Technique | Code |
|--------------|------------------------|------|
| Open ports (22, 80, 8080) | Network Service Discovery | **T1046** |
| Publicly accessible backup.zip | Gather Victim Host Information | **T1592** |
| Weak archive password | Password Cracking (Credential Access) | **T1110.002** |
| Plaintext Tomcat credentials | Credentials in Configuration Files | **T1552.003** |
| Authentication into Tomcat Manager | Valid Accounts | **T1078** |
| Upload of malicious WAR | Exploitation for Web Service | **T1190** |
| Reverse Shell via web application | Command and Scripting Interpreter | **T1059** |
| Interactive shell via netcat | Remote Services | **T1021** |
| Files user.txt, note.txt | File and Directory Discovery | **T1083** |
| Using look to read /etc/shadow | OS Credential Dumping | **T1003** |
| Cracking randy's password hash | Brute Force | **T1110** |
| Dangerous sudo privileges | Abuse Elevation Control Mechanism | **T1548** |
| Ability to modify base64.py | Modify System Partition | **T1222** |
| Python module hijacking for escalation | Hijack Execution Flow: DLL/Library Injection | **T1574** |
| Running modified base64.py as root | Privilege Escalation | **T1068** |
| Lack of monitoring | Inhibit System Recovery / Logging Disabled | **T1562.002** |

---
# Conclusion
The Corrosion2 vulnerabilities most closely align with MITRE ATT&CK tactics:
- **Initial Access**
- **Execution**
- **Privilege Escalation**
- **Credential Access**
- **Discovery**
- **Defense Evasion**
- **Persistence**

These mappings demonstrate a complete attack chain—from reconnaissance to full root compromise.
