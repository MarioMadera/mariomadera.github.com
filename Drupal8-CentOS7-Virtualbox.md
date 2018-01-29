

#Instalación de Drupal 8 sobre CentOS 7 en VirtualBox 

## Idea

Clonando la VM CentOS7-base, generar un ambiente de desarrollo para DRUPAL 8

## Software a Instalar

* PHP  7.x

* MySQL 5.5+

* APC

* Memcache

  Opcionales

* Varnish

* Composer

* Drush

* Drupal Console

* Adminer

* Drupal Dashboard


## Crear un usuario no privilegiado

```bash
useradd -m -c 'Usuario para Desarrollo en Drupal' -s /bin/bash -U drupal
passwd drupal
	use drupal1234
```

## Deshabilitar el firewall

```bash
# systemctl disable firewalld
```

```bash
# systemctl stop firewalld
# iptabless --flush
```

## Shared Folders con el hosts

Crear las carpetas locales... muestro la sección en el xml que define la VM

```xml
<SharedFolders>
	<SharedFolder name="logs" hostPath="D:\VirtualMachines\Centos7Drupal8\shared\logs" writable="true" autoMount="false"/>
	<SharedFolder name="www" hostPath="D:\VirtualMachines\Centos7Drupal8\shared\www" writable="true" autoMount="false"/>
</SharedFolders>
```

Es importante especificar que no automonte para luego poder elegir la ruta de montaje. Creo los puntos de montaje, monto las unidades y agrego los cambios en etc/fstab

```bash 
 mkdir /var/www
 mkdir /var/log/httpd
 mount -t vboxsf /www /var/www
 mount -t vboxsf logs /var/log/httpd
  cat /proc/mounts | grep vbox >> /etc/fstab
```

Compruebo

```bash
tail /etc/fstab
# Created by anaconda on Wed Dec 27 10:36:00 2017
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/cl-root     /                       xfs     defaults        0 0
UUID=c487df69-f3e4-4bee-9d7d-588fc81f296e /boot                   xfs     defaults        0 0
/dev/mapper/cl-swap     swap                    swap    defaults        0 0
www /var/www vboxsf rw,nodev,relatime 0 0
logs /var/log/httpd vboxsf rw,nodev,relatime 0 0
```

Compruebo que puedo escribir bidireccionalmente

```bash
touch /var/www/pepe
touch /var/log/httpd/pepe
```

```po
D:\VirtualMachines\Centos7Drupal8\shared
λ  d:\utiles\find . -name pepe
.\logs\pepe
.\www\pepe
```

**NOTA** luego de reiniciar, quedo en el prompt de recuperación. Edite los cambios en el fstab y reinicie. Compruebo que no levanta porque no puede montar los filesystem. No tiene el módulo vboxsf cargado.

_Posbiles Soluciones_

1. ~~--crear un archivo /etc/modules.d/vbox.conf con una sola línea. Debería cargar el módulo durante el inicio y con eso poder montar las carpetas.--~~

2. ~~En la ArchWiki parece estar la solución https://wiki.archlinux.org/index.php/VirtualBox#Mount_at_boot~~ 

3. Crear servicio de systemctl

   1. ```vi /etc/systemd/system/var-www.mount``` 

      ```
      cat /etc/systemd/system/var-www.mount
      [Unit]
      Requires=vboxadd-service.service
      After=vbox-service.service

      [Mount]
      What = www
      Where = /var/www
      Type = vboxsf
      Options = defaults

      [Install]
      WantedBy = multi-user.target
      ```

      **Nota** es requisito que el nombre del archivo sea igual a la ruta, cambiando las '/' por '-'

   2. ```systemctl daemon-reload```

   3. ```systemctl enable var-www.mount```

   4. ```systemctl start var-www.mount```

   5. ```systemctl status var-www.mount```

   6. reboot para verificar que inicia con el sistema

## Instalación de Apache

```bash
yum install httpd
systemctl enable httpd.service
systemctl start httpd.service
```

**Nota** fallo el inicio. Asumo que por las rutas de los montajes para los share folder compartiendo las rutas por defecto del sistema. Dichos montajes deben ocurrer posteriormente al inicio del HTTPD y este falla por no poder acceder.

