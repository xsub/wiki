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
