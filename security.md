# Consideraciones de seguridad

Escribir y usar un filesystem FUSE puede traer enormes problemas de seguridad que pueden o no ser obvios, pero merecen ser mencionados. En esta sección voy a hablar sobre escalada de privilegios, dejar algunas notas sobre chequeo de permisos de acceso, y mentionar las condiciones de carrera.

## Escalada de privilegios

El punto más importante es que **el filesystem ejecuta con los privilegios de acceso del proceso que lo corre**, y no con los del proceso que hace uso del filesystem.  Así es más o menos como esto resulta en algunos escenarios típicos:

### El caso más común: un usuario ejecuta el filesystem sin la opción `allow_other`

Este es el caso normal: el filesystem ejecuta con los privilegios del usuario que lo ejecutó, y sólo ese usuario puede acceder al filesystem. FUSE no expone ningún problema de seguridad en este caso.

### Un usuario corre el filesystem con la opción `allow_other`

En este caso, el filesystem corre con los privilegios del usuario que lo invocó, y no con los del usuario que haga uso del filesystem. Es responsabilidad del usuario que monta el filesystem asegurarse de no estar dándole privilegios de acceso indebidos a otros usuarios. En general, los usuarios sólo pueden hacerse daño a sí mismos de este modo, dado que sólo pueden dar privilegios que ellos ya tenían.

Es importante notar que para habilitar esta opción hay que setear la opción `user_allow_other` en el archivo `/etc/fuse.conf`.

### El usuario root corre el filesystem

A decir verdad, este caso es el mismo que los dos anteriores (dependiendo si está seteada `allow_other` o no), pero `root` es un caso lo suficientemente especial como para nombrarlo aparte. En este caso, cualquier usuario del filesystem ¡va a tener privilegios de root en ese filesystem! Si el proceso filesystem tiene acceso al filesystem real, puede ser usado para obtener prácticamente acceso ilimitado.

La próxima subsección trata sobre chequear privilegios de acceso, pero la manera más simple de escapar de este problema es evitando que `root` monte el filesystem. Debido al rol intencional del BBFS como un simple tutorial, eso es lo que hago. Al inicio del `main()` hay un fragmento de código para eso:

```
if ((getuid() == 0) || (geteuid() == 0)) {
    fprintf(stderr, "Running BBFS as root opens unnacceptable security holes\n");
    return 1;
}
```

Por supuesto, esto significa que, por más que implementé `bb_chown()`, en realidad no funciona, dado que únicamente `root` tiene permisos para cambiar el dueño de un archivo en el filesystem subyacente.

## Chequeando permisos de acceso

En general, un filesystem que pueda ser ejecutado con `allow_other` tendrá que tomar precauciones para asegurar su propia seguridad. `fuse.h` documenta que varias llamadas deben verificar que el acceso requerido esté permitido. Además de esas, hay varias otras que también requieren chequeos de acceso (como `chown()`). ¡Es responsabilidad puramente del programador asegurarse de tomar esas precauciones!

La función más importante para el chequeo de accesos es `fuse_get_context()`, la que, como su nombre indica, devuelve un puntero a un `struct fuse_context`. Los campos `uid` y `gid` de ella contienen el UID y GID del proceso realizando la operación.

## Acceso concurrente y condiciones de carrera

Por defecto, FUSE ejecuta en modo multi-thread. Esto significa, básicamente, que el filesystem puede atender una petición aunque aún no haya completado otra operación previa. Esto abre la posibilidad de que distintos threads intenten modificar la misma estructura de datos en simultáneo, cosa que causa bugs muy difíciles de debuggear.

Hay un par de cosas que se pueden hacer al respecto:

* Ejecutar el filesystem con la opción `-s`, para ejecutarlo en modo single-thread. Esto elimina el problema a costas de performance. Sinceramente, dada la naturaleza e intención de muchos filesystems FUSE, me parece que el default debería ser single-thread, y multi-thread debería requerir activar una opción. Pero yo no lo programé, así que no es algo que decida.
* Analizar el código buscando secciones críticas, e insertar las primitivas comunes de sincronización (como semáforos) para evitar las condiciones de carrera. Por supuesto, hay muchos lugares en que FUSE traduce una única llamada en una secuencia de llamadas a tus funciones - pero aún no investigué si FUSE hace algo para garantizar la atomicidad de esas llamadas. Si no lo hace (y sospecho que este sea el caso, dado que intentar hacerlo sin conocer los datos que estás exponiendo a través de tu filesystem me parece entre difícil e imposible), entonces intentar hacerlo por tus propios medios me parece muy, _muy_ difícil.

Notá que, incluso si hacés que tu filesystem sea single-thread, eso no te protege contra accesos a las estructuras por otros medios. Tomando el BBFS como ejemplo:

* Podés tener un único directorio montado a través de dos puntos de montaje distintos ejecutando el `bbfs` dos veces
* Un directorio que tenga un BBFS montado sigue siendo accesible a través del filesystem común

Cualquiera de estos dos motivos alcanza para negar por completo cualquier esfuerzo que hagas en tu filesystem para garantizar la atomicidad.

Cabe notar que el código del propio FUSE tiene cuidado de lockear su propio código y estrucutras de datos. Hasta donde se, no van a ocurrir condiciones de carrera fuera de tu código.

Seguir leyendo: [Agradecimientos y otros recursos](thanks.md)

Last modified: Sat Jan 1 21:53:06 MST 2011
