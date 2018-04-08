# Drupal 8

[TOC]

## Generales

### Permisos en Drupal 8 a nivel de filesystem

#### Propiedad

```bash
chmod -R drupadmin:apache ./drup8dev
```

#### Permisos

```bash
[root@lvwdrup8devapp ~]# cd /opt/rh/httpd24/root/var/www/drup8dev
[root@lvwdrup8devapp drup8dev]# find -type f -exec chown drupadmin:apache {} \;
[root@lvwdrup8devapp drup8dev]# find -type d -exec chown drupadmin:apache {} \;
[root@lvwdrup8devapp drup8dev]# find . -type d -exec chmod 750 {} \;
[root@lvwdrup8devapp drup8dev]# find . -type f -exec chmod 640 {} \;
[root@lvwdrup8devapp drup8dev]# find ./sites -type d -exec chmod 770 {} \;
[root@lvwdrup8devapp drup8dev]# find ./sites -type f -exec chmod 660 {} \;
[root@lvwdrup8devapp drup8dev]# chown 444 sites/default/settings.php
```

### Configuración de Proxy para salida a internet

```bash
/**
 * External access proxy settings:
 *
 * If your site must access the Internet via a web proxy then you can enter the
 * proxy settings here. Set the full URL of the proxy, including the port, in
 * variables:
 * - $settings['http_client_config']['proxy']['http']: The proxy URL for HTTP
 *   requests.
 * - $settings['http_client_config']['proxy']['https']: The proxy URL for HTTPS
 *   requests.
 * You can pass in the user name and password for basic authentication in the
 * URLs in these settings.
 *
 * You can also define an array of host names that can be accessed directly,
 * bypassing the proxy, in $settings['http_client_config']['proxy']['no'].
 */
$settings['http_client_config']['proxy']['http'] = '172.26.67.48:3128';
$settings['http_client_config']['proxy']['https'] = '172.26.67.48:3128';
$settings['http_client_config']['proxy']['no'] = ['127.0.0.1', 'localhost','drupal8dev'];

```

### Traducciones

Falla la actualización de las traducciones. Falta el directorio translation dentro de sites/default/files.

```bash
cd sites/default/files/
mkdir translations
chown drupadmin:apache translations/
chmod -R 770 translations/
```

### Backing Up

```bash
[drupadmin@lvwdrup8devapp drup8dev]$ drush sql-dump  --result-file=/opt/backup/drup8dev-20180406.sql --gzip
 [success] Database dump saved to /opt/backup/drup8dev-20180406.sql.gz
 
 cd /opt/rh/httpd24/root/var/www/
 tar zvcf /opt/backup/drup8dev-8.5.0-20180406.tar.gz ./drup8dev
```

### Enlaces interesantes

1. https://www.lucius.digital/en/blog/login-without-password-most-secure-wait-what?utm_source=drupal-newsletter&utm_medium=email&utm_campaign=drupal-newsletter-20180322
2. https://dev.acquia.com/blog/decoupling-drupal-8-core-web-services-in-core-and-the-serialization-module/20/03/2018/19271?utm_source=drupal-newsletter&utm_medium=email&utm_campaign=drupal-newsletter-20180322
3. https://www.chapterthree.com/blog/access-control-modules-for-drupal-8?utm_source=drupal-newsletter&utm_medium=email&utm_campaign=drupal-newsletter-20180322
4. https://devcentral.f5.com/questions/load-balance-drupal-site 

## Composer con Drupal 8

### Instalar Drupal con Composer

```bash
cd /var/www
mkdir <ambiente>
composer create-project drupal/drupal <ambiente> 8.5.0
```

### Como instalar varios módulos con Composer

```bash
cd /var/www/<ambiente>
composer require drupal/admin_toolbar  drupal/ctools  drupal/devel drupal/page_manager drupal/panelizer drupal/panels drupal/pathauto drupal/token
```

### Upgrade de 8.5.0 a 8.5.1 via composer

```bash
# seteo el modo de mantenimiento
drush sset system.maintenance_mode 1

# update con composer
composer update drupal/core --with-dependencies
# actualizo la base de datos
drush updatedb
#reconstruyo el cache (drush cache-rebuild, drush cr)
drush cr

# seteo el modo de mantenimiento en 0
drush sset system.maintenance_mode 

```

#### Solución de problemas con el update

