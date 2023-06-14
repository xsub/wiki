# A05 ❯ Apache: enabling mod_authn_dbd
<small>ℹ️ This article is part of AlmaLinux [System Series](/series/).</small>
<hr>
| 💡 | Experience Level  | ⭐☆☆☆☆ |
|--- | --------- | --------|
| 📆 | <small>Last modified </small>| 2023-06-14
| 🔧 | <small>Tested by <br> ↳ version \| platform \| date </small>| NOT TESTED YET |
<br> 

## 🌟 Introduction

The `mod_authn_dbd` module in Apache allows you to authenticate users using data stored in a database, such as SQL. Follow these steps in this guide to enable this module.

### 1. Install the Necessary Packages

First, install the necessary Apache and DBD packages. If you're using MySQL, install the `mod_authn_dbd` and `mod_dbd` modules, along with the corresponding MySQL packages:

```bash
sudo dnf install httpd mod_authn_dbd mod_dbd mysql
```

## 2. Enable the Modules

Next, enable the `mod_authn_dbd` and `mod_dbd` modules. You can do this by adding or uncommenting the following lines in your Apache configuration file (`/etc/httpd/conf/httpd.conf`):

```shell
LoadModule authn_dbd_module modules/mod_authn_dbd.so
LoadModule dbd_module modules/mod_dbd.so
```
You can edit this file using a text editor like vim or nano:

```shell
sudo nano /etc/httpd/conf/httpd.conf
```
Make sure to save your changes and exit the text editor.

### 3. Configure the Database Connection

Now, add the appropriate configuration to connect to your database. You'll use the DBDriver, DBDParams, DBDMin, DBDKeep, and DBDMax directives.

Here's what this might look like for a MySQL database:

```shell
<IfModule mod_dbd.c>
  DBDriver mysql
  DBDParams "dbname=mydatabase user=myuser pass=mypass"
  DBDMin  4
  DBDKeep 8
  DBDMax  20
  DBDExptime 300
</IfModule>
```
::: tip
Replace "mydatabase", "myuser", and "mypass" with your actual database name, user, and password.
:::

### 4. Restart Apache

Finally, restart Apache to enable the new modules and configuration:

```shell
sudo systemctl restart httpd
```

::: warning
Remember to adjust these instructions as necessary based on your specific system configuration and security requirements. Always backup your configuration files before making changes. 
:::


After you have configured the mod_authn_dbd and mod_dbd modules in Apache, you can test whether it's working correctly by creating a protected directory that requires authentication. Here's a step-by-step guide in markdown format:

## Testing mod_authn_dbd in Apache

### 1. Create a Protected Directory

First, create a directory that you want to protect with authentication:

```shell
sudo mkdir /var/www/html/protected
```

### 2. Configure Apache to Protect the Directory

Next, configure Apache to protect the new directory. This will involve creating a new configuration file or modifying an existing one.

Open your Apache configuration file (e.g., /etc/httpd/conf/httpd.conf), and add the following configuration. Replace "mydatabase", "mytable", "myuser", and "mypassword" with your actual database name, table, username, and password:

```shell
<Directory /var/www/html/protected>
  AuthType Basic
  AuthName "Protected Area"
  AuthBasicProvider dbd
  AuthDBDUserPWQuery "SELECT password FROM mytable WHERE user = %s"
  Require valid-user
</Directory>

<IfModule mod_dbd.c>
  DBDriver mysql
  DBDParams "dbname=mydatabase,user=myuser,pass=mypassword"
</IfModule>
```

Save your changes and exit the text editor.

### 3. Restart Apache

Restart Apache to apply the changes:

```
sudo systemctl restart httpd
```

### 4. Test the Protected Directory

Now, try to access the protected directory in a web browser by navigating to http://your-server-ip/protected. You should be prompted for a username and password. Enter the credentials of a user in your database, and you should be granted access.

If you're denied access or aren't prompted for credentials, check your Apache error logs for potential issues. The error log is usually located at `/var/log/httpd/error_log`.

### Final note
Please note that this is a basic setup and should not be used as is in a production environment, just yet. Make sure to implement the next step - encrypt the connection using SSL/TLS and next after that - securely store the passwords in the database. 


## Eenabling SSL/TLS for your Apache server

## 5. Install mod_ssl

First, you'll need to install the `mod_ssl` package, which enables SSL/TLS support in Apache:

```shell
sudo dnf install mod_ssl
```

### 6. Generate a Self-Signed SSL Certificate

For testing purposes, you can create a self-signed SSL certificate. In a real-world, production environment, you should use a certificate issued by a trusted Certificate Authority (CA).

