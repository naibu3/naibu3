---
title: Cifrado César y afín
published: true
---


# Introduccion

El *cifrado cesar*, o *cifrado por desplazamiento*, es una de las técnicas de cifrado más simples y por tanto más usadas. Toma su nombre del archiconocido emperador romano, que lo empleaba como medida de seguridad al comunicarse con sus generales. 

En cuanto al funcionamiento, se basa en cambiar el alfabeto por uno desplazado n posiciones. De esta forma, HOLA desplazado dos posiciones pasaria a ser JQNC. Este cifrado suele formar parte de cifrados más complejos como el Vigenère o el ROT13. Sin embargo, por sí solo, no ofrece gran seguridad debido a su sencillez, lo que lo hace vulnerable a ataques de fuerza bruta.


## Aritmetica modular

El algoritmo de este cifrado puede ser expresado matemáticamente mediante aritmética modular[^1], empleando en lugar de letras su posición en el alfabeto (A=0, B=1, C=2,...).

<!--Referencias-->
[^1]En matemática, la aritmética modular es un sistema aritmético para clases de equivalencia de números enteros llamadas clases de congruencia. Algunas veces se le llama, sugerentemente, aritmética del reloj, ya que los números «dan la vuelta» tras alcanzar cierto valor llamado módulo.


### Cifrado

La expresión para el cifrado sería:

	E(x) = x + n mod m

Donde x es la posicion de la letra, n el número de posiciones y m el tamaño del alfabeto.

##### Ejemplo:

Como ejemplo, encriptaremos la palabra "*HOLA*" con un desplazamiento de 4. Al cifrar el alfabeto quedaría:

| 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 | 16 | 17 | 18 | 19 | 20 | 21 | 22 | 23 | 24 | 25 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q | R | S | T | U | V | W | X | Y | Z |
| E | F | G | H | I | J | K | L | M | N | O | P | Q | R | S | T | U | V | W | X | Y | Z | A | B | C | D |

De modo que la palabra quedaría: "*LSPE*". O haciéndolo todo letra a letra:

	E("H") = E(7) = 7 + 4 = 11 = "L"
	E("O") = E(14) = 14 + 4 = 18 = "S"
	E("L") = E(11) = 11 + 4 = 15 = "P"
	E("A") = E(0) = 0 + 4 = 4 = "E"


### Descifrado

Para descifrar el código basta con realizar la operación inversa, es decir retroceder *n* posiciones. La expresión para el descifrado sería:

	D(x) = x - n mod m

Donde x es la posicion de la letra cifrada, n el número de posiciones y m el tamaño del alfabeto.

##### Ejemplo:

Como ejemplo descifraremos "*LSPE*", la palabra del ejemplo anterior, a la que aplicamos un desplazamiento de 4. Para ello simplemente aplicaríamos el procedimiento a la inversa, bien mirando la equivalencia entre alfabeto cifrado y descifrado, o matemáticamente:

	D("L") = D(11) = 11 - 4 = 7 = "H"
	D("S") = D(18) = 18 - 4 = 14 = "O"
	D("P") = D(15) = 15 - 4 = 11 = "L"
	D("E") = D(4) = 4 - 4 = 0 = "A"

De esta forma ya tendríamos el mensaje descifrado. Sin embargo, ya disponíamos el desplazamiento aplicado, pero, en caso de no tenerlo, aún sería posible descifrarlo por fuerza bruta, es decir, probando todas los posibles desplazamientos.


# Cifrado afín

