# Apache Solr

[TOC]

## Instalar jdk

Solr requiere Oracle u OpenJDK 1.8 o superior.

```bash
yum install java-1.8.0-openjdk-devel.x86_64
...
Dependencies Resolved

================================================================================================
 Package                          Arch       Version                      Repository       Size
================================================================================================
Installing:                                
 java-1.8.0-openjdk-devel         x86_64     1:1.8.0.161-0.b14.el7_4      server          9.8 M
Installing for dependencies:               
 copy-jdk-configs                 noarch     2.2-5.el7_4                  server           19 k
 fontconfig                       x86_64     2.10.95-11.el7               server          229 k
 fontpackages-filesystem          noarch     1.44-8.el7                   server          9.9 k
 giflib                           x86_64     4.1.6-9.el7                  server           40 k
 java-1.8.0-openjdk               x86_64     1:1.8.0.161-0.b14.el7_4      server          243 k
 java-1.8.0-openjdk-headless      x86_64     1:1.8.0.161-0.b14.el7_4      server           32 M
 javapackages-tools               noarch     3.4.1-11.el7                 server           73 k
 libICE                           x86_64     1.0.9-9.el7                  server           66 k
 libSM                            x86_64     1.2.2-2.el7                  server           39 k
 libXcomposite                    x86_64     0.4.4-4.1.el7                server           22 k
 libXext                          x86_64     1.3.3-3.el7                  server           39 k
 libXfont                         x86_64     1.5.2-1.el7                  server          152 k
 libXi                            x86_64     1.7.9-1.el7                  server           40 k
 libXrender                       x86_64     0.9.10-1.el7                 server           26 k
 libXtst                          x86_64     1.2.3-1.el7                  server           20 k
 libfontenc                       x86_64     1.1.3-3.el7                  server           31 k
 lksctp-tools                     x86_64     1.0.17-2.el7                 server           88 k
 nss-pem                          x86_64     1.0.3-4.el7                  server           73 k
 python-javapackages              noarch     3.4.1-11.el7                 server           31 k
 stix-fonts                       noarch     1.1.0-5.el7                  server          1.3 M
 ttmkfdir                         x86_64     3.0.9-42.el7                 server           48 k
 tzdata-java                      noarch     2017c-1.el7                  server          183 k
 xorg-x11-font-utils              x86_64     1:7.5-20.el7                 server           87 k
 xorg-x11-fonts-Type1             noarch     7.5-9.el7                    server          521 k
Updating for dependencies:                 
 chkconfig                        x86_64     1.7.4-1.el7                  server          181 k
 nspr                             x86_64     4.13.1-1.0.el7_3             server          126 k
 nss                              x86_64     3.28.4-15.el7_4              server          849 k
 nss-softokn                      x86_64     3.28.3-8.el7_4               server          310 k
 nss-softokn-freebl               x86_64     3.28.3-8.el7_4               server          214 k
 nss-sysinit                      x86_64     3.28.4-15.el7_4              server           60 k
 nss-tools                        x86_64     3.28.4-15.el7_4              server          501 k
 nss-util                         x86_64     3.28.4-3.el7                 server           74 k
                                           
Transaction Summary                        
================================================================================================
Install  1 Package  (+24 Dependent packages)
Upgrade             (  8 Dependent packages)

Total download size: 47 M
Is this ok [y/d/N]:
```

**Nota** cuantas dependencias!!! :unamused:

## Módulos de Drupal necesarios para integrar con SOLR

- search_api // Search Api
- facets // Facets no usado por el momento 
- search_api_solr // Solr Search
- search_api_solr_defaults // Solr Search Defaults

## Instalación de SOLR

Creamos un directorio para la instalacion de SOLR

```bash
[root] > mkdir -p /opt/solr-inst
[root] > cd /opt/solr-inst
[root] > wget http://www-us.apache.org/dist/lucene/solr/6.6.3/solr-6.6.3.tgz                                     
```

extraigo el script de instalacion

```
[root] > tar xzf solr-6.6.3.tgz solr-6.6.3/bin/install_solr_service.sh --strip-components=2
```