```bash
systemctl disable var-www.mount
systemctl disable var-log-httpd.mount
 mkdir -p /var/www/drupal
 mkdir -p /var/log/httpd/drupal
 # corrijo las rutas en las unidades creadas en el paso anterior y reinicio el sistema
 # efectivamente, este era el problema.
```

Se siguieron los [pasos de Hardening](./Apache.md).

Se solicitaron alias DNS para cada virtualhost y los certificados para configurar SSL.

Instalamos el módulo de SSL

```
[root@lvwdrup8devapp conf.modules.d]# yum install httpd24-mod_ssl.x86_64
Loaded plugins: product-id, search-disabled-repos, subscription-manager
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
Resolving Dependencies
--> Running transaction check
---> Package httpd24-mod_ssl.x86_64 1:2.4.27-8.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

===================================================================================================================================
 Package                                          Arch                                    Version                                         Repository                              Size
===================================================================================================================================
Installing:
 httpd24-mod_ssl                                  x86_64                                  1:2.4.27-8.el7                                  rhscl                                  110 k

Transaction Summary
===================================================================================================================================
Install  1 Package

Total download size: 110 k
Installed size: 232 k

```

Ahora debo agregar el Reenvío de Puertos hacia el host. Ver tabla al final del documento.

|                   | guest | host |
| :---------------: | :---: | :--: |
| **Default HTTP**  |  80   | 8080 |
| **Default HTTPS** |  443  | 4433 |

Solicitar Datos de las bases para los sitios a Enrique Machado



## PHP 7.1  

La razón de usar 7.1 está dada en https://www.drupal.org/docs/8/system-requirements/php 

## PHP versions supported 

| PHP version | Supported?   | Recommended? |
| ----------- | ------------ | ------------ |
| 5.5         | 5.5.9+       | No           |
| 5.6         | Yes          | No           |
| 7.0         | Yes          | No           |
| **7.1**     | **Yes**      | **Yes**      |
| 7.2         | Some issues* | No           |

Siguiendo las notas en https://webtatic.com/packages/php71/ 

```bash
rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
yum update 
yum install  php71w-cli php71w-common mod_php71w php71w-opcache php71w-gd php71w-intl php71w-mbstring php71w-mysqlnd php71w-xml 
```

Configuraciones de PHP tomadps de https://www.drupal.org/docs/8/system-requirements/php

### A shortlist based on above:

| section        |                                          | d8                              |
| -------------- | ---------------------------------------- | ------------------------------- |
| PHP version    |                                          | min 5.5.9 - Rec 5.6.5           |
| core           | allow_url_fopen                          | must be OFF or nonexistent      |
| core           | display_errors                           | off                             |
| core           | max_execution_time                       | min 30 seconds                  |
| core           | expose_php                               | must be OFF or nonexistent      |
| core           | memory_limit                             | 64MB                            |
| ctype          | ctype functions                          | enabled                         |
| cURL           | cURL support                             | enabled (avoid 7.35)            |
| cURL           | libSSH Version                           | required for install update     |
| date           | date/time support                        | enabled                         |
| dom            | DOM/XML                                  | enabled                         |
| fileinfo       | fileinfo support                         | enabled                         |
| filter         | Input Validation and Filtering           | enabled                         |
| gd             | GD Support                               | enabled                         |
| core           | magic_quotes_gpc                         | must be OFF or nonexistent      |
| core           | magic_quotes_runtime                     | must be disabled or nonexistent |
| core           | safe_mode                                | must be disabled or nonexistent |
| core           | register_globals                         | must be OFF or nonexistent      |
| hash           | hash support                             | enabled                         |
| openssl        | openssl support                          | enabled                         |
| json           | json support                             | enabled                         |
| pcre           | PCRE (Perl Compatible Regular Expressions) Support | enabled                         |
| pdo            | PHP Data Objects(PDO)                    | (specific for your database)    |
| pecl           | PECL version of PDO                      | not compatible                  |
| session        | Session Support                          | enabled                         |
| session        | session.auto_start                       | 0                               |
| session        | session.cache_limiter                    | nocache                         |
| SimpleXML      | Schema support                           | enabled                         |
| SPL            | SPL support                              | lots                            |
| Suhosin        | APC.include-once-override                | avoid                           |
| Tokenizer      | Tokenizer Support                        | enabled                         |
| XML            | XML Support                              | enabled                         |
| XML            | XML Namespace Support                    | enabled                         |
| Zend OPcache   | Opcode caching                           | up and running                  |
| Zend OPcache   | PHP 5.5+ opcache.save_comments           | 1                               |
| Zend OPcache   | PHP 5.5+ opcache.load_comments           | 1                               |
| MODULE TESTING | memory_limit                             | 256                             |

