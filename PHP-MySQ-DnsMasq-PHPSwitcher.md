# Homebrew Fresh Install PHP/MySQL

Setup a fresh (first time) local PHP/MySQL/DnsMasq/PHPSwitcher .dev using Homebrew on Sierra.

## Install HomeBrew

	/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
	brew install wget
	brew doctor
	brew tap homebrew/dupes
	brew tap homebrew/versions
	brew tap homebrew/php
	brew tap homebrew/apache
	brew update

## Install Apache

Unload original apache

	sudo apachectl stop
	sudo launchctl unload -w /System/Library/LaunchDaemons/org.apache.httpd.plist 2>/dev/null
	brew install httpd24 --with-privileged-ports --with-http2

Load new Apache version at startup

	sudo cp -v /usr/local/Cellar/httpd24/2.4.5/homebrew.mxcl.httpd24.plist /Library/LaunchDaemons
	sudo chown -v root:wheel /Library/LaunchDaemons/homebrew.mxcl.httpd24.plist
	sudo chmod -v 644 /Library/LaunchDaemons/homebrew.mxcl.httpd24.plist
	sudo launchctl load -w /Library/LaunchDaemons/homebrew.mxcl.httpd24.plist

Make sure all http instances have been stopped before going on.

	sudo apachectl -k stop
	ps -aef | grep httpd
	sudo apachectl -k start

Check for errors.

	tail -f /usr/local/var/log/apache2/error_log

Modify httpd.conf and test Apache.

	sudo nano /usr/local/etc/apache2/2.4/httpd.conf 

Edit File.

	DocumentRoot ~/Sites/active-projects

	AllowOverride All

Uncomment 

	rewrite_module

Edit User

	User YourLocalUser
	Group staff


Save File. Create an index page in webroot.

	echo "<h1>Active Projects Web Root</h1>" > ~/Sites/active-projects/index.html

##Setup PHP 5.5.x

	brew link php55
	brew install php55 --with-httpd24
	brew install --build-from-source php55-intl
	brew install php55-opcache 
	brew install php55-apcu
	brew install php55-mcrypt  
	brew rm php55-imagick && brew install --build-from-source php55-imagick
	brew install php55-pdo-dblib
	brew install php55-yaml

Add to httpd.conf so we can test the first php install.

	sudo nano /usr/local/etc/apache2/2.4/httpd.conf file
	
Edit file

	<If Module dir_module>
	DirectoryIndex index.php index.html
	</IfModule>

	<FilesMatch \.php$>
	SetHandler application/x-httpd-php
	</FilesMatch>

Test it.

	sudo apachectl -k restart

If all went well, set other php versions - unlink first.

	brew unlink php55
	
##Setup PHP 7.1.x

	brew link php71
	brew install php71 --with-httpd24
	brew install --build-from-source php71-intl
	brew install php71-opcache 
	brew install php71-apcu
	brew install php71-mcrypt  
	brew rm php71-imagick && brew install --build-from-source php71-imagick
	brew install php71-pdo-dblib
	brew install php71-yaml
	brew unlink php71

##Setup PHP 7.0.x

	brew link php70
	brew install php70 --with-httpd24
	brew install --build-from-source php70-intl
	brew install php70-opcache 
	brew install php70-apcu
	brew install php70-mcrypt  
	brew rm php70-imagick && brew install --build-from-source php70-imagick
	brew install php70-pdo-dblib
	brew install php70-yaml
	brew unlink php70

## Setup PHP 5.6.x

	brew link php56
	brew install php56 --with-httpd24
	brew install --build-from-source php56-intl
	brew install php56-opcache 
	brew install php56-apcu
	brew install php56-mcrypt  
	brew rm php56-imagick && brew install --build-from-source php56-imagick
	brew install php56-pdo-dblib
	brew install php56-yaml

We should still have php56 linked.

###PHP ini paths

	/usr/local/etc/php/5.5/php.ini
	/usr/local/etc/php/5.6/php.ini
	/usr/local/etc/php/7.0/php.ini
	/usr/local/etc/php/7.1/php.ini

We can only have one module processing PHP at a time, so for now, add all versions then comment out all but the php56 entry:

	sudo nano /usr/local/etc/apache2/2.4/httpd.conf 

Edit File.

	#LoadModule php5_module    /usr/local/opt/php55/libexec/apache2/libphp5.so
	LoadModule php5_module    /usr/local/opt/php56/libexec/apache2/libphp5.so
	#LoadModule php7_module    /usr/local/opt/php70/libexec/apache2/libphp7.so
	#LoadModule php7_module    /usr/local/opt/php71/libexec/apache2/libphp7.so

Verify php install one more time.

	sudo apachectl -k restart

