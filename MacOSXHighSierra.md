# Mac OSX High Sierra :

High Sierra bringt Apache 2.4 und PHP 7.1 bereits mit.


##### Projekt clonen

    mkdir ~/code
    cd ~/code
    git clone https://github.com/henningd/createmacdevmachine.git

## PHP Extension anpassen

Leider funktioniert das von High Sierra mit gelieferte ```xdebug``` nicht, da es nicht mit der Zend Enginge von PHP7 kompatiebel ist.


    /usr/lib/php/extensions/no-debug-non-zts-20160303/xdebug.so

Da High Sierra das Überschreiben von xdebug.so nicht zuläßt, legt man einfach ein separates Verzeichnis an, in das die neue xdebug.so kopiert wird.

Die Datei ```xdebug.so``` kopieren

    mkdir /usr/local/lib/php/extensions/71dh
    cp /Users/danielhenninger/code/createmacdevmachine/MacOSXHighSierra/php71-xdebug/2.5.5/xdebug.so /usr/local/lib/php/extensions/71dh


##### xdebug.so erzeugen, falls notwendig
Um eine funktionierende Version von xdebug zu erzeugen, sind die folgenden Schritte notwendig:

    cd ~/code
    git clone git://github.com/xdebug/xdebug.git
    cd ~/code/xdebug
    brew install autoconf
    phpize
    ./configure
    make

Die neue Datei liegt nun in ```modules/xdebug.so```.

## PHP.INI erstellen

Im Verzeichnis /etc muss eine php.ini erstellt werden.

Datei kopieren

    cp /Users/danielhenninger/code/createmacdevmachine/MacOSXHighSierra/php71/php.ini /etc

Bei manieller Erstellung der php.ini sind die folgenden Einträge zu setzen

    [xdebug]
    ;zend_extension="/usr/lib/php/extensions/no-debug-non-zts-20160303/xdebug.so"
    zend_extension="/usr/local/lib/php/extensions/71dh/xdebug.so"
    xdebug.remote_enable=true
    xdebug.remote_host=localhost
    xdebug.remote_port=9000
    xdebug.remote_handler=dbgp
    xdebug.var_display_max_children=-1
    xdebug.var_display_max_data=-1
    xdebug.var_display_max_depth=-1


## Apache anpassen

    sudo atom /private/etc/apache2/httpd.conf

Die folgenden Zeilen einkommentieren (# entfernen)

    LoadModule deflate_module libexec/apache2/mod_deflate.so
    LoadModule env_module libexec/apache2/mod_env.so
    LoadModule expires_module libexec/apache2/mod_expires.so
    LoadModule headers_module libexec/apache2/mod_headers.so
    LoadModule userdir_module libexec/apache2/mod_userdir.so
    LoadModule alias_module libexec/apache2/mod_alias.so
    LoadModule rewrite_module libexec/apache2/mod_rewrite.so
    LoadModule php7_module libexec/apache2/libphp7.so
    LoadModule perl_module libexec/apache2/mod_perl.so

    Include /private/etc/apache2/extra/httpd-userdir.conf
    Include /private/etc/apache2/extra/httpd-vhosts.conf
    Include /private/etc/apache2/other/*.conf

Dateiinhalt bearbeiten ```php7.conf```

    sudo atom /private/etc/apache2/other/php7.conf

Dateiinhalt ```php7.conf```

    <IfModule php7_module>
    	AddType application/x-httpd-php .php
    	AddType application/x-httpd-php-source .phps

    	<IfModule dir_module>
    		DirectoryIndex index.html index.php
    	</IfModule>
    </IfModule>


## VHOST anpassen

    sudo atom /private/etc/apache2/extra/httpd-vhosts.conf

## HOSTS anpassen

    sudo atom /etc/hosts

Eintrag für neue WEB-Site

    127.0.0.1 test1.dev

## Apache neu starten
    sudo apachectl graceful

## Resourcen

https://xdebug.org/index.php

