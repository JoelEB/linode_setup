# Linode Setup

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

```
sudo apt-get install nginx
tail /etc/nginx/sites-available/default -n 13 | cut -c 2- | sudo tee /etc/nginx/sites-available/joeleb.com 1> /dev/null
sudo nano /etc/nginx/sites-available/joeleb.com 
sudo cp /etc/nginx/sites-available/joeleb.com /etc/nginx/sites-available/touchdrums.com
sudo nano /etc/nginx/sites-available/touchdrums.com
sudo ln -s /etc/nginx/sites-available/joeleb.com /etc/nginx/sites-enabled
sudo ln -s /etc/nginx/sites-available/touchdrums.com /etc/nginx/sites-enabled
sudo rm /etc/nginx/sites-enabled/default
sudo systemctl restart nginx
```
