# Archivos y convenciones de nombres en este tutorial

Antes de entrar en los detalles de FUSE y el BBFS, veamos cómo están organizados estos archivos.

El código de ejemplo del BBFS está en el subdirectorio [`./src`](src/).

## [`Makefile`](src/Makefile)

Como de costumbre, usamos un `Makefile` en vez de compilar el código a mano. El código es tan simple que simplemente hardcodeé un `Makefile` en lugar de usar `automake`. Requiere `pkg-config` (y FUSE, ¡obviamente!), pero todo lo demás debería estar disponible en cualquier instalación de Linux que uses para desarrollo.

## [`bbfs.c`](src/bbfs.c)

El código que re-implementa las operaciones de archivos de Linux está en este archivo. Toda función definida en este archivo empieza con `bb_`, así que es fácil distinguir mi código del del sistema.

Este archivo contiene, también, un `struct fuse_operations` llamado `bb_oper` que contiene los punteros a las funciones (discutiremos este `struct` en la próxima sección).

Las funciones apuntadas desde los campos de la estructura `struct fuse` tienen nombres derivados de los de sus campos, prefijados por el prefijo estándar `bb_`. Entonces, por ejemplo, la función `open()` es `bb_open()`.

## [`log.c`](src/log.c) and [`log.h`](src/log.h)

El código que loggea, reportando todas las operaciones que se hacen, está en `log.c`. Esos nombres comienzan todos con `log_`, otra vez, para ayudar a distinguir el código de logging. `log.h` contiene los prototipos de las funciones que se llaman desde otros lugares (donde, obviamente, el único "otro lugar" en este proyecto es `bbfs.c`).

## [`params.h`](src/params.h)

Este archivo define qué versión de la API de FUSE se usa, define un símbolo que me da acceso a la syscall `pread()`, y define un `struct bb_state` que almacena el estado del filesystem.

---

También quiero mencionar que prácticamente toda la documentación de FUSE está en el archivo para hacer `#include`: `/usr/include/fuse/fuse.h`. Ahí están los prototipos para todas las funciones, con comentarios que describen su uso. Mi `bbfs.c` empezó como una copia de ese archivo, al que le fui insertando mi código.


Seguir leyendo: [Corriendo el BBFS](running.md)

Last modified: Thu Jun 12 17:33:14 MDT 2014
