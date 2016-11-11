# Mac OSX Sierra : Brew Apache + Mysql + PHP Switcher + DNSMasq + SSL :+1:

Things I had to do using Homebrew to get my local web dev environment back up and running after migration to OSX Sierra + Xcode 8.1. 

Note: I used ```brew reinstall``` because I had already installed most of this previously under Yosemite. Probably better ways to do this, but this is what worked for me.

	brew doctor
#
	brew tap homebrew/dupes
	brew tap homebrew/versions
	brew tap homebrew/php
	brew tap homebrew/apache
#
	brew update
#
	sudo apachectl stop
	sudo launchctl unload -w /System/Library/LaunchDaemons/org.apache.httpd.plist 2>/dev/null
	
	brew reinstall httpd24 --with-privileged-ports --with-http2
#
	sudo cp -v /usr/local/Cellar/httpd24/2.4.23_2/homebrew.mxcl.httpd24.plist /Library/LaunchDaemons

	sudo chown -v root:wheel /Library/LaunchDaemons/homebrew.mxcl.httpd24.plist
	sudo chmod -v 644 /Library/LaunchDaemons/homebrew.mxcl.httpd24.plist
	sudo launchctl load -w /Library/LaunchDaemons/homebrew.mxcl.httpd24.plist

Make sure all http instances have been stopped before going on.

	ps -aef | grep httpd
#
	sudo apachectl -k start

Check for errors.

	tail -f /usr/local/var/log/apache2/error_log
	
Useful commands.

	sudo apachectl start
	sudo apachectl stop
	sudo apachectl -k restart

Modify httpd.conf and test Apache.

	DocumentRoot ~/Sites/active-projects

	AllowOverride All

	Uncomment rewrite_module
#
	User YourLocalUser
	Group staff
#
	ServerAdmin admin@yourdomain.com

Create an index page in webroot.

	echo "<h1>Active Projects Web Root</h1>" > ~/Sites/active-projects/index.html

##Install PHP55

	brew reinstall php55 --with-apache
	brew reinstall php55-opcache 
	brew reinstall php55-apcu
	brew reinstall php55-mcrypt  
	brew reinstall php55-imagick
	brew reinstall php55-pdo-dblib
	brew reinstall php55-yaml
	brew reinstall php55-xdebug

Add to httpd.conf so we can test the first php install.

	<If	Module dir_module>
    DirectoryIndex index.php index.html
	</IfModule>

	<FilesMatch \.php$>
    SetHandler application/x-httpd-php
	</FilesMatch>
#
	sudo apachectl -k restart

If all went well, install other php versions.

	brew unlink php55
#
	brew reinstall php56 --with-apache
	brew reinstall php56-opcache 
	brew reinstall php56-apcu
	brew reinstall php56-mcrypt  
	brew reinstall php56-imagick
	brew reinstall php56-pdo-dblib
	brew reinstall php56-yaml
	brew reinstall php56-xdebug
	brew unlink php56
#
	brew reinstall php70 --with-apache
	brew reinstall php70-opcache 
	brew reinstall php70-apcu
	brew reinstall php70-mcrypt  
	brew reinstall php70-imagick
	brew reinstall php70-pdo-dblib
	brew reinstall php70-yaml
	brew reinstall php70-xdebug
	brew unlink php70
#
In my case php71 ended up needing special attention before re-install.

	brew uninstall --force php71
	brew cleanup --force -s php71
	brew prune
	sudo rm -fr  /usr/local/opt/php71
#
	brew reinstall php71 --with-apache
	brew reinstall php71-opcache 
	brew reinstall php71-apcu
	brew reinstall php71-mcrypt  
	brew reinstall php71-imagick
	brew reinstall php71-pdo-dblib
	brew reinstall php71-yaml
	brew reinstall php71-xdebug
	brew unlink php71

Back to PHP5

	brew link php56

###PHP ini paths

	/usr/local/etc/php/5.5/php.ini
	/usr/local/etc/php/5.6/php.ini
	/usr/local/etc/php/7.0/php.ini
	/usr/local/etc/php/7.1/php.ini

Modify the paths as follows:

	LoadModule php5_module    /usr/local/opt/php55/libexec/apache2/libphp5.so
	LoadModule php5_module    /usr/local/opt/php56/libexec/apache2/libphp5.so
	LoadModule php7_module    /usr/local/opt/php70/libexec/apache2/libphp7.so
	LoadModule php7_module    /usr/local/opt/php71/libexec/apache2/libphp7.so

