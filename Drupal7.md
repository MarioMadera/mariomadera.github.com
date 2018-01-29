# Drupal 7

## Problemas de Seguridad

### robots.txt

```
‎‎Martinez, Luis Alejandro‎‎ [10:52]:
  https://api.drupal.org/api/drupal/robots.txt/7.x 
  
  Esta conversación se guarda en la carpeta Bandeja de entrada de Outlook.
  
‎‎Madera Laporta, Mario Alfonso‎‎ [11:20]:
  Dato, hablando con la gente de Base de Datos me explicaron que:
- no se noto un crecimiento en archivos de datos.
- lo que se llenaba era el filesystem de los logs binarios (usados para replicar)
- la mayoría de los registros eran sentencias insert en las tablas accesslog, watchdoc y otras del sistema de cache de drupal.
‎‎Martinez, Luis Alejandro‎‎ [11:21]:
  voy a revisar el log del OWASP ZAP a ver que estaba haciendo, porque yo lo mande hacer un spider nada mas
  ese dato esta bueno gracias
‎‎Madera Laporta, Mario Alfonso‎‎ [11:28]:
  Luis o Alejandro, como prefieres? No tenemos costumbre en Uruguay de usar los dos nombres ...
‎‎Martinez, Luis Alejandro‎‎ [11:28]:
  alejandro
‎‎Madera Laporta, Mario Alfonso‎‎ [11:29]:
  Ok, Alejandro, creo haber descubierto porque no se tomo en cuenta el robots.txt.
  -rw-r-----. 1 drupadmin apache 1479 dic 18  2015 robots.txt
  por mas que el usuario apache pertenece al grupo apache, no estoy seguro que requiera ser el dueñon.
  dueño
‎‎Martinez, Luis Alejandro‎‎ [11:32]:
  ni yo, busquemos a ver que dice SAN GOOGLE
‎‎Madera Laporta, Mario Alfonso‎‎ [11:33]:
  segun se nos explico durante la instalación es una buena práctica que todo el sitio tenga un usuario distinto a apache como owner y grupo apache. Permisos 0640 para archivos y 0750 para directorios.
‎‎Martinez, Luis Alejandro‎‎ [11:34]:
  me parece muy coherente la recomendación
‎‎Martinez, Luis Alejandro‎‎ [11:39]:
  https://www.volacci.com/blog/fix-problems-drupal-default-robotstxt-file 
  The reason is that Drupal does not require the trailing slash ( / ) after the path to show you the content 
‎‎Madera Laporta, Mario Alfonso‎‎ [11:41]:
  guauuu!!!
‎‎Martinez, Luis Alejandro‎‎ [11:41]:
  todas las líneas que hacen mención a directorios tienen el / después del nombre
  no cuesta nada probarlo
‎‎Madera Laporta, Mario Alfonso‎‎ [11:42]:
  para nada... voy a comparar el robots.txt que está en drupal.org
  a ver si lo tienen corregido
  pues no.
‎‎Martinez, Luis Alejandro‎‎ [11:43]:
  nop
  ya lo mire
‎‎Madera Laporta, Mario Alfonso‎‎ [11:43]:
  chan!!!
‎‎Martinez, Luis Alejandro‎‎ [11:44]:
  fíjate que la persona que publica el workaround dice que hay que copiar los bloques # Paths (no clean URLs) y # Paths (clean URLs)   
  y dejarlos duplicados, uno con los / al final y el otro sin ellos
‎‎Madera Laporta, Mario Alfonso‎‎ [11:44]:
  Si, y encontré una razón.
  iangetz commented 4 years ago
I'd be curious to know the recommended approach as well. Using the Google Webmaster Tools robots.txt testing tool:

**http://mydomain.com/user/register/ >> Blocked by line 47: Disallow: /user/register/
**http://mydomain.com/user/register >> Allowed

As a result, **http://mydomain.com/user/register is appearing in Google SERPs.
  voy a corregir las 3 secciones con / al final
  # Directories, # Paths (clean URLS) y # Paths (no clean URLS)
‎‎Martinez, Luis Alejandro‎‎ [11:47]:
  dale y cuando lo copies pruebo el crawler sobre el pe-producción con ese cambio a ver que pasa
‎‎Madera Laporta, Mario Alfonso‎‎ [11:47]:
  ok. te aviso cuando correr.
  
  Esta conversación se guarda en la carpeta Historial de conversaciones del buzón de Outlook.
  
```

##Proxy

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

## Proceso de Mejoras durante Pentest interno

Durante un proceso de pentesting interno se saturo el filesystem de los binlogs de la base mysql

