# Configurar, compilar y ejecutar

Esta página del tutorial explica cómo buildear el programa del filesytem BBFS, y cómo usarlo para montar un directorio.

## Configurar y compilar

Este tutorial usa el sistema GNU autotools para la configuración. Como en todo proyecto que usa autotools, podés configurar y compilar el proyecto yendo al directorio raíz y tipeando:

```
./configure
make
```

Y el código debería compilarse y quedar listo para usar.

Distinto a la mayoría de los proyectos, el código de este tutorial no está pensado para ser instalado. En consecuencia, traté de deshabilitar todos los targets de instalación - si se me escapó alguno, háganmelo saber.

## Montando un filesystem BBFS

Uno monta un filesystem BBFS corriendo el commando `bbfs` (en general, los filesystems FUSE se implementan como un programa que ejecutás para montar tu FS). `bbfs` tiene dos parámetros requeridos: el directorio raíz (el que realmente contiene los datos) y el directorio de montaje. El tarball del tutorial incluye un directorio de ejemplo (`example`), que contiene dos subdirectorios (`rootdir` y `mountdir`). Podrás ver que `rootdir` tiene un único archivo llamado `bogus.txt`, mientras que `mountdir` está vacío.

Se ve algo así:

```
snowball:655$ pwd
/home/joseph/fuse-tutorial/example
snowball:656$ ls -lR
.:
total 12
-rw-r--r-- 1 joseph users  185 Jun  9 15:56 Makefile
drwxr-xr-x 2 joseph users 4096 Jun 12 17:16 mountdir/
drwxr-xr-x 2 joseph users 4096 Jun 12 17:16 rootdir/

./mountdir:
total 0

./rootdir:
total 4
-rw-r--r-- 1 joseph users 11 Jun 12 17:16 bogus.txt
```

Ahora, si vas al directorio `example` y ejecutás `../src/bbfs rootdir mountdir`, parece que todos los archivos que realmente están en `rootdir` estuvieran también en `mountdir`:

```
snowball:657$ ../src/bbfs rootdir/ mountdir/
about to call fuse_main
snowball:658$ ls -lR
.:
total 40
-rw-r--r-- 1 joseph users   185 Jun  9 15:56 Makefile
-rw-r--r-- 1 joseph users 25632 Jun 12 17:51 bbfs.log
drwxr-xr-x 2 joseph users  4096 Jun 12 17:16 mountdir/
drwxr-xr-x 2 joseph users  4096 Jun 12 17:16 rootdir/

./mountdir:
total 4
-rw-r--r-- 1 joseph users 11 Jun 12 17:16 bogus.txt

./rootdir:
total 4
-rw-r--r-- 1 joseph users 11 Jun 12 17:16 bogus.txt
```

Pero cada vez que ejecutás una operación de archivos sobre `mountdir`, la operación (y un montón de otras cosas, tanto relevantes como no) aparecen loggeadas en un nuevo archivo en el directorio actual llamado `bbfs.log`. Si ejecutás `tail -f bbfs.log` en otra terminal podés ir viendo cómo se van loggeando las operaciones.

Por último, podés ver que el sistema operativo ve a `mountdir` como otro filesystem:

```
snowball:660$ mount | grep mountdir
bbfs on /home/joseph/fuse-tutorial/example/mountdir type fuse.bbfs (rw,nosuid,nodev,relatime,user_id=1248,group_id=1005)
```

## Desmontando

Para terminar, podés desmontar el filesystem haciendo:

```
snowball:661$ fusermount -u mountdir
snowball:662$ ls -lR
.:
total 40
-rw-r--r-- 1 joseph users   185 Jun  9 15:56 Makefile
-rw-r--r-- 1 joseph users 27520 Jun 12 17:57 bbfs.log
drwxr-xr-x 2 joseph users  4096 Jun 12 17:16 mountdir/
drwxr-xr-x 2 joseph users  4096 Jun 12 17:16 rootdir/

./mountdir:
total 0

./rootdir:
total 4
-rw-r--r-- 1 joseph users 11 Jun 12 17:16 bogus.txt
```

Notá que `fusermount` no es parte de este tutorial, si no una herramienta que viene con FUSE.

## `pkg-config`

Un punto a mencionar sobre la configuración de este proyecto es la línea:

```
PKG_CHECK_MODULES(FUSE, fuse)
```

en el `configure.ac`. Esto se traduce en dos invocaciones de `pkg-config` para obtener las flags y bibliotecas de compilación de C necesarias para compilar y linkear el código de este tutorial.

```
`pkg-config fuse --cflags`
```

Esta invocación dice que se use `pkg-config` para determinar qué flags de compilación de C son necesarios para compilar un archivo fuente que usa FUSE. Las comillas invertidas alrededor del comando son importantes - indican que, en la línea de comandos, todo este comando se debe reemplazar por el resultado de su ejecución. Es importante que sean commillas invertidas ("back-quotes", o "accent graves"), y no comillas dobles, simples o ninguna otra.

El otro lugar en que se usan:

`pkg-config fuse --libs`

provee los parámetros de línea de comando necesarios para linkear el programa con la `libfuse`.

Una versión anterior de este tutorial hacía las invocaciones directamente en el `Makefile`, así:

```
bbfs : bbfs.o log.o
        gcc -g -o bbfs bbfs.o log.o `pkg-config fuse --libs`

bbfs.o : bbfs.c log.h params.h
        gcc -g -Wall `pkg-config fuse --cflags` -c bbfs.c

log.o : log.c log.h params.h
        gcc -g -Wall `pkg-config fuse --cflags` -c log.c
```

Seguir leyendo: [Callbacks y `struct fuse_operations`](callbacks.md)

Last modified: Thu Jun 12 18:12:16 MDT 2014
