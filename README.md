# User Data Collector Project

This project is a PHP-based web application designed to collect user data and store it in a MySQL database. It also allows users to upload files, which are stored on the server.

## Features
- Collects user data through a web form.
- Stores the data in a MySQL database.
- Allows file uploads and stores the files on the server.

## Setup and Configuration

### Prerequisites
- VirtualBox
- PuTTY

### Web Server Setup

1. **Install Apache:**
   ```bash
   yum install httpd -y
   ```

2. **Set Hostname:**
   Edit the file `/etc/hostname` and set the hostname to `udc.example.com`.

3. **Install PHP and Other Tools:**
   Run the following commands to enable and install PHP 8.1:
   ```bash
   dnf module list php
   dnf module -y enable php:8.1
   dnf module -y install php:8.1/common
   ```

4. **Install MySQL and PHP MySQLi Extension:**
   ```bash
   yum install mysql -y
   yum install php-mysqli -y
   ```

5. **Check PHP Version and Enable Apache Service:**
   - Verify PHP installation with:
     ```bash
     php -v
     ```
   - Enable and start the Apache service:
     ```bash
     systemctl enable --now httpd
     ```

6. **Create a Test PHP Page:**
   - Navigate to the web server root directory:
     ```bash
     cd /var/www/html
     ```
   - Create and edit a file named `php_test.php` with the following content:
     ```php
     <?php
     echo "My first PHP page";
     ?>
     ```

7. **Test the PHP Page:**
   - Open your browser and navigate to `http://<your_server_ip>/php_test.php`.
   - If it doesn't work, disable the firewall with:
     ```bash
     setenforce 0
     systemctl stop firewalld
     ```

### Database Server Setup

1. **Install MySQL Server:**
   ```bash
   dnf -y install mysql-server
   ```

2. **Configure Character Set:**
   - Edit the file `/etc/my.cnf.d/charset.cnf` and add the following content:
     ```
     [mysqld]
     character-set-server = utf8mb4

     [client]
     default-character-set = utf8mb4
     ```

3. **Enable MySQL Service:**
   ```bash
   systemctl enable --now mysqld
   ```

4. **Secure MySQL Installation:**
   ```bash
   mysql_secure_installation
   ```
   Follow the prompts (No, No, Yes, Yes, Yes).

5. **Test Database Creation:**
   - Log into MySQL:
     ```bash
     mysql -u root -p
     ```
   - Create and test a database and table:
     ```sql
     create database test_database;
     create table test_database.test_table (id int, name varchar(50), address varchar(50), primary key (id));
     insert into test_database.test_table(id, name, address) values("031", "CentOS", "India");
     select * from test_database.test_table;
     drop database test_database;
     ```
   - Exit MySQL:
     ```sql
     exit;
     ```

### User Data Collector Application Setup

1. **Create the Database and User:**
   - Log into MySQL:
     ```bash
     mysql -u root -p
     ```
   - Create the database and user:
     ```sql
     CREATE DATABASE udc;
     CREATE USER 'udc'@'%' IDENTIFIED BY 'Welcome@123';
     GRANT ALL PRIVILEGES ON udc.* TO 'udc'@'%';
     ```
   - Exit MySQL:
     ```sql
     exit;
     ```

2. **Verify Connection from Web Server:**
   - Test the connection to the database from the web server:
     ```bash
     mysql -h <database_host> -u udc -p
     ```

3. **Create the Table:**
   - Log into MySQL:
     ```bash
     mysql -u udc -p
     ```
   - Create the table:
     ```sql
     USE udc;
     CREATE TABLE users (
         id INT AUTO_INCREMENT PRIMARY KEY,
         name VARCHAR(255) NOT NULL,
         age INT NOT NULL,
         country VARCHAR(255) NOT NULL
     );
     ```
   - Exit MySQL:
     ```sql
     exit;
     ```

4. **Create Upload Directories and Set Permissions:**
   ```bash
   mkdir -p /var/udc/uploads
   chmod 777 /var/udc /var/udc/uploads
   ```

### Main Application Page

1. **Create the Main Application Page:**
   - Create and edit a file named `main.php` in the web server root directory (`/var/www/html`) with the following content:
     ```php
     <!DOCTYPE html>
     <html lang="en">
     <head>
         <meta charset="UTF-8">
         <title>User Data Collection</title>
     </head>
     <body>
         <h2>Enter User Information</h2>
         <form method="post" enctype="multipart/form-data" action="<?php echo htmlspecialchars($_SERVER["PHP_SELF"]); ?>">
             Name: <input type="text" name="name"><br><br>
             Age: <input type="number" name="age"><br><br>
             Country: <input type="text" name="country"><br><br>
             File Upload: <input type="file" name="userfile"><br><br>
             <input type="submit" value="Submit">
         </form>
         <?php
         // Database configuration
         $servername = "localhost";
         $username = "udc";
         $password = "Welcome@123";
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

         $conn->close();
         ?>
     </body>
     </html>
     ```

### Usage

1. **Start the Web Server and Database Server.**
2. **Open Your Browser and Navigate to `http://<your_server_ip>/main.php`.**
3. **Fill Out the Form and Submit It.**
4. **Check the Database to See the Stored User Data and Uploaded Files.**
