# LEMP STACK IMPLEMENTATION ON AWS EC2
The LEMP stack is a collection of software technologies that work together to host and serve dynamic websites and web applications. It is composed of the following components:

Linux: The operating system, which provides the foundation for the stack.
Nginx: The web server responsible for handling HTTP requests and serving web pages to users.
MySQL (or MariaDB): The database management system, used to store and retrieve data for your website or application.
PHP: The server-side scripting language used for dynamic content processing, often to interact with the MySQL database and render dynamic web pages.
The LEMP stack is commonly used for hosting websites, blogs, content management systems (CMS), and web applications.

### STEPS INVOLVED
- Step 0 - Preparing Prerequisites
- Step 1 - Installing the Nginx Web Server
- Step 3 - Installing PHP
- Step 4 - Configuring Nginx to use PHP Processor
- Step 5 -Testing PHP with Nginx
- Step 6 - Retrieving data from MySQL database with PHP
## Preparing the Prerequisites
- **Knowledge Requirements:** Familiarity with SQL syntax[SQL syntax and most commonly used commands](https://www.w3schools.com/sql/sql_syntax.asp) and the [Nano](https://www.linuxandubuntu.com/home/nano-cli-text-editor-for-everyone-basic-tutorials/) editor

### Instance Setup
1. In the AWS console, create a new t2.micro (or t3.micro) instance with the Ubuntu Server 24.04 LTS image.
![image1](https://github.com/user-attachments/assets/f02919cb-1e26-40fb-9732-38f6c7f882a4)
![image3](https://github.com/user-attachments/assets/c97e1139-251c-48a4-a3c8-ab02bb3d099a)
![image5](https://github.com/user-attachments/assets/10432b1f-f86c-472c-ad71-5e4f252b8fc5)

2. Create a new key pair and download it to your device.
![image6](https://github.com/user-attachments/assets/1c276354-fbee-4ec5-ab57-2e314faf3117)
![image7](https://github.com/user-attachments/assets/c3cc32e6-1b71-45ee-8b25-61b50775b5c7)

3. Set up a security group with the appropriate rules (HTTP, HTTPS, and SSH).


![alt text](images/set%20up%20the%20network%20security%20group.png)

Set up the rules as shown in the screenshot above.
![image8](https://github.com/user-attachments/assets/44a3812b-4342-4c76-a991-a6df2c29a700)

4. Launch the instance.

### Accessing the Instance
To access the instance from your terminal:
```bash
ssh -i "Private-key-name.pem" ubuntu@<Public IP address>
```


![image2](https://github.com/user-attachments/assets/d3defcd2-32b2-4556-bb3e-89e56ad5a6c1)


---
## Update and Installing the Nginx Web Server

1. Update the server package index:
   ```bash
   sudo apt update
   ```
   ![image3](https://github.com/user-attachments/assets/faa42bc4-e4c7-4e7b-b755-941015248114)

2. Install Nginx:
   ```bash
   sudo apt install nginx
   ```
![image4](https://github.com/user-attachments/assets/7289bd67-4ce1-47be-a540-fe40ae65865c)

3. When the installation is done,Check if Nginx is running:
   ```bash
   sudo systemctl status nginx
   ```
![alt text](images/Nginx%20running.png)
> If nginx is running successfully you should see see 'active:active(running) in green color like the image below
![image5](https://github.com/user-attachments/assets/60fca229-fd64-4be2-9d91-2364bf0e4b67)


### Testing Nginx
- **Locally on the Ubuntu shell with:**
  ```bash
  curl http://localhost:80
  ```
![image6](https://github.com/user-attachments/assets/06ab7c18-60e7-4267-abdb-e87239db0523)

- **From Browser:**
  Visit `http://<Public IP address>:80`.
  
![image7](https://github.com/user-attachments/assets/f37f51f2-b762-4c89-8dd6-cbc0e802cb9b)

---
## Installing MySQL

1. Install MySQL server with:
   ```bash
   sudo apt install mysql-server
   ```
![image8](https://github.com/user-attachments/assets/80d2c9ed-91eb-48ab-b68a-a0a991c30821)


2.When the installation is finished Log in to MySQL with:
   ```bash
   sudo mysql
   ```
![image9](https://github.com/user-attachments/assets/4435936d-4cbb-4d8d-90fa-c2f207ca21c9)


3. Set a password for the root user:
Use an Alter user command to set password for the root user

exit mysql, 
4. Secure the MySQL installation using the command below to configure the *VALIDATE PASSWORD PLUGIN*

```bash
 sudo mysql_secure_installation
```

![image8](https://github.com/user-attachments/assets/208ca020-5d6d-4d55-a0f6-f0fb4cc53f15)



5. Once the configuration is done and a new password is created. Trying to access mysql using
```bash
sudo mysql
```
![image8](https://github.com/user-attachments/assets/69cfca15-18ef-43fe-a513-c285fc9d8c68)


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
![image10](https://github.com/user-attachments/assets/1cf898b4-df98-40ac-8b9f-d7c3af8ab814)

2. Once the installation is done, we  check the php version with:
![image11](https://github.com/user-attachments/assets/38e61bf9-b116-4cdd-a817-f8f5b6700f14)


---
## Configuring Nginx to use PHP Processor

1. Create a directory for your project: 
 First, create a directory for the projectLEMP site inside the /var/www/ directory:
```bash
sudo mkdir /var/www/projectLEMP
```
Assign ownership of the directory to the current user (usually www-data for web servers):

```bash
sudo chown -R $USER:$USER /var/www/projectLEMP
```
![image12](https://github.com/user-attachments/assets/28dacc9a-6705-42d5-86ce-cf6261f88385)

2. Create a new server block:
Create a new server block file for the projectLEMP domain in Nginx's sites-available, for this we will be using nano :
```bash
sudo nano /etc/nginx/sites-available/projectLEMP
```
![image13](https://github.com/user-attachments/assets/324a732f-ce28-452f-9c0f-1669f1f30820)

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

![image15](https://github.com/user-attachments/assets/9edd5fff-fbd6-43b4-9e5a-1a62fa6b1ec7)



3. Enable the new server block:
Enable the new server block by creating a symbolic link to the sites-enabled directory:
```bash
sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/

```
![image16](https://github.com/user-attachments/assets/fa381c0d-9c36-409c-9efb-078a3bec5e6c)
4. Test the Nginx configuration:
Test the Nginx configuration to ensure there are no errors:
```bash
sudo nginx -t
```
Running the above returned test failed.
![image17](https://github.com/user-attachments/assets/de80d749-44ab-43a0-8f29-76a8be4adeb4)

> Unlinked the default file in sites-enabled using
 ```bash
 sudo unlink /etc/nginx/sites-enabled/default
 ```
5. Reload Nginx:
Lastly, reload Nginx to apply changes using:
```bash
sudo systemctl reload nginx
```
The projectLEMP is now active, but the web root is still empty.Create a sample index.html file in the /var/www/projectLEMP/ directory to test using using echo and the command below:
```bash
sudo echo 'Hello LEMP from hostname' $(TOKEN=curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600" && curl -H "X-aws-ec2-metadata-token: $TOKEN" -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(TOKEN=curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600" && curl -H "X-aws-ec2-metadata-token: $TOKEN" -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html
```
![image18](https://github.com/user-attachments/assets/833bd9aa-f6b7-49d2-b686-918c6df7e61b)

 Open your EC2 instance on the Console, copy the public IP address, and paste it into your browser.

 ![image19](https://github.com/user-attachments/assets/db8d5bbc-f8c6-4271-9ff5-3ab88e82a6d0)


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
 ![image20](https://github.com/user-attachments/assets/54c94a69-39e2-41c2-be6f-f45f555cd604)

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
CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';
```

Grant all privileges on the lemp_database database to the new user:
```sql
GRANT ALL PRIVILEGES ON example_database.* TO 'example_user'@'%';
```

Exit the mysql console.

Insert Sample Data into the todo_list Table with:
```sql
INSERT INTO example_database.todo_list (content) VALUES ("First task to carry out");
```
Repeat the command with different values a few times.

You can check the data in the todo_list table by running:
```sql
SELECT * FROM example_database.todo_list;
```

 Exit the MySQL console.
![image21](https://github.com/user-attachments/assets/287b8cfe-dabe-4ddc-8ed7-c7455fcc3a95)


2. Step 2: Create a PHP Script to Retrieve Data

Create a PHP file to retrieve and display the data from MySQL.

First, create a new PHP file in the projectLEMP directory:
```bash
sudo nano /var/www/projectLEMP/todo_list.php
```
Added the following PHP code to the file:
```php
<?php
$user = "example_user";
$password = "PassWord.1";
$database = "example_database";
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
![image22](https://github.com/user-attachments/assets/fb558433-ce32-4bbd-a329-082484074f72)


### Congradulation 🎈🎈✨🎉🎊

You've just completed your first real-life project by deploying a LEMP stack in the cloud. 🎉
---
