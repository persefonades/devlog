title:: wamp
banner:: ../assets/wamp_1703787568056_0.webp
icon:: 🪟
tags:: wamp, mysql, apache, php

- # Custom WAMP Setup in Windows
- ## PHP installation
	- Download `php-8.1.*-Win32-vs16-x64.zip` file from [https://windows.php.net/download#php-8.1](https://windows.php.net/download#php-8.1)
	- Unzip and rename the folder to `php81` and place it in `C:\Apps`
	- OR
	- Set `ChocolateyToolsLocation` to `C:\Apps`
	- Install php using choco
	  ```powershell
	  choco install php --version=8.1.22 --package-parameters='"/ThreadSafe"'
	  ```
	- Copy and rename `C:\Apps\php81\php.ini-production` into `C:\Apps\php81\php.ini`
	- Set extension directory in `C:\Apps\php81\php.ini`
	  ```ini
	  extension_dir="ext"
	  ```
	- Enable the extensions by commenting following lines
	  ```ini
	  extension=curl
	  extension=ftp
	  extension=fileinfo
	  extension=gd
	  extension=gettext
	  extension=mbstring
	  extension=exif      ; Must be after mbstring as it depends on it
	  extension=mysqli
	  extension=openssl
	  extension=pdo_mysql
	  extension=pdo_sqlite
	  extension=soap
	  extension=xsl
	  ```
	- ### Adding Xdebug for code coverage
		- Download xdebug file from [https://xdebug.org/files/php_xdebug-3.3.0alpha2-8.1-vs16-x86_64.dll](https://xdebug.org/files/php_xdebug-3.3.0alpha2-8.1-vs16-x86_64.dll)
		- Move it to `C:/Apps/php81/ext` and rename it to `php_xdebug.dll`
		- Update `C:\Apps\php81\php.ini` and add the line:
		  ```ini
		  zend_extension=xdebug
		  ```
- ## Apache server installation
	- Download `httpd-2.4.*-win64-VS17.zip` from [https://www.apachelounge.com/download/](https://www.apachelounge.com/download/).
	- Unzip the above zip file and move `Apache24` to `C:\Apps\Apache24`.
	- Open the terminal in administrative mode and run the below commands
	  ```powershell
	  cd C:\Apps\Apache24\bin\
	  httpd -k install
	  ```
	- httpd server will be installed and `Apache2.4` should be available in Services.
	- ### Configuring in apache server
		- Prepare a `httpd-php.conf` file with following content in `C:\Apps\Apache24\conf\extra`
		  ```apacheconf
		  #
		  # PHP-Module setup
		  #
		  
		  LoadFile "C:/Apps/php81/php8ts.dll"
		  LoadFile "C:/Apps/php81/libpq.dll"
		  LoadFile "C:/Apps/php81/libsqlite3.dll"
		  LoadModule php_module "C:/Apps/php81/php8apache2_4.dll"
		  
		  <FilesMatch "\.php$">
		      SetHandler application/x-httpd-php
		  </FilesMatch>
		  <FilesMatch "\.phps$">
		      SetHandler application/x-httpd-php-source
		  </FilesMatch>
		  
		  <IfModule php_module>
		      PHPINIDir "C:/Apps/php8"
		  </IfModule>
		  
		  <IfModule mime_module>
		      AddType text/html .php .phps
		  </IfModule>
		  
		  ScriptAlias /php-cgi/ "C:/Apps/php8/"
		  <Directory "C:/Apps/php8">
		      AllowOverride None
		      Options None
		      Require all denied
		      <Files "php-cgi.exe">
		            Require all granted
		      </Files>
		  </Directory>
		  ```
		- Include the above file in `C:\Apps\Apache24\conf\httpd.conf`.
		  ```apacheconf
		  Include conf/extra/httpd-php.conf
		  ```
- ## MySQL installation
	- Download .msi file from [https://dev.mysql.com/downloads/installer/](https://dev.mysql.com/downloads/installer/) and install
	- Set the `root` password
- ## phpMyAdmin installation
	- Download phpmyadmin zip from [https://files.phpmyadmin.net/phpMyAdmin/5.2.1/phpMyAdmin-5.2.1-all-languages.zip](https://files.phpmyadmin.net/phpMyAdmin/5.2.1/phpMyAdmin-5.2.1-all-languages.zip)
	- Unzip and copy the `phpmyadmin` folder into `C:/Apps/Apache24/htdocs`
	- Create a phpmyadmin user `pma` in mysql
	- Edit config in `C:\Apache24\htdocs\phpmyadmin\config.inc.php` (Copy from `config.sample.inc.php`)
	  ```php
	  /* Authentication type and info */
	  $cfg['Servers'][$i]['auth_type'] = 'config';
	  $cfg['Servers'][$i]['user'] = 'root';
	  $cfg['Servers'][$i]['password'] = '<mysql_password>';
	  $cfg['Servers'][$i]['extension'] = 'mysqli';
	  $cfg['Servers'][$i]['AllowNoPassword'] = false;
	  $cfg['Lang'] = 'en';
	  
	  /* User for advanced features */
	  $cfg['Servers'][$i]['controluser'] = 'pma';
	  $cfg['Servers'][$i]['controlpass'] = '<control_password>';
	  ```
	- ### Configuring Virtual Hosts and Symbolic Links
		- Create a symbolic directory from project directory
		  ```powershell
		  mklink /D C:\Apache24\htdocs\diybaazar C:\Users\purch\Documents\Projects\Github\DIY-Baazar\diybaazar-main
		  mklink /D C:\Apache24\htdocs\diybaazar-admin C:\Users\purch\Documents\Projects\Github\DIY-Baazar\diybaazar-admin
		  mklink /D C:\Apache24\htdocs\diybdocs C:\Users\purch\Documents\Projects\Github\DIY-Baazar\diybaazar-docs
		  ```
		- Add virtual host config in `httpd-vhosts.conf`
		  ```apacheconf
		  <VirtualHost *:80>
		      ServerAdmin webmaster@localhost
		  </VirtualHost>
		  
		  <VirtualHost *:80>
		      ServerAdmin webmaster@diybaazar.xyz
		      DocumentRoot "${SRVROOT}/htdocs/diybaazar"
		      ServerName diybaazar.xyz
		      ErrorLog "logs/diybaazar.xyz-error.log"
		      CustomLog "logs/diybaazar.xyz-access.log" common
		  </VirtualHost>
		  ```
	- ### Hosting Flask Service
		- Installing mod-wsgi
		  ```powershell
		   pip install mod-wsgi
		  ```
		- Run the following the command and copy the output
		  ```powershell
		   mod_wsgi-express module-config
		  ```
		- Output will look something like this:
		  ```apacheconf
		  LoadFile "C:/Python311/python311.dll"
		  LoadModule wsgi_module "C:/Python311/Lib/site-packages/mod_wsgi/server/mod_wsgi.cp311-win_amd64.pyd"
		  WSGIPythonHome "C:/Python311"
		  ```
		- Paste it in `httpd.conf`.
		- Add the following config in `httpd-vhosts.conf`.
		  ```apacheconf
		  <VirtualHost *:80>
		      DocumentRoot "${SRVROOT}/htdocs/diybdocs"
		      ServerAdmin webmaster@diybaazar.xyz
		      ServerName docs.diybaazar.xyz
		  
		      WSGIScriptAlias / "${SRVROOT}/htdocs/diybdocs/app.wsgi"
		      <Directory "${SRVROOT}/htdocs/diybdocs">
		          WSGIApplicationGroup %{GLOBAL}
		          WSGIScriptReloading On
		          allow from all
		          #Options None
		          AllowOverride All
		          Require all granted
		      </Directory>
		  
		      ErrorLog "logs/docs.diybaazar.xyz-error.log"
		      CustomLog "logs/docs.diybaazar.xyz-access.log" common
		  </VirtualHost>
		  ```