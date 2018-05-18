# Documentando lo aprendido...

Este sitio tiene la finalidad de documentar las pequeñas lecciones aprendidas mientras investigo, trabajo o simplemente uso diversos sistemas y aplicativos. Espero que compartir ayude a otros.

* [Apache](./Apache.md)

* **Cent OS 7**

  - [Centos 7 in VirtualBox](./CentOS7-VirtualBox.md)
  - [CentOS 7 VM para desarrollo Drupal 8](./Drupal8-CentOS7-Virtualbox.md)
  - [YUM](./yum.md)

* **Docker**

  * [Docker](./docker.md)
  * [Docker en Windows 7](./dockerOnWindows7.md)

* **Drupal **

  * [Drupal 7](./Drupal7.md)
  * [DRUPAL 8](./Drupal8.md)

* **GNU/Linux**

  * Piques y Trucos 
  * ​

  ​


```
root@nsgcsoptec:/etc/apt# apt update
Des:1 http://cdn-fastly.deb.debian.org/debian buster InRelease [143 kB]
Des:2 http://cdn-fastly.deb.debian.org/debian buster-updates InRelease [46,0 kB]
Des:3 http://cdn-fastly.deb.debian.org/debian-security buster/updates InRelease [38,3 kB]
Leyendo lista de paquetes... Hecho
E: Release file for http://cdn-fastly.deb.debian.org/debian/dists/buster/InRelease is not valid yet (invalid for another 5d 0h 5min 7s). Updates for this repository will not be applied.
E: Release file for http://cdn-fastly.deb.debian.org/debian/dists/buster-updates/InRelease is not valid yet (invalid for another 5d 0h 5min 6s). Updates for this repository will not be applied.
E: Release file for http://cdn-fastly.deb.debian.org/debian-security/dists/buster/updates/InRelease is not valid yet (invalid for another 4d 5h 24min 2s). Updates for this repository will not be applied.


problema en la fecha del equipo. Me di cuenta al verificar que podía navegar los repos y ver la fechas establecidas en los InRelease 
```

