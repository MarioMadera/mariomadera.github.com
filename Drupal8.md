# Drupal 8

[TOC]

## Generales

## Estructura de Archivos de Drupal Core

- **/core** - All files provided by core, that doesn't have an explicit reason to be in the / directory. More details futher down.

- **/libraries** - 3rd party libraries, eg. a wysiwyg editor. Not included by core, but common enough to warrant inclusion here.

- /modules

   \- The directory into which all custom and contrib modules go.

  - Splitting this up into the sub-directories **contrib** and **custom** can make it easier to keep track of the modules.enough to warrant mention here.

- **/profile** - contributed and custom profiles.

- **/themes** - contributed and custom (sub)themes

- **sites/[domain OR default]/{modules,themes}** - Site specific modules and themes can be moved into these directories to avoid them showing up on every site. 

- **sites/[domain OR default]/files** - Site specific files tend to go here. This could be files uploaded by users, such as images, but also includes the configuration, **active** as well as **staged** config. The configuration is read and written by Drupal, and should have the minimal amount of privileges required for the webserver, and the only the webserver, to read and modify them. 

- **/vendor** - Backend libraries that Drupal Core depends on. (Symfony, Twig, etc)

Details on the /core directory, primarily useful to know for new core hackers:

- **/core/assets** - Various external libraries used by Core. jQuery, underscore, modernizer etc.
- **/core/misc** - Frontend code that Drupal Core depends on.
- **/core/includes** - Functionality that is to low level to be modular. Such as the module system itself.
- **/core/lib** - Drupal Core classes.
- **/core/modules** - Drupal Core modules.
- **/core/profiles** - Drupal Core installation profiles. Minimal, Standard, Testing and Testing multilingual installation profiles by default. 
- **/core/scripts** - Various CLI scripts, mostly used by developers.
- **/core/tests** - Drupal Core tests.
- **/core/themes** - Drupal Core themes.

### Permisos en Drupal 8 a nivel de filesystem

#### Propiedad

```bash
chmod -R drupadmin:apache ./drup8dev
```

#### Permisos

```bash
cd /opt/rh/httpd24/root/var/www/drup8dev
find -type f -exec chown drupadmin:apache {} \;
find -type d -exec chown drupadmin:apache {} \;
find . -type d -exec chmod 750 {} \;
find . -type f -exec chmod 640 {} \;
find ./sites/default/files -type d -exec chmod 770 {} \;
find ./sites/default/files -type f -exec chmod 660 {} \;
find ./tmp -type d -exec chmod 770 {} \;
find ./tmp -type f -exec chmod 660 {} \;


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
$settings['http_client_config']['proxy']['http'] = 'X.Y.67.48:3128';
$settings['http_client_config']['proxy']['https'] = 'X.Y.67.48:3128';
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
[drupadmin@dev-drupalapp drup8dev]$ drush sql-dump --result-file=/opt/backup/drup8dev-$(date +%Y%m%d).sql --gzip
 [success] Database dump saved to /opt/backup/drup8dev-20180406.sql.gz
 
 cd /opt/rh/httpd24/root/var/www/
 tar zvcf /opt/backup/drup8dev-8.5.1--$(date +%Y%m%d).tar.gz ./drup8dev
```

### Restore

```mysql
mysql --user=<usuario> --password=<secret> --database=<base de datos> --host=<db_host> --port=<db_port> < /opt/backup/<archivo_dump>.sql
```

**nota**: ver el uso de --exclude

```tar zcvf drup7dev-7.58.tgz --exclude "drupal-dev/sites/default/files" ./drupal-dev/``` 



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
composer require drupal/admin_toolbar  drupal/ctools  drupal/devel drupal/page_manager drupal/panelizer drupal/panels drupal/pathauto drupal/token 'drupal/video_embed_field:^2.0' 'drupal/rebuild_cache_access:^1.3' 
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
drush sset system.maintenance_mode 0

