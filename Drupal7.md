# Drupal 7

## Problemas de Seguridad

### robots.txt

Hay que copiar secciones enteras al final del documento y quitar la / al final. Se muestran las secciones a continuación..

```
## Corregido,  se detecto la ejecucion de un crawler
## https://api.drupal.org/api/drupal/robots.txt/7.x
# Directories - corregido
Disallow: /includes
Disallow: /misc
Disallow: /modules
Disallow: /profiles
Disallow: /scripts
Disallow: /themes
# Paths (clean URLs) - corregido
Disallow: /admin
Disallow: /comment/reply
Disallow: /filter/tips
Disallow: /node/add
Disallow: /search
Disallow: /user/register
Disallow: /user/password
Disallow: /user/login
Disallow: /user/logout
# Paths (no clean URLs) - corregido
Disallow: /?q=admin
Disallow: /?q=comment/reply
Disallow: /?q=filter/tips
Disallow: /?q=node/add
Disallow: /?q=search
Disallow: /?q=user/password
Disallow: /?q=user/register
Disallow: /?q=user/login
Disallow: /?q=user/logout  
```

## Clickjacking

Agregar 

```
Header always append X-Frame-Options SAMEORIGIN  
```

en el .htaccess de drupal.

##Configuración de Proxy para actualizar.

```
/**
 * External access proxy settings:
 *
 * If your site must access the Internet via a web proxy then you can enter
 * the proxy settings here. Currently only basic authentication is supported
 * by using the username and password variables. The proxy_user_agent variable
 * can be set to NULL for proxies that require no User-Agent header or to a
 * non-empty string for proxies that limit requests to a specific agent. The
 * proxy_exceptions variable is an array of host names to be accessed directly,
 * not via proxy.
 */
$conf['proxy_server'] = '172.26.67.48';
$conf['proxy_port'] = 3128;
# $conf['proxy_username'] = '';
# $conf['proxy_password'] = '';
# $conf['proxy_user_agent'] = '';
$conf['proxy_exceptions'] = array('127.0.0.1', 'localhost');

```

## Bajar la cantidad de inserts a watchdog (https://www.drupal.org/docs/8/core/modules/syslog/overview)

http://developed.be/2013/02/26/write-drupal-logs-to-rsyslog-instead-of-to-dblog/ extra info

Watchdog es donde graba drupal cuando se usa el modulo dblog.

* habilitar el modulo y configuración 
  * ya estaba habilitado
  * Configuración
    * mostrar todos los mensajes
    * identidad de syslog = drupal_preprod
    * Utilidad de syslog = log_local0 (local0 para rsyslog)
    * formato de la linea de log = !base_url|!timestamp|!type|!ip|!request_uri|!referer|!uid|!link|!message
* crear el archivo /etc/rsyslog.d/drupal.conf

```
local0.* /var/log/drupal/preprod.log
& ~
```

* reiniciar el servicio 

```
service rsyslog restart
```

* php.ini needs to explicitly state that error logs are written to syslog  (ie. error_log = syslog), otherwise the syslog module will not work.Asi que editamos el php.ini y reiniciamos el servico httpd

* deshabilitamos el módulo dblog

* borramos el contenido de watchdog

  ```sql
  mysql> show table wathcdog;
  ERROR 2013 (HY000): Lost connection to MySQL server during query
  mysql> desc watchdog;
  ERROR 2006 (HY000): MySQL server has gone away
  No connection. Trying to reconnect...
  Connection id:    15165
  Current database: drupal_pre

  +-----------+---------------------+------+-----+---------+----------------+
  | Field     | Type                | Null | Key | Default | Extra          |
  +-----------+---------------------+------+-----+---------+----------------+
  | wid       | int(11)             | NO   | PRI | NULL    | auto_increment |
  | uid       | int(11)             | NO   | MUL | 0       |                |
  | type      | varchar(64)         | NO   | MUL |         |                |
  | message   | longtext            | NO   |     | NULL    |                |
  | variables | longblob            | NO   |     | NULL    |                |
  | severity  | tinyint(3) unsigned | NO   | MUL | 0       |                |
  | link      | varchar(255)        | YES  |     |         |                |
  | location  | text                | NO   |     | NULL    |                |
  | referer   | text                | YES  |     | NULL    |                |
  | hostname  | varchar(128)        | NO   |     |         |                |
  | timestamp | int(11)             | NO   |     | 0       |                |
  +-----------+---------------------+------+-----+---------+----------------+
  11 rows in set (0,43 sec)

  SELECT * FROM watchdog INTO OUTFILE '/tmp/drupal_preprod_watchdog.csv' FIELDS TERMINATED BY '|' ENCLOSED BY '"' LINES TERMINATED BY '\n';

  DELETE FROM watchdog;
  ```





## sitemap.xml generado por el módulo xmlsitemap

Lo recuperan los motores de busqueda para indexar el sitio. Se puede configurar para decirle que mostrar.

## Busquemos además que módulos están habilitados

```
drupadmin@lvwdruppreprdapp drupal-preprd]$ drush pml --status=Enabled
```

### Problemas de Cookies

Agregar estas líneas en el virtualhost de drupal y en el reverse proxy.

```
 # Mejoras de seguridad
    Header edit Set-Cookie ^(.*)$ $1;HttpOnly;Secure
    TraceEnable Off
```



### Enlaces a tener en cuenta: 

https://stackoverflow.com/questions/3161548/how-do-i-prevent-site-scraping

http://www.madirish.net/242 

https://www.drupal.org/node/264653

https://www.drupal.org/node/2364991

https://www.drupal.org/project/domain_simple_sitemap

https://www.drupal.org/node/2360859

https://www.drupal.org/docs/7/install/step-2-create-the-database

http://www.madirish.net/242

https://www.drupal.org/project/goaway

https://2bits.com/articles/presentation-28-million-page-views-day-70-million-month-one-server.html

https://www.drupal.org/forum/support/post-installation/2007-05-19/node_counter-what-is-is-and-how-do-i-use-it



### Solucionando Problemas

https://www.drupal.org/project/registry_rebuild

Como buscar info sobre módulos instalados y borrarlos cuando no estan en filesystem

```shell
drush sql-query "select filename,info from system where type='module' and name='openpublic_editors_choice'"
drush sql-query "DELETE from system where name = 'openpublic_editors_choice';"
```

La mejor opcion es recuperar nombre y version, instalar, y luego por drush

```shell
drush dis <module>
drus pm-uninstall <module>
```





PHP Fatal error:  Class 'EntityCacheControllerHelper' not found in /var/www/drupal-test/profiles/openpublic/modules/contrib/entitycache/entitycache.taxonomy.inc on line 29
solucion
    drush sql-query "truncate table registry_file;"
    https://www.drupal.org/project/registry_rebuild





https://www.drupal.org/project/menu_target/issues/2945123

```
so line 19 of menu_target.admin.inc
if (empty($enabled_menus) || empty(array_filter($enabled_menus))) {
will cause the error mentioned in the title.

this can be fixed by just doing the same thing in a different way. The following change should work.
if (empty($enabled_menus) || !count(array_filter($enabled_menus))) {
```

