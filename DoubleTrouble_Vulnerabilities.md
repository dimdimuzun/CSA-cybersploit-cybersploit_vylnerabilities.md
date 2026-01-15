# Опис вразливостей CTF DoubleTrouble

## 1. Information Disclosure / Enumeration
**Опис:**  
Виявлення IP-адреси цільової машини та відкритих сервісів за допомогою `netdiscover` та `nmap`.

**Ризик:**  
Атакувальник отримує повну карту поверхні атаки.

**Категорія:** Reconnaissance / Information Disclosure

---

## 2. Directory Enumeration
**Опис:**  
Використання `gobuster` для виявлення прихованих директорій вебзастосунку: `/backups`, `/secret`, `/batch`.

**Ризик:**  
Розкриття службових або конфіденційних ресурсів.

**Категорія:** Security Misconfiguration

---

## 3. Steganography Weakness
**Опис:**  
У зображенні знайдено приховані облікові дані за допомогою `stegcracker`.

**Ризик:**  
Компрометація облікових даних.

**Категорія:** Sensitive Data Exposure

---

## 4. Weak Authentication & File Upload
**Опис:**  
Панель керування qdPM дозволяла завантаження PHP-файлів без перевірки типу контенту.

**Ризик:**  
Можливість виконання довільного коду.

**Категорія:** Broken Access Control

---

## 5. Remote Code Execution
**Опис:**  
Завантаження PHP reverse shell дозволило отримати віддалений доступ.

**Ризик:**  
Повний контроль над сервером.

**Категорія:** Remote Code Execution

---

## 6. Privilege Escalation via Sudo Misconfiguration
**Опис:**  
Користувачу дозволено запускати `awk` через `sudo`, що дозволяє виконати `/bin/bash` з root-правами.

**Ризик:**  
Повна компрометація системи.

**Категорія:** Privilege Escalation

---

## 7. SQL Injection
**Опис:**  
Форма логіну вразлива до SQL-інʼєкції. Дані отримані через `sqlmap`.

**Ризик:**  
Витік бази даних, обхід автентифікації.

**Категорія:** Injection

---

## 8. Plaintext Credentials Storage
**Опис:**  
Паролі зберігались у відкритому вигляді в базі даних.

**Ризик:**  
Компрометація облікових записів.

**Категорія:** Sensitive Data Exposure

---

## 9. Weak SSH Credentials
**Опис:**  
Використання слабких або повторно використаних паролів для SSH-доступу.

**Ризик:**  
Несанкціонований доступ.

**Категорія:** Authentication Weakness

---

## 10. Kernel Privilege Escalation (Dirty COW)
**Опис:**  
Експлуатація уразливості ядра Linux (CVE-2016-5195).

**Ризик:**  
Отримання root-доступу локальним користувачем.

**Категорія:** Local Privilege Escalation

---

## Загальний висновок
CTF DoubleTrouble демонструє типовий ланцюг атак:
Recon → Enumeration → Initial Access → Privilege Escalation → Full Compromise.
