---
title: Apache htpasswd tricks
---

Here are all kinds of examples of htpasswd files I use on a regular basis. Hopefully they can be useful for your application.

First thing is first for any htpasswd. You need the users so we will create them here. The secure location if up to you.

{% highlight text %}
htpasswd -c /var/www/.htpasswd first_usr
Adding user first_usr
New password:
Re-type new password:

htpasswd /var/www/.htpasswd second_usr
Adding user second_usr
New password:
Re-type new password:

htpasswd /var/www/.htpasswd subsequent_usr
Adding user subsequent_usr
New password:
Re-type new password:
{% endhighlight %}

Basic .htaccess file. Create this in the directory you want to protect. Once this is installed all directories below this one will be protected.

{% highlight text %}
AuthName "Password Protected"
AuthUserFile /var/www/.htpasswd
AuthType Basic
Require valid-user
{% endhighlight %}

Blocking all users except those who authenticate or come from one of the following IP address with the X-Forwarded-For header. This is great for those who have a load balancer in-front of their solution.

{% highlight text %}
AuthName "Password Protected"
AuthUserFile /var/www/.htpasswd
AuthType Basic
Require valid-user
Order deny,allow
Deny from all
SetEnvIf X-Forwarded-For "^10\.209\.146\.46" AllowAccess
SetEnvIf X-Forwarded-For "^10\.209\.132\.232" AllowAccess
SetEnvIf X-Forwarded-For "^10\.209\.251\.158" AllowAccess
SetEnvIf X-Forwarded-For "^10\.209\.160\.144" AllowAccess
Allow from env=AllowAccess
Satisfy Any
{% endhighlight %}

How about using MySQL for authentication? I am assuming you have Apache2 and MySQL/Percona/MariaDB already installed. We are only covering the installation of the Apache module, creating the tables and activating.

For Red Hat Enterprise Linux 6 and 7, CentOS 6 and 7, Fedora, Scientific Linux and all other RHEL 6 and 7 based distro’s.

{% highlight text %}
yum install mod_auth_mysql
{% endhighlight %}

For Debian, Ubunutu and other distro’s based on Debian.

{% highlight text %}
aptitude install libapache2-mod-auth-mysql
{% endhighlight %}

{% highlight text %}
mysql
mysql>CREATE DATABASE htpasswd-auth;
mysql>GRANT SELECT, INSERT, UPDATE, DELETE ON htpasswd-auth.* TO 'htpasswd-auth_admin'@'10.%' IDENTIFIED BY 'htpasswd-auth_password';
mysql>FLUSH PRIVILEGES;
mysql>USE htpasswd-auth;
mysql>CREATE TABLE users (user_name CHAR(30) NOT NULL user_passwd CHAR(20) NOT NULL PRIMARY KEY (user_name));
mysql>CREATE TABLE groups (user_name CHAR(30) NOT NULL user_group CHAR(20) NOT NULL PRIMARY KEY (user_name, user_group));
mysql>INSERT INTO `mysql_auth` (`username`, `passwd`, `groups`) VALUES('test', MD5('test'), 'testgroup');
{% endhighlight %}

{% highlight text %}
AuthBasicAuthoritative Off
AuthUserFile /dev/null
AuthMySQL On
AuthName "Authentication required"
AuthType Basic
Auth_MySQL_Host localhost
Auth_MySQL_User examplecom_admin
Auth_MySQL_Password examplecom_admin_password
AuthMySQL_DB examplecomdb
AuthMySQL_Password_Table mysql_auth
Auth_MySQL_Username_Field username
Auth_MySQL_Password_Field passwd
Auth_MySQL_Empty_Passwords Off
Auth_MySQL_Encryption_Types PHP_MD5
Auth_MySQL_Authoritative On
require valid-user
{% endhighlight %}
