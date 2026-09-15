
# Маппінг вразливостей MoneyBox на MITRE ATT&CK (Enterprise)

Нижче наведено відповідність виявлених під час проходження CTF вразливостей до тактик і технік **MITRE ATT&CK** (версія Enterprise). Ідентифікатори технік наведені у форматі `TXXXX[.XXX]`.

> Примітка: Маппінг відображає **як саме зловмисник міг би скористатися** кожною слабкістю (тобто відповідні тактики/техніки в межах ATT&CK).

| Вразливість / спостереження | Як це може використати атакуючий | Тактика(и) | Техніка (ID та назва) |
|---|---|---|---|
| Анонімний FTP-доступ | Отримання/вивантаження файлів без автентифікації та підготовка до подальших дій | Collection, Command and Control | **T1005 – Data from Local System** (читання/збирання даних з хоста); **T1105 – Ingress Tool Transfer** (використання FTP як каналу для перенесення інструментів) |
| Конфіденційні файли у відкритому FTP | Збір чутливої інформації, що допомагає в атаці | Collection | **T1005 – Data from Local System** |
| Приховані директорії й підказки у вихідному HTML | Активне сканування та пошук відкритих директорій/артефактів | Reconnaissance | **T1595 – Active Scanning** (вкл. brute-force директорій); **T1593 – Search Open Websites/Domains** |
| Стеганографічно приховані дані у зображенні | Приховане розміщення/передача даних, уникаючи виявлення | Defense Evasion | **T1027.003 – Steganography** |
| Слабкий пароль користувача (підібрано словником) | Підбір/гессінг пароля до облікового запису | Credential Access | **T1110 – Brute Force** (напр., **T1110.001 – Password Guessing**) |
| Вхід через SSH за знайденими обліковими даними | Використання легітимних облікових записів для віддаленого доступу | Initial Access, Lateral Movement | **T1078 – Valid Accounts**; **T1021.004 – Remote Services: SSH** |
| Публічний ключ користувача `renu` у `~lily/.ssh/authorized_keys` | Стійкість/перехід між обліковими записами через `authorized_keys` | Persistence, Lateral Movement | **T1098.004 – Account Manipulation: SSH Authorized Keys**; також **T1078 – Valid Accounts** |
| `sudo perl` з `NOPASSWD` (ескалація до root) | Зловживання механізмами підвищення привілеїв та виконання коду інтерпретатором | Privilege Escalation, Defense Evasion | **T1548.003 – Abuse Elevation Control Mechanism: Sudo and Sudo Caching**; **T1059.006 – Command and Scripting Interpreter: Perl** |
| Доступ користувачів до файлів один одного (надмірні права) | Зчитування даних/секретів з локальної системи | Collection | **T1005 – Data from Local System**; за наявності секретів — **T1552 – Unsecured Credentials** |
| Секрети/ключі у вихідному коді вебсторінок | Витік/використання збережених секретів для подальшої компрометації | Credential Access | **T1552.001 – Credentials In Files** |

## Стислий огляд за тактиками
- **Reconnaissance:** T1595, T1593
- **Initial Access:** T1078
- **Credential Access:** T1110, T1552.001, T1552
- **Persistence:** T1098.004
- **Privilege Escalation:** T1548.003
- **Lateral Movement:** T1021.004, T1078
- **Defense Evasion:** T1027.003, T1548.003
- **Collection:** T1005
- **Command and Control (опціонально для FTP як каналу перенесення):** T1105

