# Ubuntu 16.04 Server Setup Guide

This guide uses **Digital Ocean** to setup a server and deploy a React app with automated deployment using **nginx** and **TravisCI**.


## Basics to know (can be skipped)

 - **traceroute** - use this to check which part of the network layer is down.
 - **vi** - use [vim cheat sheet](https://vim.rtorr.com/) for the full list of basic commands.
 - use [Explain Shell](https://explainshell.com) for commands you don't understand.
 - **iptables basics**:
	 - `sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT`
		 - **-A** append rule
		 - **-p** protocol (tcp, icmp)
		 -  **--dport** destination port
		 - **-j** jump (DROP, REJECT, ACCEPT, LOG)
	- `sudo iptables -A OUTPUT -p tcp --dport 80 -j REJECT` *Creates an iptable rule to block all outgoing HTTP connections* 
	-  `iptables -A INPUT -s 192.0.0.1 -p icmp --dport 892 -j ACCEPT` *allows icmp connections on port 892 from the IP address 192.0.0.1*
- easier way to open and close ports through**ufw** 
	- `sudo ufw allow ssh`
	- `sudo ufw enable`
	- `sudo reject out http`
- **find** [example guide](https://www.lifewire.com/uses-of-linux-command-find-2201100#billboard3-sticky_1-0)
	- searches file names
	- `find /directory -name foo.txt`
	- some useful options
		- -name
		-  -type
		-  -empty
		-  -executable
		-  -writable  
-  **grep**
	- searches file contents
	- `grep -i 'search-expression' /directory`
	- `zgrep file` to search inside a gzip file
	- `ps aux | grep node` 
- redirection operators: `|` `>` `>>` `<` `2>`
	- `|` read from stdout
	- `>` write stdout to file
	- `>>` append stdout to file
	- `<` read from stdin
	- `2>` read from stderr

## Basics to do beforehand

 - **ssh key** - use `ssh-keygen -t rsa` to generate a public and private rsa key pair if you don't already have one.
 - install **nmap** `brew install nmap`
 
## Initial Server Setup

 1. Create a droplet on Digital Ocean
 2. Log in as root. `ssh -i ~/.ssh/my_key root@$YOUR_SERVER_IP` 
 3. Change the root password. `passwd` 
 4. Use `apt update` and `apt upgrade` to upgrade all the local libraries.
 5. `apt install -y htop` to use **htop** instead of top.
 6. Create user "nida" and give it sudo permissions
	 - `adduser nida` 
	 - `usermod -aG sudo nida`
	 - check with `sudo cat /var/log/auth.log`
7. Switch to user

## User Configurations

1. Install **Nodejs** 
	- `curl -sL https://deb.nodesource.com/setup_11.x | sudo -E bash -`
	- `sudo apt install nodejs`
	- use `npm config get prefix` to check npm directory. If it says `/usr` use the [steps in this guide](https://docs.npmjs.com/resolving-eacces-permissions-errors-when-installing-packages-globally) to fix this.
2. Install **Yarn** ([yarn installation guide](https://yarnpkg.com/lang/en/docs/install/#debian-stable))
	- `curl -sSL https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -`
	- `echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list`
	- `sudo apt-get update && sudo apt-get install yarn`
3. Install **forever** `npm i -g forever` 
4. Make directory `/var/www`  
	`sudo mkdir -p /var/www` and `cd /var/www`, if it doesn't already exist.
6. Change ownership of `/var/www`
`sudo chown -R $USER:$USER /var/www`
7. Install **Git** `apt install git`
8.  `git clone https://github.com/myrepo.git`  
9. `cd myrepo` and `npm i`

## Server Security Configurations

 1. Paste the public key into the `authorized_keys` folder in the `ssh` folder to enable log in through ssh.
    `cat ~/.ssh/my_key.pub | ssh $USERNAME@$SERVER_IP "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"`
2. Test if you can log in to the server with the new user. `ssh -i my_key user@ipaddress`
3.  If you cannot, then follow these steps:
	- `sudo nano /etc/ssh/sshd_config`
		- `PermitRootLogin yes` 
		- `PasswordAuthentication yes`
	- then, restart ssh service:
	`sudo service ssh restart`
	- test log in again `ssh -i my_key user@ipaddress`
4. Disable root login and password login
	- `sudo vi /etc/ssh/sshd_config`
		- `PermitRootLogin no`
		- `PasswordAuthentication no`
	- `sudo service sshd restart`
5. Enable automatic updates on the server
	- `sudo apt install unattended-upgrades` (might already be on ubuntu) and make sure you have these configurations:
	- `cat /etc/apt/apt.conf.d/20auto-upgrades`
		- `APT::Periodic::Update-Package-Lists "1"; `
		- `APT::Periodic::Unattended-Upgrade "1"; `
	- `vi /etc/apt/apt.conf.d/50unattended-upgrades` 
		- comment out `"${distro_id}:${distro_codename}";` because we only want the security patches and not update our software all the time.
6. Install **Fail2ban** 
	- `sudo apt install fail2ban`
	- `sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local` 
	- `sudo vi /etc/fail2ban/jail.local` to make changes if you want but leave it as default.
	- `sudo tail -f /var/log/fail2ban.log` to check who's trying to access and who got banned.

## Nginx Setup

