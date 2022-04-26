---
title: Colores en terminal C
published: true
---

# Coloreando la salida en C

Para inaugurar el blog, comenzaré con un pequeño post sobre cómo añadir color a
las salidas de vuestros programas. Además, yo guardé todo en un archivo de cabecra `.h`,
y lo guardé en la carpeta `/include` para incluirlo más facilmente.


## Cómo funciona

Para cambiar el color del texto, debemos incluir al principio una cadena especial,
que C no imprime pero si toma en cuenta. A continuación dejo el listado:

```c
#define RES_COL "\x1b[0m"
#define NEGRO_T "\x1b[30m"
#define NEGRO_F "\x1b[40m"
#define ROJO_T "\x1b[31m"
#define ROJO_F "\x1b[41m"
#define VERDE_T "\x1b[32m"
#define VERDE_F "\x1b[42m"
#define AMARILLO_T "\x1b[33m"
#define AMARILLO_F "\x1b[43m"
#define AZUL_T "\x1b[34m"
#define AZUL_F "\x1b[44m"
#define MAGENTA_T "\x1b[35m"
#define MAGENTA_F "\x1b[45m"
#define CYAN_T "\x1b[36m"
#define CYAN_F "\x1b[46m"
#define BLANCO_T "\x1b[37m"
#define BLANCO_F "\x1b[47m"
```
>Los defino como macros para facilitar el uso.


De los dos números que se observan, definen si es el color de texto o fondo, o el color, respectivamente. Además, he definido RES_COL, la cadena que resetea el color del texto a aquel
por defecto. Esta cadena debe incluirse siempre al acabar.

## Ejemplo de uso

Para imprimir con `printf()`:

```c
printf("%sEste texto de mostrará azul%s\n", AZUL_T, RES_COL);
```

Tenéis el archivo en mi [github](https://github.com/naibu3/clibs).