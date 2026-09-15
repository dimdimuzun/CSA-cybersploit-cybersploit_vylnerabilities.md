# 🛡️ Маппінг вразливостей ICA1 на MITRE ATT&CK

## 📊 Таблиця відповідності

| № | Вразливість | MITRE ATT&CK Technique | ID | Опис |
|---|-------------|------------------------|----|------|
| 1 | Розкриття IP | Active Scanning | T1595 | Виявлення активних хостів у мережі |
| 2 | Сканування портів | Network Service Scanning | T1046 | Виявлення відкритих сервісів |
| 3 | Вразливий web-додаток | Exploit Public-Facing Application | T1190 | Використання вразливого веб-сервісу |
| 4 | Доступ до config файлу | Unsecured Credentials | T1552 | Отримання облікових даних з файлів |
| 5 | Доступ до MySQL | Valid Accounts | T1078 | Використання валідних облікових даних |
| 6 | Слабке хешування | Credentials from Password Stores | T1555 | Отримання паролів із БД |
| 7 | Словникова атака | Brute Force | T1110 | Підбір паролів |
| 8 | Brute-force SSH | Brute Force | T1110 | Атака на автентифікацію |
| 9 | Доступ користувачів | Account Discovery | T1087 | Виявлення акаунтів |
| 10 | SUID файл | Abuse Elevation Control Mechanism | T1548 | Використання SUID для ескалації |
| 11 | Hardcoded команда | Command and Scripting Interpreter | T1059 | Виконання системних команд |
| 12 | PATH Hijacking | Hijack Execution Flow | T1574 | Підміна виконуваних файлів |
| 13 | Root доступ | Privilege Escalation | TA0004 | Повне підвищення привілеїв |

---

## 🔗 Attack Chain (Kill Chain Mapping)

1. Reconnaissance:
   - T1595 (Active Scanning)

2. Resource Development / Initial Access:
   - T1046 (Service Scanning)
   - T1190 (Exploit Web App)

3. Credential Access:
   - T1552 (Credentials in Files)
   - T1555 (Password Extraction)
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

## 🧠 Висновки

- Основні техніки належать до категорій:
  - Credential Access
  - Privilege Escalation
  - Discovery

- Найкритичніші механізми атаки:
  - Brute Force (T1110)
  - SUID Abuse (T1548)
  - PATH Hijacking (T1574)

- Атака є класичним прикладом ланцюга MITRE ATT&CK:
  Recon → Access → Credential Dump → Privilege Escalation → Root

---

## ✅ Рекомендації

- Обмежити доступ до мережевих сервісів
- Використовувати strong authentication (MFA)
- Прибрати SUID для небезпечних бінарників
- Вказувати абсолютні шляхи в програмах
- Впровадити моніторинг MITRE технік
