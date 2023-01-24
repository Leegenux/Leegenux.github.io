---
title: Ubuntu 18.04 NextCloud Installation Guide
date: 2019-03-20 8:10
tags: [ubuntu, linux, nextcloud, apache]
---

This article breaks the installation and configuration of the NextCloud into 4 parts, and shows you the path to your own private cloud storage. These 4 parts are:

- PHP
- NextCloud tarball installation
- DB server
- Apache / Nginx server

Though this guide is Ubuntu-titled, all is applicable to other Linux distributions but the package management with `apt` part.

## PHP installation

According to the official document, for Ubuntu server user, we can type the commands below to fetch and install the PHP modules supporting the core NextCloud server hosting.

```bash
# apt-get install apache2 mariadb-server libapache2-mod-php7.2
# apt-get install php7.2-gd php7.2-json php7.2-mysql php7.2-curl php7.2-mbstring
# apt-get install php7.2-intl php-imagick php7.2-xml php7.2-zip
```

And if you need some PHP extensions, you should also install the `libapache2-mod-php7.2`. For more information, please refer to [here](https://docs.nextcloud.com/server/15/admin_manual/installation/source_installation.html#prerequisites-label).



## NextCloud Tarball Installation

Up to the day the note is finished, the latest tarball available can be downloaded through this [link](https://docs.nextcloud.com/server/15/admin_manual/installation/source_installation.html#prerequisites-label).

After validations, unzip it and move it to the document root of your web server. For example:

```bash
# unzip nextcloud-15.0.5.1.zip -d nextCloudInstall
# cp nextCloudInstall /var/www/nextcloud
# chown -R www-data:www-data /var/www/nextCloud
```



## DB server

Let take mariaDB for an example.

### Installation

Its installation is quite straightforward:

1. Add the MariaDB signing key:

   ```bash
   apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xF1656F24C74CD1D8
   ```

2. Download and execute the script to set up the mariadb apt repository:

   ```bash
   curl -sS https://downloads.mariadb.com/MariaDB/mariadb_repo_setup | sudo bash
   ```

3. Update your sources list:

   ```
   apt update
   ```

4. Install MariaDB:

   ```
   apt install mariadb-server
   ```

On finishing the installation, a prompt of setting the primary password is supposed to pop up. But under some special circumstances, it won't. So you are not able to login into the server. To solve this, you can try either reinstalling the `mariadb-server` or solution below:

1. Restart the service in `nopass` mode:

```
# systemctl restart mariadb && mysql_safe --skip-grant-tables &
```

2. Update your password

```mysql
use mysql;
update user set authentication_string=PASSWORD("") where User='root';
update user set plugin="mysql_native_password" where User='root';  # THIS LINE

flush privileges;
quit;
```

or if you encounter non-updatable warning, try followings:

```mysql
FLUSH PRIVILEGES;
USE mysql;
ALTER USER 'root'@'localhost' IDENTIFIED BY '';
FLUSH PRIVILEGES;
```

Then you can log in with password `''` now. It's recommended that you reset a stronger password rather than `''`.



### DB Configurations For NextCloud

Make sure that you can log into the mysql as root before doing followings.

```mysql
CREATE USER 'nextcloud'@'localhost' identified by 'StrongPassword';
CREATE DATABASE nextcloud;
GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@'localhost';
FLUSH PRIVILEGES;
```

After that, set the transcation isolation level to avoid problems. Also it's recommended to set the `binlog_format`:

```
[mysqld]
transaction_isolation = READ-COMMITTED
binlog_format = ROW
```

(If not present, you can just create the file and add these lines.)

Since I am using mariadb, which is an fork of mysqldb, there are other configurations necessary under `/etc/php/7.2/apache2/conf.d/mysql.ini`. Configuration below activates the connection between mariadb and apache, where option `extension`  and `mysql.default_socket` are crucial : 

```php
# configuration for PHP MySQL module
extension=pdo_mysql.so

[mysql]
mysql.allow_local_infile=On
mysql.allow_persistent=On
mysql.cache_size=2000
mysql.max_persistent=-1
mysql.max_links=-1
mysql.default_port=
mysql.default_socket=/var/lib/mysql/mysql.sock  # Debian squeeze: /var/run/mysqld/mysqld.sock
mysql.default_host=
mysql.default_user=
mysql.default_password=
mysql.connect_timeout=60
mysql.trace_mode=Off
```



## Apache Server Installation and Configurations

On Debian, Ubuntu, and their derivatives, Apache installs with a useful configuration so all you have to do is create a `/etc/apache2/sites-available/nextcloud.conf` file with these lines in it, replacing the Directory and other filepaths with your own filepaths:

```nginx
Alias /nextcloud "/var/www/nextcloud/"

<Directory /var/www/nextcloud/>
  Options +FollowSymlinks
  AllowOverride All
  Satisfy Any

 <IfModule mod_dav.c>
  Dav off
 </IfModule>

 SetEnv HOME /var/www/nextcloud
 SetEnv HTTP_HOME /var/www/nextcloud

</Directory>
```

Then enable the newly created site: `# a2ensite nextcloud.conf`.

Also, you should enable the module `mod_rewrite`: `# a2enmod rewrite`

Other recommended modules are:

```bash
# a2enmod headers
# a2enmod env
# a2enmod dir
# a2enmod mime
```

You must disable any server-configured authentication for Nextcloud, as it uses Basic authenticationi internally for DAV services. Following the above example configuration file, add the following line in the <Directory> section of in `nextcloud.conf`:

```
Satisfy Any
```

After all above, renable the apache server:

```bash
# a2enmod rewrite dir mime env headers
# systemctl restart apache2
```

Now your site is reachable. Note that the firewall might block outer connections.



## Trouble Shooting Tips

If you get Internal Server Error at your first access to the site. Find some  information from `/var/log/apache2/error.log` and `/var/log/php*`. 

For example, if you found `chmod()` related error. You may solve it with

```
# chown -R www-data:www-data /var/www/html && chown -R www-data:www-data /var/www/nextcloud
```

