# LEMP (Linux, EngineX, mySQL, PHP) setup

Directions for setting up virtual hosting on Linode (two domain names pointing to one ip address)


1. Start [here](https://linode.com/docs/getting-started/).

```
apt-get update && apt-get upgrade
hostnamectl set-hostname example_hostname
dpkg-reconfigure tzdata
adduser example_user
adduser example_user sudo
```

2. Follow first steps [here](https://linode.com/docs/security/securing-your-server/).

  * Make sure you add you host name to the hosts file, otherwise, `sudo` will complain. 
  ```
  sudo nano /etc/hosts
  ```

```
127.0.0.1       localhost
127.0.1.1       your_host_name
```

3. Follow [this guide](https://linode.com/docs/web-servers/lemp/how-to-install-a-lemp-server-on-ubuntu-16-04/). 

#### nginx and virtial host config

```
sudo apt-get install nginx
tail /etc/nginx/sites-available/default -n 13 | cut -c 2- | sudo tee /etc/nginx/sites-available/joeleb.com 1> /dev/null
sudo nano /etc/nginx/sites-available/joeleb.com 
```
```
server {
        listen 80;
        listen [::]:80;

        server_name www.joeleb.com joeleb.com;

        root /var/www/html/joeleb.com/public_html/;
        index index.html;

        location / {
                try_files $uri $uri/ =404;
        }
        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                include fastcgi_params;
                fastcgi_pass unix:/run/php/php7.0-fpm.sock;
                fastcgi_param SCRIPT_FILENAME /var/www/html/joeleb.com/public_html$fastcgi_script_name;
        }
}
```
```
sudo cp /etc/nginx/sites-available/joeleb.com /etc/nginx/sites-available/touchdrums.com
sudo nano /etc/nginx/sites-available/touchdrums.com
```
```
server {
        listen 80;
        listen [::]:80;

        server_name www.touchdrums.com touchdrums.com;

        root /var/www/html/touchdrums.com/public_html/;
        index index.html;

        location / {
                try_files $uri $uri/ =404;
        }
        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                include fastcgi_params;
                fastcgi_pass unix:/run/php/php7.0-fpm.sock;
                fastcgi_param SCRIPT_FILENAME /var/www/html/touchdrums.com/public_html$fastcgi_script_name;
        }
}
```

```
sudo ln -s /etc/nginx/sites-available/joeleb.com /etc/nginx/sites-enabled
sudo ln -s /etc/nginx/sites-available/touchdrums.com /etc/nginx/sites-enabled
sudo mkdir -p /var/www/html/joeleb.com/public_html
sudo mkdir -p /var/www/html/touchdrums.com/public_html
sudo chown -R joel21987:joel21987 /var/www/html/joeleb.com/public_html/
sudo chown -R joel21987:joel21987 /var/www/html/touchdrums.com/public_html/
sudo rm /etc/nginx/sites-enabled/default
sudo systemctl restart nginx
```

#### PHP

```
sudo apt-get install php7.0-cli php7.0-cgi php7.0-fpm
sudo systemctl restart php7.0-fpm nginx

```

#### mySQL

```
sudo apt-get install mysql-server php7.0-mysq
mysql -u root -p
sudo systemctl restart php7.0-fpm
```

#### Test
```
sudo nano /var/www/html/touchdrums.com/public_html/phptest.php
```
```
<html>
<head>
    <title>PHP Test</title>
</head>
    <body>
    <?php echo '<p>Hello World</p>';

    // In the variables section below, replace user and password with your own MySQL credentials as created on your server
    $servername = "localhost";
    $username = "webuser";
    $password = "password";

    // Create MySQL connection
    $conn = mysqli_connect($servername, $username, $password);

    // Check connection - if it fails, output will include the error message
    if (!$conn) {
        exit('<p>Connection failed: <p>' . mysqli_connect_error());
    }
    echo '<p>Connected successfully</p>';
    ?>
</body>
</html>
```

Vist site to test. 

### Ghost

#### Install nodejs & ghost

```
sudo apt install build-essential
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
sudo apt install nodejs
sudo npm install -g ghost-cli@latest
sudo ghost install
```

Durring Ghost installation, you are asked to setup nginx or not. After choosing yes, a new `/etc.nginx/sites-avalable/joeleb.com/conf` file was created and linked. Here is what the default looks like:

```

server {
    listen 80;
    listen [::]:80;

    server_name joeleb.com;
    root /var/www/html/joeleb.com/public_html/system/nginx-root;

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

Tried mashing the two to gether, like so:

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

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        include fastcgi_params;
        fastcgi_pass unix:/run/php/php7.0-fpm.sock;
        fastcgi_param SCRIPT_FILENAME /var/www/html/joeleb.com/public_html$fastcgi_script_name;
    }

    client_max_body_size 50m;
}
```

Which resulted in a 502 Bad Gateway. 