Veo como se ejecuta el comando

```bash
[root] > [root@dev-drupalapp solr-inst]# ./install_solr_service.sh --help
ERROR: Specified Solr installation archive --help not found!
Usage: install_solr_service.sh <path_to_solr_distribution_archive> [OPTIONS]
  The first argument to the script must be a path to a Solr distribution archive, such as solr-5.0.0.tgz
    (only .tgz or .zip are supported formats for the archive)
  Supported OPTIONS include:

    -d     Directory for live / writable Solr files, such as logs, pid files, and index data; defaults to /var/solr
    -i     Directory to extract the Solr installation archive; defaults to /opt/
             The specified path must exist prior to using this script.
    -p     Port Solr should bind to; default is 8983
    -s     Service name; defaults to solr
    -u     User to own the Solr files and run the Solr process as; defaults to solr
             This script will create the specified user account if it does not exist.
    -f     Upgrade Solr. Overwrite symlink and init script of previous installation.
    -n     Do not start Solr service after install, and do not abort on missing Java
 NOTE: Must be run as the root user
```

Ejecuto  la instalación.                                 

```bash
[root] > ./install_solr_service.sh ./solr-6.6.3.tgz -u drupadmin -d /opt/solr-6.6.3/var
```

El valor para la opción ```-d``` lo determine luego de revisar el código fuente del instalador, teniendo en cuenta que es en ```/opt```donde tengo disponible espacio suficiente y para dejar los datos de SOLR bajo el directorio de instación.

Generamos el core selector

```bash
[root] > /opt/solr/bin/solr create -c ute_search -n data_driven_schema_configs 
```

y falla

```
  WARNING: Creating cores as the root user can cause Solr to fail and is not advisable. Exiting.
  If you started Solr as root (not advisable either), force core creation by adding argument -force
```

Reviso los permisos del comando

```bash
ls -l /opt/solr/bin/solr
-rwxr-xr-x 1 root root 72389 Mar  2 15:33 /opt/solr/bin/solr
```

Cualquiera puede usar el comando solr y la carpeta ```SOLR_VAR_DIR/data``` le pertenece al ```SOLR_USER``` que no es ```root```.
Pruebo crearlo como ese usuario.

```bash
[root] > su - drupadmin -c '/opt/solr/bin/solr create -c ute_search -n data_driven_schema_configs'
Copying configuration to new core instance directory:
/opt/solr-6.6.3/var/data/ute_search
Creating new core 'ute_search' using command:
http://localhost:8983/solr/admin/cores?action=CREATE&name=ute_search&instanceDir=ute_search

{
  "responseHeader":{
    "status":0,
    "QTime":1556},
  "core":"ute_search"}
```

Compruebo la creación y veo que creo

```
[root] > cd /opt/solr/var/data/ute_search/
[root] > ls -la
total 20
drwxrwxr-x 4 drupadmin drupadmin 4096 Apr 30 10:11 .
drwxr-x--- 3 drupadmin drupadmin 4096 Apr 30 10:11 ..
drwxrwxr-x 3 drupadmin drupadmin 4096 Apr 30 10:02 conf
-rw-rw-r-- 1 drupadmin drupadmin   80 Apr 30 10:11 core.properties
drwxrwxr-x 5 drupadmin drupadmin 4096 Apr 30 10:11 data
```

Obviamente, luego de instalar los pasos se realizan como el ```SOLR_USER```

```bash
[root] > su - drupadmin
[drupadmin] > mv conf conf.orig
[drupadmin] > mkdir conf
[drupadmin] > cp  ~/drup8dev/modules/contrib/search_api_solr/solr-conf/6.x/* conf/
```

verifico

```bash
[drupadmin] > diff ~/drup8dev/modules/contrib/search_api_solr/solr-conf/6.x/ conf/
```

No debería haber diferencias y no las hubo.

Comparando los permisos con el directorio anterior, los archivos tienen ```0664``` y los directorios ```0775```
```conf.orig/lang``` no está en ```conf```. No se si es requerido, lo copio

