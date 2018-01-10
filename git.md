# GIT

## Atrás de un proxy

```
git config --global http.proxy http://proxyuser:proxypwd@proxy.server.com:8080
```

## Comandos Básicos

Siempre en la raíz de la copia de trabajo.

**Clonar un repositorio**
```
git clone <url>
```

**Bajo los cambios de la rama master remota**
```
git pull 
```


**Agregar un archivo al versionado**
```
git add <filename>
```

**Realiza un commit de los cambios al brunch master local**
```
git commit -m 'comentario'
```

**Muevo los cambios de la rama master local a la rama origen remota **
_Normalmente la rama origen remota es la master remota._
```
git push master origin
```