```

#### Solución de problemas con el update

```
[drupadmin@dev-drupalapp drup8dev]$ composer update drupal/core --with-dependencies
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
[drupadmin@dev-drupalapp drup8dev]$ diff composer.json composer.json.8.5.0-orig
9d8
<         "drupal/core": "^8.5",
17a17,19
>     },
>     "replace": {
>         "drupal/core": "^8.5"
```

Luego de eso, 

```bash
[drupadmin@dev-drupalapp drup8dev]$ composer update drupal/core --with-dependencies
Loading composer repositories with package information
Updating dependencies (including require-dev)
Package operations: 1 install, 0 updates, 0 removals
  - Installing drupal/core (8.5.1): Loading from cache
> Drupal\Core\Composer\Composer::vendorTestCodeCleanup
Writing lock file
Generating autoload files
> Drupal\Core\Composer\Composer::preAutoloadDump
> Drupal\Core\Composer\Composer::ensureHtaccess
[drupadmin@dev-drupalapp drup8dev]$ drush updatedb
 [success] No database updates required.
[drupadmin@dev-drupalapp drup8dev]$ drush cr
 [success] Cache rebuild complete.
[drupadmin@dev-drupalapp drup8dev]$ drush status
 Drupal version   : 8.5.1
 Site URI         : default
 DB driver        : mysql
 DB hostname      : X.Y.161.95
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

### Claves publica/privada en drupadmin

```
[drupadmin@dev-drupalapp ~]$ ssh-keygen -t rsa -b 4096
Generating public/private rsa key pair.
Enter file in which to save the key (/home/drupadmin/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/drupadmin/.ssh/id_rsa.
Your public key has been saved in /home/drupadmin/.ssh/id_rsa.pub.
The key fingerprint is:
43:07:ca:42:12:9c:1c:8e:77:9b:00:d3:e0:11:9d:c7 drupadmin@dev-drupalapp.localdomain
The key's randomart image is:
+--[ RSA 4096]----+
|+B*++   .        |
|.=*= E . .       |
|..+ + o . .      |
| . o + . .       |
|    o   S        |
|         .       |
|                 |
|                 |
|                 |
+-----------------+

[drupadmin@dev-drupalapp ~]$ ssh-copy-id drupadmin@drupaltst
The authenticity of host 'drupaltst (X.Y.161.148)' can't be established.
ECDSA key fingerprint is e8:98:22:f3:c9:cf:78:7e:78:ed:71:f2:95:4d:28:7b.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
drupadmin@drupaltst's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'drupadmin@drupaltst'"
and check to make sure that only the key(s) you wanted were added.

[drupadmin@dev-drupalapp ~]$ ssh drupadmin@drupaltst
Last login: Tue Apr 10 11:33:50 2018 from pc147048.localdomain

```



### Instalacción

```
composer global require drush/drush
```

Editar el archivo ```~/.bashrc``` y agregar la siquiente línea

```
export PATH="$HOME/.config/composer/vendor/bin:$PATH"
```

Actualización

```
composer global update
```

### Configuración de aliases

Se requiere crear un archivo llamado ```drupla8.site.yml``` en el directorio ```~/.drush``` del usuario drupadmin. El contenido es como sigue

```
dev:
   root: /opt/rh/httpd24/root/var/www/drup8dev
   uri: https://drup8dev.localdomain

agesic:
   root: /opt/rh/httpd24/root/var/www/drup8agesic
   uri: http://drup8agesic.localdomain

test:
   host: drupaltst
   user: drupadmin
   root: /opt/rh/httpd24/root/var/www/drup8test
   uri: https://drup8test.localdomain
```

### One time link para cambiar la contraseña

```
[drupadmin@dev-drupalapp drup8agesic]$ drush uli -l https://drup8agesic.localdomain
https://drup8agesic.localdomain/user/reset/1/1519217557/E1XwkM1LXCsW5aSyT3E_W-GeboeQ4UwXUVNlKtyqFe4/login

[drupadmin@dev-drupalapp drup8agesic]$ drush help uli
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

### Creación de usuarios

```
drush user:create newuser --mail="person@example.com" --password="letmein"
```

### Agregar un usuario (o una lista a un rol

```
drush urol <rol>  <user1>[,user2,..,usern]
```

### Update de cambios en esquemas de entidades 

```
[drupadmin@dev-drupalapp ~]$ drush @drupal8.dev entup
The following updates are pending:

page entity type :
El tipo de entidad Page necesita ser instalado.
page_variant entity type :
El tipo de entidad Page Variant necesita ser instalado.
pathauto_pattern entity type :
El tipo de entidad Pathauto pattern necesita ser instalado.

 Do you wish to run all pending updates? (yes/no) [yes]:
 > yes


 [success] Cache rebuild complete.
 [success] Finished performing updates.
[drupadmin@dev-drupalapp ~]$ drush help entup
Apply pending entity schema updates.

Options:
  --cache-clear[=CACHE-CLEAR] Set to 0 to suppress normal cache clearing; the caller should then clear if needed. [default: "true"]
  --no-cache-clear            Negate --cache-clear option.

Aliases: entup, entity-updates

```

### Sincronizar Ambientes

#### File System

```
drush rsync @alias.origen @alias.destino
drush rsync @alias.origen @alias.destino --exclude-path=files
```

Base de Datos

```
drush sql:sync @alias.origen @alias.destino
```

### Configuraciones

#### Ver el uuid del sitio

```
 drush @drupal8.dev cget system.site | grep -i UUID
```



```

```

5up3r4Dm;n-t3St superadmin
5op4Pp1$$t3St





[./Solr.md]: 