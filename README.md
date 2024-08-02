# LAMP
Install Lamp stack on Almalinux. Then host a website called "php.table.com". The "php.table.com" must contain a file called table.php. When someone opens the link "http://php.table.com/table.php", the page must display the contents from a database. 

### Step 1
Run system update<br>
`dnf update`

### Step 2
The next step is to install the Apache webserver on AlmaLinux<br>
`dnf install httpd httpd-tools`

### Step 3
Once the webserver is installed, let’s start its service and also make it automatically up with the system boot. This will ensure whenever you boot AlmaLinux you won’t need to start Apache manually.<br>
`systemctl start httpd`<br>
`systemctl enable httpd`

After installing Apache and applying its settings, we will now check its Status:<br>
`systemctl status httpd`

### Step 4
If you want to access the Apache webserver outside your local machine using some browser, then first we have to open ports 80 and 443 on our AlmaLinux server.<br>
`firewall-cmd --permanent --zone=public --add-service=http`<br>
`firewall-cmd --permanent --zone=public --add-service=https`<br>
`firewall-cmd --reload`

### Step 5
Install mariadb
`dnf install mariadb-server mariadb -y`<br>
`systemctl start mariadb`<br>
`systemctl enable mariadb`<br>
`systemctl Status mariadb`

### Step 6
This step  will give some options to follow and set some settings so that we can secure of Database from any common future threats.<br>
`mysql_secure_installation`

### Step 7
Install php
First, check what are PHP versions available to install:<br>
`dnf module list php`

To install PHP 8.2, first enable the module as provided.<br>
`dnf module reset php`<br>
`dnf module enable php:8.2`

install php<br>
`dnf install php php-common php-opcache php-cli php-gd php-curl php-mysqlnd`

To get better performance for various applications using PHP, we can start (if not already) and enable PHP-FPM (FastCGI Process Manager) using the below commands:
`systemctl start php-fpm`<br>
`systemctl enable php-fpm`

now verify the installation<br>
`php -v`

### Step 8
Create and configure the website<br>
Create Document Root Directory:<br>
`mkdir -p /var/www/php.table.com`

Set permissions<br>
`chown -R apache:apache /var/www/php.table.com`<br>
`chmod -R 755 /var/www/php.table.com`

configure your DNS settings to point php.table.com to your server's IP address.<br>
`vim /etc/hosts`<br>
`192.168.1.89 php.table.com`

Create Apache Configuration File for php.table.com: Create a new file in /etc/httpd/conf.d/:<br>
`vim /etc/httpd/conf.d/php.table.com.conf`
```<VirtualHost *:80>
ServerAdmin webmaster@php.table.com
DocumentRoot /var/www/php.table.com
ServerName php.table.com
ErrorLog /var/log/httpd/php.table.com-error.log
CustomLog /var/log/httpd/php.table.com-access.log combined
</VirtualHost>
```
Restart httpd<br>
`systemctl restart httpd`

### Step 9
Set up the database<br>
Log in to MariaDB:<br>
`mysql -u root -p`

`Create Database and Table:`<br>
`CREATE DATABASE clado;`<br>
`USE clado;`<br>
`CREATE TABLE students ( name CHAR(20), mark INT(3) );`<br>
`INSERT INTO students (name, mark) VALUES ('Alice', 85);`<br>
`INSERT INTO students (name, mark) VALUES ('Bob', 90);`<br>
`INSERT INTO students (name, mark) VALUES ('Charlie', 78);`<br>
`EXIT;`

### Step 10
Create table.php in the Document Root:<br>
`vim /var/www/php.table.com/table.php`

Add the following code<br>
```<?php // Database configuration
$servername = "localhost";
$username = "root";
$password = "1234";
$dbname = "clado"; // Create connection
$conn = new mysqli($servername, $username, $password, $dbname); // Check connection
if ($conn->connect_error)
{ die("Connection failed: " . $conn->connect_error); } // Fetch data
$sql = "SELECT name, mark FROM students";
$result = $conn->query($sql);
if ($result->num_rows > 0)
{ // Output data of each row echo "<table border='1'><tr><th>Name</th><th>Mark</th></tr>";
while($row = $result->fetch_assoc())
{ echo "<tr><td>" . $row["name"]. "</td><td>" . $row["mark"]. "</td></tr>"; }
echo "</table>";
}
else
{ echo "0 results"; }
$conn->close();
?>
```
### Step 11
Save and close the file. Ensure the correct permissions:<br>
`chown apache:apache /var/www/php.table.com/table.php`
`chmod 644 /var/www/php.table.com/table.php`


Open your web browser and navigate to `http://php.table.com/table.php`. You should see a table displaying the contents from the students database.


