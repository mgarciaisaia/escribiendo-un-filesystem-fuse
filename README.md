# Escribiendo un Filesystem FUSE: un tutorial

Joseph J. Pfeiffer, Jr., Ph.D.
Emeritus Professor
Department of Computer Science
New Mexico State University
pfeiffer@cs.nmsu.edu

Version of 2016-03-25

Una de las grandes contribuciones de Unix fue la visión de que "todo es un archivo". Una cantidad tremenda de objetos de tipos radicalmente diferentes, desde el almacenamiento de datos a conversiones de formatos a estructuras de datos internas del sistema operativo - todas fueron representadas a través de la abstracción archivo.

Una de las más recientes direcciones que tomó esta visión fueron los sistemas de archivos en espacio de usuario (FUSE, Filesystems in User Space - no, el acrónimo no está bueno). La idea acá es que, si podés ver la interacción con un objeto en términos de una estructura de directorios y operaciones de filesystem, podés escribir un filesystem FUSE que provea esa interacción. Simplemente escribís código que implementa operaciones de archivo como `open()`, `read()` y `write()`. Cuando montan tu sistema de archivos, los programas pueden acceder los datos usando las llamadas a sistema estándar de archivos, que llaman a tu código.

Se han escrito filesystems FUSE para hacer de todo, desde proveer acceso remoto a archivos en otra máquina sin usar NFS o CIFS (ver SSHFS en https://github.com/libfuse/sshfs) hasta implementar un filesystem para hablar con dispositivos usando el Media Transfer Protocol (ver jmtpfs en https://github.com/kiorky/jmtpfs), hasta organizar colecciones de música en directorios basados en sus tags de MP3 (ver id3fs en http://erislabs.net/ianb/projects/id3fs/id3fsd.html), hasta - bueno, cualquier cosa. ¡Las posibilidades sólo están limitadas por tu imaginación!

Hay muchos documentos en la web describiendo cómo funciona FUSE y cómo instalar y usar un filesystem FUSE, pero no me crucé ninguno que intente describir cómo _escribir_ uno. El objetivo de este tutorial es que exista ese documento.

Este tutorial presenta FUSE usando un filesystem que llamo el "Big Brother File System" (Sistema de Archivos del Gran Hermano, por eso de que "El Gran Hermano está mirando"). El filesystem simplemente delega cada operación a un directorio subyacente, pero loggeando la operación.

Este tutorial, junto con su filesystem de ejemplo asociado, está disponible en un tarball en http://www.cs.nmsu.edu/~pfeiffer/fuse-tutorial.tgz.

**Audiencia:** Este tutotial apunta a desarrolladores que tienen cierta familiaridad con la programación general en Linux (y sistemas Unix-like en general), por lo que ya deberías saber descomprimir un tarball, cómo funcionan los Makefiles, y cosas por el estilo. No voy a entrar en detalles sobre cómo hacer estas cosas - me voy a enfocar en las cosas específicas de los filesystem FUSE que necesitás saber.

No estoy afiliado al proyecto FUSE de ningún modo, excepto como usuario. Mis descripciones de la interface de FUSE, y de las técnicas para trabajar con él, son el resultado de destilar lo que leí en la documentación existente, y mi experiencia trabajando con él. Consecuentemente, cualquier error es mío - ¡y las correcciones son bienvenidas!

## Organización

Vas a encontrar dos subdirectorios bajo este:

* `src` contiene el código del BBFS filesystem
* `example` contiene un par de directorios que podés usar para explorar el BBFS

El tutorial en sí, en formato Markdown, se encuentra `index.md`. Sugiero que [clickees acá](index.md) para empezar a leerlo.


## Consultoría

Contestaré las preguntas que puedas tener respecto al BBFS o FUSE en general con gusto. Además, estoy disponible para hacer consultoría sobre FUSE o desarrollo sobre Linux o microprocesadores PIC en general. Si estás interesado, contactame en joseph@pfeifferfamily.net

## Traducción

La traducción al español (castellano argentino) de este tutorial se encuentra en <https://github.com/mgarciaisaia/escribiendo-un-filesystem-fuse>. Fue realizada por Matías García Isaía para la cátedra de Sistemas Operativos de la Universidad Tecnológica Nacional - Facultad Regional Buenos Aires.

Cualquier consulta o reporte de error será más que bien recibida en forma de issue en este repositorio, al igual que las correcciones en forma de pull request.

## License

Writing a FUSE Filesystem: a Tutorial by Joseph J. Pfeiffer, Jr., PhD is licensed under a Creative Commons Attribution-NonCommercial-ShareAlike 3.0 Unported License.

The code found in `src/bbfs.c` is derived from the function prototypes found in `/usr/include/fuse/fuse.h`, which is licensed under the LGPLv2. My code is being released under the GPLv3. See the file `src/COPYING`

Last modified: Sat Mar 26 21:25:03 MDT 2016
