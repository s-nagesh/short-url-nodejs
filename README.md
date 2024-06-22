# Node.js Deployment

> Steps to deploy a Node.js app to DigitalOcean using PM2, NGINX as a reverse proxy and an SSL from LetsEncrypt

## 1. Create Free AWS Account
Create free AWS Account at https://aws.amazon.com/

## 2. Create and Lauch an EC2 instance and SSH into machine
I would be creating a t2.medium ubuntu machine for this demo.

## 3. Install Node and NPM
```
curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install nodejs

node --version
```

## 4. Clone your project from Github
```
git clone https://github.com/s-nagesh/short-url-nodejs.git
```

## 5. Install dependencies and test app
```
sudo npm i pm2 -g
pm2 start index

# Other pm2 commands
pm2 show app
pm2 status
pm2 restart app
pm2 stop app
pm2 logs (Show log stream)
pm2 flush (Clear logs)

# To make sure app starts when reboot
pm2 startup ubuntu
```

## 6. Setup Firewall
```
sudo ufw enable
sudo ufw status
sudo ufw allow ssh (Port 22)
sudo ufw allow http (Port 80)
sudo ufw allow https (Port 443)
```

## 7. Install NGINX and configure
```
sudo apt install nginx

sudo nano /etc/nginx/sites-available/default
```
Add the following to the location part of the server block
```
    server_name yourdomain.com www.yourdomain.com;

    location / {
        proxy_pass http://localhost:8001; #whatever port your app runs on
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
```
```
# Check NGINX config
sudo nginx -t

# Restart NGINX
sudo nginx -s reload
```

## 8. Add SSL with LetsEncrypt
```
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install python3-certbot-nginx
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Only valid for 90 days, test the renewal process with
certbot renew --dry-run
```

or 
using cicd with github action 

## Getting Started

-   STEP1 - Login to AWS console and create EC2 instance
-   STEP2 - Setup GitHub Repo and Push your project
-   STEP3 - Login to EC2 instance
-   STEP4 - Setup GitHub Action runner on EC2 instance
-   STEP5 - Create GitHub Secrets for managing environment variables
-   STEP6 - Create CI/CD Workflow using GitHub Action
-   STEP7 - Install nodejs and nginx on EC2 instance

```bash
sudo apt update

curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -

sudo apt-get install -y nodejs

sudo apt-get install -y nginx
```

-   STEP8 - Install pm2

```bash
sudo npm i -g pm2
```

-   STEP9 - Config nginx and restart it

```bash
cd /etc/nginx/sites-available

sudo nano default

location /api {
	rewrite ^\/api\/(.*)$ /api/$1 break;
	proxy_pass  http://localhost:8002;
	proxy_set_header Host $host;
	proxy_set_header X-Real-IP $remote_addr;
	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}

sudo systemctl restart nginx
```

-   STEP10 - Run backend api in the background as a service using pm2

```bash
pm2 start server.js --name=BackendAPI
```

-   STEP11 - Add the command in yml script of project to restart the nodejs api server after every push to the repo

```bash
run: pm2 restart BackendAPI
```