```
[drupadmin@lvwdrup8devapp drup8dev]$ composer update drupal/core --with-dependencies
Package "drupal/core" listed for update is not installed. Ignoring.
Loading composer repositories with package information
Updating dependencies (including require-dev)

  [Composer\Downloader\TransportException]
  The "http://packagist.org/p/symfony/config%24896fcd3d69dc85947d44b4ded2c2d68be5b13a45c691089743ed7a44bcbe3eb3.json" file could not be downloaded (HTTP/1.1 404 Not Found)

update [--prefer-source] [--prefer-dist] [--dry-run] [--dev] [--no-dev] [--lock] [--no-custom-installers] [--no-autoloader] [--no-scripts] [--no-progress] [--no-suggest] [--with-dependencies] [--with-all-dependencies] [-v|vv|vvv|--verbose] [-o|--optimize-autoloader] [-a|--classmap-authoritative] [--apcu-autoloader] [--ignore-platform-reqs] [--prefer-stable] [--prefer-lowest] [-i|--interactive] [--root-reqs] [--] [<packages>]...


```

Dato clave, **Package "drupal/core" listed for update is not installed. Ignoring.**

Entonces en https://drupal.stackexchange.com/questions/254226/core-updates-after-install-with-composer-create-project-drupal-drupal dan la clave, modificar el composer.json, eliminando la sección replace y pasadon drupal/core a la sección, require

```bash
[drupadmin@lvwdrup8devapp drup8dev]$ diff composer.json composer.json.8.5.0-orig
9d8
<         "drupal/core": "^8.5",
17a17,19
>     },
>     "replace": {
>         "drupal/core": "^8.5"
```

Luego de eso, 

```bash
[drupadmin@lvwdrup8devapp drup8dev]$ composer update drupal/core --with-dependencies
Loading composer repositories with package information
Updating dependencies (including require-dev)
Package operations: 1 install, 0 updates, 0 removals
  - Installing drupal/core (8.5.1): Loading from cache
> Drupal\Core\Composer\Composer::vendorTestCodeCleanup
Writing lock file
Generating autoload files
> Drupal\Core\Composer\Composer::preAutoloadDump
> Drupal\Core\Composer\Composer::ensureHtaccess
[drupadmin@lvwdrup8devapp drup8dev]$ drush updatedb
 [success] No database updates required.
[drupadmin@lvwdrup8devapp drup8dev]$ drush cr
 [success] Cache rebuild complete.
[drupadmin@lvwdrup8devapp drup8dev]$ drush status
 Drupal version   : 8.5.1
 Site URI         : default
 DB driver        : mysql
 DB hostname      : 172.26.161.95
 DB port          : 3306
 DB username      : drupaldev
 DB name          : drupal_dev
 Database         : Connected
 Drupal bootstrap : Successful
 Default theme    : bartik
 Admin theme      : seven
 PHP binary       : /opt/rh/rh-php71/root/usr/bin/php
 PHP config       : /etc/opt/rh/rh-php71/php.ini
 PHP OS           : Linux
 Drush script     : /home/drupadmin/.config/composer/vendor/bin/drush
 Drush version    : 9.0.0
 Drush temp       : /tmp
 Drush configs    : /home/drupadmin/.drush/drush.yml
                    /home/drupadmin/.config/composer/vendor/drush/drush/drush.yml
 Install profile  : standard
 Drupal root      : /opt/rh/httpd24/root/var/www/drup8dev
 Site path        : sites/default
 Files, Public    : sites/default/files
 Files, Temp      : /tmp

```

## Drush 9 con Drupal 8

### One time link para cambiar la contraseña

```
[drupadmin@lvwdrup8devapp drup8agesic]$ drush uli -l https://drup8agesic.corp.ute.com.uy
https://drup8agesic.corp.ute.com.uy/user/reset/1/1519217557/E1XwkM1LXCsW5aSyT3E_W-GeboeQ4UwXUVNlKtyqFe4/login

[drupadmin@lvwdrup8devapp drup8agesic]$ drush help uli
Display a one time login link for user ID 1, or another user.

Examples:
drush user:login                            Open default web browser and browse to homepage, logged in as uid=1.
drush user:login --name=ryan node/add/blog  Open default web browser (if configured or detected) for a one-time 
                                            login link for username ryan that redirects to node/add/blog.
drush user:login --browser=firefox --mail=drush@example.org  Open firefox web browser, and login as the user 
                                                             with the e-mail address drush@example.org.
Arguments:
  [path] Optional path to redirect to after logging in.

Options:
  --name[=NAME]                 A user name to log in as. If not provided, defaults to uid=1. [default: "1"]
  --browser[=BROWSER]           Optional value denotes which browser to use (defaults to operating system 
                                default). Use --no-browser to suppress opening a browser. [default:"true"]
  --redirect-port=REDIRECT-PORT A custom port for redirecting to (e.g., when running within a Vagrant 
                                environment)
  --no-browser                  Negate --browser option.
Aliases: uli, user-login
```

5up3r4Dm;n-t3St superadmin
5op4Pp1$$t3St



