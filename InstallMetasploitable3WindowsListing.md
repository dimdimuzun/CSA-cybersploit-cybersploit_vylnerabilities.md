## Коротка інструкція з встановлення Metasploitable3 на Windows (10,11)

#### 1. Запуск PowerShell

```sh
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

Install the latest PowerShell for new features and improvements! https://aka.ms/PSWindows
```

#### 2. Завантаження та запуск імеджів Metasploitable3 (з урахуванням, що VBox вже встановлений)

```sh
PS C:\Users\UserName> Invoke-WebRequest -Uri "https://raw.githubusercontent.com/rapid7/metasploitable3/master/Vagrantfile" -OutFile "Vagrantfile"
PS C:\Users\UserName> vagrant up
```
#### 3. Очікуваний вивід в консолі

```sh
Bringing machine 'ub1404' up with 'virtualbox' provider...
Bringing machine 'win2k8' up with 'virtualbox' provider...
==> ub1404: Box 'rapid7/metasploitable3-ub1404' could not be found. Attempting to find and install...
    ub1404: Box Provider: virtualbox
    ub1404: Box Version: >= 0
==> ub1404: Loading metadata for box 'rapid7/metasploitable3-ub1404'
    ub1404: URL: https://vagrantcloud.com/api/v2/vagrant/rapid7/metasploitable3-ub1404
==> ub1404: Adding box 'rapid7/metasploitable3-ub1404' (v0.1.12-weekly) for provider: virtualbox
    ub1404: Downloading: https://vagrantcloud.com/rapid7/boxes/metasploitable3-ub1404/versions/0.1.12-weekly/providers/virtualbox/unknown/vagrant.box
    ub1404:
==> ub1404: Successfully added box 'rapid7/metasploitable3-ub1404' (v0.1.12-weekly) for 'virtualbox'!
==> ub1404: Importing base box 'rapid7/metasploitable3-ub1404'...
==> ub1404: Matching MAC address for NAT networking...
==> ub1404: Checking if box 'rapid7/metasploitable3-ub1404' version '0.1.12-weekly' is up to date...
==> ub1404: Setting the name of the VM: Metasploitable3-ub1404
==> ub1404: Clearing any previously set network interfaces...
==> ub1404: Preparing network interfaces based on configuration...
    ub1404: Adapter 1: nat
    ub1404: Adapter 2: hostonly
==> ub1404: Forwarding ports...
    ub1404: 22 (guest) => 2222 (host) (adapter 1)
==> ub1404: Running 'pre-boot' VM custoUserNameions...
==> ub1404: Booting VM...
==> ub1404: Waiting for machine to boot. This may take a few minutes...
    ub1404: SSH address: 127.0.0.1:2222
    ub1404: SSH username: vagrant
    ub1404: SSH auth method: password
    ub1404:
    ub1404: Inserting generated public key within guest...
    ub1404: Removing insecure key from the guest if it's present...
    ub1404: Key inserted! Disconnecting and reconnecting using new SSH key...
==> ub1404: Machine booted and ready!
==> ub1404: Checking for guest additions in VM...
    ub1404: No guest additions were detected on the base box for this VM! Guest
    ub1404: additions are required for forwarded ports, shared folders, host only
    ub1404: networking, and more. If SSH fails on this machine, please install
    ub1404: the guest additions and repackage the box to continue.
    ub1404:
    ub1404: This is not an error message; everything may continue to work properly,
    ub1404: in which case you may ignore this message.
==> ub1404: Setting hostname...
==> ub1404: Configuring and enabling network interfaces...
==> win2k8: Box 'rapid7/metasploitable3-win2k8' could not be found. Attempting to find and install...
    win2k8: Box Provider: virtualbox
    win2k8: Box Version: >= 0
==> win2k8: Loading metadata for box 'rapid7/metasploitable3-win2k8'
    win2k8: URL: https://vagrantcloud.com/api/v2/vagrant/rapid7/metasploitable3-win2k8
==> win2k8: Adding box 'rapid7/metasploitable3-win2k8' (v0.1.0-weekly) for provider: virtualbox
    win2k8: Downloading: https://vagrantcloud.com/rapid7/boxes/metasploitable3-win2k8/versions/0.1.0-weekly/providers/virtualbox/unknown/vagrant.box
    win2k8:
==> win2k8: Successfully added box 'rapid7/metasploitable3-win2k8' (v0.1.0-weekly) for 'virtualbox'!
==> win2k8: Importing base box 'rapid7/metasploitable3-win2k8'...
==> win2k8: Matching MAC address for NAT networking...
==> win2k8: Checking if box 'rapid7/metasploitable3-win2k8' version '0.1.0-weekly' is up to date...
==> win2k8: Setting the name of the VM: UserName_win2k8_1769513867490_80414
==> win2k8: Fixed port collision for 22 => 2222. Now on port 2200.
==> win2k8: Clearing any previously set network interfaces...
A host only network interface you're attempting to configure via DHCP
already has a conflicting host only adapter with DHCP enabled. The
DHCP on this adapter is incompatible with the DHCP settings. Two
host only network interfaces are not allowed to overlap, and each
host only network interface can have only one DHCP server. Please
reconfigure your host only network or remove the virtual machine
using the other host only network.
```
4. Переходимо до VBox та перевіряємо коректність розгортання образів 
##### Логін по замовчанню: vagrant
##### Пароль по замовчанню: vagrant