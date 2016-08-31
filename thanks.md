# Agradecimientos

Muchos lectores con buen ojo encontraron errores en este tutorial, y algunos incluso mandaron las correcciones. ¡Gracias!

| Quién | Por qué |
|------|--------|
| Neeraj Rawat | El código no era portable a FreeBSD. Gracias a Neeraj cambié a GNU autotools e hice varios cambios para ser compatible con FreeBSD |
| David Terret | `bb_readlink()` estaba devolviendo la cantidad de bytes leídos en lugar de 0 |
| Patrick Ulam | `bb_release()` no estaba cerrando el archivo |
| Patrick Ulam y Bernardo F Costa | `bb_readlink()` no estaba terminando el string en `NULL`. ¡Patrick y Bernardo me mandaron los parches! El código actual de `bb_readlink()` es de Bernardo. |
| Bernardo F Costa | El parseo de la línea de comandos no estaba manejando el `-o` correctamente |
| Bernardo F Costa | Me hizo pensar en problemas de seguridad |
| Diogo Molo y Sivan Tal | El orden de los parámetros del Makefile era incorrecto y causaba errores de linkeo a algunos (mas no todos) usuarios |
| Skippy VonDrake | Me hizo notar que el código de parseo de línea de comandos que usaba era confuso, por lo que terminé reescribiéndolo |

# Otros recursos

* [Homepage de FUSE](http://fuse.sourceforge.net/) (en inglés)
* [Desarrolla tu propio filesystem FUSE](http://www.ibm.com/developerworks/linux/library/l-fuse/), por Sumit Singh, en IBM Developerworks (en inglés)
* [Documentación de FUSE en OpenSolaris](http://hub.opensolaris.org/bin/view/Project+fuse/Documentation) (en inglés)

Last modified: Thu Jun 12 18:49:24 MDT 2014
