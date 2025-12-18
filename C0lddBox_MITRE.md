
## Мапінг вразливостей на MITRE ATT&CK

### **TA0043 – Reconnaissance (Розвідка)**

| Вразливість                                                   | Technique                          |
| ------------------------------------------------------------- | ---------------------------------- |
| Відсутність сегментації мережі, виявлення через `netdiscover` | **T1595 – Active Scanning**        |
| Відкриті порти, сканування `nmap`                             | **T1595.001 – Scanning IP Blocks** |

**Пояснення:**
Атакувальник безперешкодно отримує інформацію про мережу та сервіси.

---

### **TA0001 – Initial Access (Початковий доступ)**

| Вразливість                     | Technique                                      |
| ------------------------------- | ---------------------------------------------- |
| Доступний WordPress login       | **T1078 – Valid Accounts**                     |
| Відсутність rate limiting       | **T1110 – Brute Force**                        |
| User Enumeration через `wpscan` | **T1589 – Gather Victim Identity Information** |

---

### **TA0006 – Credential Access (Доступ до облікових даних)**

| Вразливість                    | Technique                                                    |
| ------------------------------ | ------------------------------------------------------------ |
| Слабкий пароль (`rockyou.txt`) | **T1110.002 – Password Cracking**                            |
| Повторне використання паролів  | **T1078 – Valid Accounts**                                   |
| Читання `wp-config.php`        | **T1552.001 – Credentials in Files**                         |
| Base64 як «захист»             | **T1555 – Credentials from Password Stores** (концептуально) |

---

### **TA0002 – Execution (Виконання коду)**

| Вразливість                         | Technique                                     |
| ----------------------------------- | --------------------------------------------- |
| Редагування PHP-коду через WP Admin | **T1059 – Command and Scripting Interpreter** |
| PHP reverse shell                   | **T1059.004 – Unix Shell**                    |

---

### **TA0003 – Persistence (Закріплення в системі)**

| Потенційна можливість          | Technique                             |
| ------------------------------ | ------------------------------------- |
| Зміна системних файлів         | **T1546 – Event Triggered Execution** |
| Створення backdoor-користувача | **T1136 – Create Account**            |

*(У CTF не реалізовано явно, але технічно можливо після root)*

---

### **TA0004 – Privilege Escalation (Підвищення привілеїв)**

| Вразливість                | Technique                                         |
| -------------------------- | ------------------------------------------------- |
| Misconfigured sudoers      | **T1548.003 – Sudo and Sudo Caching**             |
| GTFOBins (vim, chmod, ftp) | **T1068 – Exploitation for Privilege Escalation** |

---

### **TA0005 – Defense Evasion (Обхід захисту)**

| Вразливість                        | Technique                                 |
| ---------------------------------- | ----------------------------------------- |
| Відсутність SELinux / AppArmor     | **T1562 – Impair Defenses**               |
| Використання легітимних бінарників | **T1218 – Signed Binary Proxy Execution** |

---

### **TA0007 – Discovery (Внутрішня розвідка)**

| Дія                         | Technique                               |
| --------------------------- | --------------------------------------- |
| Перегляд файлів, прав, sudo | **T1087 – Account Discovery**           |
| `sudo -l`                   | **T1069 – Permission Groups Discovery** |

---

### **TA0011 – Command and Control (C2)**

| Вразливість                | Technique                                  |
| -------------------------- | ------------------------------------------ |
| Reverse shell через netcat | **T1071 – Application Layer Protocol**     |
| Shell over TCP             | **T1095 – Non-Application Layer Protocol** |

---

### **TA0008 – Lateral Movement (потенційно)**

| Потенціал                     | Technique                   |
| ----------------------------- | --------------------------- |
| Повторне використання паролів | **T1021 – Remote Services** |

*(У межах CTF не реалізовано, але логічно випливає)*

---

##  Узагальнювальна таблиця (коротко)

| Фаза ATT&CK          | Техніки      |
| -------------------- | ------------ |
| Reconnaissance       | T1595        |
| Initial Access       | T1078, T1110 |
| Credential Access    | T1552, T1110 |
| Execution            | T1059        |
| Privilege Escalation | T1548, T1068 |
| Defense Evasion      | T1562, T1218 |
| Discovery            | T1087, T1069 |
| C2                   | T1071, T1095 |

---

## Методичний висновок

Цей CTF **чітко відображає повний ATT&CK-ланцюг**:

> **Recon → Initial Access → Credential Access → Execution → Privilege Escalation → Full Compromise**

Ідеальний кейс для:

* навчання SOC-аналітиків,
* пояснення різниці між **вразливістю** та **технікою атаки**,
* демонстрації, як **одна misconfiguration** (sudoers) завершує весь kill chain.

