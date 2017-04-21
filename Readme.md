# Squid Proxy Server
This will install Squid Proxy Server and configure additional IP addresses to it.

## Prerequisites

We need to have a controller node/vps where you'll be running commands to launch new squid proxy servers. We have set up that server for you. You just need to follow instructions given below to get it done. The choice of client environment is Windows in this document.

### Login to Controller Node
**Below Mentioned Steps are performed on ```on  Windows Machine```.**

PuTTY is an SSH and telnet client for the Windows platform.
1. Download [Putty.exe](http://www.putty.org/).
2. Run the ```Putty.exe``` file
3. Add the Public IP address of Ansible Controller Node to Putty and connect.
```
Default username is root.
Password as given on your VPS provider.
```

## Bootstrap

### Login to New Squid Server Node
**Below Mentioned Steps are performed on ```on Windows Machine```.**

PuTTY is an SSH and telnet client, developed originally by Simon Tatham for the Windows platform.
1. Download [Putty.exe](http://www.putty.org/).
2. Run the ```Putty.exe``` file
3. Add the Public IP address of Squid Server Node to Putty and connect.
```
Default username is root.
Password is given by cloud provider.
```

### To Add Newly launched Squid/Proxy Server Node on Controller
Use below mentioned commands to Bootstrap the New Proxy/Squid Server into controller.

**Below Mentioned Steps are performed on ```Ansible Controller Node```.**
1. Login to ```Ansible Controller Node``` using above defined method.
2. Create SSH-key in Ansible Controller Node (if does not Exist).
```
sudo su root
ssh-keygen
```
follow the instructions of `ssh-keygen` to create key.

3. Once key is created, we need to copy it into other node. Copy the output of below mentioned commands.
```
sudo su root
cat ~/.ssh/id_rsa.pub
```

**Below Mentioned Steps are performed on ```New Proxy/Squid Server``` Node.**

* Login/SSH to ```New Proxy/Squid Server``` using just like we did before.
* Save the previously copied content of key to ```/root/.ssh/authorized_keys``` file on New Proxy/Squid Server Node/VPS. Use the below mentioned commands to complete the bootstrapping process.
```
mkdir /root/.ssh
```
This command will Create Folder in Proxy Server.
```
touch /root/.ssh/authorized_keys
```
This command will create an empty file.
```
nano /root/.ssh/authorized_keys
```
This command will open ```nano editor```. Now Paste the content of last command that we performed on ```Controller Node```.


* Disable/Stop Firewalld Service
We need to have firewall disabled to install Squid Proxy Server. Run commands mentioned below to stop the firewalld service on ```New Squid/Proxy Server```
```
service firewalld stop
systemctl disable firewalld
```
4. We also need to have Seliunx disabled on ```New Squid/Proxy Server```.
By Default it is disabled by vultr cloud provider. We just need to confirm it by using below mentioned command.
```
getenforce
```
Output of above mentioned command should be
```
Disabled
```

Add the Public IP address of Squid Proxy Server to ```inventory``` file under ```squid-server``` block.


#### Checking Connectivity
```
ansible -m ping squid-server -i inventory
```
Output Should be like this
```
xxx.xxx.xxx.xxx | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## Configuration

This section contains configurable options which can be edited if required.

**Following Steps are performed on ```Ansible Controller Node```.**

Set Squid Port, allowed_ip_address_ranges in ```group_vars/all.yml```
* **squid_port:** 3128

This Default Port of Squid Proxy Server but we can change it any other number from ```(1025-65535)```

* **allowed_ip_address_ranges:** ["192.168.56.1/32","172.16.0.1"]

we can add any number of ip address either in ```range format or single ip address``` both are supported.
```Above mentioned IP addresses are only allowed to authenticate to Squid Proxy Server.```

## Authentication


## Installation


Run the below Mentioned to install proxy server on remote instance/node.
```
ansible-playbook main.yml -i inventory
```


## Backup
** In order to backup squid proxy server, following steps are performed on ```Ansible Controller Node```.**

This will take backup of squid configuration file to specified location on proxy server.
Set Squid backup path in ```group_vars/all.yml```
* **squid_backup_path:** /etc/squid

```/etc/squid``` is default path for backup location but we can change to any other location on server.

Run using below mentioned command.
```
ansible-playbook backup.yml -i inventory
```

## Adding Additional IP Addresses to Squid Server
**Below Mentioned Steps are Performed on ```Ansible Controller Node```.**
This will add addtional ip address to squid proxy.
```
IP addresses and Netmask addresses must be in sequential order.
These IP Address and Netmask Address will be provided by VPS provider,
when you add additional IP addresses to Squid/Proxy Server.
```

Add Squid additional addresses and netmask in ```roles/sec-address/vars/main.yml```
* **list_of_ip_address:** ["45.32.202.35","45.76.234.57"]
* **list_of_netmask_address:** ["255.255.252.0","255.255.254.0"]

Below mentioned variable is total number of secondary IP addresses you want to add. It will be count of above mentioned ip addresses (```list_of_ip_address```).
* **total_number_additional_address:** 2


Run below mentioned command.
```
ansible-playbook sec.yml -i inventory
```
After Running above mentioned command.

```Restart your VPS server from cloud provider console to take effect.```

## Test
Add Public IP addresses of Squid/Proxy Node to firefox with port number 3128 in proxy configuration.

Every client have their own way of configuring proxy, Covering all of them is beyond scope of this document. We are covering Firefox to use our squid proxy below:

1. Open firefox
2. Go to settings/preferences
3. Go to Advanced tab from left side
4. Then select the Network Setting from tab menu
5. open Connection settings
6. Select the Manual Proxy  Radio Button
7. Add the Public IP address and Port Number 3128
8. check the ```Use this proxy server for all protocols``` button
9. save the settings it .
10. Now try to open any website it will go through Proxy Server

## Notes*
```
* Ansible 2.2.1.0 or higher version.
* All roles/tasks are tested on Centos 7.
* All/Squid Proxy port should be allowed from your VPS provider firewall.
```