##Install php switcher

	sudo curl -L https://gist.github.com/w00fz/142b6b19750ea6979137b963df959d11/raw > /usr/local/bin/sphp
	sudo chmod +x /usr/local/bin/sphp

It's IMPORTANT at this stage to fully stop the Apache sever, and start it again. Do not just restart it!

	sudo apachectl -k stop
	sudo apachectl start

Add LoadModule for sphp switcher to httpd.conf (final PHP change)

	sudo nano /usr/local/etc/apache2/2.4/httpd.conf 

Edit File

	LoadModule php5_module /usr/local/lib/libphp5.so
	#LoadModule php7_module /usr/local/lib/libphp7.so

Test php switcher. You will be asked for permissions if needed.

	sphp 55
	sphp 70
	sphp 71
	sphp 56

##Virtual hosts

Apache already comes preconfigured to support this behavior but it is not enabled.

First you will need to uncomment the following lines in your httpd.conf file:

	sudo nano /usr/local/etc/apache2/2.4/httpd.conf
Uncomment

	LoadModule vhost_alias_module libexec/mod_vhost_alias.so

and:

	Include /usr/local/etc/apache2/2.4/extra/httpd-vhosts.conf

and add the following entries:

	<VirtualHost *:80>
		DocumentRoot "~/Sites/active-projects"
		ServerName localhost
	</VirtualHost>

	<VirtualHost *:80>
		DocumentRoot "~/Sites/active-projects/myproject.dev"
		ServerName myproject.dev
	</VirtualHost>

Make the directory

	mkdir ~/Sites/active-projects/myproject.dev

##Install DnsMasq

	brew install dnsmasq
	echo 'address=/.dev/127.0.0.1' > /usr/local/etc/dnsmasq.conf
	sudo brew services start dnsmasq
	sudo mkdir -v /etc/resolver
	sudo bash -c 'echo "nameserver 127.0.0.1" > /etc/resolver/dev'

Test it!

	ping fakedomain.dev

##Generate Self-Signed SSL Cert

	sudo openssl genrsa -des3 -passout pass:x -out local.dev.pass.key 2048
	sudo openssl rsa -passin pass:x -in local.dev.pass.key -out local.dev.key
	sudo rm local.dev.pass.key
	sudo openssl req -new -key local.dev.key -out local.dev.csr
	sudo openssl x509 -req -days 365 -in local.dev.csr -signkey local.dev.key -out local.dev.crt

Move them in place local.dev.key and local.dev.crt (example).

	sudo mv local.dev.crt /usr/local/etc/apache2/2.4/

Edit httpd.conf again and enable SSL and add virtual host (example).

	sudo nano /usr/local/etc/apache2/2.4/httpd.conf

	<VirtualHost _default_:443>
	    ServerName myproject.dev
	    DocumentRoot "~/Sites/active-projects/myproject.dev"
	    SSLEngine on
	 	SSLCertificateFile "/usr/local/etc/apache2/2.4/local.dev.crt"
		SSLCertificateKeyFile "/usr/local/etc/apache2/2.4/local.dev.key"
	</VirtualHost>

	# Secure (SSL/TLS) connections
	Include /usr/local/etc/apache2/2.4/extra/httpd-ssl.conf
	#
	# Note: The following must must be present to support
	#       starting without SSL on platforms with no /dev/random equivalent
	#       but a statically compiled-in mod_ssl.
	#
	<IfModule ssl_module>
		SSLRandomSeed startup builtin
		SSLRandomSeed connect builtin
	</IfModule>

Edit ssl configuration.

	sudo nano /usr/local/etc/apache2/2.4/extra/httpd-ssl.conf

Adjust paths and settings based on your local setup as needed.

Test it.

	sudo apachectl -k restart

##Install MySQL

	brew install mysql
	brew services start mysql

or 

	mysql.server start
	
Secure it.

	mysql_secure_installation

Tweak as needed.

##Download and Install Composer Globally

	php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
	
	php -r "if (hash_file('SHA384', 'composer-setup.php') === '55d6ead61b29c7bdee5cccfb50076874187bd9f21f65d8991d46ec5cc90518f447387fb9f76ebae1fbbacf329e583e30') 	{ echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo 	PHP_EOL;"
	
	php composer-setup.php
	
	php -r "unlink('composer-setup.php');"
	
	mv composer.phar /usr/local/bin/composer

Note: If the above fails due to permissions, you may need to run it again with sudo.

Note: On some versions of OSX the /usr directory does not exist by default. If you receive the error "/usr/local/bin/composer: No such file or directory" then you must create the directory manually before proceeding: mkdir -p /usr/local/bin.

Note: For information on changing your PATH, please read the Wikipedia article and/or use Google.

Now just run composer in order to run Composer instead of php composer.phar.

	composer update -vv









