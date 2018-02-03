# linode_setup

Directions for setting up linode


1. Start [here](https://linode.com/docs/getting-started/).

2. Follow first steps [here](https://linode.com/docs/security/securing-your-server/).

3. Follow [this guide](https://linode.com/docs/websites/hosting-a-website/) but using [this Ubuntu 16.04 varient](https://linode.com/docs/web-servers/lamp/install-lamp-stack-on-ubuntu-16-04/).


  
```
sudo apt install apache2
sudo nano /etc/apache2/apache2.conf 
sudo nano  /etc/apache2/mods-available/mpm_prefork.conf 
sudo a2dismod mpm_event
```
  * Make sure you add you host name to the hosts file 
  ```
  sudo nano /etc/hosts
  ```

```
127.0.0.1       localhost
127.0.1.1       your_host_name

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

```   
sudo a2dismod mpm_event
sudo a2enmod mpm_prefork
sudo systemctl restart apache2
sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/joeleb.com.conf
sudo nano /etc/apache2/sites-available/joeleb.com.conf 
sudo cp /etc/apache2/sites-available/joeleb.com.conf /etc/apache2/sites-available/touchdrums.com.conf
sudo nano /etc/apache2/sites-available/touchdrums.com.conf
```

```
sudo mkdir -p /var/www/html/joeleb.com/{public_html,logs,backups}
sudo mkdir -p /var/www/html/touchdrums.com/{public_html,logs,backups}
sudo a2ensite joeleb.com.conf 
sudo a2ensite touchdrums.com.conf 
sudo a2dissite 000-default.conf
sudo systemctl reload apache2
```
```
sudo apt install mysql-server
mysql -u root -p
sudo apt install php7.0 libapache2-mod-php7.0 php7.0-mysql
sudo apt install php7.0-curl php7.0-json php7.0-cgi
sudo nano /etc/php/7.0/apache2/php.ini 
sudo mkdir /var/log/php
sudo chown www-data /var/log/php
sudo systemctl restart apache2
sudo nano /var/www/html/joeleb.com/public_html/phptest.php
```

At this point, you should be able to access test php or html files from both website directories. 

4. Stop apache before moving on to next step

```
sudo service apache2 stop
```


5. Install nginx, node.js, npm, and ghost using this [guide](https://github.com/linode/docs/blob/master/docs/websites/cms/how-to-install-ghost-cms-on-ubuntu-16-04.md). 

```
sudo apt install build-essential
sudo apt install nginx
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
sudo apt install nodejs
sudo npm install -g ghost-cli@latest
sudo chown -R joel21987:joel21987 /var/www/html/joeleb.com/public_html/
sudo chown -R joel21987:joel21987 /var/www/html/touchdrums.com/public_html/
sudo nano /etc/apache2/sites-available/joeleb.com.conf 
sudo nano /etc/apache2/sites-available/touchdrums.com.conf 
sudo systemctl restart apache2
```

6. This is where things usually get difficult. You should now have a Ghost site running. But you'll probably not see the second site correctly. It will likely be pointing to the Apache index.hmtl file located in /var/www/html. To remedy this, I disabled (deleted?) the default nginx .conf file, similar to how the default apache .conf file was in previous steps. 


```
sudo rm /etc/nginx/sites-enabled/default 
sudo service nginx reload
sudo ls -la /etc/nginx/sites-enabled/
```

After this, I was no longer seeing the defautl apache index.html page, however, now my second site was just pointing to my first. Furthermore, I could not access the phptest.php file or any other file that was supposed to be linked in the apache virtual host file (`/var/www/html/secondsite.com/public_html`). 

7. Then I copied and linked the .conf file that was created (when ghost was installed?) for my second site. Not sure if this was actually necessary or not. 

```
sudo cp joeleb.com.conf touchdrums.com.conf
sudo nano touchdrums.com.conf
sudo ln -s /etc/nginx/sites-available/touchdrums.com.conf /etc/nginx/sites-enabled/touchdrums.com.conf 
sudo service nginx reload
```

It was about this time I noticed that restarting apache was returnign an error. Checking on the status of apache with `/etc/init.d/apache2 status` confirmed that apache was dying as soon as it started. 

[This site](https://linode.com/docs/web-servers/nginx/how-to-configure-nginx/) was helpful. 

But, [This guide](https://linode.com/docs/uptime/loadbalancing/use-nginx-as-a-front-end-proxy-and-software-load-balancer/) seemed to offer the solution. The problem is, I don't know if it's a proper solution. I followed most of the steps in `Configure Apache for Port Listening`, buti t was about that time I noticed the nginx .conf files might just need their root altered, just like it was in the apache .conf files (`root /var/www/html/example.com;`)

After those changes, everythign seems to be running happily, including apache. And, both sites are now pointing to their appropriate `/var/www/html/site.com/public_html` directories! 


---

#### nginx .conf file 

`sudo nano /etc/nginx/sites-available/joeleb.com.conf`

```
server {
    listen 80;
    listen [::]:80;

    server_name www.joeleb.com joeleb.com;
    root /var/www/html/joeleb.com/public_html/;

    location / {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $http_host;
        proxy_pass http://127.0.0.1:2368;
    }

    location ~ /.well-known {
        allow all;
    }

    client_max_body_size 50m;
}
```

#### apache .conf file

`sudo nano /etc/apache2/sites-available/joeleb.com.conf`