Analizando los datos disponibles podemos ver que:

1. habían mas de 300 procesos en espera para la base de datos.
2. se generaron varios GB de binary logs

la lista de procesos se grabo a texto y luego fue analizado.

```
awk -F "|" '{print $9}' drupalpre_processlist | cut -d" " -f2 | sort | uniq -c
     98
      2 CREATE
      1 FLUSH
      1 Info
    296 INSERT
      3 show
    212 UPDATE
```

Ahora analizamos los insert y updates.

```
cat drupalpre_processlist | grep INSERT |awk -F"|" '/INSERT/ {print $9}' | cut -d" " -f4 | sort | uniq -c
    109 cache_page
      3 captcha_sessions
    184 watchdog
```

```
λ cat drupalpre_processlist | grep UPDATE |awk -F"|" '{print $9}' | cut -d" " -f3 | sort | uniq -c
    117 cache
     95 node_counter
```

Tambien se detecto que OWASP ZAP era capaz de recuperar un archivo llamado sitemap.xml con un contenido como el que sigue

```xml
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
	<url>
		<loc>http://portal.ute.com.uy/</loc>
		<changefreq>daily</changefreq>
		<priority>1.0</priority>
	</url>
	<url>
		<loc>http://portal.ute.com.uy/institucional-medio-ambiente/rescate-arqueol%C3%B3gico-en-punta-del-tigre</loc>
	</url>
	<url>
		<loc>http://portal.ute.com.uy/institucional-medio-ambiente/gesti%C3%B3n-exportaci%C3%B3n-de-residuos-pcb</loc>
	</url>
	<url>
		<loc>http://portal.ute.com.uy/institucional-medio-ambiente/campo-electromagn%C3%A9tico-cem</loc>
	</url>
	<url>
      	<loc>http://portal.ute.com.uy/institucional-medio-ambiente/planta-de-celulosa-de-botnia</loc>
	</url>
	<url>
		<loc>http://portal.ute.com.uy/institucional/certificaciones</loc>
	</url>
...
```

como se puede apreciar, permite conocer cada una de las páginas válidas dentro del sitio. Parseando el código html de cada uno de ellas sigue obteniendo recursos tales como 

* archivos
* formularios
* scripts
* otros enlaces

### Iteración 1 para soluciones

#### Bajar la cantidad de inserts a watchdog (https://www.drupal.org/docs/8/core/modules/syslog/overview)

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

* desahbilitamos el módulo dblog

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

  ​

  Analizando los binlogs que me paso base de datos

  | archivo          | # lineas | Mb   |
  | ---------------- | -------- | ---- |
  | mysql-bin.000032 | 21943    | 48   |
  | mysql-bin.000860 | 16768    | 67   |
  | mysql-bin.000861 | 31547    | 101  |
  | mysql-bin.000862 | 26429    | 101  |
  | mysql-bin.000863 | 25165    | 101  |
  | mysql-bin.000864 | 23562    | 101  |
  | mysql-bin.000865 | 20418    | 101  |
  | mysql-bin.000866 | 22646    | 101  |
  | mysql-bin.000867 | 24317    | 101  |
  | mysql-bin.000868 | 24418    | 101  |
  | mysql-bin.000869 | 24844    | 101  |
  | mysql-bin.000870 | 25270    | 101  |
  | mysql-bin.000871 | 24846    | 101  |
  | mysql-bin.000872 | 23756    | 101  |
  | mysql-bin.000873 | 25783    | 101  |
  | mysql-bin.000874 | 24919    | 101  |
  | mysql-bin.000875 | 24905    | 101  |
  | mysql-bin.000876 | 25296    | 101  |
  | mysql-bin.000877 | 25243    | 101  |
  | mysql-bin.000878 | 9277     | 40   |
  | mysql-bin.index  | 19       | 1    |

  for i in `ls *bin*`; do echo "|$i| $(wc -l $i|  cut -d" " -f1)| $(du -sm  $i| cut -d" " -f1 )|"; done + Notepad++ ;-

* ​

Busquemos además que módulos están habilitados

