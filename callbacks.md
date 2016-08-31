# Callbacks y `struct fuse_operations`

Un filesystem FUSE es un programa que escucha en un socket peticiones de operaciones de archivo a realizar, y las realiza. La biblioteca FUSE (`libfuse`) provee toda la comunicación con el socket, y le pasa los pedidos a tu código. Para hacerlo, usa un mecanismo de "callbacks".

Los callbacks son un conjunto de funciones que vos programás para implementar las operaciones, y un `struct fuse_operations` que contiene punteros a cada una de ellas.

En el caso del BBFS, el `struct` de callbacks se llama `bb_oper`. En total existen 34 operaciones de archivo definidas en `bbfs.c` apuntadas desde `bb_oper`. La inicialización usa una sintaxis con la que no todo el mundo está familiarizado. Parte de esa inicialización se ve así:

```
 struct fuse_operations bb_oper = {
  .getattr = bb_getattr,
  .readlink = bb_readlink,
  .open = bb_open,
  .read = bb_read
};
```

Por supuesto que esta no es la estructura completa - simplemente son las necesarias para darnos una idea de qué está pasando.

Esto indica que `bb_oper.getattr` apunta a `bb_getattr()`, `bb_oper.readlink` apunta a `bb_readlink()`, `bb_oper.open` apunta a `bb_open()`, y `bb_oper.read` apunta a `bb_read()`. Cada una de estas funciones es mi reimplementación de las funciones correspondientes de filesystem: cuando un programa llama a `read()`, se termina llamando a mi función `bb_read()`. En general, lo que hacen todas mis reimplementaciones es loggear algo de información sobre la llamada, y después llamar a la implementación original de esa función en el filesystem subyacente.

Peguémosle un vistazo a dos operaciones en particular: `bb_open()` (mi reimplementación de `open()`), y `bb_read()` (mi reimplementación de `read()`).

Acá está `bb_open()`:

```
int bb_open(const char *path, struct fuse_file_info *fi)
{
    int retstat = 0;
    int fd;
    char fpath[PATH_MAX];

    bb_fullpath(fpath, path);

    log_msg("bb_open(fpath\"%s\", fi=0x%08x)\n",
	    fpath,  (int) fi);

    fd = open(fpath, fi->flags);
    if (fd < 0)
	retstat = bb_error("bb_open open");

    fi->fh = fd;
    log_fi(fi);

    return retstat;
}
```

Cuando se llama a la función, se le pasan dos parámetros: una ruta de archivo (un path) relativa a la raíz del filesystem montado, y un puntero al `struct fuse_file_info` que se usa para mantener información sobre el archivo.

`bb_open()` comienza traduciendo la ruta relativa que le pasaron a una ruta absoluta en el filesystem subyacente usando mi función `bb_fullpath()`. Después loggea el path absoluto, y la dirección del puntero `fi`. Luego hace la llamada al filesystem subyacente, y se fija si fue exitosa. Si lo fue, guarda el descriptor de archivo devuelto por `open()` para poder usarlo más adelante, y devuelve 0. Si falla, devuelve `-errno`. Sobre el valor de retorno:

1. 0 indica un éxito. Por defecto, todas las funciones en las bibliotecas se comportan así, salvo cuando se documente lo contrario.
1. Un valor negativo indica un error. Si devuelvo un valor de `-i`, quien haya llamado a la función va a recibir un `-1`, y la variable `errno` se setea en `i`. Mi función `bb_error()` se fija el valor de `errno` que seteó `open()`, loggea el error, y devuelve `-errno` para poder pasárselo al usuario.

Notar que FUSE hace algunas traducciones. La documentación de `open()` dice que devuelve un descriptor de archivos en lugar de 0 - entonces, cuando yo devuelvo 0 a FUSE, él le devuelve el descriptor de archivos apropiados a quien la haya llamado (¡que no es necesariamente el mismo que recibí en mi `open()`!). Mientras tanto, tengo el archivo subyacente abierto, y me guardé su descriptor en `fi`. Las llamadas futuras que se hagan a mi código van a recibir ese puntero, por lo que voy a poder pedirle mi descriptor para operar sobre él. Recapitulando, el programa de usuario abrió un archivo en el filesystem montado, y tiene un descriptor de ese archivo. Cuando ese programa hace cualquier operación sobre ese descriptor, el kernel intercepta la operación y se la envía al programa `bbfs`. En mi programa, yo también tengo un archivo del directorio subyaciente abierto, y un descriptor de archivos. Cuando le llegue la operación a mi programa, voy a loggearla y hacer esa misma operación en mi archivo.

Para que quede más concreto, veamos la función `bb_read()`:

```
int bb_read(const char *path, char *buf, size_t size, off_t offset, struct fuse_file_info *fi)
{
    int retstat = 0;

    log_msg("bb_read(path=\"%s\", buf=0x%08x, size=%d, offset=%lld, fi=0x%08x)\n",
	    path,  (int) buf, size,  offset,  (int) fi);

    retstat = pread(fi->fh, buf, size, offset);
    if (retstat < 0)
	retstat = bb_error("bb_read read");

    return retstat;
}
```

Esta función nos permite leer datos desde un offset especificado desde el principio de un archivo (por lo que corresponde más a `pread()` que a `read()`).

Lo más importante a destacar sobre esta función es que en ella uso mi descriptor de archivos (el que puse en `fi` cuando ejecuté `open()`) para leerlo. Además, si `pread()` retorna algo que no es un error, le paso ese valor a quien me haya llamado. En este caso FUSE no hace ninguna traducción - simplemente retorna el valor que yo le de. Para devolver un error, uso la misma técnica que en `bb_open()`.

Seguir leyendo: [Parseando la línea de comandos e inicializando FUSE](init.md)

Last modified: Thu Jun 12 18:12:31 MDT 2014
