---
title: Apuntes ficheros en C
published: true
---

# Ficheros

   En C, el manejo de ficheros se realiza mediante ciertas funciones predefinidas que 
toman como parámetro un tipo especial de estructura, `FILE(stdio.h)`. Esta estructura contiene elementos como un *buffer* para almacenar la E/S, o un *cursor* que guarda la posición de lectura/escritura en el archivo. Esta estructura puede variar entre sistemas operativos, por lo que de lo único de lo que hay que preocuparse es de asignar un puntero de tipo `FILE*` que se pasará de parámetro a las funciones. En casó de algún error será de tipo `NULL`.


## Ficheros de texto

   Los ficheros de texto almacenan la información en texto plano, de forma que es 
entendible por las personas. Para guardar información en ellos, se necesita de *separadores*, que dividan los bloques de información.

###### Apertura

   La aperturase realizará mediante la función:

```c
	FILE* fopen (const char* nombre, const char* modo);
```

   Devolverá un puntero a la estructura `FILE` asociada al archivo, en caso de error, 
el puntero será null, por lo que se suele llamar de la siguiente manera:

```c
	if( (FILE* ptr=fopen("nombreFich.txt", "modo"))==NULL ){
		
		printf("[!]ERROR AL ABRIR EL ARCHIVO\n");
		exit(-1);
	}
```

   Se pasará como parámetro el nombre del archivo y el modo, que será otro 
string que dirá si se puede leer, escribir,...etc, según la siguiente tabla:

| Modo | Accion | Si existe | Si no existe|
|---|---|---|---|---|
| r | Lectura | Inicio | Abre | Error |
| w | Escritura | Inicio | Borra | Crea |
| a | Adición | Final | Abre, Agrega al final | Crea |
| r+ | L/E | Inicio | Abre, agrega al princ. sobreescr. | Error |
| w+ | L/E | Inicio | Borra | Crea |
| a+ | L/Adición | Final | Abre, Agrega al final | Crea |

###### Cierre

   El cierre se efectuará mediante la función:

```c
	int fclose (FILE* f);
```

   La función recibe un puntero a `FILE` creado con `fopen`, así interrumpirá la 
conexión creada y se vaciará el buffer, __es importante hacerlo de esta forma para cerrar el flujo de datos y evitar que queden cabeceras incompletas que corrompan el archivo__. Devuelve cero en caso de cerrarse bien o EOF (-1) si hay algún problema.