## Adminer

Adminier es un reemplazo a phpmyadmin pero tambien permite administrar otras motores de base de datos. TAmbien posée varios plugins y 'pieles' controladas mediante archivos de hojas de estilo.

https://www.adminer.org/

```bash
 curl -OL https://github.com/vrana/adminer/releases/download/v4.3.1/adminer-4.3.1.zip
 unzip adminer-4.3.1.zip
 mv adminer-4.3.1 adminer
 mv adminer /var/www/
 chown -R apache:apache /var/www/adminer
 find /var/www/adminer -type d -exec chmod 775 {} \;
 find /var/www/adminer -type f -exec chmod 664 {} \;
 
```

Recordar editar el archivo ```/etc/selinux/config``` y dejé la variable ```SELINUX=disable```  y reiniciar.

Crear un VirtualHost en `/etc/httpd/conf.d/adminer.conf`

    <VirtualHost *:80>
    	ServerAdmin webmaster@dummy-host.example.com
    	DocumentRoot /var/www/adminer
    	ServerName adminer
    	ServerAlias adminer
    	ErrorLog  /var/log/httpd/adminer_error.log
    	CustomLog /var/log/httpd/adminer_custom.log common
    </VirtualHost>	
Agregar la enrtrada correspondiente en `/etc/hosts` 

````
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
127.0.0.1   adminer
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
````

En el host windows editar el archivo `c:\windows\system32\drivers\etc\hosts` y agregar una entrada similar

Ahora podemos acceder desde el host usando http://adminer:8080/adminer

## Instalacion de Composer, Drush y Drupal 8 

Usando las notas de http://valuebound.com/resources/blog/Installing-drupal-with-drush-the-basics

### Composer

```bash
curl -sS https://getcomposer.org/installer | php
```

Falló. Pide modificar a on el  ```allow_url_fopen```

    [root@localhost ~]# curl -sS https://getcomposer.org/installer | php
    Some settings on your machine make Composer unable to work properly.
    Make sure that you fix the issues listed below and run this script again:
    The allow_url_fopen setting is incorrect.
    Add the following to the end of your php.ini:
    allow_url_fopen = On
    The php.ini used by your command-line PHP is: /etc/php.ini
    
    If you can not modify the ini file, you can also run php -d option=value to modify ini values on the fly. You can use -d multiple times.
    [root@localhost ~]# cp /etc/php.ini /etc/php.ini.drupal8
    [root@localhost ~]# vi /etc/php.ini
Lanzo de nuevo y ahora sí

```bash
curl -sS https://getcomposer.org/installer | php
All settings correct for using Composer
Downloading...

Composer (version 1.5.6) successfully installed to: /root/composer.phar
Use it: php composer.phar
mv composer.phar /usr/local/bin/composer
```

Las proximas configuraciones las hago en el usurio drupal

```bash
# su - drupal
$ vim .bash_profile
...
export PATH="$HOME/.config/composer/vendor/bin:$PATH"
:wq!

$ source ~/.bash_profile
```

### Instalación de drush

