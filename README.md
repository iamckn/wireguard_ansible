# wireguard_ansible

This is the ansible automation of the Wireguard VPN set up described here https://www.ckn.io/blog/2017/11/14/wireguard-vpn-typical-setup/

This will create ten VPN client profiles when done.

This project has also been restructured as an ansible role for inclusion in other ansible playbooks.

# Requirements

This assumes an ubuntu 16.04 client. It should also work on other platforms with minimal tweaking.

## Install git
```bash
sudo apt-get install git
```

## Install ansible
```bash
sudo apt-add-repository ppa:ansible/ansible -y
sudo apt-get update && sudo apt-get install ansible -y
```

# Server set up

This assumes you have an Ubuntu 16.04 server with ssh access on port 22.
Ensure that you've already added the server key to your known hosts file by sshing into it at least once.
If you are using an SSH key, then you can forgo that.

# Quick setup

On the client

```bash
git clone https://github.com/iamckn/wireguard_ansible
cd wireguard_ansible
```

Edit the hosts file in that folder and fill in the IP field with the VPN server IP

Begin the remote installation process by running

```bash
ansible-playbook wireguard.yml -u root -k -i hosts
```

If you're using an SSH key for authentication run this instead

```bash
ansible-playbook wireguard.yml -u root -i hosts
```

Give it a few minutes and the server set up will be complete.

Ten client configs labeled client_1.conf, client_2.conf... and so on will be created in the home folder of the root user on the VPN server (/root/). They will also be downloaded to the **wireguard_role/profiles** folder on your local host.


Assuming you're using the first client config (client_1.conf), copy it to **/etc/wireguard/** and you can start using the VPN tunnel on your client.

To bring up the VPN interface 
```bash
sudo wg-quick up client_1
```


To bring down the VPN interface
```bash
sudo wg-quick down client_1
```

To view connection details
```bash
sudo wg show
```

# Advanced use

You have the option of determining the vpn network subnet you prefer your clients to use by editing the file **wireguard_role/defaults/main.yml**, and setting the vpn_network variable as desired. You can also change the vpn server port and the number of client profiles you want generated in the same file:


```bash 
vpn_network: '10.200.200'

vpn_port: '51820'

clients: 10
```

## Adding a client

If you want to generate an additional client profile in future, edit the following two variables in **wireguard_role/tasks/main.yml**:

```bash
    new_client: newclient
    new_client_ip: 10.200.200.11
```

Then run the setup process again but now with the tag **add_client** specified:

```bash
ansible-playbook wireguard.yml -u root -k -i hosts -t add_client
```

The new client config will then be downloaded to the **wireguard_role/profiles** folder on your local host.

Note: This needs to be run from the directory the initial setup was done from and not from a newly cloned one.

## Use as an ansible role

This project has been structured as an ansible role. You can therefore include it in other ansible playbooks

	- name: Setup Wireguard VPN
	  hosts: all
	  gather_facts: true
	  roles:
	    - {role: 'wireguard_role', tags: 'wireguard'}


# DNS issues

If there is another service listening on port 53, you will have issues with getting DNS resolution working.
It is therefore advisable to either disable or change the port of any service already using port 53.
An example of this is the **systemd-resolved** service on Ubuntu 18.04. You should switch off binding to port 53 by editing the file **/etc/systemd/resolved.conf** as follows:

```bash
DNSStubListener=no
```

Reboot the VPN server and DNS resolution will work as expected.