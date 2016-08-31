# Datos privados

Otro dato súper útil sobre el uso de FUSE es cómo te permite almacenar y acceder datos definidos por vos.

FUSE tiene un `struct fuse_context` que contiene un poco de información extra sobre el filesystem. Un campo particularmente útil en este `struct` es un `void *private_data` - un puntero a información arbitraria almacenada por tu filesystem.

Desde dentro de cualquier operación FUSE vas a poder obtener el contexto llamando a `fuse_get_context()` - esto significa que podemos usar `fuse_get_context()->private_data` para obtener los datos privados.

Podemos ver cómo usar esto viendo mi función `log_msg()`. Es esta:

```
void log_msg(const char *format, ...)
{
    va_list ap;
    va_start(ap, format);

    vfprintf(BB_DATA->logfile, format, ap);
}
```

Usa una macro que definí así:

```
#define BB_DATA ((struct bb_state *) fuse_get_context()->private_data)
```

para obtener el campo de datos privados y castearlo a un `struct bb_state *`.

Podés ver cómo creé el `struct bb_struct` mirando el `params.h`, en el que definí la estructura:

```
struct bb_state {
    FILE *logfile;
    char *rootdir;
};
```

Tiene dos campos: el `FILE *` para el archivo de log, y el path al directorio que estamos accediendo a través del BBFS.

Yo `malloc()`eo la estructura y le seteo los valores de los campos en mi función `main()` (mirá la última sección para más info), así los tengo disponibles más adelante.

Seguir leyendo: [Información extra en las funciones obscuras de FUSE](unclear.md)

Last modified: Thu Jun 12 18:14:08 MDT 2014
