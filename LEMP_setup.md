# LEMP (Linux, EngineX, mySQL, PHP) setup

Directions for setting up virtual hosting on Linode (two domain names pointing to one ip address)


1. Start [here](https://linode.com/docs/getting-started/).

```
apt-get update && apt-get upgrade
hostnamectl set-hostname example_hostname
dpkg-reconfigure tzdata
```

  * Make sure you add you host name to the hosts file, otherwise, `sudo` will complain. 
  ```
  nano /etc/hosts
  ```
  
  ```
  127.0.0.1       localhost
  127.0.1.1       your_host_name
  ```
2. Follow first steps [here](https://linode.com/docs/security/securing-your-server/).

```
adduser example_user
adduser example_user sudo
```



3. Follow [this Linode LEMP guide](https://linode.com/docs/web-servers/lemp/how-to-install-a-lemp-server-on-ubuntu-16-04/). 

#### nginx and virtial host config

Don't bother making config file for Ghost site (joeleb.com), as ghost install will do that for you. 

```
sudo apt-get install nginx
tail /etc/nginx/sites-available/default -n 13 | cut -c 2- | sudo tee /etc/nginx/sites-available/touchdrums.com 1> /dev/null
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
sudo ln -s /etc/nginx/sites-available/touchdrums.com /etc/nginx/sites-enabled
sudo mkdir -p /var/www/html/joeleb.com/public_html
sudo mkdir -p /var/www/html/touchdrums.com/public_html
sudo chown -R joel21987:joel21987 /var/www/html/joeleb.com/public_html/
sudo chown -R joel21987:joel21987 /var/www/html/touchdrums.com/public_html/
sudo rm /etc/nginx/sites-enabled/default
sudo rm /etc/nginx/sites-available/default
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
Pending all the correct info was put in during ghost setup, Ghost should now be started, and you should be able to visit the ghost admin site. 

