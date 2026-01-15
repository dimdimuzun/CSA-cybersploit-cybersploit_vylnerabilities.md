# Мапінг вразливостей CTF DoubleTrouble на OWASP Top 10 (2021)

| № | Виявлена вразливість | Опис | OWASP Top 10 Категорія |
|---|----------------------|------|-------------------------|
| 1 | Information Disclosure / Recon | Виявлення IP, портів, сервісів через netdiscover, nmap | **A05: Security Misconfiguration** |
| 2 | Directory Enumeration | Відкриті директорії `/backups`, `/secret`, `/batch` | **A05: Security Misconfiguration** |
| 3 | Steganography (Hidden Credentials) | Облікові дані заховані у зображенні | **A02: Cryptographic Failures** |
| 4 | Unrestricted File Upload | Завантаження PHP-файлів без фільтрації | **A03: Injection / A05: Security Misconfiguration** |
| 5 | Remote Code Execution | Виконання PHP reverse shell | **A03: Injection** |
| 6 | Sudo Misconfiguration | Можливість запуску awk через sudo | **A05: Security Misconfiguration** |
| 7 | SQL Injection | Інʼєкція у формі логіну | **A03: Injection** |
| 8 | Plaintext Password Storage | Паролі зберігаються відкритим текстом | **A02: Cryptographic Failures** |
| 9 | Weak SSH Credentials | Повторне використання слабких паролів | **A07: Identification and Authentication Failures** |
|10 | Kernel Privilege Escalation (Dirty COW) | Локальне підвищення привілеїв | **A04: Insecure Design / A06: Vulnerable and Outdated Components** |

---

## Деталізований мапінг

### A02 – Cryptographic Failures
- Зберігання паролів у відкритому вигляді.
- Відсутність захисту конфіденційних даних у файлах.

### A03 – Injection
- SQL Injection у формі логіну.
- Виконання довільного коду через завантаження shell.

### A04 – Insecure Design
- Відсутність ізоляції користувацьких прав.
- Можливість локальної ескалації без додаткових барʼєрів.

### A05 – Security Misconfiguration
- Відкриті директорії.
- Небезпечні sudo-права.
- Відсутність обмежень на завантаження файлів.

### A06 – Vulnerable and Outdated Components
- Старе ядро Linux (Dirty COW).
- Застарілий PHP.

### A07 – Identification and Authentication Failures
- Повторне використання паролів.
- Відсутність механізмів захисту автентифікації.

---

## Висновок

CTF DoubleTrouble демонструє класичний ланцюг компрометації:
Misconfiguration → Injection → Credential Exposure → Privilege Escalation → Full System Compromise.

Матеріал добре підходить для навчання студентів аналізу OWASP Top 10 на практиці.
