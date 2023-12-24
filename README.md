# ftb-server
FTB Modpack Debian Linux Server  
From a fresh install of Debian 12  

## Debian setup (from root)
```bash
apt update && apt upgrade -y
```
### Install sudo, curl, tmux, vim and Java  
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
#### Configure a static IP (optional)  
```bash
sudo cp /etc/network/interfaces /etc/network/interfaces.orig && \
echo -e "auto lo eno1\niface lo inet loopback\n\niface eno1 inet static\n    address 192.168.0.10\n    netmask 255.255.255.0\n    gateway 192.168.0.1\n    dns-nameservers 1.1.1.1" \
| sudo tee -a /etc/network/interfaces
```
The above outputs to this:  
```
auto lo eno1
iface lo inet loopback

iface eno1 inet static
    address 192.168.0.10
    netmask 255.255.255.0
    gateway 192.168.0.1
    dns-nameservers 1.1.1.1
```
### Restart and login to the new IP
```bash
sudo systemctl reboot
```

### Add your ssh key for easier login (optoinal)
#### Create a key if you don't have one 
From a Windows CMD  
```
ssh-keygen -t ed25519 -C "email"
```
Copy the public key   
```
start notepad .ssh\id_ed25519.pub
```
#### Back on the Debian host
```bash
mkdir -p $HOME/.ssh && touch $HOME/.ssh/authorized_keys && \
echo "ssh-ed25519 random-string email" | tee $HOME/.ssh/authorized_keys
```
Logout and back in to verify  

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
sudo -u minecraft -H sh -c "cd /srv/minecraft/ftb_skies; ./serverinstall_*"
```
#### Remove the install script
```bash
sudo rm /srv/minecraft/ftb_skies/serverinstall_*
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

## Editing Server files
First, stop the server
```bash
sudo systemctl stop ftb-skies
```
server.properties, for enabling flight, e.g...
```bash
sudo -u minecraft -H sh -c "vim /srv/minecraft/ftb_skies/server.properties"
```

## Security
### Debian Firewall
### Whitelisting