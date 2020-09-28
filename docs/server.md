title: Server
template: extrahead.html

### Update & Upgrade your server

```bash
sudo apt-get update
```
```bash
DEBIAN_FRONTEND=noninteractive apt-get -y -o Dpkg::Options::="--force-confdef" -o Dpkg::Options::="--force-confold" upgrade
```


### Nginx

```bash
sudo apt-get install -y nginx
```

### MySql

Installing mysql + set your `root` password. Do not use `CHANGETHIS`!
```bash
sudo debconf-set-selections <<< "mysql-server mysql-server/root_password password CHANGETHIS"
sudo debconf-set-selections <<< "mysql-server mysql-server/root_password_again password CHANGETHIS"
```

```bash
sudo apt-get install -y mysql-server
sudo wget http://repo.mysql.com/mysql-apt-config_0.8.9-1_all.deb
sudo -E dpkg -i mysql-apt-config_0.8.9-1_all.deb
sudo apt-get update
sudo apt-get install -y mysql-server
```

### PHP

```bash
sudo add-apt-repository -y ppa:ondrej/php && sudo apt-get update
sudo apt-get install -y php7.3-cli php7.3-fpm php7.3-mysql php7.3-curl php-memcached php7.3-dev php7.3-sqlite3 php7.3-mbstring php7.3-xml freetds-common freetds-bin unixodbc php7.3-sybase
sudo apt install -y php7.3-bcmath
sudo apt install -y zip unzip php7.3-zip
sudo apt-cache search php7.3
sudo apt install -y php7.3-zip
```

For that php.ini

```bash
sudo sed -i.bak 's/^;cgi.fix_pathinfo.*$/cgi.fix_pathinfo = 1/g' /etc/php/7.3/fpm/php.ini
```

Restarting php7.3-fpm

```bash
sudo service php7.3-fpm restart
```

### ACL & NPM

```bash
sudo apt-get install acl
sudo apt install -y npm
```

### Sendmail

If you don't want to use sendmail, then you dont need to install it. Change the hostname to your own.

```bash
sudo apt-get install -y sendmail
sudo hostnamectl set-hostname silkroad-laravel.de
```

### Database

Change `CHANGETHIS` to your choosen root password.
```bash
echo mysql_upgrade -u root -pCHANGETHIS --force
echo "CREATE DATABASE DB_SILKROAD" | mysql -uroot -pCHANGETHIS
echo "FLUSH PRIVILEGES" | mysql -uroot -pCHANGETHIS
echo "ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'root';" | mysql -uroot -pCHANGETHIS
```

### Host configration

If you have an ssl certificate you can use this config:

- Change the `ssl_certificate` & `ssl_certificate_key` to your own certificate path.
- Change the `root` to your project root path.

```bash
cat << 'EOF' > /etc/nginx/sites-available/default
server {
    listen 80;
    server_name localhost;
    return 301 https://$host$request_uri;
}

server {
  listen 443 ssl;

  server_name localhost;

  ssl_certificate     /var/www/ssl-script/silkroad-laravel.crt;
  ssl_certificate_key /var/www/ssl-script/device.key;

	# Useful logs for debug.
	access_log /var/log/nginx/localhost_access.log;
	error_log /var/log/nginx/localhost_error.log;
	rewrite_log on;

	# The location of our projects public directory.
	root  /var/www/public;

	# Point index to the Laravel front controller.
	index index.php index.html;

	location / {
		try_files $uri $uri/ /index.php?$query_string;
	}

	# Remove trailing slash to please routing system.
	if (!-d $request_filename) {
		rewrite ^/(.+)/$ /$1 permanent;
	}

	# PHP FPM configuration.
	location ~* \.php$ {
		fastcgi_pass unix:/run/php/php7.3-fpm.sock;
		fastcgi_index index.php;
		fastcgi_split_path_info ^(.+\.php)(.*)$;
		include /etc/nginx/fastcgi_params;
		fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
	}

	# We don't need .ht files with nginx.
	location ~ /\.ht {
		deny all;
	}

	# Set header expirations on per-project basis
	location ~* \.(?:ico|css|js|jpe?g|JPG|png|svg|woff)$ {
		expires 365d;
	}
}
EOF
```

### Finish

```bash
sudo service nginx restart
sudo service php7.3-fpm restart
sudo service mysql restart
```

Finished with the server Configuration.
