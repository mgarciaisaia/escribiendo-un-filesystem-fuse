# Información extra en las funciones obscuras de FUSE

La intención de esta página es dar un poco de información extra sobre las llamadas que parecen más oscuras en la documentación de FUSE. Hasta ahora aparecieron dos casos que requieren explicaciones extra: `readdir()` y el manejo que hace FUSE de los flags de creación de archivos en `open()`.

## Directorios y `readdir()`

FUSE provee un mecanismo para registrar las entradas de la estrucutra de directorio. La estructura de directorio en sí misma es opaca, por lo que el mecanismo básico es crear los datos y llamar una función provista por FUSE para completar la estructura.

Cuando invocan tu callback de `readdir()`, uno de sus parámetros es una función llamada `filler()`. El propósito de esta función es insertar entradas de directorio en la estructura de directorio, que convenientemente también le pasan a tu función como `buf`.

El prototipo de `filler()` se ve así:

```
int fuse_fill_dir_t(void *buf, const char *name,
				const struct stat *stbuf, off_t off);
```

Para insertar una entrada en `buf` (el mismo buffer que le pasan a `readdir()`) llamando a la función `filler()` con el nombre del archivo y, opcionalmente, un puntero a un `struct stat` conteniendo su tipo de archivo.

`bb_readdir()` usa `filler()` en su forma más simple para simplemente copiar en el directorio montado los nombres de los archivos del directorio subyacente. Notá que ignoramos el offset que se le pasa a `bb_readdir()` - a `filler()` le pasamos siempre un offset 0. Esto le dice a `filler()` que maneje los offsets en la estructura de directorio él mismo. Acá está el código:

```
int bb_readdir(const char *path, void *buf, fuse_fill_dir_t filler, off_t offset,
               struct fuse_file_info *fi)
{
    int retstat = 0;
    DIR *dp;
    struct dirent *de;

    log_msg("bb_readdir(path=\"%s\", buf=0x%08x, filler=0x%08x, offset=%lld, fi=0x%08x)\n",
            path, (int) buf, (int) filler,  offset, (int) fi);

    dp = (DIR *) (uintptr_t) fi->fh;

    // Cada directorio contiene al menos dos entradas: . y ..
    // Si mi primer llamada al readdir() del sistema devuelve NULL,
    // entonces tengo un error. Hasta donde se, esa es la unica
    // condicion en que puedo obtener un error de readdir()
    de = readdir(dp);
    if (de == 0)
        return -errno;

    // Esto va a copiar todo el directorio en el buffer. El loop termina
    // cuando readdir() devuelve NULL, o filler() devuelve no-cero.
    // El primer caso significa que leimos el directorio completo; el
    // segundo significa que se lleno el buffer.
    do {
        log_msg("calling filler with name %s\n", de->d_name);
        if (filler(buf, de->d_name, NULL, 0) != 0)
            return -ENOMEM;
    } while ((de = readdir(dp)) != NULL);

    log_fi(fi);

    return retstat;
}
```

## Flags de creación de archivos

La documentación de la llamada al sistema `open()` dice que recibe tanto el modo de acceso al archivo como los flags de creación (los flags de creación son `O_CREAT`, `O_EXCL`, `O_NOCTTY` y `O_TRUNC`). `fuse.h` documenta que `O_CREAT` y `O_EXCL` no le llegan a tu función `open()`, e incluso que `O_TRUNC` no te lo manda por defecto.

La razón para esto es que FUSE maneja esos flags internamente, y cambia cuáles de tus funciones llama dependiendo de su estado y los resultados de llamar a tu `getattr()` (siempre te van a llamar `getattr()` antes de abrir el archivo). Si los flags están seteados, las llamadas se manejan así:

### `O_CREAT`

Si el archivo no existía, se llama a tu función `create()` en lugar de `open()` - si existía, entonces se llama a `open()`. Luego de llamar a tu función `create()`, llaman a tu `fgetattr()`, aunque todavía no entendí muy bien por qué. Un posible uso es que podrías usarla para modificar la semántica de crear un archivo al que no tenés acceso (la semántica estándar aplica unícamente al modo de acceso de archivo de los `open()`s subsiguientes).

Si el archivo no existía y el flag no estaba seteado, FUSE sólo llama a tu función `getattr()` (en este caso no llama ni a tu `create()` ni a tu `open()`).

### `O_EXCL`

El comportamiento de este flag sólo está definido cuando también está especificado `O_CREAT`. Si el archivo no existía, se llama a tus funciones `create()` y `fgetattr()`; si existía, FUSE devuelve un error luego de la llamada a `getattr()` (por lo que no llama ni a `open()` ni a `create()` en este caso).

### `O_NOCTTY`

Por lo que pude determinar hasta ahora, este flag es sencillamente ignorado.

### `O_TRUNC`

El valor de este flag depende de si se monta el filesystem con la opción `-o atomic_o_trunc` o no. Si no, FUSE va a llamar a tu función `truncate()` antes de llamar a `open()`. Si, en cambio, `-o atomic_o_trunc` estaba activado, la función `open()` recibe este flag `O_TRUNC`. Notá que esto significa que no tengo código que maneje este flag explícitamente - si se lo pasan a `bb_open()`, éste se lo pasa a `open()` y ya.

Seguir leyendo: [Consideraciones de seguridad y condiciones de carrera](security.md)

Last modified: Sat Jan 1 21:45:06 MST 2011
