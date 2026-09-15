
# Маппінг вразливостей Corrosion2 на MITRE ATT&CK

Нижче наведено відповідність між виявленими вразливостями та техніками MITRE ATT&CK (Enterprise Matrix).

---

# Таблиця відповідності

| Вразливість | MITRE ATT&CK Техніка | Код |
|-------------|------------------------|------|
| Відкриті порти (22, 80, 8080) | Network Service Discovery | **T1046** |
| Відкритий файл backup.zip | Gather Victim Host Information | **T1592** |
| Слабкий пароль архіву | Password Cracking (Credential Access) | **T1110.002** |
| Збереження паролів Tomcat у відкритому вигляді | Credentials in Configuration Files | **T1552.003** |
| Авторизація в Tomcat Manager | Valid Accounts | **T1078** |
| Завантаження шкідливого WAR | Exploitation for Web Service | **T1190** |
| Reverse Shell через веб-додаток | Command and Scripting Interpreter | **T1059** |
| Отримання інтерактивної оболонки через netcat | Remote Services | **T1021** |
| Файли user.txt, note.txt | Discovery: File and Directory Discovery | **T1083** |
| Використання look для читання /etc/shadow | OS Credential Dumping | **T1003** |
| Злам хешу користувача randy | Brute Force | **T1110** |
| Небезпечні sudo-привілеї | Abuse Elevation Control Mechanism | **T1548** |
| Можливість змінити base64.py | Modify System Partition | **T1222** |
| Підміна Python модуля для ескалації | Hijack Execution Flow: DLL/Library Injection | **T1574** |
| Запуск модифікованого base64.py з правами root | Privilege Escalation | **T1068** |
| Відсутність моніторингу | Inhibit System Recovery / Logging Disabled | **T1562.002** |

---

# Висновок
Вразливості Corrosion2 найбільше відповідають таким тактикам MITRE ATT&CK:
- **Initial Access**
- **Execution**
- **Privilege Escalation**
- **Credential Access**
- **Discovery**
- **Defense Evasion**
- **Persistence**

Отримані відповідності демонструють повний ланцюг атаки від Recon до Root.
