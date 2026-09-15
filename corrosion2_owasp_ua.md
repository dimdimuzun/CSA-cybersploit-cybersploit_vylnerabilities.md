
# Маппінг вразливостей Corrosion2 на OWASP Top 10

Нижче наведено відповідність між виявленими вразливостями та категоріями OWASP Top 10 (2021).

## OWASP Top 10 Категорії
1. **A01: Broken Access Control**
2. **A02: Cryptographic Failures**
3. **A03: Injection**
4. **A04: Insecure Design**
5. **A05: Security Misconfiguration**
6. **A06: Vulnerable and Outdated Components**
7. **A07: Identification and Authentication Failures**
8. **A08: Software and Data Integrity Failures**
9. **A09: Security Logging and Monitoring Failures**
10. **A10: Server-Side Request Forgery (SSRF)**

---

# Таблиця відповідності

| Вразливість | Опис | OWASP Top 10 Категорія |
|------------|-------|--------------------------|
| Відкриті порти 22, 80, 8080 | Немає обмежень доступу до сервісів | **A05: Security Misconfiguration** |
| Відкритий файл backup.zip | Конфіденційні дані у відкритому доступі | **A05: Security Misconfiguration** |
| Слабкий пароль архіву | Незахищені криптографічні механізми | **A02: Cryptographic Failures** |
| Збереження паролів Tomcat у відкритому вигляді | Plain-text креденшіали в конфігурації | **A07: Identification and Authentication Failures** |
| Можливість завантажувати WAR у Tomcat Manager | Відсутність контролю доступу / небезпека RCE | **A01: Broken Access Control**, **A05: Security Misconfiguration** |
| Розгортання зворотної оболонки (revshell) | Виконання довільного коду | **A03: Injection**, **A05: Security Misconfiguration** |
| Слабкий пароль користувача randy | Уразливість до brute-force | **A07: Identification and Authentication Failures** |
| Небезпечні sudo-привілеї (python script root) | Неналежні налаштування привілеїв | **A05: Security Misconfiguration** |
| Можливість редагувати системний модуль base64.py | Порушення цілісності компонентів | **A08: Software and Data Integrity Failures** |
| Ескалація привілеїв через модифікацію модуля Python | Виконання шкідливого коду з правами root | **A01: Broken Access Control**, **A08: Software and Data Integrity Failures** |
| Відсутність моніторингу та контролю цілісності | Відсутній аудит небезпечних подій | **A09: Security Logging and Monitoring Failures** |

---

# Висновок
Вразливості Corrosion2 найбільше відповідають категоріям:
- **A05: Security Misconfiguration** (помилки конфігурації),
- **A07: Identification and Authentication Failures** (слабкі/некоректні креденшіали),
- **A08: Software and Data Integrity Failures** (порушення довіри до компонентів),
- **A01: Broken Access Control** (неправильне керування правами).

Ці недоліки дали змогу пройти всю атаку від початкового доступу до повного захоплення root.
