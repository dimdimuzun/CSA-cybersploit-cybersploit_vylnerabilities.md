# Мапінг вразливостей Vikings 1 на MITRE ATT&CK

Цей документ містить зіставлення виявлених вразливостей машини **Vikings 1** із фреймворком атак **MITRE ATT&CK** для кращого розуміння тактик та технік, що були використані.

---

## 1. Зведена таблиця мапінгу

| Вразливість | Тактика (Tactics) | Техніка (Technique) | ID |
| :--- | :--- | :--- | :--- |
| **Відкрита поверхня атаки** | Reconnaissance | Active Scanning | [T1595](https://attack.mitre.org/techniques/T1595/) |
| **Directory Enumeration** | Discovery | File and Directory Discovery | [T1083](https://attack.mitre.org/techniques/T1083/) |
| **Слабкий пароль архіву** | Credential Access | Brute Force | [T1110](https://attack.mitre.org/techniques/T1110/) |
| **Стеганографія** | Defense Evasion | Steganography | [T1027.003](https://attack.mitre.org/techniques/T1027/003/) |
| **Слабка аутентифікація** | Initial Access | Valid Accounts | [T1078](https://attack.mitre.org/techniques/T1078/) |
| **Rpyc_classic (RCE)** | Execution | Exploitation for Client Execution | [T1203](https://attack.mitre.org/techniques/T1203/) |
| **Конфігурація .profile** | Persistence | Unix Shell Configuration Modification | [T1546.004](https://attack.mitre.org/techniques/T1546/004/) |

---

## 2. Деталізація за етапами атаки

### 2.1. Етап: Розвідка та Початковий доступ
* **Техніка:** Active Scanning ([T1595](https://attack.mitre.org/techniques/T1595/))
    * **Опис:** Використання засобів сканування портів (Nmap) для виявлення сервісів SSH та HTTP. Це перший крок для визначення вектору атаки.
* **Техніка:** File and Directory Discovery ([T1083](https://attack.mitre.org/techniques/T1083/))
    * **Опис:** Використання Gobuster для перебору директорій на вебсервері з метою пошуку прихованих скриптів або файлів конфігурації.

### 2.2. Етап: Отримання облікових даних
* **Техніка:** Brute Force ([T1110](https://attack.mitre.org/techniques/T1110/))
    * **Опис:** Атака за словником на ZIP-архів. Вразливість полягає у низькій ентропії пароля, що дозволяє відновити доступ до вмісту.
* **Техніка:** Steganography ([T1027.003](https://attack.mitre.org/techniques/T1027/003/))
    * **Опис:** Приховування паролів або ключів всередині цифрових зображень для обходу візуального аналізу та засобів виявлення витоку даних.

### 2.3. Етап: Виконання та Підвищення привілеїв
* **Техніка:** Exploitation for Client Execution ([T1203](https://attack.mitre.org/techniques/T1203/))
    * **Опис:** Експлуатація сервісу `rpyc_classic`. Оскільки сервіс дозволяє віддалене виконання Python-коду, це призводить до виконання довільних команд у контексті власника процесу (Root).
* **Техніка:** Exploitation for Privilege Escalation ([T1068](https://attack.mitre.org/techniques/T1068/))
    * **Опис:** Прямий наслідок використання небепечного RPC-сервісу, що дозволяє отримати максимальні права в системі.

### 2.4. Етап: Закріплення в системі (Persistence)
* **Техніка:** Unix Shell Configuration Modification ([T1546.004](https://attack.mitre.org/techniques/T1546/004/))
    * **Опис:** Модифікація файлу `.profile`. Зловмисник може додати шкідливі команди, які будуть виконуватися щоразу при вході користувача в систему, що забезпечує сталий доступ.

---

## 3. Висновок та рекомендації
Згідно з фреймворком MITRE, ланцюг атаки базується на **Discovery** та **Credential Access**. Основним критичним вектором є **Execution** через `rpyc_classic`.

**Рекомендується:**
1. Впровадити сегментацію мережі для закриття доступу до сервісів відладки (RPC).
2. Використовувати механізми контролю цілісності для файлів конфігурації оболонок (наприклад, Auditd).