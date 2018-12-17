
# Ubuntu 16.04 Server Setup Guide

This guide is written for my own future reference. These are basically notes I compiled from the [FullStack Part 1](https://frontendmasters.com/courses/full-stack/) and [FullStack Part 2](https://frontendmasters.com/courses/full-stack-v2/) courses on Frontend Masters by @jemyoung. I use **Digital Ocean** to setup a server and deploy a NodeJs using **Nginx**.


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
- **redirection operators**: `|` `>` `>>` `<` `2>`
	- `|` read from stdout
	- `>` write stdout to file
	- `>>` append stdout to file
	- `<` read from stdin
	- `2>` read from stderr
- **changing shells**:
	- `cat /etc/shells` list of acceptable shells to change to
	- `chsh -s /bin/sh` change to shell `sh`
	- `su $USERNAME` to login into new shell to see the change
	- `chsh -s /bin/bash` change shell back to bash
- **shell scripting** 
	- create a `load.sh` file with the following commands.
		- `#!/bin/sh`
		- `cat /proc/loadavg | awk '{print $1"-"$2"-"$3}'` 
		- use command `sudo chmod 755 ./load.sh` to make your script file `load.sh` an executable. See [linux chmod permissions cheat sheet](https://isabelcastillo.com/linux-chmod-permissions-cheat-sheet) for more details on changing permissions.
		- run it with `./load.sh`
	- node shell scripting
		- `mkdir ~/workspace` -> `cd~/workspace` -> `touch index.js`
		-  `npm init` and add `"bin": { "loadscript":"index.js"},` to package.json
		- `vi index.js` 
			- `#!/usr/bin/node`
				 `const exec = require('child_process').exec;`
				``const stat = exec(`cat /proc/loadavg | awk '{print $1"-"$2"-"$3}'`);``
				 `stat.stdout.on('data', function(data) { console.log(data);
	});`
			- `npm i -g` and then type`loadscript` to run the file.

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
	-  check with `sudo cat /var/log/auth.log`
7. Switch to user

## User Configurations

#### 1. Install **Nodejs** 
- `curl -sL https://deb.nodesource.com/setup_11.x | sudo -E bash -`
- `sudo apt install nodejs`
- use `npm config get prefix` to check npm directory. If it says `/usr` use the [steps in this guide](https://docs.npmjs.com/resolving-eacces-permissions-errors-when-installing-packages-globally) to fix this.
#### 2. Install **Yarn** ([yarn installation guide](https://yarnpkg.com/lang/en/docs/install/#debian-stable))
- `curl -sSL https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -`
- `echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list`
- `sudo apt-get update && sudo apt-get install yarn`
#### 3. Install **forever** `npm i -g forever` 
#### 4. Make directory `/var/www`  
`sudo mkdir -p /var/www` and `cd /var/www`, if it doesn't already exist.
#### 5. Change ownership of `/var/www`
`sudo chown -R $USER:$USER /var/www`
#### 6. Install **Git** `apt install git`
#### 7.  `git clone https://github.com/myrepo.git`  
#### 8. `cd myrepo` and `npm i`

## Server Security Configurations

####  1. Paste the public key into the `authorized_keys` folder in the `ssh` folder to enable log in through ssh.
    `cat ~/.ssh/my_key.pub | ssh $USERNAME@$SERVER_IP "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"`
#### 2. Test if you can log in to the server with the new user. `ssh -i my_key user@ipaddress`
#### 3.  If you cannot, then follow these steps:
 - `sudo nano /etc/ssh/sshd_config`
	 ```
	 PermitRootLogin yes` 
	 PasswordAuthentication yes`
	```
 - then, restart ssh service: 	`sudo service ssh restart`
 - test log in again `ssh -i my_key user@ipaddress`
#### 4. Disable root login and password login 
 - `sudo vi /etc/ssh/sshd_config`
	```
	PermitRootLogin no
	PasswordAuthentication no
	```
- `sudo service sshd restart`
#### 5. Enable automatic updates on the server
- `sudo apt install unattended-upgrades` (might already be on ubuntu) and make sure you have these configurations:
- `cat /etc/apt/apt.conf.d/20auto-upgrades`
	```
	APT::Periodic::Update-Package-Lists "1"; 
	APT::Periodic::Unattended-Upgrade "1"; 
    ```
- `vi /etc/apt/apt.conf.d/50unattended-upgrades` 
	 - comment out `"${distro_id}:${distro_codename}";` because we only want the security patches and not update our software all the time.
#### 6. Install **Fail2ban** 
- `sudo apt install fail2ban`
	```
	sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
	``` 
- `sudo vi /etc/fail2ban/jail.local` to make changes if you want but leave it as default.
- `sudo tail -f /var/log/fail2ban.log` to check who's trying to access and who got banned.

## Nginx Setup
Refer to the [nginx setup guide](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-16-04) and the [server block guide](https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-server-blocks-virtual-hosts-on-ubuntu-16-04) for more details.
#### 1. Install nginx `apt install nginx`.
#### 2. Set up server blocks for domains and sub domains.
- Make the following directories for your projects:
```
sudo mkdir -p /var/www/example.com/
sudo mkdir -p /var/www/test.com/
```
 - Change ownership of the directories to the user.
 ```
sudo chown -R $USER:$USER /var/www/example.com/
sudo chown -R $USER:$USER /var/www/test.com/
```
  - Clone the repos from github inside each folder for `example.com` and `test.com`.
```
git clone https://github.com/nida/my_repo.git
```
 - Create server block config files for both domains.
	- ```
	  sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/example.com
      ```
    - ```
	  sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/test.com
      ``` 
	- Edit the config files for each:
		    `sudo vi /etc/nginx/sites-available/example.com`
		    `sudo vi /etc/nginx/sites-available/test.com`
		- *Make sure only one server block has the  `default_server`  option enabled.*
		- Change root to `root /var/www/example.com;`
		- Modify `server_name` to `server_name example.com www.example.com;`
		- Create symlinks from these files to the `sites-enabled` directory: 
			      ```
			      sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
			      sudo ln -s /etc/nginx/sites-available/test.com /etc/nginx/sites-enabled/
		          ```
	- To avoid a possible hash bucket memory problem that can arise from adding additional server names, open nginx.conf
		     `sudo nano /etc/nginx/nginx.conf` 
		     and find the `server_names_hash_bucket_size` directive. Remove the `#` symbol to uncomment the line: `server_names_hash_bucket_size 64;`
	- Test to make sure there are no errors `sudo nginx -t`
	- Restart nginx if no problems were found `sudo systemctl restart nginx`
#### 3. Alternative if you want to edit the `default` file and run a node app:
   Edit the `default` file in `/etc/nginx/sites-available/default` and change the line of `try_files $uri $uri/ =404;` to `proxy_pass http://127.0.0.1:3001;`, then run `sudo nginx -t` to make sure there are no syntax errors and restart nginx with `sudo service nginx restart`. You can then use `forever start app.js` to forever run the node app.
   [See here for more details on setting up a node app for production](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-node-js-application-for-production-on-ubuntu-16-04)
#### 4. Setup the firewall
 ```
sudo ufw allow 443
sudo ufw allow ssh
sudo ufw enable
sudo ufw status	 
```
	
You should be able to see a table like below:
	  
| To | Action | From |
|-|-|-|
|443| ALLOW | Anywhere |
| 22 | ALLOW | Anywhere |
| 443 (v6) | ALLOW | Anywhere |
| 22 (v6)  | ALLOW | Anywhere |

> Note: I  had to use `sudo ufw allow 'Nginx Full'` because the firewall rules for ipv6 on port 80 were not set.


#### 5. [Setup an SSL certificate](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-16-04)
- `sudo add-apt-repository ppa:certbot/certbot`
- `sudo apt update`
- `sudo apt install python-certbot-nginx`
- `sudo certbot --nginx` (Remember to press 2)
- `sudo certbot renew --dry-run`
#### 6. Setup a `cron` jon to get the certificate renewed automatically:
 `min hr dayOfMonth month dayOfWeek echo "Hello" | pbcopy`
 e.g : `00 12 * * 1`
`* is used for every min/hr/dayOfMonth/month/dayOfWeek`

   Use [crontab.guru](https://crontab.guru) for help
- `sudo crontab -e`
- Add `00 12 * * 1 certbot renew` to the file.
 
#### 7. Enable `gzip`  if it isn't aready by opening `/etc/nginx/nginx.conf` and make sure you have these: 
```
gzip on;
gzip_disable "msie6";
gzip_comp_level 5		
```
  Once you have edited the file run these commands like before `sudo nginx -t` and `sudo service nginx reload`
  
####  8. Set up [caching](https://www.digitalocean.com/community/tutorials/how-to-implement-browser-caching-with-nginx-s-header-module-on-ubuntu-16-04) for static files to improve performance. 
-  [Web Caching Basics](https://www.digitalocean.com/community/tutorials/web-caching-basics-terminology-http-headers-and-caching-strategies)
-  [Understanding Nginx HTTP Proxying, Load Balancing, Buffering, and Caching](https://www.digitalocean.com/community/tutorials/understanding-nginx-http-proxying-load-balancing-buffering-and-caching)
- [A Guide to Caching with NGINX and NGINX Plus](https://www.nginx.com/blog/nginx-caching-guide/)
  
Inside `etc/nginx/sites-available/example.com`, add this before the server block 
```
#Expires map
map $sent_http_content_type $expires {
    default                    off;
    text/html                  epoch;
    text/css                   max;
    application/javascript     max;
    ~image/                    max;
}			  
``` 
and then inside the server block add `expires $expires;`

#### 9.  Extra cache configuration for slow files:
  Add this outside the server block:
```
proxy_cache_path /tmp/nginx levels=1:2 keys_zone=slowfile_cache:10m inactive=60m use_temp_path=off;
proxy_cache_key "$request_uri";
```
and then create a slowfile location block with these directives:
```
location /slowfile {
	proxy_cache_valid 1m;
	proxy_ignore_headers Cache-Control;
	add_header X-Proxy-Cache $upstream_cache_status;
	proxy_cache slowfile_cache;
	proxy_pass http://127.0.0.1:3001/slowfile;
}
```
#### Explanation
We use a couple of directives to setup the cache:
The first directive is `proxy_cache_path`

|Parameters for `proxy_cache_path`|Explanation|
|--|--|
|/tmp/nginx| Where we want the cache (which is a file) to live `/tmp/nginx` tmp periodically gets cleaned by the filesystem so it's a good location.|
|levels|How far and how many levels deep should the cache be within the file system i.e. `/static/cats/cute_cats/cat.jpeg` or just `static/cat1.jpeg`.
|  keys_zone| The name of the cache and its size (10m is 10 Mb). |
|  inactive| How long the cache should be inactive before it gets deleted. |
|  use_temp_path = off| Instead of writing to a temporary file and then writing to the cache, keep it in memory and write it to cache. (If we have a lot of requests coming in we wouldn't set it to off) |
 
#### Note about inactive:
>  A file that has not been requested for 60 minutes is automatically deleted from the cache by the cache manager process, regardless of whether or not it has expired. The default value is 10 minutes (`10m`). Inactive content differs from expired content. NGINX does not automatically delete content that has expired as defined by a cache control header (`Cache-Control:max-age=120` for example). Expired (stale) content is deleted only when it has not been accessed for the time specified by `inactive`. When expired content is accessed, NGINX refreshes it from the origin server and resets the `inactive` timer. ([Source](https://www.nginx.com/blog/nginx-caching-guide/))

|Parameters for `proxy_cache_key`|Explanation|
|--|--|
|$request_uri | To set the key that will be used to store cached values. This same key is used to check whether a request can be served from the cache | 



|Directives|Explanation|
|--|--|
|proxy_cache_valid | How long will the cache be valid before we expire it (1m is 1 minute). | 
|proxy_ignore_headers | If you send `Cache-Control` as a header, it won't read the cache and get a fresh copy every time. So we ignore the `Cache-Control` header for the browser to get a cache copy on every page hit. Chrome by default sends `Cache-Control: No Cache` when you first hit a page to try and get a fresh copy. | 
|add_header|To add headers. Here we add `X-Proxy-Cache` and set it to to the value of the `$upstream_cache_status` variable. Basically, this sets a header that allows us to see if the request resulted in a cache hit, a cache miss, expired or more (see below for more detail). This is useful for debugging. On the first page load it should be a miss because nothing is cached, the second time it should be a hit.|
|proxy_cache| The name of the cache file we are going to use|
|proxy_pass|When a request matches a location with a `proxy_pass`directive inside, the request is forwarded to the URL given by the directive. (Refer to [this guide](https://www.digitalocean.com/community/tutorials/understanding-nginx-http-proxying-load-balancing-buffering-and-caching)) |


> Beware of setting the time `proxy_cache_valid` directive for too long, because once your caching on the server, there is nothing you can do on the client side to get a fresh copy. Hard reload won't work.
> 
The following are the possible values for  [$upstream_cache_status](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#var_upstream_cache_status): 

|Values for `upstream_cache_status` | Explanation |
|--|--|
|`MISS` | The response was not found in the cache and so was fetched from an origin server. The response might then have been cached. |
|`BYPASS`|The response was fetched from the origin server instead of served from the cache because the request matched a  `proxy_cache_bypass`  directive (see  [Can I Punch a Hole Through My Cache?](https://www.nginx.com/blog/nginx-caching-guide/#caching-guide-faq-hole-punch).) The response might then have been cached.|
|`EXPIRED`|The entry in the cache has expired. The response contains fresh content from the origin server.|
|`STALE`| The content is stale because the origin server is not responding correctly, and  `proxy_cache_use_stale`  was configured.|
|`UPDATING`|The content is stale because the entry is currently being updated in response to a previous request, and  `proxy_cache_use_stale updating`  is configured.|
|`REVALIDATED`|The `[proxy_cache_revalidate](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_revalidate)`  directive was enabled and NGINX verified that the current cached content was still valid (`If-Modified-Since`or  `If-None-Match`).|
| `HIT` |The response contains valid, fresh content direct from the cache.|



Refer to the caching guide '[A Guide to Caching with NGINX and NGINX Plus](https://www.nginx.com/blog/nginx-caching-guide/)' and '[Understanding Nginx HTTP Proxying, Load Balancing, Buffering, and Caching](https://www.digitalocean.com/community/tutorials/understanding-nginx-http-proxying-load-balancing-buffering-and-caching)' for more details.

  #### 10.  Enable websockets:
  Add these two directives in the location block:
```
location / {
	proxy_set_header Upgrade $http_upgrade;
	proxy_set_header Connection "upgrade";
	.....
}
```
These headers are used to upgrade the connection from HTTP to WebSocket.
([more on websockets](https://www.nginx.com/blog/websocket-nginx/))

#### 11. Enable http/2

 Inside the `/etc/nginx/sites-available/your-domain` file find these lines:

````
listen [::]:443 ssl ipv6only=on; # managed by Certbot
listen 443 ssl; # managed by Certbot
````
and change them to these:
````
listen [::]:443 http2 ssl ipv6only=on; # managed by Certbot
listen 443 http2 ssl; # managed by Certbot
````
Check for any syntax errors through `sudo nginx -t` and reload nginx.

Refer to [this guide](https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-with-http-2-support-on-ubuntu-16-04) for more details.

#### 12. Enable Service Workers
[Enabling Service Worker in NGINX](https://blog.hasura.io/strategies-for-service-worker-caching-d66f3c828433)
....More details coming soon...


-----------
#### Some More Tutorials to Read
Server optimization:
[An Introduction to Load Testing](https://www.digitalocean.com/community/tutorials/an-introduction-to-load-testing)
[How to Benchmark a Website with Firefox, Siege, and Sproxy On Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-benchmark-a-website-with-firefox-siege-and-sproxy-on-ubuntu-16-04)
[How To Increase PageSpeed Score By Changing Your Nginx Configuration on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-increase-pagespeed-score-by-changing-your-nginx-configuration-on-ubuntu-16-04)
[How To Install and Secure Memcached on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-memcached-on-ubuntu-16-04)
For details on setting up a node or react app
[Setting up a Node App for Production](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-node-js-application-for-production-on-ubuntu-16-04)
[Deploying a React App](https://www.digitalocean.com/community/tutorials/deploying-react-applications-with-webhooks-and-slack-on-ubuntu-16-04)
