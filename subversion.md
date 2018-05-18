# Subversion

## Respaldo y Recuperación

You could use [`svnadmin dump`](http://svnbook.red-bean.com/nightly/en/svn.ref.svnadmin.c.dump.html). For example, to create a compressed backup file in Linux run:

```
svnadmin dump -q /path/to/repo | bzip2 -9 > repo_backup.bz2
```

To restore a backup use [`svnadmin load`](http://svnbook.red-bean.com/nightly/en/svn.ref.svnadmin.c.load.html):

```
svnadmin create /path/to/newrepo
bzip2 -cd repo_backup.bz2 | svnadmin load /path/to/newrepo
```

See also [Repository data migration using svnadmin](http://svnbook.red-bean.com/nightly/en/svn.reposadmin.maint.html#svn.reposadmin.maint.migrate.svnadmin) in the SVN Book.

### Agregar la lista de archivos o directorios no versionados a la working copy.

```bash
for target in (svn st | awk '/^?/ {print $2}'); do svn add target ;done
svn commit -m "Comentario util para los cambios introducidos... ;-) "
```

Excluir un directorio o archivo para ser considerado por SVN

```bash
cd /ruta/al/dir/padre
svn propset svn:ignore <directorio_hijo> .
svn propset svn:ignore <archivo_hijo> .
```

```borrar una propiedad
cd /ruta/al/dir/padre
svn propdel svn:ignore  .
```

```bash
cd /ruta/al/dir/padre
svn propedit svn:ignore  .
```

Para poder editar debemos exportar la variable de entorno SVN_EDITOR con la ruta a nuestro editor de preferencia, edtando el  .basrhrc

```
# .bashrc

# Source global definitions
if [ -f /etc/bashrc ]; then
        . /etc/bashrc
fi

# Uncomment the following line if you don't like systemctl's auto-paging feature:
# export SYSTEMD_PAGER=

# User specific aliases and functions

# Configuraciones de drupadmin
export SVN_EDITOR=/usr/bin/vi

```





