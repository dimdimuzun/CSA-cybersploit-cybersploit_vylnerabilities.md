# 🛡️ Mapping ICA1 Vulnerabilities to MITRE ATT&CK

## 📊 Mapping Table

| # | Vulnerability | MITRE ATT&CK Technique | ID | Description |
|---|-------------|------------------------|----|-------------|
| 1 | IP Address Discovery | Active Scanning | T1595 | Identification of active hosts in the network |
| 2 | Port Scanning | Network Service Discovery | T1046 | Discovering open services and ports |
| 3 | Vulnerable Web Application | Exploit Public-Facing Application | T1190 | Exploiting exposed web application vulnerabilities |
| 4 | Access to Config File | Unsecured Credentials | T1552 | Retrieving credentials from configuration files |
| 5 | MySQL Access | Valid Accounts | T1078 | Using legitimate credentials for authentication |
| 6 | Weak Hashing | Credentials from Password Stores | T1555 | Extracting credentials from database |
| 7 | Dictionary Attack | Brute Force | T1110 | Password guessing using wordlists |
| 8 | SSH Brute Force | Brute Force | T1110 | Repeated authentication attempts |
| 9 | User Enumeration | Account Discovery | T1087 | Identification of valid accounts |
| 10 | SUID Binary Abuse | Abuse Elevation Control Mechanism | T1548 | Using SUID binaries to escalate privileges |
| 11 | Hardcoded Command | Command and Scripting Interpreter | T1059 | Execution of system commands |
| 12 | PATH Hijacking | Hijack Execution Flow | T1574 | Manipulating execution path to run malicious binaries |
| 13 | Root Access | Privilege Escalation | TA0004 | Achieving full administrative control |

---

## 🔗 Attack Chain (Kill Chain Mapping)

1. Reconnaissance:
   - T1595 (Active Scanning)

2. Initial Access:
   - T1046 (Service Discovery)
   - T1190 (Exploit Public-Facing Application)

3. Credential Access:
   - T1552 (Unsecured Credentials)
   - T1555 (Credential Dumping)
   - T1110 (Brute Force)

4. Lateral Movement / Persistence:
   - T1078 (Valid Accounts)

5. Privilege Escalation:
   - T1548 (SUID Abuse)
   - T1574 (PATH Hijacking)

6. Execution:
   - T1059 (Command Execution)

7. Impact:
   - TA0004 (Privilege Escalation)

---

## 🧠 Key Insights

- The most prominent techniques belong to:
  - Credential Access
  - Privilege Escalation
  - Discovery

- Critical attack methods:
  - Brute Force (T1110)
  - SUID Abuse (T1548)
  - PATH Hijacking (T1574)

- The attack chain follows a typical MITRE ATT&CK pattern:
  Reconnaissance → Initial Access → Credential Access → Privilege Escalation → Full Compromise

---

## ✅ Recommendations

- Restrict access to external services
- Implement strong authentication (MFA)
- Remove unnecessary SUID binaries
- Use absolute paths in privileged applications
- Implement monitoring aligned with MITRE ATT&CK techniques
