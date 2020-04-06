**Simple setup openVPN on VPS/VDS server.** </br >
**The configuration will be carried out on the example of Ubuntu server 18.04.**

*If you do not have an ssh key, you will need to generate it on the local machine and add the public key to your hosting control panel.*

* *The first time you access the server, you need to update the system:*

   `ssh root@server_domain_or_IP`

   `apt update && apt upgrade`

   reboot the system if necessary:

   `reboot` or `shutdown now -r`

* :exclamation: *If you are a root user, remember that you may lose data and access to the server.* :exclamation: </br >

   *Therefore, it is recommended to have an individual user:*

   `apt install sudo`

   Add User (where 'user' is the username):

   `adduser user`

   Adding a user to the sudo group:

   `usermod -aG sudo user`

   Login with user account:

   `su - user`
   
   
* *UFW setup:*
   
   Open SSH Access; </br >
   
   :exclamation: If you changed the default port 22 for SSH, also add it to the UFW rule. :exclamation:
   
   Also add port 1194/udp for OpenVPN to the rule.
      
   `ufw allow OpenSSH`
   
   `ufw allow 1194/udp`
   
   Edit the config:
   
   `nano /etc/default/ufw`
   
   Edit the line to 'ACCEPT':
   
   >DEFAULT_FORWARD_POLICY="ACCEPT"
   
   Enabling UFW </br >
   
   :exclamation: Before enabling UFW, make sure that you open the ports you need :exclamation:
   
   `ufw enable`
   
   To view the status of UFW:
   
   `ufw status`
   
* *Setup SSH:*

   `nano /etc/ssh/sshd_config`

   Disable login with blank password:

   >PermitEmptyPasswords no

   Disable user root login:

   >PermitRootLogin no

   It is also recommended that you change the default port 22. </br >
   
  :exclamation: Use this item with caution; the changed port must be specified in the URW rules. :exclamation:

   >Port 22
   
   To apply the settings, you must restart the SSH service:
   
   `service ssh restart`
   
* *Setup OpenVPN:*

   `apt install openvpn easy-rsa`
   
   Add or [server.conf](https://github.com/de956/openVPN/blob/master/server.conf) to the directory:
   
   /etc/openvpn/server.conf
   
   Enable IP forwarding on the server:
   
   `echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf`
   
   `sysctl -p`
   
   View network interface name:
   
   `ifconfig`
   
   Masquerade the internet traffic coming from the VPN network (10.8.0.0/24) to systems local network interface (eth0). Where 10.8.0.0 is VPN network and eth0 is the network interface of system:
   
   `modprobe iptable_nat`
   
   `iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE`
   
   Adding trusted certificates for servers and clients:
   
   `make-cadir /etc/openvpn/openvpn-ca/`
   
   `cd /etc/openvpn/openvpn-ca/`
   
   Edit vars file:
   
   `nano vars`
   
   Update the below values as required, but do not leave them empty:
   
   >export KEY_COUNTRY="US" </br >
   >export KEY_PROVINCE="CA" </br >
   >export KEY_CITY="SanFrancisco" </br >
   >export KEY_ORG="MyOrg" </br >
   >export KEY_EMAIL="info@myorg.com" </br >
   >export KEY_OU="Security" </br >
   
   Create a symlink for openssl.cnf file: </br >
   * *There may be several different versions of the openssl-x.x.x.cnf configuration file in the / etc / openvpn / easy-rsa directory.*
   
   
   `ln -s openssl-1.0.0.cnf openssl.cnf`
   
   Load the values:
   
   `source vars`
   
   The script will execute without errors, we will see the following:
   
   >NOTE: If you run ./clean-all, I will be doing a rm -rf on /etc/openvpn/easy-rsa/keys
   
   
   
   





   


   
   
   
   
   
   
   