```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/pki/tls/private/apache-selfsigned.key -out /etc/pki/tls/certs/apache-selfsigned.crt
```

You'll be prompted to enter some information for the certificate.

### 7. Configure Apache to Use SSL

Open the SSL configuration file:

```
sudo nano /etc/httpd/conf.d/ssl.conf
```

Find the sections for `SSLCertificateFile` and `SSLCertificateKeyFile` and update them to point to the certificate and key you just generated:

```
SSLCertificateFile /etc/pki/tls/certs/apache-selfsigned.crt
SSLCertificateKeyFile /etc/pki/tls/private/apache-selfsigned.key
```
Save your changes and exit the text editor.

### 8. Redirect HTTP to HTTPS

To ensure that all connections use HTTPS, you can set up a redirect in your Apache configuration. Open your Apache configuration file (e.g., /etc/httpd/conf/httpd.conf), and add the following configuration:

```shell
<VirtualHost *:80>
    ServerName www.example.com
    Redirect "/" "https://www.example.com/"
</VirtualHost>
Replace "www.example.com" with your actual domain name.
```
Save your changes and exit the text editor.

### 9. Restart Apache

Finally, restart Apache to apply the changes:

```
sudo systemctl restart httpd
```

### 10. Test SSL

You can now access your site using https:// and you should see that the connection is secure, though browsers will display a warning because the certificate is self-signed and not trusted.


### Final note: Certificates
Please note, in a production environment, it's essential to use a certificate from a trusted certificate authority (CA) rather than a self-signed certificate. The process to obtain such a certificate varies depending on the CA. Some providers, such as Let's Encrypt, offer free certificates and automated tools to easily install and renew them.

### Secure Password Storage

Implementing secure password storage involves hashing passwords with a strong algorithm and a unique salt before storing them in the database. Unfortunately, Apache's mod_authn_dbd module, which we have been using so far, doesn't support password hashing out of the box. Instead, it expects the AuthDBDUserPWQuery directive to return a plaintext or MD5 hashed password.

To securely store hashed and salted passwords, you'd need to implement the hashing in your application code (i.e., the code that handles user registration and authentication). Since it's not possible to cover all scenarios (as this would depend on your specific programming language, framework, and database system), I'll provide a general example using PHP and MySQL for illustrative purposes:

## Implementing Secure Password Hashing With PHP and MySQL

### 11. Install PHP and MySQL

If not already installed, you would need PHP and MySQL:

```shell
sudo dnf install php php-mysqlnd mariadb-server
```

And start the services:

```shell
sudo systemctl start httpd
sudo systemctl start mariadb
```
### 12. Create a User Registration Page

In your PHP application, when a user registers, you'll hash their password using PHP's built-in password_hash function. Here's a simple example:

```php
<?php
$servername = "localhost";
$username = "username";
$password = "password";
$dbname = "myDB";

// Create connection
$conn = new mysqli($servername, $username, $password, $dbname);

// Check connection
if ($conn->connect_error) {
  die("Connection failed: " . $conn->connect_error);
}

// Hash the user's password
$hashed_password = password_hash($_POST['password'], PASSWORD_DEFAULT);

// Insert the new user into the database
$sql = "INSERT INTO Users (username, password) VALUES (?, ?)";
$stmt = $conn->prepare($sql);
$stmt->bind_param("ss", $_POST['username'], $hashed_password);
$stmt->execute();

$stmt->close();
$conn->close();
?>
```
In this example, replace 'username', 'password', and 'myDB' with your MySQL username, password, and database name. Users should be replaced with the name of your table storing the user credentials.

# 13. Create a User Login Page

When a user logs in, you'll need to fetch their hashed password from the database and use PHP's password_verify function to check it against the password they entered. Here's a simple example:

```php
<?php
$servername = "localhost";
$username = "username";
$password = "password";
$dbname = "myDB";

// Create connection
$conn = new mysqli($servername, $username, $password, $dbname);

// Check connection
if ($conn->connect_error) {
  die("Connection failed: " . $conn->connect_error);
}

// Get the hashed password for the user from the database
$sql = "SELECT password FROM Users WHERE username = ?";
$stmt = $conn->prepare($sql);
$stmt->bind_param("s", $_POST['username']);
$stmt->execute();
$result = $stmt->get_result();
$row = $result->fetch_assoc();
$hashed_password = $row['password'];

// Verify the password the user entered against the hashed password
if (password_verify($_POST['password'], $hashed_password)) {
  // The passwords match, the user is authenticated
} else {
  // The passwords don't match, the user is not authenticated
}

$stmt->close();
$conn->close();
?>
```

Again, replace 'username', 'password', and 'myDB' with your MySQL username,
