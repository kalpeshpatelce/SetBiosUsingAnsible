# Remotly Set Bios Using Ansible
Basic Idea behind the SetBiosAnsible is that Configure Bios of HP System Remotly Using Ansible

Basic Idea behind the SetBiosAnsible is that everyday i deal with 1000+ Computers so to 
set BiosSetting like Wakeupon Lan, Power loss restore option & Boot Computer in Network
Bootup mode i have to go each computers to set above Bios Configuration it is very 
difficult & Boring Work for me. Instead of doing those long Process. I Developed 
Ansible Playbook which automize task for me. how please read this repos 
basically i done it with HP System only but you can try with other vendor also by finding their 
Bios Configuration Utility.
Find Bios Utilty based on System i am using HPBiosUtility to configure Bios.

# Get Bios Configuration Utility From Vendor Website
Please Download HP Bios Configuration Utility From Link

```bash
https://ftp.hp.com/pub/caps-softpaq/cmit/HP_BCU.html
````

When You install Downloaded Utility It will Installed at Following Location

```bash
C:\Program Files (x86)\HP\BIOS Configuration Utility\
````
You Will find two Files like
1.	BiosConfigUtility.exe #for 32 bit OS
2.	BiosConfigUtility64.exe #For 64 bit OS (We use in this scenario)
Copy BiosConfigUtility64.exe to http or ftp Server look like as 
For example http://ServerIP/BiosConfigUtility64.exe or ftp://ServerIP/BiosConfigUtility64.exe

# How To Use Bios Configuration Utility to Get Bios
if you want to get current configuration of Bios. You need to run following Command
make dir at C:\BCU Copy BiosConfigUtility64.exe to BCU Folder from C:\Program Files (x86)\HP\BIOS Configuration Utility\
```bash
C:\BCU\BiosConfigUtility64.exe /Set:c:\bcu\SetBios.txt
````
You will Get All the current configuration to SetBios.txt
Please Change the Bios Setting as you need in Setbios.txt

# How To Use Bios Configuration Utility to Set Bios
if you want set Bios configuration in current system
After the change in above Setbios.txt. Please run Following Command
```bash
C:\BCU\BiosConfigUtility64.exe /Set:c:\bcu\SetBios.txt
`````

# Enter the Entry in Ansible host file
copy following block to vi /etc/ansible/hosts file
```bash
[LAB]
172.16.15.10
# list of Computers IP
[LAB:vars]
ansible_ssh_user=Administrator
ansible_ssh_pass=password
ansible_ssh_port=5986
ansible_connection=winrm
ansible_winrm_server_cert_validation=ignore
````

# To Set Bios Configuration Utility in Bulk Computers

Create SetBios.yml in ansible Control Machine as below

```bash
---
- name: Change Bios of Remote PC
  hosts: LAB
  gather_facts: no
  become_method: runas

  vars:
    ansible_become_password: password

  tasks:
  - name: ping
    win_ping:

  - name: Remove BCU folder if Exist
    win_shell: rm -r c:\BCU | out-null
    ignore_errors: yes
    become: yes
    become_user: Administrator

  - name: Make BCU Directory
    win_shell: mkdir c:\BCU
    ignore_errors: yes

  - name: Copy BiosConfigurationUtility
    win_shell: wget http://ServerIP/BiosConfigUtility64.exe -OutFile C:\bcu\BiosConfigUtility64.exe

  - name: Copy config File
    win_shell: wget http://ServerIP/SetBios.txt -OutFile C:\bcu\SetBios.txt

  - name: Configure Bios of Remote PC
    win_shell: C:\bcu\BiosConfigUtility64.exe /Set:c:\bcu\SetBios.txt
    ignore_errors: yes
    become: yes
    become_user: Administrator

  - name: Remove BCU folder
    win_shell: rm -r c:\BCU | out-null
    ignore_errors: yes
    become: yes
    become_user: Administrator

  - name: Reboot Computer
    win_shell: shutdown -r -f -t 00
    become: yes
    become_user: Administrator
````
You must reboot computer after setting Bios to take effect
Also share Setbios.yml file with this repository

