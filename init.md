# Parseando la línea de comandos e inicializando FUSE

La verdad es que tenés que hacer bastante poco parseo de parámetros de la línea de comandos: FUSE entiende una gran cantidad de parámetros de línea de comandos, y los parsea él solito. Más importante aún, FUSE espera que el punto de montaje sea uno de los componentes del `argv[]`.

Yo me simplifiqué las cosas asumiendo que el directorio raíz y el punto de montaje van a ser los últimos dos parámetros de la línea de comandos, con el directorio raíz precediendo al punto de montaje. Entonces, todo el parseo de líneas que hago se limita a dos secciones cortitas. Primero, hago un poco de chequeo de sanidad: me aseguro que tengo parámetros suficientes en la línea de comadnos, y que los últimos dos parámetros no son opciones:

```
if ((argc < 3) || (argv[argc-2][0] == '-') || (argv[argc-1][0] == '-'))
    bb_usage();
```

Algunas líneas después saco el directorio raíz de la lista de argumentos, lo traduzco a una ruta absoluta usando la función `realpath()` de la biblioteca de C, y la grabo en mis datos privados (ya hablaré de los datos privados en la próxima sección).

```
bb_data->rootdir = realpath(argv[argc-2], NULL);
argv[argc-2] = argv[argc-1];
argv[argc-1] = NULL;
argc--;
```

Lo último que hago antes de devolverle el control a FUSE es abrir el archivo de log, y guardarme su puntero también en mis datos privados.

Una vez listo para iniciar el filesystem, llamo a la función `fuse_main()`. Sus parámetros son `argc` y `argv` (modificado por mi función `main()`), la estructura `bb_oper` con los punteros a todas mis reimplementaciones de las operaciones de archivos de POSIX, y un `struct bb_data` usado para almacenar datos privados.

`fuse_main()` parsea el resto de la línea de comandos, monta el directorio especificado en la línea de comandos, y hace otras inicializaciones necesarias. Luego llama a una función de inicialización para hacer cualquier inicialización que mi código haya definido. `bb_oper->init` apunta a mi función `bb_init()`, por lo que se llama a esa función. Mi función `bb_init()` es realmente chica: todo lo que hace es loggear el hecho de que fue llamada.

Seguir leyendo: [Datos privados y temas relacionados](private.md)

Last modified: Thu Jun 12 18:13:36 MDT 2014
