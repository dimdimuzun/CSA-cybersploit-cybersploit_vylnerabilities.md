# Мапінг проходження CTF "Empire Breakout" на MITRE ATT&CK

Ось відповідність кожного етапу твого walkthrough до тактик і технік MITRE ATT&CK (Enterprise Matrix).

| Крок у CTF | Тактика (Tactic) | Техніка / Sub-technique | ID |
|---|---|---|---|
| `netdiscover`, `nmap -sV` — пошук хоста та сканування портів | Reconnaissance | Active Scanning: Scanning IP Blocks / Vulnerability Scanning | T1595 / T1595.001 |
| `dirb` — брутфорс директорій на порту 80 | Reconnaissance | Active Scanning: Wordlist Scanning | T1595.003 |
| Знайдений у HTML-коментарі зашифрований (Brainfuck) пароль | Credential Access | Unsecured Credentials: Credentials In Files | T1552.001 |
| Декодування Brainfuck-рядка вручну/онлайн-декодером | Credential Access | Steal or Forge Authentication Certificates *(умовно)* — точніше просто ручний аналіз, MITRE-техніки немає; фактично це продовження T1552.001 (виявлення прихованих креденшлів) | T1552.001 |
| `enum4linux -a` — пошук користувачів (SMB/NetBIOS) | Discovery | Account Discovery: Domain/Local Account (через SMB enum) | T1087.001/.002 |
| Вхід через Usermin (порт 20000) з отриманими кредами | Initial Access | Valid Accounts + Exploit Public-Facing Application (Usermin як веб-панель) | T1078, T1190 |
| Доступ до веб-терміналу Usermin, виконання `id`, `ls` | Execution | Command and Scripting Interpreter: Unix Shell | T1059.004 |
| Знаходження `user.txt` | Collection | Data from Local System | T1005 |
| Python reverse shell (`socket`, `pty.spawn`) | Execution | Command and Scripting Interpreter: Python | T1059.006 |
| З'єднання назад на `nc -lvp` атакуючого | Command and Control | Application Layer Protocol / Non-Standard Port | T1071 / T1571 |
| `cat /etc/issue`, `uname -a` — збір інфи про ОС/ядро | Discovery | System Information Discovery | T1082 |
| Пошук CVE під версію ядра (без результату) | *(підготовчий етап, техніки немає)* | — | — |
| `find / -perm -4000`, `getcap -r /` — пошук SUID/capabilities | Discovery | Permission Groups Discovery / File and Directory Discovery | T1069, T1083 |
| Використання `cap_dac_read_search` для обходу прав доступу | Privilege Escalation / Defense Evasion | Abuse Elevation Control Mechanism (через Linux capabilities) | T1548 |
| Спроба зламати `/etc/shadow` через John the Ripper | Credential Access | Brute Force: Password Cracking (offline) | T1110.002 |
| Читання `.old_pass.bak` через `tar` (bypass прав) | Credential Access | Unsecured Credentials: Credentials In Files | T1552.001 |
| `su root` з отриманим паролем | Privilege Escalation | Valid Accounts: Local Accounts | T1078.003 |
| Читання `/root/rOOt.txt` | Collection | Data from Local System | T1005 |

## Загальна картина по kill chain

```
Reconnaissance      → T1595, T1595.003
Initial Access      → T1190, T1078
Execution           → T1059.004, T1059.006
Discovery           → T1082, T1087, T1069, T1083
Credential Access   → T1552.001, T1110.002
Privilege Escalation→ T1548, T1078.003
Collection          → T1005
Command and Control → T1071 / T1571
```
