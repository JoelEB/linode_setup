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
```

At this point, you should be able to access test php or html files from both website directories. 

4. Stop apache before moving on to next step

```
sudo service apache2 stop
```


5. Install nginx, node.js, npm, and ghost using this [guide](https://github.com/linode/docs/blob/master/docs/websites/cms/how-to-install-ghost-cms-on-ubuntu-16-04.md). 