```bash
[drupadmin] > cp -r conf.orig/lang/ conf/
```

Corrijo los permisos de los archivos de conf

```bash
[drupadmin] > find -type f -exec chmod 0664 {} \;
```

Ahora tengo que volver a root para reiniciar el servicio

```bash
[root] > service solr restart
Sending stop command to Solr running on port 8983 ... waiting up to 180 seconds to allow Jetty process 44740 to stop gracefully.
Warning: Available entropy is low. As a result, use of the UUIDField, SSL, or any other features that require
RNG might not work properly. To check for the amount of available entropy, use 'cat /proc/sys/kernel/random/entropy_avail'.

Waiting up to 180 seconds to see Solr running on port 8983 [\]
Started Solr server on port 8983 (pid=45319). Happy searching!

[root] > service solr status

Found 1 Solr nodes:

Solr process 45319 running on port 8983
{
  "solr_home":"/opt/solr-6.6.3/var/data",
  "version":"6.6.3 d1e9bbd333ea55cfa0c75d324424606e857a775b - sarowe - 2018-03-02 15:09:34",
  "startTime":"2018-04-30T13:24:16.470Z",
  "uptime":"0 days, 0 hours, 0 minutes, 9 seconds",
  "memory":"32.3 MB (%6.6) of 490.7 MB"}
```

## Analizando el installer

```
| Option  | Varaible         | Default Value|
| -i      | SOLR_EXTRACT_DIR | [/opt]{ Debe existir o ser creado antes de instalar }
| -d      | SOLR_VAR_DIR     | [/var/$SOLR_SERVICE] { no puede ser igual que $SOLR_INSTALL_DIR porque causa 
|         |                  |                        problemas de permiso }
| -u      | SOLR_USER        | [solr]
| -S      | SOLR_SERVICE     | [solr]
| -p      | SOLR_PORT        | [8983]
| -f      | SOLR_UPGRADE     | [NO]
| -n      | SOLR_START       | [true]

Otras Variables
SOLR_ARCHIVE      | ruta hasta el archivo .tgz que se pasa como parámetro
SOLR_INSTALL_FILE | nombre del archivo extraído desde SOLR_ARCHIVE
SOLR_DIR          | nombre del instalador sin la extension tgz.
SOLR_INSTALL_DIR  | $SOLR_EXTRACT_DIR/$SOLR_DIR

# Chequeo de prerequisitos
tar, unzip, service, java, lsof
```

${SOLR_INSTALL_FILE: -4} -> devuelve los últimos 4 caracteres del valor de la variable....

**Nota** no tiene un uninstall.... pero puede deducirse de ver el código del installer. 

```
service stop solr
cd /opt
ls -la
> total 44
> drwxr-xr-x   8 root      root       4096 Apr 30 09:47 .
> dr-xr-xr-x. 19 root      root       4096 Nov 13 17:46 ..
> drwxr-xr-x   2 drupadmin drupadmin  4096 Apr 25 19:42 backup
> drwx------   2 root      root      16384 Jan 18 15:47 lost+found
> drwxr-xr-x   5 root      root       4096 Mar  5 08:42 rh
> lrwxrwxrwx   1 root      root         15 Apr 30 09:47 solr -> /opt/solr-6.6.3
> drwxr-x---   4 drupadmin drupadmin  4096 Apr 30 09:47 solr-6-6-3
> drwxr-xr-x   9 root      root       4096 Apr 30 09:47 solr-6.6.3
> drwxr-xr-x   3 root      root       4096 Apr 30 09:00 solr-inst
rm -rf solr solr-6-6-3 solr-6.6.3
cd /etc/default
rm -f solr.in.sh
cd /etc/init.d
rm -rf solr
```

## Anexo - Salida del Instalador

Analizarla para entender mejor el script

