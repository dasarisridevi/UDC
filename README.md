User Data Collector Project
This project is a PHP-based web application designed to collect user data and store it in a MySQL database. It also allows users to upload files, which are stored on the server.

Features
Collects user data through a web form.
Stores the data in a MySQL database.
Allows file uploads and stores the files on the server.
Setup and Configuration
Prerequisites
VirtualBox
PuTTY
Web Server Setup
Install Apache:

Run the command: yum install httpd -y
Set hostname:

Edit the file /etc/hostname and set the hostname to udc.example.com.
Install PHP and other tools:

Run the commands to enable and install PHP 8.1:
dnf module list php
dnf module -y enable php:8.1
dnf module -y install php:8.1/common
Install MySQL and PHP MySQLi extension:
yum install mysql -y
yum install php-mysqli -y
Check PHP version and enable httpd service:

Verify PHP installation with: php -v
Enable and start the Apache service: systemctl enable --now httpd
Create a test PHP page:

Navigate to the web server root directory: cd /var/www/html
Create and edit a file named php_test.php with the following content:
<!DOCTYPE html>
<html>
<body>
<h1>My first PHP page</h1>
<?php echo "Hello World!"; ?>
</body>
</html>
Test the PHP page:

Open your browser and navigate to http://<Web Server IP Address>/php_test.php.
If it doesn't work, disable the firewall with:
setenforce 0
systemctl stop firewalld
Database Server Setup
Install MySQL Server:

Run the command: dnf -y install mysql-server
Configure character set:

Edit the file /etc/my.cnf.d/charset.cnf and add the following content:
[mysqld]
character-set-server = utf8mb4
[client]
default-character-set = utf8mb4
Enable MySQL service:

Run the command: systemctl enable --now mysqld
Secure MySQL installation:

Run the command: mysql_secure_installation
Follow the prompts (No, No, Yes, Yes, Yes).
Test database creation:

Log into MySQL with: mysql -u root -p
Create and test a database and table with the following commands:
create database test_database;
create table test_database.test_table (id int, name varchar(50), address varchar(50), primary key (id));
insert into test_database.test_table(id, name, address) values("031", "CentOS", "India");
select * from test_database.test_table;
drop database test_database;
Exit MySQL with: exit;
User Data Collector Application Setup
Create the database and user:

Log into MySQL with: mysql -u root -p
Create the database and user with the following commands:
CREATE DATABASE udc;
CREATE USER 'udc'@'%' IDENTIFIED BY 'Welcome@123';
GRANT ALL PRIVILEGES ON udc.* TO 'udc'@'%';
Exit MySQL with: exit;
Verify connection from web server:

Test the connection to the database from the web server with:
mysql -h <DB Server IP> -u udc -p
Create the table:

Log into MySQL with: mysql -u udc -p
Create the table with the following commands:
USE udc;
CREATE TABLE users (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(255) NOT NULL, age INT NOT NULL, country VARCHAR(255) NOT NULL);
Exit MySQL with: exit;
Create upload directories and set permissions:

Run the commands:
mkdir -p /var/udc/uploads
chmod 777 /var/udc /var/udc/uploads
Main Application Page
Create the main application page:
Create and edit a file named main.php in the web server root directory (/var/www/html) with the following content:
<!DOCTYPE html>
<html>
<head>
<title>User Data Collection</title>
</head>
<body>
<?php
// MySQL database configuration
$servername = "192.168.29.43";
$username = "sridevi";
$password = "welcome@123";
$dbname = "udc";

// Create a database connection
$conn = new mysqli($servername, $username, $password, $dbname);

// Check connection
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}

if ($_SERVER["REQUEST_METHOD"] == "POST") {
    // Collect user data
    $name = $_POST["name"];
    $age = $_POST["age"];
    $country = $_POST["country"];

    // Insert data into the MySQL database
    $sql = "INSERT INTO users (name, age, country) VALUES ('$name', $age, '$country')";
    if ($conn->query($sql) === TRUE) {
        echo "User data has been successfully stored in the database.<br>";
    } else {
        echo "Error: " . $sql . "<br>" . $conn->error;
    }

    // Handle file upload
    $uploadDir = '/var/udc/uploads/';
    $uploadFile = $uploadDir . basename($_FILES['userfile']['name']);
    if (move_uploaded_file($_FILES['userfile']['tmp_name'], $uploadFile)) {
        echo "File is valid, and it has been successfully uploaded.<br>";
    } else {
        echo "File upload failed.<br>";
    }
}
?>
<h2>Enter User Information</h2>
<form method="post" enctype="multipart/form-data">
    Name: <input type="text" name="name"><br>
    Age: <input type="number" name="age"><br>
    Country: <input type="text" name="country"><br>
    File Upload: <input type="file" name="userfile"><br>
    <input type="submit" value="Submit">
</form>
<?php
// Close the database connection
$conn->close();
?>
</body>
</html>
Usage
Start the web server and database server.
Open your browser and navigate to http://<Web Server IP Address>/main.php.
Fill out the form and submit it.
Check the database to see the stored user data and uploaded files.
