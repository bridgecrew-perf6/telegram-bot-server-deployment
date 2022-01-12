# telegram-bot-server-deployment

This repository will help you launch your Telegram Bot Server on a dedicated server using Docker Compose and set up a secure protocol connection `https`

If you are using this tutorial I will assume that you already have `Docker` and `Docker Compose` installed on your dedicated server.

## docker-compose.yml

```
version: '3.7'

services:
  telegram-bot-server:
    image: aiogram/telegram-bot-api:5.5
    container_name: bot-server
    environment:
      TELEGRAM_API_ID: "you-api-id-here"
      TELEGRAM_API_HASH: "you-api-hash-here"
      TELEGRAM_STAT: 1
    volumes:
      - telegram-bot-server-data:/var/lib/telegram-bot-api
    ports:
      - 8081:8081
      - 8082:8082
    restart: always

volumes:
  telegram-bot-server-data:

```
In principle, this could have ended, input 
 - `<host-ip>:8081/bot<bot-token>/getMe` - viewing basic information on the bot
 -  `<host-ip>:8082` - viewing information on the server

Further instructions for setting up your own secure subdomain for access to the server

## Install and Configure Nginx

`sudo apt update`
`sudo apt install nginx`

### Configure Nginx server block

`sudo vim /etc/nginx/sites-available/default`

```
server {
   server_name   {sub-domain.your-domain-name.com};
   location / {
       proxy_pass         http://localhost:8081;
       proxy_http_version 1.1;
       proxy_set_header   Upgrade $http_upgrade;
       proxy_set_header   Connection keep-alive;
       proxy_set_header   Host $host;
       proxy_cache_bypass $http_upgrade;
       proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_set_header   X-Forwarded-Proto $scheme;
   }
}

```
`sudo service nginx restart`

### Accessing HTTPS through a firewall

You can view the current settings with the following command:

`sudo ufw status`

Perhaps the profile will look like this, that is, only HTTP traffic will be allowed on the web server:

```
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                  
Nginx HTTP                 ALLOW       Anywhere                  
OpenSSH (v6)               ALLOW       Anywhere (v6)             
Nginx HTTP (v6)            ALLOW       Anywhere (v6)
```

To allow HTTPS traffic, enable the Nginx Full profile and remove the redundant Nginx HTTP profile:

```
sudo ufw allow 'Nginx Full'
sudo ufw delete allow 'Nginx HTTP'
```

Your status should now look like this:

`sudo ufw status`

```
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
Nginx Full                 ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
Nginx Full (v6)            ALLOW       Anywhere (v6)
```

## Obtaining an SSL Certificate

Let’s Encrypt is a certification authority (CA) that makes it easy to obtain and install free TLS / SSL certificates, allowing you to use encrypted HTTPS traffic on your web servers.

### Installing Certbot

`sudo apt install certbot python3-certbot-nginx`

### Obtaining an SSL Certificate

`sudo certbot --nginx -d {sub-domain.your-domain-name.com}`

### Checking Certbot Automatic Updates

`sudo systemctl status certbot.timer`

```
Output
● certbot.timer - Run certbot twice daily
     Loaded: loaded (/lib/systemd/system/certbot.timer; enabled; vendor preset: enabled)
     Active: active (waiting) since Mon 2020-05-04 20:04:36 UTC; 2 weeks 1 days ago
     Trigger: Thu 2020-05-21 05:22:32 UTC; 9h left
     Triggers: ● certbot.service
```

Чтобы протестировать процесс обновления, можно сделать запуск «вхолостую»:

`sudo certbot renew --dry-run`

If there are no errors, everything is fine. Certbot will renew your certificates when required and reload Nginx to activate the changes.

Now you can test the server operation on a personal subdomain using a secure protocol:

`{sub-domain.your-domain-name.com}/bot<bot-token>/getMe`

### If this manual was useful to you, you can thank me here <code> <b>--></b> [Mono Bank donate](https://send.monobank.com.ua/NNG8cy25) "</code> or here <code> <b>--></b> [TON donate](https://t.me/CryptoBot?start=IVzvtl4RU4q8) "</code>
