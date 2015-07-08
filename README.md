# localhost-config
####Instructions for setting up a localhost developer environment.
Includes creating Virtual Hosts, securing phpMyAdmin and something useful you can do in your Wordpress `wp-config.php` file, once you've set it all up.

## Table of Contents

* [Wordpress](#wordpress-bonus)
* [MAMP APPENDIX](#mamp-appendix)

## Tools

- WAMP - http://www.wampserver.com/en/
- MAMP - https://www.mamp.info/en/

## Files

I like to have all my local config files quickly available, so I create a new project in Sublime Text, open all the files I need, and save the sublime text project files to a folder like `localhost-config`, somewhere in my `Developer` folder.

Files you will need to edit:

**Windows**

- WAMP - `C:\wamp\bin\apache\apache2.4.9\conf\httpd.conf`
- hosts - `C:\Windows\System32\Drivers\etc\hosts`
- phpMyAdmin - `C:\wamp\apps\phpmyadmin4.1.14\config.inc.php`

**OS X**

- MAMP - `/Applications/MAMP/conf/apache/httpd.conf`
- hosts - `/private/etc/hosts`
- phpMyAdmin - `/Applications/MAMP/bin/phpMyAdmin/config.inc.php`

I also create my own VHOSTS config file. You can call this whatever you want and place it wherever you want - we will point to it in Apache's default `httpd.conf` file.

This means that re-installing WAMP or MAMP, won't result in the loss of all your VHOST configs, you'll only have to point to your custom file again.

I call mine `local-development.conf` and place it in a folder called `localhost-config` in my `Developer` directory.

## httpd.conf

- WAMP - `C:\wamp\bin\apache\apache2.4.9\conf\httpd.conf`
- MAMP - `/Applications/MAMP/conf/apache/httpd.conf`

Add this line to the bottom of your default `httpd.conf` file

```PHP
Include D:/your_path/local-development.conf
```

## local-development.conf

First, we're going to set (override) some options that are already present in Apache's default `httpd.conf` file.
The default `httpd.conf` file contains many comments explaining what each setting does. I encourage you to look through the `httpd.conf` file itself if you're still learning.

By default, your localhost folder is located here:

- Windows - `C:\wamp\www`
- Mac - `/Applications/MAMP/htdocs`

The following changes the default location that files will be served from to a location of your choice and opens up the permissions for that location.

**NB:** The line for `Require local` (or `Require all granted`) is only required for Apache versions >= 2.4. If you include it in lower versions it will result in **500 Internal Server Error**.
```ApacheConf
# First we open up the permissions
<Directory "D:/path_to/your_default_localhost_folder/">
    Options Indexes FollowSymLinks
    AllowOverride All
    Require local # Remove for Apache versions < 2.4
</Directory>

# Then we set the host name
NameVirtualHost *
<VirtualHost *:80>
DocumentRoot "D:/path_to/your_default_localhost_folder/"
ServerName localhost
</VirtualHost>
```

Simply type `localhost` into your browser and this is where it will take you.

You can still type `localhost/phpmyadmin` to get to your local phpMyAdmin interface.

Next, we'll add some virtual hosts. We'll add a vhost located within the folder used above, so no need to repeat the permissions part.

For each virtual host you want, add the following:
```ApacheConf
<VirtualHost *:80>
DocumentRoot "D:/path_to/your_default_localhost_folder/and_vhost_folder/"
ServerName yourdomain1.dev
</VirtualHost>
```

If the location is elsewhere, just include the permissions as well, like this:
```ApacheConf
<VirtualHost *:80>
DocumentRoot "C:/another_folder/"
ServerName anothersite.dev
<Directory "C:/another_folder/">
    Options Indexes FollowSymLinks
    AllowOverride All
    Require local
</Directory>
</VirtualHost>
```

**Don't forget to add the `ServerName` for each vhost to your systems hosts file!**

...and restart Apache each time you make any changes, if it is already running.

## hosts

- Windows - `C:\Windows\System32\Drivers\etc\hosts`
- Mac - `/private/etc/hosts`

Add one of each of these to the bottom of your `hosts` file, for each virtual host you need

```
127.0.0.1       yourdomain1.dev
```

## phpMyAdmin

Even in a local environment, it's still good practice to secure phpMyAdmin with at least a password.

There are a couple of things to do to make this work for you in a good way.

First, set a password for the `root` user through the phpMyAdmin interface, usually accessed by visiting http://localhost/phpmyadmin in your browser.

- Remove any 'anonymous' users.

Open the phpMyAdmin configuration file:

- Windows - `C:\wamp\apps\phpmyadmin4.1.14\config.inc.php`
- Mac - `/Applications/MAMP/bin/phpMyAdmin/config.inc.php`

Find the line with:
```PHP
$cfg['blowfish_secret'] = '';
```
and enter a random string.

Find
```PHP
$cfg['Servers'][$i]['auth_type'] = '';
```
and set to `cookie`.

You could also add or set
```PHP
$cfg['Servers'][$i]['AllowNoPassword'] = false;
```


You will now be asked to enter your username and password when you access phpMyAdmin.

After a period of inactivity, you will be asked for your username and password again. I find the default timeout to be too short, and if you are developing locally you will probably find this annoying. To increase the time, look for the following line:

```PHP
$cfg['Servers'][$i]['LoginCookieValidity'] = 1440;
```

The number `1440` is the number of seconds before you will be requested for your password again. Change it to something longer. e.g.

```PHP
$cfg['Servers'][$i]['LoginCookieValidity'] = 3600 * 3;
```

## Wordpress Bonus

This doesn't really belong here but I'll add it as a bonus anyway.

If you have the above VHOSTS configuration setup for your local Wordpress development, you can take advantage of a nice trick for easily switching databases, depending on environment.

In your `wp_config.php` file, comment out the `DB_NAME`, `DB_USER` and `DB_PASSWORD` lines in the standard database settings section
```PHP
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', '');

/** MySQL database username */
define('DB_USER', '');

/** MySQL database password */
define('DB_PASSWORD', '');

/** MySQL hostname */
define('DB_HOST', 'localhost');

/** Database Charset to use in creating database tables. */
define('DB_CHARSET', 'utf8');

/** The Database Collate type. Don't change this if in doubt. */
define('DB_COLLATE', '');
```
and then immediately below, put the following
```PHP
switch($_SERVER['SERVER_NAME'])
{
    case 'domain1.dev':
        define('DB_NAME', '');
        define('DB_USER', '');
        define('DB_PASSWORD', '');
    break;

    default:
        define('DB_NAME', '');
        define('DB_USER', '');
        define('DB_PASSWORD', '');
    break;
}
```

The `domain1.dev` case should contain the database settings for your localhost setup

The `default` case should be your live production servers settings.

This simple example just reduces a little bit of admin when pushing a Wordpress installation from a local environemt to the live server.

Other settings can be included in these switch statements too. For example, you could include
```PHP
define('WP_DEBUG', true);
```
in your localhost section.

...or set your home and site url's dynamically, reducing database queries:

```PHP
    /** Set the home and site urls dynamically */
    define('WP_HOME', 'http://'.$_SERVER['HTTP_HOST'].'');
    define('WP_SITEURL', 'http://'.$_SERVER['HTTP_HOST'].'');
```

## MAMP APPENDIX

MAMP can be a little tedious sometimes, so here are a few things that should make your experience with MAMP a little easier.

1. **MAMP prompts for your password on every startup and shutdown**

    This occurs when you're using a port lower than 1024. If you don't want to have to append the port everytime you view one of your local sites, e.g. `localhost:8888`, then you can set MAMP to use the default port `80`. Unfortunately, this causes OS X to prompt for your password to allow it. There is no good (secure) workaround for this that I've discovered.

2. **You are prompted to allow mysql network access everytime it starts**

    Despite being set as an allowed application in the firewall, I was still prompted for this every single time.
    The answer is to create a self signed certificate and apply it to mysql.

    See this StackExchange Question http://apple.stackexchange.com/questions/3271/how-to-get-rid-of-firewall-accept-incoming-connections-dialog
    
    Just in case it ever vanishes:
    
  ```
  1) Create your own code signing cert:

  In Keychain Access, Keychain Access > Certificate Assistant > Create a certificate. This launches the Certificate Assistant:

  Name: Enter some arbitrary string here that you can remember. Avoid spaces otherwise you'll need to escape the cert's name when using codesign from the command line.

  Identity type: Self Signed Root

  Certificate Type: Code Signing

  Check the box "Let me override defaults", this is quite important

  Serial number: 1 (OK as long as the cert name/serial no. combination is unique)

  Validity Period: 3650 (gives you 10 years)

  Email, Name, etc. fill out as you wish.

  Key pair info: set to RSA, 2048 bits. Does not really matter IMHO.

  From "Key usage extension" up to "Subject Alternate Name Extension": accept the defaults.

  Location: login keychain.

  Once it is created, set to "Always trust" in the Login keychain.

  2) Re-signing an app: codesign -f -s <certname> /path/to/app --deep

  3) Verify that it worked: codesign -dvvvv /path/to/app
  ```

3. **Use MAMP's MySQL from the Terminal**

  Add the following to your `.bash_profile`:

  ```
  alias mysql='/Applications/MAMP/Library/bin/mysql -uroot -p'
  ```

  Now, just typing `mysql` in the Terminal will attempt to launch mysql as `root` (MySQL `root`, not your OS X `root`) and   will prompt you for the `root` password.
