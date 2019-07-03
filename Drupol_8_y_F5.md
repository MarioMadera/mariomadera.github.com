# Drupal8 & F5

El acceso inicial a https://drup8testhttps.localdomain nos devuelve 

```The provided host name is not valid for this server.```

Solución

Agregar en el settings.php, dentro del array trusted host, la dirección del balanceador.

```php
$settings['trusted_host_patterns'] = array(
    '^drup8test$',
    '^drup8test\.corp\.ute\.com\.uy$',
    '^drup8testHTTPS$',
    '^drup8testHTTPS\.corp\.ute\.com\.uy$',
 );
```

Ahora reviso en los logs a ver como graba la línea, 

Las líneas aparacen bien, nada que tocar... pero tengo muchas líneas del estilo

```
X.Y.171.211 - 2018-05-10 11:19:56 GET GET /  200 153003 14 749 154274 "-" "-"
X.Y.171.210 - 2018-05-10 11:20:00 GET GET /  200 153003 14 749 156975 "-" "-"
X.Y.171.211 - 2018-05-10 11:20:01 GET GET /  200 153003 18 749 156975 "-" "-"
X.Y.171.210 - 2018-05-10 11:20:05 GET GET /  200 153003 15 749 156975 "-" "-"
X.Y.171.211 - 2018-05-10 11:20:06 GET GET /  200 153003 14 749 156975 "-" "-"
X.Y.171.210 - 2018-05-10 11:20:10 GET GET /  200 153003 13 749 156975 "-" "-"
X.Y.171.211 - 2018-05-10 11:20:11 GET GET /  200 153003 14 749 156975 "-" "-"
X.Y.171.210 - 2018-05-10 11:20:15 GET GET /  200 153003 14 749 156975 "-" "-"
X.Y.171.211 - 2018-05-10 11:20:16 GET GET /  200 153003 16 749 156975 "-" "-"
X.Y.171.210 - 2018-05-10 11:20:20 GET GET /  200 153003 14 749 156975 "-" "-"
X.Y.171.211 - 2018-05-10 11:20:21 GET GET /  200 153003 14 749 156975 "-" "-"
```

Si son los probes del F5, podría ignorarlos...

## Separar en logs distintos las entradas de los probes. 

Seteo una variable de entorno para evitar loguear los probes
```
    SetEnvIf Remote_Addr X.Y.171.210  probe-no-log
    SetEnvIf Remote_Addr X.Y.171.211  probe-no-log
    
    <IfModule log_config_module>
       LogFormat "%a %u %{%Y-%m-%d %H:%M:%S}t %m %r %q %>s %B %{ms}T %I %O \"%{Referer}i\" \"%{User-Agent}i\"" ute-wls_style
       ErrorLog  /var/log/httpd24/drupal/drup8-test_error.log
       CustomLog /var/log/httpd24/drupal/drup8test_access.log ute-wls_style  env=!probe-no-log
    
       ErrorLog  /var/log/httpd24/drupal/drup8-test_probes-error.log
       CustomLog /var/log/httpd24/drupal/drup8test_probes-access.log ute-wls_style  env=probe-no-log
    </IfModule>
```
