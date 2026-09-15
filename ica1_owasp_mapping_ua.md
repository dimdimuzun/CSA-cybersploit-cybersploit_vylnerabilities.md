# 🛡️ Маппінг вразливостей ICA1 на OWASP Top 10 (2021)

## 📊 Таблиця відповідності

| № | Вразливість | OWASP Top 10 | Опис відповідності |
|---|-------------|--------------|--------------------|
| 1 | Розкриття IP-адреси | A05: Security Misconfiguration | Відсутність мережевої ізоляції та контролю доступу |
| 2 | Відкриті порти та сервіси | A05: Security Misconfiguration | Неправильна конфігурація сервісів і firewall |
| 3 | Вразлива версія qdPM | A06: Vulnerable and Outdated Components | Використання застарілого ПЗ |
| 4 | Доступ до databases.yml | A02: Cryptographic Failures | Витік облікових даних |
| 5 | Доступ до MySQL | A05: Security Misconfiguration | Відсутність обмежень доступу до БД |
| 6 | MD5 хешування | A02: Cryptographic Failures | Використання слабкого алгоритму |
| 7 | Слабкі паролі | A07: Identification and Authentication Failures | Недостатня складність паролів |
| 8 | Brute-force SSH | A07: Identification and Authentication Failures | Відсутність захисту від перебору |
| 9 | Надлишкові права користувачів | A01: Broken Access Control | Неправильне управління доступом |
| 10 | SUID файли | A05: Security Misconfiguration | Небезпечні налаштування прав |
| 11 | Hardcoded команда | A03: Injection | Потенційне виконання небезпечних команд |
| 12 | PATH Hijacking | A05: Security Misconfiguration | Неправильне використання PATH |
| 13 | Privilege Escalation | A01: Broken Access Control | Отримання root доступу |

---

## 🧠 Висновки

- Найбільша частина вразливостей належить до категорії **Security Misconfiguration (A05)**.
- Критичну роль відіграють також проблеми з автентифікацією (**A07**) та контролем доступу (**A01**).
- Ланцюг атак демонструє класичний сценарій: misconfiguration → credential exposure → privilege escalation.

---

## ✅ Рекомендації

- Регулярно оновлювати програмне забезпечення (A06)
- Використовувати сучасні алгоритми шифрування (A02)
- Впроваджувати захист від brute-force (A07)
- Мінімізувати привілеї користувачів (A01)
- Регулярно перевіряти конфігурацію систем (A05)
