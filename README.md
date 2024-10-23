Installing PM2:

Install PM2 from npm: npm i pm2.
Check the list of apps PM2 is running: pm2 list.
Start your app with PM2: pm2 start <filePath>.
Setting up Nginx Web Server for Reverse
Proxying

1. Install and verify NGINX:

Install NGINX: apt install nginx.
Check NGINX status: systemctl status nginx.
Start NGINX if not active: systemctl start nginx.

Configuring Nginx Server Blocks
1. Create a custom path for the web server block:
Create a new directory: mkdir -p /var/www/<domain>.TLD/html.
Create an index file: nano index.html.
Create a server block file: nano /etc/nginx/sites-available/<domain-name>.

2. Server block configuration:
server {
listen 80;
listen [::]:80;
root /var/www/<example>.TLD/html;
index index.html index.htm index.nginx-debian.html;
server_name <example>.TLD www.<example>.TLD;
location / {
try_files $uri $uri/ =404;
}
}

3. Enable the server block:
Add a soft link from sites-enabled to sites-available: ln -s /etc/nginx/sitesavailable/<
domain-name> /etc/nginx/sites-enabled/.
Test the configuration: nginx -t.
Restart NGINX: systemctl restart nginx.
Reverse proxying node app with NGINX
nano etc/nginx/sites-available/4
We have to put the location tag in the file that we wrote earlier. Now write this in the location tag ->
location / {
proxy_pass http://localhost:<PORT>;
proxy_http_version 1.1;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection 'upgrade';
proxy_set_header Host $host;
proxy_set_header X-Forwarded-For $remote_addr;
proxy_cache_bypass $http_upgrade;
}
Now you can remove the PORT 3000 (or whatever your PORT is) from the Firewall Rules section of the VPC
networks.
Now your app is only accessible via the domain, without any port. It won’t be accessible via the PORT number.

Serving Multiple Node Apps in Single Host
1. Run multiple apps on different PORT numbers:
Deploy your additional node apps in the server as root.
Ensure the routes of the apps do not conflict.
Add a prefix to the second app if necessary.

2. Reverse proxy traffic for multiple apps:
location /<new-route> {
proxy_pass http://localhost:<new-PORT>;
proxy_http_version 1.1;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection 'upgrade';
proxy_set_header Host $host;
proxy_cache_bypass $http_upgrade;
}

3. Restart NGINX:
Save the configuration file.
Test the configuration: nginx -t.
Restart NGINX: systemctl restart nginx.

Configuring and Installing SSL Certificates
with CertBot
1. Install CertBot:
Update packages: apt-get update, apt-get upgrade.
Add repositories: add-apt-repository universe, add-apt-repository
ppa:certbot/certbot.
Install CertBot: apt-get install certbot python-certbot-nginx.
2. Generate and apply SSL certificates:
Run CertBot: certbot --nginx.
Follow the instructions on the TLDmand line.
3. Set up automatic renewal of SSL certificates:
Test the renewal process: certbot renew --dry-run.

Redirecting IP to Domain
1. Add a server block to redirect IP to the domain:
server {
listen 80;
server_name <IP address>;
return 301 https://<domain-name>$request_uri;
}
2. Add a server block to drop IP access:
server {
listen 80;
server_name <IP address>;
return 444;
}

Setting up Sub-Domains
1. Add a record set for the subdomain in Cloud DNS:
Add your subdomain (e.g., app.).
Use the IP of your current VM or another VM for another app.
2. Reverse-proxying for sub-domains:
server {
listen 80;
listen [::]:80;
root /var/www/<example.TLD>/html;
index index.html index.htm index.nginx-debian.html;
server_name <sub-domain>.<example.TLD>;
location / {
proxy_pass http://localhost:<PORT>;
proxy_http_version 1.1;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection 'upgrade';
proxy_set_header Host $host;
proxy_cache_bypass $http_upgrade;
}
}
3. Restart NGINX:
Test the configuration: nginx -t.
Restart NGINX: systemctl restart nginx.