We can only have one module processing PHP at a time, so for now, comment out all but the php56 entry:

	#LoadModule php5_module    /usr/local/opt/php55/libexec/apache2/libphp5.so
	LoadModule php5_module    /usr/local/opt/php56/libexec/apache2/libphp5.so
	#LoadModule php7_module    /usr/local/opt/php70/libexec/apache2/libphp7.so
	#LoadModule php7_module    /usr/local/opt/php71/libexec/apache2/libphp7.so

Verify php install one more time.

##Install php switcher.

	curl -L https://gist.github.com/w00fz/142b6b19750ea6979137b963df959d11/raw > /usr/local/bin/sphp
	chmod +x /usr/local/bin/sphp
Add LoadModule for `sphp` switcher to httpd.conf (final PHP change)

	LoadModule php5_module /usr/local/lib/libphp5.so
	#LoadModule php7_module /usr/local/lib/libphp7.so

It's IMPORTANT at this stage to fully stop your Apache sever, and start it again. Do not just restart it!

	sudo apachectl -k stop
	sudo apachectl start

I had some error messages regarding php*-intl, this fixed it. Had to be run for each version of php as do any other updates.

	sphp71
	brew reinstall --build-from-source php71-intl
	brew update
	brew upgrade
#
	sphp55
	brew reinstall --build-from-source php55-intl
	brew update
	brew upgrade
#
	sphp56
	brew reinstall --build-from-source php56-intl
	brew update
	brew upgrade
#
	sphp70
	brew reinstall --build-from-source php70-intl
	brew update
	brew upgrade

# Virtual hosts
Apache already comes preconfigured to support this behavior but it is not enabled. 

First you will need to uncomment the following lines in your /usr/local/etc/apache2/2.4/httpd.conf file:


	LoadModule vhost_alias_module libexec/mod_vhost_alias.so
	
and:

	Include /usr/local/etc/apache2/2.4/extra/httpd-vhosts.conf
and:

	<VirtualHost *:80>
    DocumentRoot "~/Sites/active-projects"
    ServerName localhost
	</VirtualHost>

	<VirtualHost *:80>
    DocumentRoot "~/Sites/active-projects/myproject.dev"
    ServerName myproject.dev
	</VirtualHost>

##Install DnsMasq

	brew reinstall dnsmasq
	echo 'address=/.dev/127.0.0.1' > /usr/local/etc/dnsmasq.conf
	sudo brew services start dnsmasq
#
	sudo mkdir -v /etc/resolver
	sudo bash -c 'echo "nameserver 127.0.0.1" > /etc/resolver/dev'

Test it!

	ping fakedomain.dev

## Generate Self-Signed SSL Cert

	openssl genrsa -des3 -passout pass:x -out local.dev.pass.key 2048
	openssl rsa -passin pass:x -in local.dev.pass.key -out local.dev.key

#
	rm local.dev.pass.key
#	
	openssl req -new -key local.dev.key -out local.dev.csr
	openssl x509 -req -days 365 -in local.dev.csr -signkey local.dev.key -out local.dev.crt

## Remove MySQL completely

1. Open the Terminal
2. Use `mysqldump` to backup your databases
3. Check for MySQL processes with: `ps -ax | grep mysql`
4. Stop and kill any MySQL processes
5. Analyze MySQL on HomeBrew: 
        
    ```
    brew remove mysql
    brew cleanup
    ```

6. Remove files: 

    ```
    sudo rm /usr/local/mysql
    sudo rm -rf /usr/local/var/mysql
    sudo rm -rf /usr/local/mysql*
    sudo rm ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist
    sudo rm -rf /Library/StartupItems/MySQLCOM
    sudo rm -rf /Library/PreferencePanes/My*
    ```

7. Unload previous MySQL Auto-Login: 
        
    ```
    launchctl unload -w ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist
    ```
        
8. Remove previous MySQL Configuration: 

    ```
    subl /etc/hostconfig` 
    # Remove the line MYSQLCOM=-YES-
    ```
        
9. Remove previous MySQL Preferences: 
    
    ```
    rm -rf ~/Library/PreferencePanes/My*
    sudo rm -rf /Library/Receipts/mysql*
    sudo rm -rf /Library/Receipts/MySQL*
    sudo rm -rf /private/var/db/receipts/*mysql*
    ```
    
10. Restart your computer just to ensure any MySQL processes are killed
11. Try to run mysql, **it shouldn't work**

##Install MySQL

	brew install mysql
	brew services start mysql
	
	
