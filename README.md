# ZRECore provides a Mobile and E-Commerce boiler-plate application, based on the Zend Framework.

This code is provided to you under the terms of the GNU General Public License V3 or higher. It can be found at http://www.gnu.org/copyleft/gpl.html

# Requirements
This installation is made to work on Unix/Linux operating systems. If you're really inclined to make this application work on a Windows machine, feel free to port it.

* Apache 2 or higher with the PHP module, Rewrite module, and the SSL module enabled. 
* SSL Certificate - You can create a "snake oil" SSL certificate for development purposes on your own machine, but you should purchase and install a reputable SSL certificate on a production machine.
* Zend Framework 1.10 or higher - See http://code.google.com/p/zend/ if you want to install it via PEAR or http://framework.zend.com if you plan to copy the Zend/ folder directly into the application library/ folder
* PHP 5.3 or higher with the Standard PHP Library (SPL, usually included by default)
* SQLight3 or higher (Use your package manager tool such as yast on CentOS/Red Hat or apt-get on Debian based servers. You will need `sqlite` and `php5-sqlite` or higher. Remember to restart Apache after installing. You can also download from http://www.sqlite.org/download.html if you don't use a package manager)

# Notes
I was inspired by some of the nicer features found in Wordpress, Joomla, and a few things I did right during the ZRECommerce project. 
I can be contacted via my blog at http://www.alexventure.com

# Backing up
Always back up. Just copy your ZRECore installation directory somewhere safe every so often (or commit to your own github fork). That way, if you ever need to undo a boo-boo, you'll be fine.
Your `data.sq3` file is especially important. Wherever you decide to keep it, don't forget to back it up too. (Keep reading, you'll see what that's all about further down.)

# Installation
Downloads can be found at https://github.com/aalbino/ZRECore/downloads
Additionally, you can fork me off of http://github.com/aalbino/ZRECore/ and then check out your own fork onto your local machine using git (read below).

## File Setup
Extract the contents of the ZRECore download archive into /var/www/ZRECore/ (or whatever directory you plan to host these files from)
For example, let's say you downloaded aalbino-ZRECore-a6a4749.tar.gz and extracted the archive into /home/User/Downloads/aalbino-ZRECore-a6a4749/

If you are going to run this installation under /var/www/www.mywebsite.com/ you could do the following:

cd /home/User/Downloads/aalbino-ZRECore-a6a4749/
mv * /var/www/www.mywebsite.com/

For those of you who prefer the bleeding edge of technology, you can just check out a copy from github.com directly into /var/www/www.mywebsite.com/

## Database
The .sq3 database file location can be specified in [your installation directory]/application/configs/application.ini under the '; Database' comment

You can use the default sqlite3 database file in [your installation directory]/data/sqlite/data.sq3 ...Feel free to use the sqlite3 client to open the .sq3 file and then use the `.read` command to (re)create whatever table you want. Each file is individually wrapped in its own .sql file for your convenience.
Then, move or copy data.sq3 into some directory outside your installation and update application.ini accordingly.

*NOTE: You are responsible for setting permissions on the copied/moved data.sq3 file in order to keep it secure, yet accessible by this application.*

## Virtual Host

You will need to create two files in /etc/apache2/sites-available ...one for the HTTP version of your installation, and one for the HTTPS version.

Here is a sample (HTTP) Apache2 virtual host configuration file named www.example.com (/etc/apache2/sites-available/www.example.com):

    <VirtualHost *:80>
        ServerName www.example.com
        DocumentRoot /var/www/www.example.com/public
     
        SetEnv APPLICATION_ENV "development"
     
        <Directory /var/www/www.example.com/public>
            DirectoryIndex index.php
            AllowOverride All
            Order allow,deny
            Allow from all
        </Directory>
    </VirtualHost>

Here is a sample (HTTPS) Apache2 virtual host configuration file named www.example.com-ssl (/etc/apache2/sites-available/www.example.com-ssl):

    <VirtualHost *:443>
    	ServerName www.example.com
    	DocumentRoot /var/www/www.example.com/public
    
    	SetEnv APPLICATION_ENV "development"
    
    	<Directory /var/www/www.example.com/public>
    		DirectoryIndex index.php
    		AllowOverride All
    		Order allow,deny
    		Allow from all
    	</Directory>
    
    	SSLEngine on
    
    	SSLCertificateFile    /etc/ssl/certs/ssl-cert-snakeoil.pem
    	SSLCertificateKeyFile /etc/ssl/private/ssl-cert-snakeoil.key
    
    	<FilesMatch "\.(cgi|shtml|phtml|php)$">
    		SSLOptions +StdEnvVars
    	</FilesMatch>
    	<Directory /usr/lib/cgi-bin>
    		SSLOptions +StdEnvVars
    	</Directory>
    	BrowserMatch "MSIE [2-6]" \
    	nokeepalive ssl-unclean-shutdown \
    	downgrade-1.0 force-response-1.0
    	BrowserMatch "MSIE [17-9]" ssl-unclean-shutdown
    </VirtualHost>

*Remember to modify these host files to reflect the correct server name, document root, and SSL certificate file locations accordingly.*

If you are installing on your own local development machine, you may need to add the following entry into your /etc/hosts file as well (under the last entry for 127.0.0.1):
`127.0.0.1	www.example.com`
That way, when you brows to www.example.com, you will actually request your local machine's virtual host "www.example.com"

Finally, type `a2ensite www.example.com` and hit enter, then `a2ensite www.example.com-ssl` and hit enter, then type `sudo service apache2 reload` to reload the Apache2 server configuration files.
This enables the HTTP and HTTPS versions of your virtual hosts, and reloads the Apache 2 server.

If you did everything correctly, you should be able to open up your browser and go to www.example.com (or whatever fake URL you've been using in its place up to this point) and see a working installation.

# Samples

## Observer/Subject environment
To enable/disable an observer plug-in, simple add or comment out the corresponding file path entry in `/application/configs/observers.ini`
If you would like to learn what this section is all about, read http://softcoded.com/pdfs/subject_observer.pdf
The Observer Design Pattern used here is actually based off the PHP  SplSubject/SplObserver classes (interfaces) http://php.net/manual/en/class.splobserver.php

*Sample observer* - Hooks into our Observer/Subject environment and echos "Hello World!" when the "Auth" subject's "authenticate()" method is called. This sample is an introduction to creating your own custom observer classes.
* See: /custom/Auth.HelloWorld.php

*Sample shopping cart for service providers* - Overrides the shopping cart system behavior (via the Observer/Subject environment) to provide a "Service" oriented shopping cart.


### Available Observer Subjects
Available observer subjects are in associative array format and can be retrieved anywhere within the application as follows:

    $observerSubjects = Zend_Registry::get('OBSERVER_SUBJECTS');
    $theSubject = $observerSubjects['NameOfObserverSubject'];

Simply replace the key 'NameOfObserverSubject' with one of the following:

* `Auth` - Notifies attached observers when the following methods are called:
    * `setIsAuthenticate(bool $bool)` - Sets the authenticated flag to TRUE or FALSE.

* `Cart` - Notifies attached observers when the following methods are called:
    * `renderForm(string $formKey, Zend_View $view)` - Renders the requested form view helper.
    * `redirectToForm(string $currentFormKey, string $newFormKey, array $data )` - Notifies the observers that the controller logic requests a redirect to a new form, using the supplied data.
    * `redirectToURL(string $currentFormKey, string $returnFormKey, array $data, string $method = 'POST')` - Notifies the observers that the controller logic requests a redirect to another URL, using the supplied data, and using the method requested. The method specified can be GET or POST (POST is the default).
    * `redirectFromURL(string $returnFormKey, array $data)` - Notifies the observers that we have received a response from a previously supplied URL, and gives the observer a copy of the returned data.
    * `addToCart(array $item_data)` - 
    * `onSubmitForm(string $formKey, array $input)` - Notifies observers that the specified form has been submitted with the provided input.