# Provision & Setup Ubuntu Desktop 22.04

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Network Topology](#network-topology)
3. [Ubuntu Overview](#ubuntu-overview)
    - [What is Linux & Ubuntu?](#what-is-linux--ubuntu)
    - [How is Linux used?](#how-is-linux-used)
4. [Security Implications](#security-implications)
5. [Setup Ubuntu](#setup-ubuntu)
    - [Step 1](#step-1)
    - [Step 2](#step-2)
6. [Connect Ubuntu Desktop to Active Directory](#connect-ubuntu-desktop-to-active-directory)
    - [Realmd + SSSD](#realmd--sssd)
        - [Step 1](#step-1)
        - [Step 2](#step-2)
        - [Step 3](#step-3)
        - [Step 4](#step-4)
        - [Step 5](#step-5)
    - [Samba Winbind](#samba-winbind)
        - [Step 1](#step-1)
        - [Step 2](#step-2)
        - [Step 3](#step-3)
        - [Step 4](#step-4)
        - [Step 5](#step-5)
        - [Step 6](#step-6)
        - [Step 7](#step-7)
        - [Step 8](#step-8)
        - [Step 9](#step-9)
        - [Step 10](#step-10)
        - [Step 11](#step-11)
        - [Step 12](#step-12)
        - [Step 13](#step-13)

---

## Prerequisites
1. VirtualBox installed.
2. Virtual Machine with Ubuntu 22.04 ISO has been configured and provisioned (the ISO should be attached to the new VM).
3. Windows Server 2025 with AD Directory Services (ADDS) configured.

---

## Network Topology

---

## Ubuntu Overview

### What is Linux & Ubuntu?
**Linux** is an open-source operating system kernel that serves as the foundation for various distributions (distros) like Ubuntu, Debian, Fedora, and CentOS. It is known for its flexibility, stability, and security, making it a popular choice for servers, desktops, and embedded systems.

**Ubuntu** is a Linux distribution based on Debian, developed and maintained by Canonical. It is designed to be user-friendly, making it a go-to choice for beginners while remaining robust enough for advanced users and enterprise environments. Ubuntu is available in various editions: Desktop, Server, and Core (for IoT).

Key Features:
- **Open-Source**: Free to use, modify, and distribute.
- **Wide Compatibility**: Supports a variety of hardware and software.
- **Active Community**: Backed by a vast community and regular updates.

### How is Linux used?
Linux, and specifically Ubuntu, is utilized across various fields for diverse purposes:

1. **Servers and Hosting**
   - **Web Servers**: Ubuntu is a leading choice for hosting websites, applications, and databases using services like Apache, Nginx, and MySQL.
   - **Cloud Computing**: Powers major cloud platforms like AWS, Google Cloud, and Azure.

2. **Development and Testing**
   - Popular among developers for its built-in tools, package management (APT), and scripting capabilities.
   - Ideal for DevOps workflows with support for Docker, Kubernetes, and CI/CD pipelines.

---

## Security Implications
While Linux and Ubuntu are inherently more secure than many other operating systems, they are not immune to threats. Understanding their security implications is crucial for safe and effective usage.

**Common Threats:**
1. **Privilege Escalation**: Misconfigured sudo or excessive permissions can allow attackers to gain root access.
2. **Unpatched Vulnerabilities**: Delays in applying updates can leave systems exposed to exploits like kernel vulnerabilities.
3. **Weak SSH Configurations**: Using default settings or weak passwords can lead to brute-force attacks.
4. **Malware and Rootkits**: Though less common, Linux-specific malware and rootkits exist and can compromise systems.
5. **Supply Chain Attacks**: Threats can arise from malicious packages or software downloaded from untrusted sources.

---

## Setup Ubuntu

### Step 1
Hit **Enter**.  
Choose **Install Ubuntu**.  
Proceed through the keyboard layout, choose defaults for **Updates and other software**.  
Choose **Erase Disk and Install Ubuntu**. Then select **Install Now**.

Select **Continue**.  
Choose your region.  
Add the following information.

Wait for Ubuntu to install. Let the virtual machine restart and press **Enter** when prompted to remove the installation medium.

Go through the wizard. Unselect **Location Services**.

### Step 2
Go to **Settings → Network**.  
Choose the **+** symbol to add a new network.  
Name the new Wired connection **“Linux AD”**. Then navigate to **IPv4**.

Add the following information to set a static IP address and the Domain Controller as the DNS. Select the green **Add** button to save changes.

Make sure the Linux Desktop can reach the Windows Server Domain Controller.

You may not be able to ping `corp.project-x-dc.com`, that is okay at this time.

**Take Snapshot!**

---

## Connect Ubuntu Desktop to Active Directory

Since Ubuntu (and Linux-native operating systems) are not native to the Microsoft ecosystem, connecting Ubuntu (and Debian-based systems) to Active Directory can be accomplished in a couple of ways. The easiest way is to connect Ubuntu to Active Directory with **realmd** and **SSSD (System Security Services Daemon)**. **Samba Winbind** can also be used if realmd/SSSD is not working.

### Realmd + SSSD

#### Step 1
Open a new terminal session.  
Update the system with:

sudo apt update
#### Step 2

Add the following under the [Time] block.
```bash
sudo nano /etc/systemd/timesyncd.conf
```

Install the necessary packages:
```bash
sudo apt install realmd sssd sssd-tools samba-common krb5-user packagekit libnss-sss libpam-sss adcli samba-common-bin
```

#### Step 3
Use the realm command to discover the domain.

#### Step 4
Enter the following command and provide the Administrator password:
```bash
sudo realm join --verbose --user=Administrator corp.project-x-dc.com
```

#### Step 5
If no output is shown in the console, then the VM has been connected.
Enter the following command to confirm:
```bash
realm list
```


### Samba Wind

#### Step 1
Open a new terminal session.
Update the system with:
```bash
sudo apt update
```

Install the necessary packages:
```bash
sudo apt -y install winbind libpam-winbind libnss-winbind krb5-config samba-dsdb-modules samba-vfs-modules
```

#### Step 2
Move the smb.conf.org file:
```bash
sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.org
```

#### Step 3
Edit the smb.conf file:
```bash
sudo nano /etc/samba/smb.conf
```
Replace realm and workgroup with the following:
```bash
[global]
   kerberos method = secrets and keytab
   realm = CORP.PROJECT-X-DC.COM
   workgroup = CORP
   security = ads
   template shell = /bin/bash
   winbind enum groups = Yes
   winbind enum users = Yes
   winbind separator = +
   idmap config * : rangesize = 1000000
   idmap config * : range = 1000000-19999999
   idmap config * : backend = autorid

```
#### Step 4
Confirm passwd and group have winbind set as a value:
```bash
sudo nano /etc/nsswitch.conf
```

#### Step 5
For domain users, create a home directory on login. Run:
```bash
sudo pam-auth-update
```

#### Step 6
Change DNS settings to refer to AD:
```bash
sudo nano /etc/resolv.conf
```

#### Step 7
Join the domain with Administrator:
```bash
sudo net ads join -U Administrator
```

#### Step 8
Restart winbind:
```bash
systemctl restart winbind
```

#### Step 9
Get Active Directory services information listing:
```bash
net ads info
```

#### Step 10
List all available users:
```bash
wbinfo -u
```

#### Step 11
Create Jane's AD account in your Domain Controller.
Go to Server Manager, then Tools → Active Directory Users and Computers.
Right-click Users, then select New → User.

Add the following information. Make sure Jane’s username is `janed@corp.project-x-dc.com`.

Set Jane’s password (@password123!).

#### Step 12
Step 12
Clear the winbind cache by restarting the service:
```bash
sudo systemctl restart winbind
```
Check the changes with:
```bash
wbinfo -u
```

#### Step 13
Login as Jane:
```bash
sudo login
```
Verify by issuing:
```bash
id
```
