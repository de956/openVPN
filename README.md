**Simple setup openVPN on VPS/VDS server.
The configuration will be carried out on the example of Ubuntu server 18.04.**

>If you do not have an ssh key, you will need to generate it on the local machine and add the public key to your hosting control panel.

* *The first time you access the server, you need to update the system:*

   `ssh root@server_domain_or_IP`

   `apt update && apt upgrade`

   reboot the system if necessary:

   `reboot` or `shutdown now -r`

* :exclamation: *If you are a root user, remember that you may lose data and access to the server.*
*Therefore, it is recommended to have an individual user:*

   `apt install sudo`

   Add User (where 'user' is the username):

   `adduser user`

   Adding a user to the sudo group:

   `usermod -aG sudo user`

   Login with user account:

   `su - user`
   
   
* *UFW setup:*
   
   Open SSH Access;
   
   :exclamation: If you changed the default port 22 for SSH, also add it to the UFW rule.
   
   Also add port 1194/udp for OpenVPN to the rule.
      
   `ufw allow OpenSSH`
   
   `ufw allow 1194/udp`
   
   Edit the config:
   
   `nano /etc/default/ufw`
   
   Edit the line to 'ACCEPT':
   
   >DEFAULT_FORWARD_POLICY="ACCEPT"
   
   Enabling UFW (:exclamation: Before enabling UFW, make sure that you open the ports you need):
   
   `ufw enable`
   
   To view the status of UFW:
   
   `ufw status`
   
* *Setup SSH:*
   
   
   
   
   
   
   
   





