# LEMP STACK IMPLEMENTATION ON AWS EC2
This guide walks through the steps to install and configure a LEMP (Linux, Nginx, MySQL, PHP) stack on an AWS EC2 t2.micro instance running Ubuntu 24.04 LTS. This setup will allow you to host and manage a dynamic website with a MySQL database and PHP processing on an Nginx web server.

### STEPS INVOLVED
- Step 0 -  [Preparing the Prerequisites](#preparing-the-prerequisites)
-  Step 1 - [Installing the Nginx Web Server](#installing-the-nginx-web-server)
- Step 2 - [Installing MySQL](#installing-mysql)
- Step 3 - [Installing PHP](#installing-php)
- Step 4 - [Configuring Nginx to use PHP Processor](#configuring-nginx-to-use-php-processor)
- Step 5 -[Testing PHP with Nginx](#testing-php-with-nginx)
- Step 6 - [Retrieving data from MySQL database with PHP](#retrieving-data-from-mysql-database-with-php)

## Preparing the Prerequisites
- **Knowledge Requirements:** Familiarity with SQL syntax and the Nano editor.
- **AWS Account:** Required to set up the Ubuntu server.
- **Terminal:** Use Git Bash or any terminal that supports SSH.


### Instance Setup
1. In the AWS console, create a new t2.micro (or t3.micro) instance with the Ubuntu Server 24.04 LTS image.
2. Create a new key pair and download it to your device.


![alt text](images/create%20key%20pair.png)

3. Set up a security group with the appropriate rules (HTTP, HTTPS, and SSH).


![alt text](images/set%20up%20the%20network%20security%20group.png)

Set up the rules as shown in the screenshot above.

4. Launch the instance.

### Accessing the Instance
To access the instance from your terminal:
```bash
ssh -i "Private-key-name.pem" ubuntu@<Public IP address>
```


![alt text](images/ssh%20into%20ubuntuaws.png)

---
## Installing the Nginx Web Server

Inside the terminal, on the Ubuntu server,  first update the server package index, then install nginx using the following commands.

1. Update the server package index:
   ```bash
   sudo apt update
   ```
2. Install Nginx:
   ```bash
   sudo apt install nginx
   ```

3. When the installation is done,Check if Nginx is running:
   ```bash
   sudo systemctl status nginx
   ```
![alt text](images/Nginx%20running.png)
> If nginx is running successfully you should see see 'active:active(running) in green color.


### Testing Nginx
- **Locally on the Ubuntu shell with:**
  ```bash
  curl http://localhost:80
  ```
![alt text](images/testing%20nginx%20locally.png)

- **From Browser:**
  Visit `http://<Public IP address>:80`.
  
![alt text](images/testin%20via%20pubip.png)
---
## Installing MySQL
For this LEMP stack implementation, we are making use of MySQL for the database management  system (DBMS), Install MySQL server to access MySQL.

1. Install MySQL server with:
   ```bash
   sudo apt install mysql-server
   ```


2.When the installation is finished Log in to MySQL with:
   ```bash
   sudo mysql
   ```
![alt text](images/sqlafterinstall.png)

3. Set a password for the root user:
Use an Alter user command to set password for the root user

![alt text](images/set%20mysql%20default%20password.png)

exit mysql, 
4. Secure the MySQL installation using the command below to configure the *VALIDATE PASSWORD PLUGIN*

```bash
 sudo mysql_secure_installation
```

![alt text](images/mysql%20secure%20install.png)


5. Once the configuration is done and a new password is created. Trying to access mysql using
```bash
sudo mysql
```
![alt text](images/mysql%20access%20denied.png)

It will return the above error.

The actual way to access mysql after creating the new password is by using:
```bash
sudo mysql -p
```
The above will prompt for the new password used after changing the root user password.

Exit mysql.

---
## Installing PHP
To continue the LEMP stack setup, 
1. let's proceed with installing PHP and configuring it to work with Nginx:
Using the command:
```bash
sudo apt install php-fpm php-mysql
```


PHP-FPM (FastCGI Process Manager):
PHP-FPM is a version of PHP optimized for handling web requests. In a LEMP stack, Nginx passes PHP requests to PHP-FPM for processing. It's faster and more efficient, especially for high-traffic sites.

PHP-MySQL Extension:
php-mysql enables PHP to communicate with MySQL databases. It's essential for dynamic websites that rely on MySQL for data storage and retrieval.

2. Once the installation is done, we  check the php version with:
![alt text](images/checkphpv.png)

---
## Configuring Nginx to use PHP Processor

Now that PHP is installed, we must configure Nginx to handle PHP files using PHP-FPM.
In Nginx web servers, we can create server blocks(quite similar to virtual hosts in Apache)
For this Implementation, we will use "projectLEMP" as the sample domain name.

1. Create a directory for your project: 
 First, create a directory for the projectLEMP site inside the /var/www/ directory:
```bash
sudo mkdir /var/www/projectLEMP
```
Assign ownership of the directory to the current user (usually www-data for web servers):

```bash
sudo chown -R $USER:$USER /var/www/projectLEMP
```
2. Create a new server block:
Create a new server block file for the projectLEMP domain in Nginx's sites-available, for this we will be using nano :
```bash
sudo nano /etc/nginx/sites-available/projectLEMP
```
The above will create a blank file, and paste it in the following configuration. 
```nginx
server {
    listen 80;
    server_name projectLEMP www.projectLEMP;
    root /var/www/projectLEMP;

    index index.php index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
````
![alt text](images/nano%20editor.png)

The Nginx server block configuration above sets up the projectLEMP domain and specifies how Nginx should handle requests for projectLEMP. Below is a brief explanation of what each part does. 

> - **listen 80**: Configures Nginx to listen on port 80 (HTTP) for incoming traffic.
>
> - **server_name projectLEMP www.projectLEMP**: Defines the domain names that this server block will respond to (`projectLEMP` and `www.projectLEMP`).
>
> - **root /var/www/projectLEMP**: Sets the root directory for this website, where Nginx will look for the website files (HTML, PHP, etc.).
>
> - **index index.php index.html index.htm index.php**: Specifies the default files Nginx will serve when the root directory is accessed. It prioritizes `index.php` and falls back to other index files.
>
> - **location /**: Handles requests for the root URL (`/`). It tries to serve the requested file or directory, and if it doesn't exist, it returns a `404 Not Found` error.
>
> - **location ~ \.php$**: Handles PHP file requests. It uses PHP-FPM (via `fastcgi_pass`) to process PHP files by passing them to the PHP processor (`php8.3-fpm.sock`).
>
> - **location ~ /\.ht**: Denies access to `.ht` files (like `.htaccess`), which are usually used by Apache for configuration but shouldn't be accessible in an Nginx setup.

3. Enable the new server block:
Enable the new server block by creating a symbolic link to the sites-enabled directory:
```bash
sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/
```
4. Test the Nginx configuration:
Test the Nginx configuration to ensure there are no errors:
```bash
sudo nginx -t
```
Running the above returned test failed.
![alt text](images/nginxtfailed.png)


To troubleshoot what could be wrong, I used ls  to check if the file /etc/nginx/sites-enabled/projectLEMP exists
![alt text](images/running%20test%20again.png)
>which showed ProjectLEMP exists in sites-enabled

![alt text](<images/unlink default.png>)
> Unlinked the default file in sites-enabled using
 ```bash
 sudo unlink /etc/nginx/sites-enabled/default
 ```

 The test still failed, following google search results, I checked  read permissions for the configuration file using:
 ```
 sudo chmod 644 /etc/nginx/sites-available/projectLEMP

sudo chmod 644 /etc/nginx/sites-enabled/projectLEMP

 ```
 ![alt text](images/chmod.png)
 > It brought up cannot operate on dangling symlink for /etc/nginx/sites-enabled/projectLEMP

 To resolve the issue, We remove the Dangling Symlink: Since it's pointing to a non-existent file, let's remove the broken symlink first:
 ```bash
 sudo rm /etc/nginx/sites-enabled/projectLEMP
```
Ensure the source file still exists in "/etc/nginx/sites-available/projectLEMP" by running:
```bash
sudo nano /etc/nginx/sites-available/projectLEMP
```
Recreate the symlink by using:

```bash
sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/
```

Verify permissions again
```bash
sudo chmod 644 /etc/nginx/sites-available/projectLEMP
```

Then finally run the below again:
```bash
sudo nginx -t
```
![alt text](images/nginxtsuccessful.png)
> It returned "successful" !.

5. Reload Nginx:
Lastly, reload Nginx to apply changes using:
```bash
sudo systemctl reload nginx
```
The projectLEMP is now active, but the web root is still empty.Create a sample index.html file in the /var/www/projectLEMP/ directory to test using using echo and the command below:
```bash
sudo echo 'Hello LEMP from hostname' $(TOKEN=curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600" && curl -H "X-aws-ec2-metadata-token: $TOKEN" -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(TOKEN=curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600" && curl -H "X-aws-ec2-metadata-token: $TOKEN" -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html
```
 Open your EC2 instance on the Console, copy the public IP address, and paste it into your browser.

 ![alt text](images/projectlemp%20purl.png)

 ---
## Testing PHP with Nginx

Now that Nginx is configured to use PHP, we test to see if PHP is processing correctly.

1. Create a PHP info file:
Create a PHP info file in the projectLEMP root directory:
```bash
sudo nano /var/www/projectLEMP/info.php
```
Paste the below into the new file.
```php
<?php
phpinfo();
?>
```
2. Access the PHP info file via your server's public IP:
Test PHP processing by accessing the info.php file via your server's public IP in your browser:
```
http://<your-server-public-ip>/info.php
```
 ![alt text](images/phpinfopage.png)
 > The above page will be displayed showing detailed information about the server.

3. Remove the info.php file afterward for security:
Remove the info.php file once you confirm it's working, as it can expose sensitive server details:
```bash
sudo rm /var/www/projectLEMP/info.php
```
--- 
## Retrieving data from MySQL database with PHP

Now that we have successfully set up the LEMP stack, we will retrieve data from a MySQL database using PHP.

1. Step 1: Create a MySQL Database and Table

First, log in to the MySQL console by running:
```bash
sudo mysql -p
```
>Running the above will ask for the password created earlier in this guide.

Create a new Database in mysql:
```sql
CREATE DATABASE lemp_database;
```
Create a new user (e.g., ayopo) with a password in mysql:
```sql
CREATE USER 'ayopo'@'%' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';
```

Grant all privileges on the lemp_database database to the new user:
```sql
GRANT ALL PRIVILEGES ON lemp_database.* TO 'ayopo'@'%';
```
 ![alt text](images/createdbusergrantaccess.png)

Exit the mysql console.

Test the permissions by logging in to the mysql console again, by using the newly created custom user credentials:
```bash
mysql -u ayopo -p
```
 ![alt text](images/access%20newdbwithuser.png)
 >the above prompts for the custom user password.

 Next we will create a test table named *todo_list* in lemp_database from MySQL console:

 ![alt text](images/createtodo_listtable.png) 
 
Insert Sample Data into the todo_list Table with:
```sql
INSERT INTO lemp_database.todo_list (content) VALUES ("First task to carry out");
```
Repeat the command with different values a few times.

You can check the data in the todo_list table by running:
```sql
SELECT * FROM lemp_database.todo_list;
```
 ![alt text](images/view%20table%20content.png)

 Exit the MySQL console.


2. Step 2: Create a PHP Script to Retrieve Data

Create a PHP file to retrieve and display the data from MySQL.

First, create a new PHP file in the projectLEMP directory:
```bash
sudo nano /var/www/projectLEMP/todo_list.php
```
Added the following PHP code to the file:
```php
<?php
$user = "ayopo";
$password = "PassWord.1";
$database = "lemp_database";
$table = "todo_list";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h2>TODO</h2><ol>";
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}
```
Save the above file and exit the nano editor.

3. Step 3: Test the PHP Script
Now, you can test the PHP script by navigating to the following URL in your browser:
```
http://<your-server-public-ip>/todo_list.php
```

 ![alt text](images/web%20todo%20list.png)
 >the inserted data from the todo_list table  will be displayed in your browser.

### Lessons Learned and Obstacles Overcome

Throughout the process of setting up the LEMP stack on AWS EC2, I encountered several challenges that provided valuable learning experiences:

1. **Database Access Issues**: Initially, I faced access problems when attempting to log into MySQL after setting a new root password. I mistakenly used the command `sudo mysql`, which led to an "Access Denied" error. The correct approach was to use `sudo mysql -p`, prompting for the new password. This taught me the importance of double-checking command syntax, especially when dealing with authentication.

2. **Nginx Configuration Challenges**: While configuring Nginx to serve the PHP files, I encountered issues with the server block not being recognized due to a dangling symlink. After several troubleshooting steps, including checking file permissions and verifying the existence of the configuration files, I learned the importance of understanding how symlinks function in Linux. This experience highlighted the need for careful attention to file management when setting up web servers.
3. **PHP and Database Interaction**: Successfully retrieving data from MySQL using PHP was a rewarding experience. I learned how to use PDO for secure database connections and query execution, reinforcing best practices in PHP development.

### Conclusion

The LEMP stack setup on AWS EC2 has been a valuable project that deepened my understanding of Linux, Nginx, MySQL, and PHP. By documenting the obstacles I faced and the solutions I found, I hope to provide insights for others who may undertake a similar journey. Each challenge was an opportunity to learn, and I feel more confident in my DevOps skills moving forward. 

---