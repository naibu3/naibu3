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

   Devolverá un puntero a la estructura *FILE* asociada al archivo, en caso de error, 
el puntero será null, por lo que se suele llamar de la siguiente manera:

```c
	if( (FILE* ptr=fopen("nombreFich.txt", "modo"))==NULL ){
		
		printf("[!]ERROR AL ABRIR EL ARCHIVO\n");
		exit(-1);
	}
```

   Se pasará como parámetro el nombre del archivo y el modo, que será otro 
string que dirá si se puede leer, escribir,...etc, según la siguiente tabla:

| Modo | Accion | Posicion | Si existe | Si no existe|
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

   La función recibe un puntero a *FILE* creado con `fopen`, así interrumpirá la 
conexión creada y se vaciará el buffer, __es importante hacerlo de esta forma para cerrar el flujo de datos y evitar que queden cabeceras incompletas que corrompan el archivo__. Devuelve cero en caso de cerrarse bien o EOF (-1) si hay algún problema.

###### E/S con formato

```c
	int fprintf (FILE* f, const char* formato, ...);
```

Escribe en el archivo *f* la cadena *formato* con el formato indicado. Dvuelve el número de carácteres escritos o un valor negativo si se produce un error.

```c
	int fscanf (FILE *f, const char *formato, ...);
```

Lee datos del fichero *f* con la conversión establecida en *formato*. Devuelve EOF si alcanza el final del fichero o se produce algún error, en caso contrario devuelve el número de objetos convertidos.

###### E/S de líneas

```c
	char* fgets (char* s, int n, FILE* f);
```

Lee como mucho n-1 caracteres del fichero referenciado por *f* y los copia en la cadena *s*, parando cuando encuentra '\n'. Devuelve *s* o NULL si alcanza el EOF o sucede un error.

```c
	int fputs (const char* s, FILE* f);
```

Escribe la cadena s en el fichero referenciado por f. Devuelve un valor no negativo si tiene éxito ó EOF si ocurre un error de escritura.

###### E/S de carácteres

```c
	int fgetc (FILE *f); 
	int getc (FILE *f);
```

Devuelven el siguiente carácter del fichero referenciado por *f*, avanza el cursor una posición hasta el siguiente carácter. Devuelven EOF si encuentra el fin del fichero o hay error.

```c
	int fputc (intc, FILE *f); int putc (int c, FILE *f);
```

Escribe el carácter  en el fichero referenciado porfy lo devuelve, si hay error devuelve EOF.

###### Otras funciones

```c
	int remove (const char* nombre);
```

Borra el fichero llamado *nombre*, devuelve un valor distinto de cero en caso de error.

```c
	int rename (const char* viejo, const char* nuevo);
```

Cambia el nombre del fichero llamado *viejo* por el de *nuevo*, devuelve un valor distinto de cero en caso de error.

```c
	int fflush (FILE* f);
```

Fuerza a que el fichero *f* se libere: se vacía el buffer asociado al fichero de salida. El efecto está indefinido para ficheros de entrada (por lo que no se recomienda emplear para *stdin*). Devuelve EOF sihay algún error de escritura y cero en otro caso.

`fflush(NULL)` -> Libera todos los ficheros de salida.
`fflush(stdout)` -> Libera el buffer del dispositivo de salida.


## Ficheros binarios

Los ficheros binarios guardan la información, como su nombre indica, en formato binario, como una copia de la memoria. De ésta forma, estará dividido en *registros* por lo qu no hará falta emplear separadores, pero no será legible por humanos. Un fichero binario se compone entre otras cosas de:

- **Registro activo**, aquel que está siendo procesado.
- **Cursor**, marca interna que apunta al registro activo, se incrementa cada vez que se procesa un registro (se lee o escribe).
- **Clave**, valor (o un conjunto de valores) de un registro que lo identifica unívocamente. Es diferente para cada uno de los registros.

###### Apertura

   La aperturase realizará mediante la función:

```c
	FILE* fopen (const char* nombre, const char* modo);
```

   Devolverá un puntero a la estructura *FILE* asociada al archivo, en caso de error, 
el puntero será null, por lo que se suele llamar de la siguiente manera:

```c
	if( (FILE* ptr=fopen("nombreFich", "modo"))==NULL ){
		
		printf("[!]ERROR AL ABRIR EL ARCHIVO BINARIO\n");
		exit(-1);
	}
```

   Se pasará como parámetro el nombre del archivo y el modo, que será otro 
string que dirá si se puede leer, escribir,...etc, según la siguiente tabla:

| Modo | Accion | Posicion | Si existe | Si no existe|
|---|---|---|---|---|
| rb | Lectura | Inicio | Abre | Error |
| wb | Escritura | Inicio | Borra | Crea |
| ab | Adición | Final | Abre, Agrega al final | Crea |
| r+b | L/E | Inicio | Abre, agrega al princ. sobreescr. | Error |
| w+b | L/E | Inicio | Borra | Crea |
| a+b | L/Adición | Final | Abre, Agrega al final | Crea |

###### E/S en ficheros binarios

```c
	size_t fread(void* ptr, size_t tam, size_t num, FILE* f);
```

Lee un máximo de *num* objetos de tamaño *tam* del fichero referenciado por *f* y los deja en el buffer referenciado por *ptr*. Devuelve el número de objetos leídos, que puede ser menor que *num* si hay algún error o se alcanza el fin del fichero. El cursor del fichero avanza los bytes leídos.

```c
	size_t fwrite(const void* ptr,size_t tam,size_t num, FILE* f);
```

Escribe en el fichero referenciado por *f* un máximo de *num* objetos de tamaño *tam* tomados del buffer referenciado por *ptr*. Devuelve el número de objetos escritos, que puede ser menor que *num* si hay algún error. El cursor del fichero avanza los bytes escritos.

###### Funciones de posicionamiento

```c
	long ftell (FILE* f);
```

Devuelve el número de bytes desde el principio hasta la posición actual del cursor, –1L si hay error.

```c
	int fseek (FILE* f, long desp, int origen);
```

Devuelve cierto(=! 0) si se produce algún error. Establece la posición actual del cursor. La nueva posición se establece como un desplazamiento de *desp* bytes a partir de *origen*, que puede ser:

- *SEEK_SET* (0): inicio del fichero.
- *SEEK_CUR* (1): posición actual del cursor.
- *SEEK_END* (2): fin del fichero.

