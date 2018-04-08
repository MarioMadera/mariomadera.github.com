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