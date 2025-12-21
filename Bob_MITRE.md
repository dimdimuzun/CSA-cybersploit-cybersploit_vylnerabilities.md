##  MITRE ATT&CK — Mapping для CTF VM *Bob*

> Матриця: **Enterprise ATT&CK**
> Тактики наведені у логічному порядку атаки

---

##  MITRE ATT&CK Mapping Table

| Етап атаки           | Тактика (TA) | Техніка (T) | Назва техніки                             | Прояв у CTF                                    |
| -------------------- | ------------ | ----------- | ----------------------------------------- | ---------------------------------------------- |
| Recon                | TA0043       | T1595       | Active Scanning                           | Виявлення хоста через `netdiscover`, ARP-scan  |
| Recon                | TA0043       | T1595.002   | Active Scanning: Network Service Scanning | `nmap -sC -sV -p-`                             |
| Recon                | TA0043       | T1592       | Gather Victim Host Information            | Визначення відкритих сервісів і портів         |
| Initial Access       | TA0001       | T1190       | Exploit Public-Facing Application         | Доступ до `dev_shell.php`                      |
| Execution            | TA0002       | T1059       | Command and Scripting Interpreter         | Виконання `id`, `whoami`, `bash`               |
| Execution            | TA0002       | T1059.004   | Unix Shell                                | Bash reverse shell                             |
| Persistence          | TA0003       | —           | —                                         | ❌ Не реалізовано (CTF фокусується на PE)       |
| Privilege Escalation | TA0004       | T1068       | Exploitation for Privilege Escalation     | Використання `sudo ALL=(ALL) ALL`              |
| Credential Access    | TA0006       | T1552       | Unsecured Credentials                     | `.old_passwordfile.html`, `theadminisdumb.txt` |
| Credential Access    | TA0006       | T1552.001   | Credentials In Files                      | Паролі у відкритих файлах                      |
| Credential Access    | TA0006       | T1555       | Credentials from Password Stores          | Парольна фраза для GPG                         |
| Discovery            | TA0007       | T1083       | File and Directory Discovery              | `ls -la`, перегляд `/home/*`                   |
| Discovery            | TA0007       | T1033       | System Owner/User Discovery               | `whoami`, `id`, користувачі                    |
| Lateral Movement     | TA0008       | T1078       | Valid Accounts                            | Перехід на `bob` через валідні креденшали      |
| Defense Evasion      | TA0005       | T1078       | Valid Accounts                            | Легітимний доступ без тригерів безпеки         |
| Privilege Escalation | TA0004       | T1548.003   | Abuse Elevation Control Mechanism: Sudo   | `sudo -i` → root                               |
| Impact               | TA0040       | T1499       | Endpoint Denial of Service                | ❌ Не застосовувалось                           |
| Impact               | TA0040       | T1489       | Service Stop                              | ❌ Не застосовувалось                           |

---

##  Узагальнений Attack Flow (ATT&CK view)

```
Recon (T1595)
 → Initial Access (T1190)
 → Execution (T1059)
 → Credential Access (T1552)
 → Discovery (T1083)
 → Lateral Movement (T1078)
 → Privilege Escalation (T1548.003)
 → Root Compromise
```

---

##  Навчальні акценти 

### Що важливо розуміти

* **Жодної zero-day**
* **Жодного складного експлойту**
* Вся атака — це **misconfiguration + poor hygiene**

### ✔ Чому це реалістично

* ATT&CK техніки — **low-noise**
* Атака виглядає як **звичайна робота адміністратора**
* IDS / SIEM без кореляції **нічого не помітять**

