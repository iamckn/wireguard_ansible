# wireguard_ansible

This is the ansible automation of the Wireguard VPN set up described here https://www.ckn.io/blog/2017/11/14/wireguard-vpn-typical-setup/
This will create ten VPN client profiles when done.

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

This guide assumes assumes your VPN server's internet facing interface is **eth0**. If otherwise change the interface name in line **59** of the **firewall.yml** file to what you have.

# Usage

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

Ten client configs labeled one.conf, two.conf... and so on will be created in the home folder of the root user on the VPN server (/root).

You can then move them to your clients using scp.

Assuming you're using the first client config (one.conf), copy it to **/etc/wireguard/** and you can start using the VPN tunnel on your client.

To bring up the VPN interface 
```bash
sudo wg-quick up one
```


To bring down the VPN interface
```bash
sudo wg-quick down one
```

To view connection details
```bash
sudo wg show
```

More details on the setup can be found in the post referenced above.
