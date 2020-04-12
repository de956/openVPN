**Simple setup openVPN on VPS/VDS server.** </br >
**The configuration will be carried out on the example of Ubuntu server 18.04.**

* *If you do not have an ssh key, you will need to generate it on the local machine and add the public key to your hosting control panel.*

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
   
  :exclamation: Use this item with caution; the changed port must be specified in the UFW rules. :exclamation:

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
   
   * *You may also need to generate openssl certificates:* </br >
   
   `openssl req -newkey rsa:2048 -new -nodes -keyout key.pem -out csr.pem`
   
   `openssl x509 -req -days 365 -in csr.pem -signkey key.pem -out server.crt`
   
   Load the values:
   
   `source vars`
   
   The script will execute without errors, we will see the following:
   
   >NOTE: If you run ./clean-all, I will be doing a rm -rf on /etc/openvpn/easy-rsa/keys
      
   Now use:
   
   `./clean-all`
   
   `./build-ca`
   
   Generate server certificate files:
   
   `cd /etc/openvpn/openvpn-ca/`
   
   `./build-key-server server`
   
   Generate a strong Diffie-Hellman key:
   
   `openssl dhparam -out /etc/openvpn/dh2048.pem 2048`
   
   Make more secure TLS integrity verification capabilities of the server and copy them to /etc/openvpn directory:
   
   `openvpn --genkey --secret /etc/openvpn/openvpn-ca/keys/ta.key`
   
   `cd /etc/openvpn/openvpn-ca/keys`
   
   `cp ca.crt ta.key server.crt server.key /etc/openvpn`
   
   Let’s start the OpenVPN service:
   
   `systemctl start openvpn@server`
   
   `systemctl status openvpn@server`
   
   There should be no errors in the output, and the final line of the log should report successful work:
   
   >... Initialization Sequence Completed
   
   You should also make sure that there is a new network interface named tun0:
   
   `ifconfig` or `ifconfig tun0`
   
   Generate client configuration:
   
   `mkdir /etc/openvpn/client`
   
   `cd /etc/openvpn/client`
   
   Add [make-vpn.sh](https://github.com/de956/openVPN/blob/master/make-vpn.sh) script to the directory.
   
   Modify the string to your IP server address:
   
   `OPENVPN_SERVER="your_ip"`
   
   Set the execute permission:
   
   `chmod +x ./make-vpn.sh`
   
   Now generate the client file by passing its name in the line parameters:
   
   `./make-vpn.sh myopenvpn`
   
    Transfer the .ovpn file to the local machine, go to the folder with the file:
    
    `sudo openvpn --auth-nocache --config  myopenvpn.ovpn`
    
    You can also install the add-on for Network Manager:
    
    `sudo apt-get install network-manager-openvpn-gnome`
    
    Then create a new connection and import the .ovpn configuration file for VPN !
    
    Starting with the opeVPN v2.3.9+, support for secure DNS queries is
    enabled. To do this, add the following line in the generated .ovpn file:
    
    `block-outside-dns`
    
    ---
    
* *An alternative use of openVPN is:* [PiVPN](https://pivpn.io/)
    
    * PiVPN easier to install and configure and also supports OpenVPN 2.4 / WireGuard.
    
    To install, run the command:
    
    `apt install openvpn`
    
    `curl -L https://install.pivpn.io | bash`
    
    More info https://github.com/pivpn/pivpn
    
    ---
    
* *To block ads and statistics, you can use:* [Pi-hole](https://pi-hole.net/)
    
    To install, run the command:
    
    `curl -sSL https://install.pi-hole.net | bash`

     More info https://github.com/pi-hole/pi-hole
     
     [The Big Blocklist Collection](https://firebog.net)
     
     ---
     
* *It is possible to use DNS over TLS (DoT) encryption:*
     
     [Stubby](https://github.com/getdnsapi/stubby) or [Unbound](https://github.com/NLnetLabs/unbound)
     
     :exclamation: Before configuring, you must have your ports free (if for example you are already using Pi-hole) 
     and update the rules for UFW :exclamation:
     
     
