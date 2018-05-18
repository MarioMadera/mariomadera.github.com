# Jugando con Docker 

Todas estas notas son tomadas sobre la base de [https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/#install-using-the-repository]

## Instalacion de Docker en Ubuntu
> **Nota**: Recordar setear *proxy* en la sesion antes de iniciar y de configurar sudo para que mantenga las 
>           variables de entorno definidas.

1. sudo apt-get update
2. sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
3. curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
4. sudo apt-key fingerprint 0EBFCD88

> OK
> root@myhost:~# apt-key fingerprint 0EBFCD88
> pub   4096R/0EBFCD88 2017-02-22
>      Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
> uid                  Docker Release (CE deb) <docker@docker.com>
> sub   4096R/F273FCD8 2017-02-22

5. sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

6. sudo apt-get update
7. sudo apt-get install docker-ce

> **Nota** en ambientes productivos se recomienda instalar una version específica 
>         y no la ultima liberada. Ver en enlace anterior por mas información.


8. root@myhost:~# systemctl status docker.service
``` 
● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
   Active: active (running) since lun 2017-12-11 14:06:42 -03; 12s ago
     Docs: https://docs.docker.com
 Main PID: 21384 (dockerd)
   CGroup: /system.slice/docker.service
           ├─21384 /usr/bin/dockerd -H fd://
           └─21393 docker-containerd -l unix:///var/run/docker/libcontainerd/docker-containerd.sock --metrics-interval=0 --start-ti

dic 11 14:06:41 myhost dockerd[21384]: time="2017-12-11T14:06:41.581396299-03:00" level=warning msg="Your kernel does not support s
dic 11 14:06:41 myhost dockerd[21384]: time="2017-12-11T14:06:41.581528846-03:00" level=warning msg="Your kernel does not support c
dic 11 14:06:41 myhost dockerd[21384]: time="2017-12-11T14:06:41.581569311-03:00" level=warning msg="Your kernel does not support c
dic 11 14:06:41 myhost dockerd[21384]: time="2017-12-11T14:06:41.582897337-03:00" level=info msg="Loading containers: start."
dic 11 14:06:42 myhost dockerd[21384]: time="2017-12-11T14:06:42.300975975-03:00" level=info msg="Default bridge (docker0) is assig
dic 11 14:06:42 myhost dockerd[21384]: time="2017-12-11T14:06:42.599447028-03:00" level=info msg="Loading containers: done."
dic 11 14:06:42 myhost dockerd[21384]: time="2017-12-11T14:06:42.631223287-03:00" level=info msg="Docker daemon" commit=19e2cf6 gra
dic 11 14:06:42 myhost dockerd[21384]: time="2017-12-11T14:06:42.631468676-03:00" level=info msg="Daemon has completed initializati
dic 11 14:06:42 myhost systemd[1]: Started Docker Application Container Engine.
dic 11 14:06:42 myhost dockerd[21384]: time="2017-12-11T14:06:42.684906617-03:00" level=info msg="API listen on /var/run/docker.soc
```

9. root@myhost:~# apt-get install docker-compose

## Pruebas

### sudo docker run hello-world

Falla con este mensaje:
```
root@myhost:~# docker run hello-world
Unable to find image 'hello-world:latest' locally
docker: Error response from daemon: Get https://registry-1.docker.io/v2/: dial tcp: lookup registry-1.docker.io on X.Y.55.1:53: read udp X.Y.67.235:58239->X.Y.55.1:53: i/o timeout.
See 'docker run --help'.
```

Docker no tiene salida a internet, encuentro como agregar el proxy en https://stackoverflow.com/questions/26550360/docker-ubuntu-behind-proxy

> Ubuntu 16.04 LTS
> For Ubuntu 16.04 LTS who uses Systemd, you can follow this post:
>
> (1) Create a systemd drop-in directory:
> mkdir /etc/systemd/system/docker.service.d
> (2) Add proxy in /etc/systemd/system/docker.service.d/http-proxy.conf file:
>
> ```
> vi /etc/systemd/system/docker.service.d/http-proxy.conf
> [Service]
> Environment="HTTP_PROXY=http://X.Y.67.48:3128/"
> Environment="HTTPS_PROXY=http://X.Y.67.48:3128/"
> Environment="NO_PROXY=localhost,127.0.0.1,localaddress,.localdomain.com,X.Y*,192.168.*,*.mi.dominio.com"
> ```
> (3) Flush changes:
>
> systemctl daemon-reload
> (4) Restart Docker:
>
> systemctl restart docker


Volvemos a probar
```
root@myhost:~# docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
ca4f61b1923c: Pull complete
Digest: sha256:be0cd392e45be79ffeffa6b05338b98ebb16c87b255f48e297ec7f98e123905c
Status: Downloaded newer image for hello-world:latest

Hello from Docker! 
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
1. The Docker client contacted the Docker daemon.
2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
   (amd64)
3. The Docker daemon created a new container from that image which runs the
   executable that produces the output you are currently reading.
4. The Docker daemon streamed that output to the Docker client, which sent it
   to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
$ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
https://cloud.docker.com/

For more examples and ideas, visit:
https://docs.docker.com/engine/userguide/
```

### Manejo de permisos
Ejecutando como usuario no previlegiado mediante sudo, no hay problema.
Intentaré agregar un usuario al grupo docker y ejecutqar el hello-world

`sudo usermod -a -G docker sopaplic1`

reiniciar sesión como el usuario modificado

```
sopaplic1@myhost:~$ docker run hello-world
 
 Hello from Docker!
 This message shows that your installation appears to be working correctly.
 
 To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
$ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
https://cloud.docker.com/

For more examples and ideas, visit:
https://docs.docker.com/engine/userguide/
```

## Crear una imagen a medida

Cree una carpeta docker-projects y una dentro de ella para nginx.
Luego genere directorios adicionales para config, logs y contenido.
content es un enlace a /var/www/kb (kb de prueba)

Genero un dockerfile usando alpine 3.5 como base
```
FROM alpine:3.5                                    
MAINTAINER Mario Madera <mmadera@localdomain>  
 
# Defino las variables de entorno para el proxy
ENV http_proxy=http://X.Y.67.48:3128
ENV https_proxy=http://X.Y.67.48:3128
ENV no_proxy=localhost,127.0.0.1
ENV HTTP_PROXY=http://X.Y.67.48:3128
ENV HTTPS_PROXY=http://X.Y.67.48:3128
ENV NO_PROXY=localhost,127.0.0.1
 
# Instalacion de paquetes necesarios
RUN apk update
RUN apk add bash
RUN apk add nginx
RUN apk add curl
RUN rm -rf  /var/cache/apk/*                                
# Creacion de usuario y ruta base,
# en /wwww montaremos el directorio compartido
# con el host
RUN adduser -D -g 'www' www
RUN mkdir /www
RUN chown -R www:www /var/lib/nginx
RUN chown -R www:www /www                                   
# Cargo el archivo de configuracion
RUN mv /etc/nginx/nginx.conf /etc/nginx/nginx.conf.orig     
COPY ./config/nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]                          
```

El comando copy carga el archivo de configuracion de nginx al contendor
``` 
user                            www;
worker_processes                1;

error_log                       /var/log/nginx/error.log warn;
pid                             /var/run/nginx.pid;

events {
    worker_connections          1024;
    }

http {
    include                     /etc/nginx/mime.types;
    default_type                application/octet-stream;
    sendfile                    on;
    access_log                  /var/log/nginx/access.log;
    keepalive_timeout           3000;
    server {
        listen                  80;
        root                    /var/www;
        index                   index.html index.htm;
        server_name             localhost;
        client_max_body_size    32m;
        error_page              500 502 503 504  /50x.html;
        location = /50x.html {
              root              /var/lib/nginx/html;
        }
    }
}
```
### Construyo la imagen
    docker build -t ute:nginx .

### Creo un contendor con esa imagen, monto una rutal local al host y nateo el puerto
    docker run -d -p 127.0.0.1:8080:80 --name nginx_01 --volume /var/www/kb:/var/www ute:nginx    

### Para bajar el contenedor
    docker stop nginx_01

### Estadísticas de los docker 
    docker stats nginx_01    

### Para borrar el contendor
    docker rm nginx_01

### Acceder a la linea de comando dentro del contenedor
    docker exec -it nginx_01 bash

### Donde quedan las imágenes de docker?
en /var/lib/docker

### Comandos
Listar Imagenes -- docker ls

[Index](./README.md)
