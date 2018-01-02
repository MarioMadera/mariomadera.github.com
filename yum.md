## Configuraciones de YUM en CentOS y RHEL

### Proxy

```bash
drupal $ sudo vi /etc/yum/yum.conf
```

dentro del la sección [main] agregar

```
proxy=http://proxyhost:proxyport
#proxy_username=username
#proxy_password=secret
```

**Nota** en nuestro caso no es necesario porque CNTLM no requiere credenciales.

