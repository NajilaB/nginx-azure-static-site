TO BE CONTINUED ... last modified 23/11/2026 

PRACTICING SERVING A WEBSITE USING VM AND NGINX AS A WEB SERVER.

First things first 

I have created a small game-self-contained-like html file using AI.
The game is a type-the-words-kind of mini-game.
After copying the .html locally I then created the projects folder to get things started using WSL.

Installed Nginx;
Located Nginx configuration files;
set the website root to /var/www/mywebsite;
configured listen to port 80, 22 and 443;
configured server name after acquiring a sub-domain from DUCKDNS
self-signed certification
Configured Firewall 
Test the Nginx configuration for validation
Reloaded Nginx and run it locally to verify integrity 

PROBLEMS
I run into a few problems, Nginx worked well on the computer, ports and firewall rules were set correctly and even after turning off firewall I could not get it to work on my personal phone.
Computer and phone was on the same network.
After a few failed attempts at getting it to work I created a Azure subscription and followed the same steps.

Creation of VM  
VM details: Standard B2ats v2 - vCpus 2 -RAM 1 GiB - Ubuntu image

Ran into a few problems. Azure has quota limits due to the abuse of cryptocurrency bots, which limited the use of "B" virtual machines-cheaper than others. 
Requested extra use of the VM for this project.

Opened ports
Add Inbound rules 22 to SSH in, 80 for HTTP requests and 443 for later HTTPS

Copied local html files to the VM using: 
scp -i wherekeyis wherelocalfileis azureuseraandpublicvmip:~/pathinvm
scp i ~/.ssh/az_lab.pem index.html azureuser@20.xxx.xxx.65:~/projeto/gamelab

Finding where nginx files are: 
etc/nginx/sites-available/default
Open default file using sudo vim, nano or vi

CHANGES TO BE MADE INSIDE DEFAULT 
* SET ROOT /VAR/WWW/
set root /var/www/mywebsite - /mywebsite here is the folder you create yourself inside the /var/www/ containing your website files e.g. index.html, main.css, JS

* SERVER_NAME - Add your own domain 
remove the # Hashtags from both 443 SSL default server
add the ssl_certificate keys after generating one
$ sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -subj "/CN=localhost" -keyout /etc/ssl/private/mywebsite.key -out /etc/ssl/certs/mywebsite.crt

Test Nginx configuration
$ sudo nginx -t && sudo systemctl reload nginx
Expected results: syntax is ok / test is successful

Testing it locally
curl -kI https://yourdomain.org 
Expect results : HTTP/1.1 200 