```bash
[root@localhost ~]# curl -OL https://github.com/drush-ops/drush/releases/download/8.1.15/drush.phar
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   599    0   599    0     0    623      0 --:--:-- --:--:-- --:--:--   624
100 5912k  100 5912k    0     0   200k      0  0:00:29  0:00:29 --:--:--  144k

[root@localhost ~]# php drush.phar core-status
 PHP configuration      :  /etc/php.ini
 PHP OS                 :  Linux
 Drush script           :  /root/drush.phar
 Drush version          :  8.1.15
 Drush temp directory   :  /tmp
 Drush configuration    :
 Drush alias files      :

[root@localhost ~]# chmod +x drush.phar
[root@localhost ~]# mv drush.phar /usr/local/bin/drush

[drupal@localhost ~]$ drush init

 Modify /home/drupal/.bashrc to include Drush configuration files? (yes/no) [yes]:
 >

 [notice] mkdir ["/home/drupal/.drush"]
 [notice] Writing to /home/drupal/.drush/drush.yml.
 [notice] Writing to /home/drupal/.drush/drush.bashrc.
 [ok] Copied Drush bash customizations to /home/drupal/.drush/drush.bashrc
 [notice] Writing to /home/drupal/.drush/drush.prompt.sh.
 [ok] Copied Drush prompt customizations to /home/drupal/.drush/drush.prompt.sh
 [notice] Writing to /home/drupal/.bashrc.
 [ok] Updated bash configuration file /home/drupal/.bashrc
 [ok] Start a new shell in order to experience the improvements (e.g. `bash`).

```

## Instalacion de Drupal

### Previas, agregar el usuario ```drupal``` al grupo ```apache```

```bash
usermod -a -G apache drupal
cat /etc/group | grep apache
apache:x:48:drupal

chown -R apache:apache /var/www/drupal/
chmod -R 775 /var/www/drupal/
```

### Bajamos el core y lo expandimos

```bash
curl -OL https://ftp.drupal.org/files/projects/drupal-8.4.2.tar.gz
tar zxf drupal-8.4.2.tar.gz
mv drupal-8.4.2 drupal-dev
```

### Creamos la base de datos para el ambiente y el usuario en mysql

```mysql
mysql -u root -p -e "CREATE DATABASE drupaldev CHARACTER SET utf8 COLLATE utf8_general_ci"
mysql -u root -p
CREATE USER drupdev@localhost IDENTIFIED BY 'drupdev';
GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER, CREATE TEMPORARY TABLES ON drupaldev.* TO 'drupdev'@'localhost' IDENTIFIED BY 'drupdev';
```

### Crear el virtual host apache

```
<VirtualHost *:80>
    ServerAdmin webmaster@dummy-host.example.com
    DocumentRoot /var/www/drupal/drupal8-dev
    <Directory /var/www/drupal/drupal8-dev>
       Options FollowSymLinks
       AllowOverride All
       Require all granted
       # Clean URL
       RewriteEngine On
       RewriteBase /
       RewriteCond %{REQUEST_FILENAME} !-f
       RewriteCond %{REQUEST_FILENAME} !-d
       RewriteRule ^(.*)$ index.php?q=$1 [L,QSA]
     </Directory>
    ServerName drupal8dev
    ServerAlias drupal8dev
    ErrorLog  /var/log/httpd/drupal/drupal8-dev_error.log
    CustomLog /var/log/httpd/drupal/drupal8-dev_custom.log common
</VirtualHost>
```

Luego de esto agregamos una entrada en ```/etc/hosts``` para resolver el server name

```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
127.0.0.1   drupal8dev
127.0.0.1   adminer
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
```

y reiniciamos

```
systemctl restart httpd.service
```

Debemos reconfigurar el proxy a nivel de la vm para que los accesos al sitio no intente salir por proxy, editando la variable no_proxy en ```/etc/profile.d/proxy.sh```

```
export no_proxy=127.0.0.1,localhost,drupal8dev
```

Luego debemos reconfigurar host y proxy en nuestra PC windows. Editamos primero el archivo  ```c:\windows\system32\drivers\etc\hosts``` agregando una linea similar a la agregada en la vm linux. Luego debemos configurar el navegador para que ignore el proxy para ese dominio.

Luego podremos acceder como http://drupal8dev:8080/

Luego fue, acceder al sitio y seguir los pasos del install.

## Herramientas instaladas

* http://127.0.0.1:8080/adminer/adminer/



## Port Forwarding

|                   | guest | host  |
| :---------------: | :---: | :---: |
|      **SSH**      |  22   | 2222  |
| **Default HTTP**  |  80   | 8080  |
| **Default HTTPS** |  443  | 4433  |
|     **MySQL**     | 3306  | 13306 |

## Miscelaneas

### Vim

Agregar VIM enhanced. VIM minimal no puede reemplazarse porque es requisito de sudo

```bash
yum install vim-enhanced.x86_64 
yumn install tmux
```

drupal8dev/drup4l8.dev

Administrator/admin.1234