```
drupadmin@lvwdruppreprdapp drupal-preprd]$ drush pml --status=Enabled
 Package                          Name                                                               Type    Version
 Access control                   Content Access (content_access)                                    Module  7.x-1.2-beta2
 Administration                   Module filter (module_filter)                                      Module  7.x-2.0-alpha2+0-dev
 Chaos tool suite                 Chaos tools (ctools)                                               Module  7.x-1.9
 Chaos tool suite                 Custom rulesets (ctools_access_ruleset)                            Module  7.x-1.9
 Chaos tool suite                 Page manager (page_manager)                                        Module  7.x-1.9
 Chaos tool suite                 Views content panes (views_content)                                Module  7.x-1.9
 Content analysis                 Alchemy (alchemy)                                                  Module  7.x-1.0-beta1
 Content analysis                 Alchemy Content Analyzer (alchemy_contentanalysis)                 Module  7.x-1.0-beta1
 Content analysis                 Content Analysis API (contentanalysis)                             Module  7.x-1.0-beta6
 Content analysis                 Content Optimizer (contentoptimizer)                               Module  7.x-2.0-beta4
 Context                          Context (context)                                                  Module  7.x-3.6
 Context                          Context Bool Field (context_bool_field)                            Module  7.x-1.0
 Context                          Context Breadcrumb Current Page (context_breadcrumb_current_page)  Module  7.x-1.0
 Context                          Context Condition Admin Theme (context_condition_admin_theme)      Module  7.x-1.0
 Context                          Context UI (context_ui)                                            Module  7.x-3.6
 Core                             Block (block)                                                      Module  7.43
 Core                             Color (color)                                                      Module  7.43
 Core                             Comment (comment)                                                  Module  7.43
 Core                             Content translation (translation)                                  Module  7.43
 Core                             Contextual links (contextual)                                      Module  7.43
 Core                             Dashboard (dashboard)                                              Module  7.43
 Core                             Field (field)                                                      Module  7.43
 Core                             Field SQL storage (field_sql_storage)                              Module  7.43
 Core                             Field UI (field_ui)                                                Module  7.43
 Core                             File (file)                                                        Module  7.43
 Core                             Filter (filter)                                                    Module  7.43
 Core                             Help (help)                                                        Module  7.43
 Core                             Image (image)                                                      Module  7.43
 Core                             List (list)                                                        Module  7.43
 Core                             Locale (locale)                                                    Module  7.43
 Core                             Menu (menu)                                                        Module  7.43
 Core                             Node (node)                                                        Module  7.43
 Core                             Number (number)                                                    Module  7.43
 Core                             Options (options)                                                  Module  7.43
 Core                             Path (path)                                                        Module  7.43
 Core                             PHP filter (php)                                                   Module  7.43
 Core                             RDF (rdf)                                                          Module  7.43
 Core                             Search (search)                                                    Module  7.43
 Core                             Shortcut (shortcut)                                                Module  7.43
 Core                             Statistics (statistics)                                            Module  7.43
 Core                             Syslog (syslog)                                                    Module  7.43
 Core                             System (system)                                                    Module  7.43
 Core                             Taxonomy (taxonomy)                                                Module  7.43
 Core                             Text (text)                                                        Module  7.43
 Core                             Toolbar (toolbar)                                                  Module  7.43
 Core                             Update manager (update)                                            Module  7.43
 Core                             User (user)                                                        Module  7.43
 Date/Time                        Date (date)                                                        Module  7.x-2.8
 Date/Time                        Date API (date_api)                                                Module  7.x-2.8
 Date/Time                        Date Popup (date_popup)                                            Module  7.x-2.8
 Date/Time                        Date Repeat API (date_repeat)                                      Module  7.x-2.8
 Date/Time                        Date Repeat Field (date_repeat_field)                              Module  7.x-2.8
 Distribution Management          Apps (apps)                                                        Module  7.x-1.0
 Features                         Default Content (defaultcontent)                                   Module  7.x-1.0-alpha9
 Features                         Features (features)                                                Module  7.x-2.3
 Fields                           Bundle copy (bundle_copy)                                          Module  7.x-1.1
 Fields                           Field Group (field_group)                                          Module  7.x-1.4
 Fields                           Link (link)                                                        Module  7.x-1.3
 Fields                           Multiupload Filefield Widget (multiupload_filefield_widget)        Module  7.x-1.13
 Fields                           Node Reference (node_reference)                                    Module  7.x-2.1
 Fields                           References (references)                                            Module  7.x-2.1
 Fields                           User Reference (user_reference)                                    Module  7.x-2.1
 Insight                          Insight (insight)                                                  Module  7.x-1.0-alpha2
 Juno                             Block Animation (block_animation)                                  Module  7.x-1.0
 Juno                             Juni Widget (widget)                                               Module
 Juno                             Juno Shortcode (bs_shortcodes)                                     Module
 Juno                             Views Juno Custom (views_custom)                                   Module
 Keyword Research                 Keyword Research (kwresearch)                                      Module  7.x-1.0-alpha3+1-dev
 Keyword Research                 Keyword Research Google (kwresearch_google)                        Module  7.x-1.0-alpha3+1-dev
 Mail                             SMTP Authentication Support (smtp)                                 Module  7.x-1.0
 Media                            File entity (file_entity)                                          Module  7.x-1.4
 Media                            IMCE (imce)                                                        Module  7.x-1.9
 Media                            Media (media)                                                      Module  7.x-1.4
 Media                            Media Internet Sources (media_internet)                            Module  7.x-1.4
 Media                            Video Embed Field (video_embed_field)                              Module  7.x-2.0-beta11
 MegaDrupal                       MD Slider (md_slider)                                              Module  7.x-2.8
 Multilanguage                    Translation helpers (translation_helpers)                          Module  7.x-1.0
 OpenPublic                       OpenPublic API (openpublic_api)                                    Module  7.x-1.7
 OpenPublic                       OpenPublic Base Fields (openpublic_base_fields)                    Module  7.x-1.7
 OpenPublic                       OpenPublic Captcha (openpublic_captcha)                            Module  7.x-1.7
 OpenPublic                       OpenPublic Comments (openpublic_comments)                          Module  7.x-1.7
 OpenPublic                       OpenPublic Document (openpublic_document)                          Module  7.x-1.7
 OpenPublic                       OpenPublic Menu (openpublic_menu)                                  Module  7.x-1.7
 OpenPublic                       OpenPublic Roles/Permissions (openpublic_users)                    Module  7.x-1.7
 OpenPublic                       OpenPublic Splash Page (openpublic_splash)                         Module  7.x-1.7
 OpenPublic                       OpenPublic Utility Menu (openpublic_menu_utility)                  Module  7.x-1.7
 OpenPublic                       OpenPublic Webform (openpublic_webform)                            Module  7.x-1.7
 Other                            AddThis (addthis)                                                  Module  7.x-2.1-beta1
 Other                            Automated Logout (autologout)                                      Module  7.x-4.3
 Other                            Back To Top (back_to_top)                                          Module  7.x-1.4
 Other                            Backup and Migrate (backup_migrate)                                Module  7.x-3.0
 Other                            Block Title Link (block_titlelink)                                 Module  7.x-1.5
 Other                            Colorbox (colorbox)                                                Module  7.x-2.10
 Other                            Comment Notify (comment_notify)                                    Module  7.x-1.2
 Other                            Conditional Stylesheets (conditional_styles)                       Module  7.x-2.2
 Other                            CSS3PIE (css3pie)                                                  Module  7.x-2.1
 Other                            Diff (diff)                                                        Module  7.x-3.2
 Other                            Elysia Cron (elysia_cron)                                          Module  7.x-2.1
 Other                            Entity API (entity)                                                Module  7.x-1.6
 Other                            Entity Autocomplete (entity_autocomplete)                          Module  7.x-1.0-beta3
 Other                            Entity tokens (entity_token)                                       Module  7.x-1.6
 Other                            Features Template (features_template)                              Module  7.x-1.0-beta1
 Other                            Follow (follow)                                                    Module  7.x-2.0-alpha1
 Other                            Grid builder (gridbuilder)                                         Module  7.x-1.0-alpha2
 Other                            Image URL Formatter (image_url_formatter)                          Module  7.x-1.4
 Other                            Job Scheduler (job_scheduler)                                      Module  7.x-2.0-alpha3
 Other                            Job Scheduler Trigger (job_scheduler_trigger)                      Module  7.x-2.0-alpha3
 Other                            JSON2 javascript library (json2)                                   Module  7.x-1.1
 Other                            Libraries (libraries)                                              Module  7.x-2.2
 Other                            Menu attributes (menu_attributes)                                  Module  7.x-1.0-rc2+9-dev
 Other                            Menu target (menu_target)                                          Module  7.x-1.3
 Other                            OpenPublic Accessibility (openpublic_accessibility)                Module  7.x-1.7
 Other                            Panels responsive layouts (layout)                                 Module  7.x-1.0-alpha6
 Other                            Pathauto (pathauto)                                                Module  7.x-1.0-beta1+0-dev
 Other                            Redirect (redirect)                                                Module  7.x-1.0-rc1
 Other                            Scheduler (scheduler)                                              Module  7.x-1.3
 Other                            Secure Pages (securepages)                                         Module  7.x-1.0-beta2
 Other                            Special menu items (special_menu_items)                            Module  7.x-2.0+0-dev
 Other                            Strongarm (strongarm)                                              Module  7.x-2.0
 Other                            Token (token)                                                      Module  7.x-1.5
 Panels                           Mini panels (panels_mini)                                          Module  7.x-3.4
 Panels                           Panels (panels)                                                    Module  7.x-3.4
 Path management                  Global Redirect (globalredirect)                                   Module  7.x-1.5
 Performance and scalability      Entity cache (entitycache)                                         Module  7.x-1.2
 Printer, email and PDF versions  Printer-friendly pages (print)                                     Module  7.x-2.0
 Printer, email and PDF versions  Printer-friendly pages UI (print_ui)                               Module  7.x-2.0
 Rules                            Rules (rules)                                                      Module  7.x-2.7
 Rules                            Rules Scheduler (rules_scheduler)                                  Module  7.x-2.7
 Rules                            Rules UI (rules_admin)                                             Module  7.x-2.7
 SEO                              Metatag (metatag)                                                  Module  7.x-1.0-rc2
 SEO                              Metatag: Context (metatag_context)                                 Module  7.x-1.0-rc2
 SEO                              Metatag: Facebook (metatag_facebook)                               Module  7.x-1.0-rc2
 SEO                              SEO Tools (seotools)                                               Module  7.x-1.0-alpha6
 Sharing                          ShareThis (sharethis)                                              Module  7.x-2.9+0-dev
 Shortcode                        Shortcode (shortcode)                                              Module  7.x-2.1
 Shortcode                        Shortcode Basic Tags (shortcode_basic_tags)                        Module  7.x-2.1
 Shortcode                        Shortcode Embed Content Tag (shortcode_embed_content)              Module  7.x-2.1
 Shortcode                        Shortcode Video Macro (shortcode_video)                            Module  7.x-2.1
 Spam control                     CAPTCHA (captcha)                                                  Module  7.x-1.2
 Spam control                     Image CAPTCHA (image_captcha)                                      Module  7.x-1.2
 Spam control                     reCAPTCHA (recaptcha)                                              Module  7.x-1.11
 Statistics                       Google Analytics (googleanalytics)                                 Module  7.x-2.1
 tb_megamenu                      TB Mega Menu (tb_megamenu)                                         Module  7.x-1.0-beta5+0-dev
 Theme Tools                      Delta API (delta)                                                  Module  7.x-3.0-beta11
 Theme Tools                      Delta Blocks (delta_blocks)                                        Module  7.x-3.0-beta11
 Theme Tools                      Delta UI (delta_ui)                                                Module  7.x-3.0-beta11
 User interface                   Ajax links API (ajax_links_api)                                    Module  7.x-1.83
 User interface                   CKEditor (ckeditor)                                                Module  7.x-1.16
 User interface                   CKEditor SWF (ckeditor_swf)                                        Module  7.x-1.0-beta1
 User interface                   Gallery Formatter (galleryformatter)                               Module  7.x-1.0-beta5+0-dev
 User interface                   jCarousel (jcarousel)                                              Module  7.x-1.0-beta1
 User interface                   jQuery plugins (jquery_plugin)                                     Module  7.x-1.0-beta1+0-dev
 User interface                   jQuery Update (jquery_update)                                      Module  7.x-2.7
 User interface                   Popup Message (popup_message)                                      Module  7.x-1.2
 User interface                   Superfish (superfish)                                              Module  7.x-1.9+33-dev
 UTE                              UTE - Crear novedades (ute_crear_novedades)                        Module  7.x-1.0
 Views                            Node Reference View Formatter (node_reference_view_formatter)      Module  7.x-1.0
 Views                            Views (views)                                                      Module  7.x-3.11
 Views                            Views PHP (views_php)                                              Module  7.x-1.0-alpha1
 Views                            Views Slideshow (views_slideshow)                                  Module  7.x-3.1
 Views                            Views Slideshow: Cycle (views_slideshow_cycle)                     Module  7.x-3.1
 Views                            Views UI (views_ui)                                                Module  7.x-3.11
 Webform                          Webform (webform)                                                  Module  7.x-3.24
 Webform                          Webform Clear (webform_clear)                                      Module  7.x-2.0
 XML sitemap                      XML sitemap (xmlsitemap)                                           Module  7.x-2.2
 XML sitemap                      XML sitemap engines (xmlsitemap_engines)                           Module  7.x-2.2
 XML sitemap                      XML sitemap menu (xmlsitemap_menu)                                 Module  7.x-2.2
 XML sitemap                      XML sitemap node (xmlsitemap_node)                                 Module  7.x-2.2
 Core                             Seven (seven)                                                      Theme   7.43
 Otro(s)                          juno_college (juno_college)                                        Theme   7.x
 Otro(s)                          Spartan (spartan)                                                  Theme   7.x-1.7

```

* sitemap.xml generado por el módulo xmlsitemap

Enlaces a tener en cuenta:

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