```
[root@dev-drupalapp solr-inst]# sh -x ./install_solr_service.sh ./solr-6.6.3.tgz -u drupadmin -d /opt/solr-6.6.3/var
+ [[ 0 -ne 0 ]]
+ for command in '"grep -E \"^NAME=\" /etc/os-release"' '"lsb_release -i"' '"cat /proc/version"' '"uname -a"'
++ eval grep -E '"^NAME="' /etc/os-release
+ distro_string='NAME="Red Hat Enterprise Linux Server"'
+ unset distro
+ [[ name="red hat enterprise linux server" == *\d\e\b\i\a\n* ]]
+ [[ name="red hat enterprise linux server" == *\r\e\d\ \h\a\t* ]]
+ distro=RedHat
+ [[ -n RedHat ]]
+ break
+ [[ ! -n RedHat ]]
+ '[' -z ./solr-6.6.3.tgz ']'
+ SOLR_ARCHIVE=./solr-6.6.3.tgz
+ '[' '!' -f ./solr-6.6.3.tgz ']'
+ SOLR_INSTALL_FILE=solr-6.6.3.tgz
+ is_tar=true
+ '[' .tgz == .tgz ']'
+ SOLR_DIR=solr-6.6.3
+ SOLR_START=true
+ '[' 5 -gt 1 ']'
+ shift
+ true
+ case $1 in
+ [[ -z drupadmin ]]
+ [[ d == \- ]]
+ SOLR_USER=drupadmin
+ shift 2
+ true
+ case $1 in
+ [[ -z /opt/solr-6.6.3/var ]]
+ [[ / == \- ]]
+ SOLR_VAR_DIR=/opt/solr-6.6.3/var
+ shift 2
+ true
+ case $1 in
+ '[' '' '!=' '' ']'
+ break
+ [[ -n true ]]
+ tar --version
+ [[ true == \t\r\u\e ]]
+ service --version
+ java -version
+ lsof -h
+ '[' -z '' ']'
+ SOLR_EXTRACT_DIR=/opt
+ '[' '!' -d /opt ']'
+ '[' -z '' ']'
+ SOLR_SERVICE=solr
+ '[' -z /opt/solr-6.6.3/var ']'
+ '[' -z drupadmin ']'
+ '[' -z '' ']'
+ SOLR_PORT=8983
+ '[' -z '' ']'
+ SOLR_UPGRADE=NO
+ '[' '!' NO = YES ']'
+ '[' -f /etc/init.d/solr ']'
+ '[' -e /opt/solr ']'
+ '[' -f /etc/init.d/solr ']'
++ id -u drupadmin
+ solr_uid=500
+ '[' 0 -ne 0 ']'
+ SOLR_INSTALL_DIR=/opt/solr-6.6.3
+ '[' '!' -d /opt/solr-6.6.3 ']'
+ echo -e '\nExtracting ./solr-6.6.3.tgz to /opt\n'

Extracting ./solr-6.6.3.tgz to /opt

+ true
+ tar zxf ./solr-6.6.3.tgz -C /opt
+ '[' '!' -d /opt/solr-6.6.3 ']'
+ chown -R root: /opt/solr-6.6.3
+ find /opt/solr-6.6.3 -type d -print0
+ xargs -0 chmod 0755
+ find /opt/solr-6.6.3 -type f -print0
+ xargs -0 chmod 0644
+ chmod -R 0755 /opt/solr-6.6.3/bin
+ '[' -h /opt/solr ']'
+ '[' -e /opt/solr ']'
+ echo -e '\nInstalling symlink /opt/solr -> /opt/solr-6.6.3 ...\n'

Installing symlink /opt/solr -> /opt/solr-6.6.3 ...

+ ln -s /opt/solr-6.6.3 /opt/solr
+ echo -e '\nInstalling /etc/init.d/solr script ...\n'

Installing /etc/init.d/solr script ...

+ cp /opt/solr-6.6.3/bin/init.d/solr /etc/init.d/solr
+ chmod 0744 /etc/init.d/solr
+ chown root: /etc/init.d/solr
+ sed_expr1='s#SOLR_INSTALL_DIR=.*#SOLR_INSTALL_DIR="/opt/solr"#'
+ sed_expr2='s#SOLR_ENV=.*#SOLR_ENV="/etc/default/solr.in.sh"#'
+ sed_expr3='s#RUNAS=.*#RUNAS="drupadmin"#'
+ sed_expr4='s#Provides:.*#Provides: solr#'
+ sed -i -e 's#SOLR_INSTALL_DIR=.*#SOLR_INSTALL_DIR="/opt/solr"#' -e 's#SOLR_ENV=.*#SOLR_ENV="/etc/default/solr.in.sh"#' -e 's#RUNAS=.*#RUNAS="drupadmin"#' -e 's#Provides:.*#Provides: solr#' /etc/init.d/solr
+ '[' '!' -d /etc/default ']'
+ '[' -f /opt/solr-6.6.3/var/solr.in.sh ']'
+ '[' -f /etc/default/solr.in.sh ']'
+ echo -e '\nInstalling /etc/default/solr.in.sh ...\n'

Installing /etc/default/solr.in.sh ...

+ cp /opt/solr-6.6.3/bin/solr.in.sh /etc/default/solr.in.sh
+ mv /opt/solr-6.6.3/bin/solr.in.sh /opt/solr-6.6.3/bin/solr.in.sh.orig
+ mv /opt/solr-6.6.3/bin/solr.in.cmd /opt/solr-6.6.3/bin/solr.in.cmd.orig
+ echo 'SOLR_PID_DIR="/opt/solr-6.6.3/var"
SOLR_HOME="/opt/solr-6.6.3/var/data"
LOG4J_PROPS="/opt/solr-6.6.3/var/log4j.properties"
SOLR_LOGS_DIR="/opt/solr-6.6.3/var/logs"
SOLR_PORT="8983"
'
+ chown root:drupadmin /etc/default/solr.in.sh
+ chmod 0640 /etc/default/solr.in.sh
+ mkdir -p /opt/solr-6.6.3/var/data
+ mkdir -p /opt/solr-6.6.3/var/logs
+ '[' -f /opt/solr-6.6.3/var/data/solr.xml ']'
+ cp /opt/solr-6.6.3/server/solr/solr.xml /opt/solr-6.6.3/server/solr/zoo.cfg /opt/solr-6.6.3/var/data/
+ '[' -f /opt/solr-6.6.3/var/log4j.properties ']'
+ cp /opt/solr-6.6.3/server/resources/log4j.properties /opt/solr-6.6.3/var/log4j.properties
+ chown -R drupadmin: /opt/solr-6.6.3/var
+ find /opt/solr-6.6.3/var -type d -print0
+ xargs -0 chmod 0750
+ find /opt/solr-6.6.3/var -type f -print0
+ xargs -0 chmod 0640
+ [[ RedHat == \R\e\d\H\a\t ]]
+ chkconfig solr on
+ echo 'Service solr installed.'
Service solr installed.
+ echo 'Customize Solr startup configuration in /etc/default/solr.in.sh'
Customize Solr startup configuration in /etc/default/solr.in.sh
+ [[ true == \t\r\u\e ]]
+ service solr start
Warning: Available entropy is low. As a result, use of the UUIDField, SSL, or any other features that require
RNG might not work properly. To check for the amount of available entropy, use 'cat /proc/sys/kernel/random/entropy_avail'.

Waiting up to 180 seconds to see Solr running on port 8983 [\]
Started Solr server on port 8983 (pid=44740). Happy searching!
                                                                                                                                                                                      + sleep 5
+ service solr status

Found 1 Solr nodes:

Solr process 44740 running on port 8983
{
  "solr_home":"/opt/solr-6.6.3/var/data",
  "version":"6.6.3 d1e9bbd333ea55cfa0c75d324424606e857a775b - sarowe - 2018-03-02 15:09:34",
  "startTime":"2018-04-30T13:02:45.155Z",
  "uptime":"0 days, 0 hours, 0 minutes, 8 seconds",
  "memory":"31 MB (%6.3) of 490.7 MB"
```

_____________________________________________

## TODO

​	Scripts que arranquen el servicio de solr automaticamente en caso que se reinicie el servidor.
	Proteger la ruta donde esta el servidor de solr para que nadie tenga acceso desde afuera.