El cifrado cesar, pese a ser uno de los más conocidos, es una generalización del cifrado afín. Este, es un cifrado por sustitución, que cambia cada letra por otra según una función matemática. Otorga mayor seguridad al ser más complejo, pero aún es posible descifrarlo fácilmente con fuerza bruta, método que se analizará en la [segunda parte del post](https://naibu3.github.io/Desifrando-afin-python).


### Cifrado

La fórmula aplicando la aritmética binaria para el cifrado sería:

	E(x) = (a * x + b) mod m

Donde x es el índice de cada letra en el alfabeto, a y b, las constantes de decimación y desplazamiento; y m el número de elementos del alfabeto.

Además, deben cumplirse ciertos requisitos:

- __a__ debe ser prima entre sí con el número de elementos (que el máximo común divisor sea 1).
- __a__ debe ser distinta de 0.
- __b__ debe estar entre 0 y m, ambos inclusive.

##### Ejemplo

Para ilustrar el método encriptemos la palabra `"HOLA"` según el siguiente alfabeto:

	[A, B, C, D, E, F, G, H, I, J, K, L, M, N, O, P, Q, R, S, T, U, V, W, X, Y, Z]

Donde `A=0` y `Z=25`. Además emplearemos una `a = 19` y `b = 25` que cumplen las condiciones previamente mencionadas. Aplicando la fórmula al alfabeto, quedaría una equivalecia como esta:

| 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 | 16 | 17 | 18 | 19 | 20 | 21 | 22 | 23 | 24 | 25 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q | R | S | T | U | V | W | X | Y | Z |
| Z | S | L | E | X | Q | J | C | V | O | H | A | T | M | F | Y | R | K | D | W | P | I | B | U | N | G |

Y ya sólo quedaría sustituir según la tabla. O bien se puede calcular individualmente para cada una de las letras:

	E("H") = E(7) = (19 * 7 + 25) mod 26 = 2 = "C"
	E("O") = E(14) = (19 * 14 + 25) mod 26 = 5 = "F"
	E("L") = E(11) = (19 * 11 + 25) mod 26 = 0 = "A"
	E("A") = E(0) = (19 * 0 + 25) mod 26 = 25 = "Y"

| texto claro | H | O | L | A |
|-------------|---|---|---|---|
| indice del alfabeto normal | 7 | 14 | 11 | 0 |
| indice del alfabeto cifrado | 2 | 5 | 0 | 25 |
| texto cifrado | C | F | A | Y |


### Descifrado

La fórmula aplicando la aritmética binaria para el cifrado sería:

	D(x) = (a⁻¹ * (x - b)) mod m

Donde x es el índice de cada letra en el alfabeto, a⁻¹ la inversa multiplicativa de a, b la constante de desplazamiento y m el número de elementos del alfabeto.

Para descifrarlo al igual que con el *cifrado césar* deberemos aplicar la fórmula inversa. Para aplicarla, debemos encontrar la inversa multiplicativa de *a*, por algún método como el *algoritmo de Euclides extendido*. Una vez hayada podremos sustituir en la expresión para cada una de las letras, siempre y cuando conozcamos también *a* y *b*, en caso contrario será necesario recurrir a un análisis de frecuencias o a fuerza bruta.

##### EJEMPLO

Como ejemplo, descifraremos la palabra "*CFAY*", que ciframos en el ejemplo anterior. Como ya conocemos *a* y *b*, solo será necesario aplicar la expresión antes mencionada:

	a⁻¹= 11 #Calculado previamente con python3 (pow(a, -1, m))

	D("C") = D(2) = (11 * (2 - 25)) mod 26 = 7 = "H"
	D("F") = D(5) = (11 * (5 - 25)) mod 26 = 14 = "O"
	D("A") = D(0) = (11 * (0 - 25)) mod 26 = 11 = "L"
	D("Y") = D(25) = (11 * (25 - 25)) mod 26 = 0 = "A"

Así quedaría descifrado el mensaje. En caso de querer una explicación de cómo implementar una función para decodificar en python3, puedes echarle un ojo a mi [siguiente post](https://naibu3.github.io/Desifrando-afin-python).

# Fuentes
- [Wikipedia-Cifrado Cesar](https://es.wikipedia.org/wiki/Cifrado_C%C3%A9sar)
- [Wikipedia-Cifrado Afin](https://es.wikipedia.org/wiki/Cifrado_af%C3%ADn)
- [Wikipedia-Aritmetica Modular](https://es.wikipedia.org/wiki/Aritm%C3%A9tica_modular)
- [La güeb de Juaquín-La cifra afín](http://joaquin.medina.name/web2008/documentos/informatica/documentacion/seguridad/criptografia/CifraAfin/2013_08_22_LaCifraAfin.html)
-[DelfStack-Inverse-mod-python](https://www.delftstack.com/es/howto/python/mod-inverse-python/)
