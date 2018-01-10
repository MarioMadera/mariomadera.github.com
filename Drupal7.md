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

