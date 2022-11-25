## Launching Linux Instance on AWS

### Provisioning an EC2 Instance
![Screenshot from 2022-11-24 21-01-44](https://user-images.githubusercontent.com/84657461/203866509-fb69e3c9-6328-4d85-84c1-55e8a6afde16.png)
___
 
### SSH into the Instance
```
  cd ~/Downloads
  sudo chmod 0400 <private-key-name>. pem
  ssh -i <private-key-name>. pem ubuntu@<Public-IP-address>
 ```
![Screenshot from 2022-11-24 21-02-22](https://user-images.githubusercontent.com/84657461/203866624-7aadc8a9-c324-4ffc-bb18-f3568b55b56d.png)
___

### Installing Apache Server
```
  #Update a list of packages in package manager
  sudo apt update

  #Run apache2 package installation
  sudo apt install apache2
```
![Screenshot from 2022-11-24 21-25-44](https://user-images.githubusercontent.com/84657461/204040794-d3f03077-81ec-435a-88f8-de7a96a96d8c.png)
___

#### Check the status of apache2 service in the OS
```
  sudo systemctl status apache2
```
![Screenshot from 2022-11-24 21-31-37](https://user-images.githubusercontent.com/84657461/204040479-934ca13c-2b22-46e8-8868-8392c8f3c8e9.png)
___

#### Input the instance public IP address in the browser to verify Apache2 Installation
![Screenshot from 2022-11-24 21-41-44](https://user-images.githubusercontent.com/84657461/204041001-925e30b3-7cd5-4343-8434-90ba1b7a713d.png)
___

### Install MYSQL
When prompted, confirm installation by typing Y, and then ENTER.
```
  sudo apt install mysql-server
```
___

#### Log in to mysql after installation
This will connect to the MySQL server as the administrative database user root, which is inferred by the use of sudo when running this command.

```
  sudo mysql
```
![Screenshot from 2022-11-24 21-45-19](https://user-images.githubusercontent.com/84657461/204041890-4004b558-bc8e-4769-a52d-5618a8ed77fc.png)

___

#### Set a password for the root user using mysql_native_password as default authentication method
```
  ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '< password >';
```
Exit mysql

```
  exit
```

It’s recommended that you run a security script that comes pre-installed with MySQL. This script will remove some insecure default settings and lock down access to your database system.

```
  sudo mysql_secure_installation
```
![Screenshot from 2022-11-24 21-48-30](https://user-images.githubusercontent.com/84657461/204042910-bce6a77f-a8db-4568-aa34-8d175f8c3d4e.png)

This will ask if you want to configure the VALIDATE PASSWORD PLUGIN.

Note: Enabling this feature is something of a judgment call. If enabled, passwords which don’t match the specified criteria will be rejected by MySQL with an error. It is safe to leave validation disabled, but you should always use strong, unique passwords for database credentials.

Answer Y for yes, or anything else to continue without enabling. Follow the prompt
___

#### Confirm Mysql was successfully installed by logging in to the console
```
  sudo mysql -p
```
Notice the -p flag in this command, which will prompt you for the password used after changing the root user password.

![Screenshot from 2022-11-24 21-50-31](https://user-images.githubusercontent.com/84657461/204043654-9883cfd7-c01c-42f2-b8d9-bfd77315b78b.png)


Exit mysql
```
  exit
```
___

### Installing PHP

You have Apache installed to serve your content and MySQL installed to store and manage your data. PHP is the component of our setup that will process code to display dynamic content to the end user. In addition to the php package, you’ll need php-mysql, a PHP module that allows PHP to communicate with MySQL-based databases. You’ll also need libapache2-mod-php to enable Apache to handle PHP files. Core PHP packages will automatically be installed as dependencies.

```
  sudo apt install php libapache2-mod-php php-mysql
```
___

### Creating a Virtaul Host Using Apache
We will set up a domain called projectlamp, but you can replace this with any domain of your choice.

Apache on Ubuntu 20.04 has one server block enabled by default that is configured to serve documents from the /var/www/html directory.
We will leave this configuration as is and will add our own directory next next to the default one.
Create the directory for projectlamp using ‘mkdir’ command and assign ownership as follows:
```
  sudo mkdir /var/www/projectlamp
  sudo chown -R $USER:$USER /var/www/projectlamp
```
![Screenshot from 2022-11-24 22-02-51](https://user-images.githubusercontent.com/84657461/204043920-ffa3d740-15dd-426e-a27d-7c83f5167733.png)

Then, create and open a new configuration file in Apache’s sites-available directory using your preferred command-line editor. Here, we’ll be using vi or vim (They are the same by the way):
```
  sudo vi /etc/apache2/sites-available/projectlamp.conf
```
![Screenshot from 2022-11-24 22-04-28](https://user-images.githubusercontent.com/84657461/204044081-2ca0c912-c9d7-4578-8e09-988a8f0e1456.png)

This will create a new blank file. Paste in the following bare-bones configuration by hitting on i on the keyboard to enter the insert mode, and paste the text:

To save and close the file, simple hit esc on your keyboard and type :wq
: for command
w for write 
q for quit

```
<VirtualHost *:80>

ServerName projectlamp

ServerAlias www.projectlamp

ServerAdmin webmaster@localhost

DocumentRoot /var/www/projectlamp

ErrorLog ${APACHE_LOG_DIR}/error.log

CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```
![Screenshot from 2022-11-24 21-54-37](https://user-images.githubusercontent.com/84657461/204044230-a5d5777a-fc04-4a2c-ab6f-480efe0959a4.png)

Use the ls command to show the new file in the sites-available directory
```
  sudo ls /etc/apache2/sites-available
```
The out should have:  000-default.conf default-ssl.conf projectlamp.conf

With this VirtualHost configuration, we’re telling Apache to serve projectlamp using /var/www/projectlampl as its web root directory
You can now use a2ensite command to enable the new virtual host:
```
  sudo a2ensite projectlamp
```
Disable the default website that comes installed with Apache and check if your configuration file doesnt have syntax errors and finally reload Apache do these chnages takes effect.
```
  sudo a2dissite 000-default
  sudo apache2ctl configtest
  sudo systemctl reload apache2
```
Your new website is now active, but the web root /var/www/projectlamp is still empty. Create an index.html file in that location so that we can test that the virtual host works as expected:
```
  sudo echo 'Hello LAMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectlamp/index.html
```

![Screenshot from 2022-11-24 22-02-51](https://user-images.githubusercontent.com/84657461/204045686-bf231166-e3db-49e9-8deb-4c49a6030189.png)

Now go to your browser and try to open your website URL using IP address:
```
  http://<Public-IP-Address>:80
```
___

### Enable Php on the website
With the default DirectoryIndex settings on Apache, a file named index.html will always take precedence over an index.php file. This is useful for setting up maintenance pages in PHP applications, by creating a temporary index.html file containing an informative message to visitors. Because this page will take precedence over the index.php page, it will then become the landing page for the application. Once maintenance is over, the index.html is renamed or removed from the document root, bringing back the regular application page.

In case you want to change this behavior, you’ll need to edit the /etc/apache2/mods-enabled/dir.conf file and change the order in which the index.php file is listed within the DirectoryIndex directive:
```
  sudo vim /etc/apache2/mods-enabled/dir.conf
```
![Screenshot from 2022-11-24 22-04-28](https://user-images.githubusercontent.com/84657461/204046184-5fd7404b-994e-455e-b978-f8106419a29e.png)

Copy and paste the below, save.
```
<IfModule mod_dir.c>

#Change this:

#DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm

#To this:

DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm

</IfModule>
```
After saving and closing the file, you will need to reload Apache so the changes take effect:
```
  sudo systemctl reload apache2
```
Finally, we will create a PHP script to test that PHP is correctly installed and configured on your server.

Now that you have a custom location to host your website’s files and folders, we’ll create a PHP test script to confirm that Apache is able to handle and process requests for PHP files.

Create a new file named index.php inside your custom web root folder:
```
  vim /var/www/projectlamp/index.php
```
This will open a blank file. Add the following text, which is valid PHP code, inside the file:
```
  <?php
  phpinfo();
```
### When you are finished, save and close the file, refresh the page
