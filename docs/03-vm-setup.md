#Virtual machine Setup
This section documents the creation and configuration of the vm used for the rule-based HIDS.
Three vm were created:
- Wazuh Manager (Ubuntu OS)
- Wazuh Client (Ubuntu OS)
- Attack vm (Kali Linux) 

##Virtual Machine Specifications
WazuhManager:
- RAM: 6GB
- CPU: 2 cores
- Disk: 40GB
- IP: 192.168.0.42
WazuhClient:
- RAM:4GB
- CPU: 1 core
- Disk: 30GB
- IP:192.168.0.78
AtckVM:
- RAM: 4GB
- CPU: 1 core
- Disk: 30GB
- IP: 192.168.0.247

##Steps of installing and configuring

###1 Installation of VMs:
Creating a new VM -> machine + new
Enter VM name
Select the iso image, can be found in ubuntu and kali website
Assigning the correct components for each VM (RAM, CPU, Storage)
Set Network to be briged for each VM
Launch the VM and follow the default configurations

###2 after the installations
Ubuntu VM:
- sudo apt update && sudo apt upgrade -y
- sudo apt install net-tools -y (ifconfig)
- enable ssh:
  - sudo apt install openssh-server -y
  - sudo systemctl enable ssh
  - sudo systemctl start ssh
Kali VM:
- sudo apt update && sudo apt upgrade -y

ifconfig - for all vm to discover the ip addresses

Test connection:
- ping <ip address>
