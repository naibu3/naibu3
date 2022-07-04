---
title: Eliminar salto de linea en C
published: true
---

# Procedimiento

Al leer strings desde la terminal con `fgets()`, leemos también el salto de línea, por lo que debemos sustituirlo por `\0`. Para ello utilizaremos una función definida en `<string.h>`, `strcspn()`. Dicha función toma dos parámetros:

```c
	size_t strcspn(const char *s, const char *reject)
```

La cadena `s` será la cadena (buffer) leída por `fgets()`, y `reject` la cadena que buscará la función. En nuestro caso, esta última valdrá `"\n"`. Quedaría:

```c
	buf[strcspn(buf, "\n")]='\0';
```

Donde `buf` es la cadena devuelta por `fgets()`. De forma que buscaremos en `buf` la posición del salto de línea e intercambiaremos dicho carácter por el carácter `'\0'`.