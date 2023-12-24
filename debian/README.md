# FTB Modpack Server Setup for Debian 12
[Home](/README.md)  

---
From a fresh install of Debian 12  
## Initial setup (from root)
```bash
apt update && apt upgrade -y
```
### Install sudo, curl, tmux, vim (optional) and Java  
```bash
apt install sudo curl vim tmux default-jre
```
### Add yourself to sudo  
```bash
usermod -a -G sudo `whoami`
```
### Logout and back in as you  
```bash
exit
```
Maybe setup your shell...  
#### Configure a static IP (optional)  
```bash
sudo cp /etc/network/interfaces /etc/network/interfaces.orig && sudo vim /etc/network/interfaces
```
Replace with (adjust accordingly - interface name / IP scheme):  
```bash
auto lo eno1
iface lo inet loopback

iface eno1 inet static
    address 192.168.0.10
    netmask 255.255.255.0
    gateway 192.168.0.1
    dns-nameservers 1.1.1.1
```
Restart and login to the new IP  
You could just restart the service and log back in. I like to verify everything survives a reboot  
```bash
sudo systemctl reboot
```

## FTB Modpack Server Installation
#### Create a system user to run the programs. System users have no password, home directory and can't login  
```bash
sudo useradd -r minecraft
```
#### Server Files
https://feed-the-beast.com/modpacks/server-files  
### FTB Skies Example  
#### Create a folder for the files and download the install script
```bash
sudo mkdir -p /srv/minecraft/ftb_skies && sudo chown -R minecraft:minecraft /srv/minecraft && \
sudo usermod -d /srv/minecraft minecraft && \
sudo -u minecraft -H sh -c "cd /srv/minecraft/ftb_skies; curl -JLO 'https://api.modpacks.ch/public/modpack/103/11446/server/linux' && chmod +x serverinstall_103_11446"
```
#### Run the install script. Accept all the defaults
```bash
sudo -u minecraft -H sh -c "cd /srv/minecraft/ftb_skies; ./serverinstall_* --auto"
```
#### Remove the install script (optional)
```bash
sudo rm /srv/minecraft/ftb_skies/serverinstall_*
```
You can keep the script and use it to update the server without downloading a newer script version  
```bash
sudo -u minecraft -H sh -c "cd /srv/minecraft/ftb_skies; ./serverinstall_* --auto --latest"
```
## Systemd Control Scripts  
#### Run the server manually once to accept the EULA and verify it works
```bash
sudo -u minecraft -H sh -c '/usr/bin/bash /srv/minecraft/ftb_skies/start.sh'
```
Stop the server
```bash
stop
```
### Create the systemd script
Create a systemd service file
```bash
sudo touch /etc/systemd/system/ftb-skies.service && sudo chmod +x /etc/systemd/system/ftb-skies.service
```
Copy and paste from the example.service file. Adjust accordingly   
```bash
sudo vim /etc/systemd/system/ftb-skies.service
```
Reload Systemd  
```bash
sudo systemctl daemon-reload
```
Start it  
```bash
sudo systemctl start ftb-skies
```
### Verify it's running
```bash
sudo -u minecraft -H sh -c "tmux -L minecraft attach -t ftb-skies"
```
Ctrl-b then d to detatch  
### Enable it
```bash
sudo systemctl enable ftb-skies
```

## Netfilter (firewall)
Edit /etc/nftables.conf
```bash
sudo cp /etc/nftables.conf /etc/nftables.conf.orig && sudo vim /etc/nftables.conf
```
Replace with:  
```bash
#!/usr/sbin/nft -f

flush ruleset

table inet filter {
    chain input {
        type filter hook input priority 0; policy drop;
        iif lo counter accept comment "accept loopback";
        ct state established,related counter accept comment "accept established,related";
        ct state invalid counter drop comment "drop invalid packets";

        # basic things needed for IPv6 to function correctly. optional
        icmpv6 type { nd-neighbor-solicit, nd-router-advert, nd-neighbor-advert } counter accept;

        # icmp / ping. optional
        ip6 nexthdr icmpv6 icmpv6 type echo-request limit rate 3/second counter accept;
        ip protocol icmp icmp type echo-request limit rate 3/second counter accept;

        tcp dport ssh counter accept comment "accept ssh";
        tcp dport 25565 counter accept comment "accept minecraft";
        counter comment "input drops"
    }
    chain forward {
        type filter hook forward priority 0; policy drop;
        counter comment "forward drops"
    }
    chain output {
        type filter hook output priority 0; policy accept;
        counter comment "output packets"
    }
}
```
Start and enable    
```bash
sudo systemctl start nftables && sudo systemctl enable nftables
```

### Editing server.properties
Before editing anything, stop the server  
```bash
sudo systemctl stop ftb-skies
``` 
```bash
sudo -u minecraft -H sh -c "vim /srv/minecraft/ftb_skies/server.properties"
```
### Editing user_jvm_args.txt
Edit server.properties  
```bash
sudo -u minecraft -H sh -c "vim /srv/minecraft/ftb_skies/user_jvm_args.txt"
```
### Restart the server
```bash
sudo systemctl start ftb-skies
``` 

### Whitelisting
Attach to the tmux session  
```bash
sudo -u minecraft -H sh -c "tmux -L minecraft attach -t ftb-skies"
```
Add the user  
```
whtielist add <usrename>
```