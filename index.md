# Organización

El tutorial se divide en las siguientes secciones:

## [Archivos y convenciones de nombres en este tutorial](files.md)
Esta sección describe los archivos distribuídos como parte de este tutorial, y las convenciones de nombres que usan las funciones del BBFS. Está más que nada para proveer una visión global del tutorial y del filesystem, más que para dar información específica de FUSE.

## [Corriendo el BBFS](running.md)

Un poco de información sobre cómo montar un BBFS, revisar los logs y desmontarlo.

## [Callbacks y `struct fuse_operations`](callbacks.md)

Este es el corazón de un filesystem FUSE, y de este tutorial. Acá se abarcan los conceptos centrales.

## [Parseando la línea de comandos e inicializando FUSE](init.md)

Arrancando tu programa.

## [Datos privados y temas relacionados](private.md)

Manteniendo estado en el filesystem.

## [Información extra en las funciones obscuras de FUSE](unclear.md)

La intención de esta sección es dar algo de información extra sobre algunas funciones de FUSE que no quedan del todo claras. De momento, esta sección cubre `readdir()` y el manejo que hace FUSE de los flags de creación de archivos en `open()`.

## [Consideraciones de seguridad y condiciones de carrera](security.md)

Ejecutar un filesystem FUSE puede acarrear algunos temas de seguridad de los que deberías estar al tanto. Además, dado que el servidor de FUSE es multihilo, pueden haber algunas condiciones de carrera a tener en cuenta.

## [Agradecimientos y otros recursos](thanks.md)

Agradecimientos a quienes colaboraron encontrando errores, o de otro modo. Además, este no es el único recurso de FUSE en la web.

Seguir leyendo: [Files and Naming Conventions in This Tutorial](files.md)

Last modified: Thu Jun 12 18:15:04 MDT 2014
