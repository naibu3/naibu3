---
title: Descifrando afín
published: true
---

# Introducción

En este post trataremos la resolución de un ejercicio para un trabajo de la asignatura de Matemática Discreta del grado de Ingeniería Informática. El enunciado pide lo siguiente:

*Suponiendo que tenemos un alfabeto como el indicado en la pregunta 1 (indicado a continuación), utilizar unsistema de cifrado César adecuado para descifrar el siguiente texto: FDEXDXZTFDTPLCZDPXKWX*

Así, tenemos como datos el alfabeto `[A, B, C, D, E, F, G, H, I, J, K, L, M, N, O, P, Q, R, S, T, U, V, W, X, Y, Z]` y la expresión codificada `FDEXDXZTFDTPLCZDPXKWX`. Además, el profesor nos dio la pista de que se trata de [cifrado afín](https://naibu3.github.io/Cifrado-Cesar-y-afin).

Para resolverlo utilizaremos python, en su versión 3.8, ya que permite un manejo cómodo de listas y posee una sintaxis sencilla. La metodología será un ataque de fuerza bruta, con la que intentaremos *romper* la contraseña probando todas las combinaciones posibles. Ya definido el problema, vamos con el código.


# Código

Empezaremos con la cabecera clásica y el tipo de encoding, además importaremos la función del máximo común divisor de la librería matemática para usarla más adelante.

```python
#!/usr/bin/python3
#_*_utf-8_*_

from math import gcd

```

A continuación definiremos tres variables con los datos del problema para utilizarlos más cómodamente.

```python
n_elem=26

abcedario=['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M',
	 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z']

codigo="FDEXDXZTFDTPLCZDPXKWX"
```

A continuación, implementaremos la __función de descifrado__. En el post anterior explicamos como se cifra y descifra, así como las fórmulas y procedimientos. En concreto, la función que descifra sería 
`D(x) = a⁻¹*(x-b) mod m`, donde x es el índice de cada letra en el alfabeto, a y b, las constantes de decimación y desplazamiento; y m el número de elementos del alfabeto.

```python
def descifrar(code, abc, elem, a, b):

	#Dclaramos una lista para guardar el resultado
	code_descif=[]

	#Recorremos la cadena de codigo cifrado
	for i in code:

		#Guardamos el indice en el alfabeto de cada elemento cifrado
		ind = int(abc.index(i))

		#Aplicamos la formula inversa y lo añadimos a la lista 
		ind_orig=int( (ind-b)*pow(a, -1, elem) % elem )

		code_descif.append(abc[ind_orig])
	
	#Devolvemos la lista de elementos descifrados convertida a cadena
	return "".join(code_descif)
```

En el __bucle principal__, probaremos todas las posibles combinaciones de a y b para el alfabeto dado. Esto implica que solo servirán las *a* que sean primas entre sí con el número de elementos (el máximo común divisor es 1), condición que detallo en el [post sobre cifrado cesar y afin](https://naibu3.github.io/Cifrado-Cesar-y-afin).

```python
if __name__=="__main__":

	for a in range(0, n_elem+1):

		if (gcd(a, n_elem)==1):

			for b in range(0,n_elem+1):

				#Imprimo el codigo descifrado junto a la a y b correspondientes
				print(descifrar(codigo, abcedario, n_elem, a, b) + " -> a=" + str(a) + ", b=" + str(b))
				print("")
```

Tras la ejecución obtendremos una lista de más de 300 elementos por pantalla, que podremos leer uno a uno para encontrar aquel que sea legible. Puede parecer tedioso pero es la forma más simple de resolverlo. Sin embargo, en caso de que se quiera mejorar el script, estoy trabajando en una implementación con un análisis de frecuencia que muestre las opciones más probables primero para facilitar este último paso.

```
[...]

XHCLHLBFXHFZTMBHZLYQL -> a=5, b=20

CMHQMQGKCMKEYRGMEQDVQ -> a=5, b=21

HRMVRVLPHRPJDWLRJVIAV -> a=5, b=22

MWRAWAQUMWUOIBQWOANFA -> a=5, b=23

RBWFBFVZRBZTNGVBTFSKF -> a=5, b=24

WGBKGKAEWGEYSLAGYKXPK -> a=5, b=25

BLGPLPFJBLJDXQFLDPCUP -> a=5, b=26

XTIHTHLZXTZRJELTRHUSH -> a=7, b=0

IETSESWKIEKCUPWECSFDS -> a=7, b=1

TPEDPDHVTPVNFAHPNDQOD -> a=7, b=2

EAPOAOSGEAGYQLSAYOBZO -> a=7, b=3

PLAZLZDRPLRJBWDLJZMKZ -> a=7, b=4

AWLKWKOCAWCUMHOWUKXVK -> a=7, b=5

LHWVHVZNLHNFXSZHFVIGV -> a=7, b=6

WSHGSGKYWSYQIDKSQGTRG -> a=7, b=7

HDSRDRVJHDJBTOVDBRECR -> a=7, b=8

SODCOCGUSOUMEZGOMCPNC -> a=7, b=9

DZONZNRFDZFXPKRZXNAYN -> a=7, b=10

OKZYKYCQOKQIAVCKIYLJY -> a=7, b=11

[...]
```

Como ya encontré la frase cifrada pude comprobar rápidamente si el script funcionaba:

```bash
❯ python3 afindecoder.py | grep SUERTE

OSDESEAMOSMUCHASUERTE -> a=19, b=25

```

# Fuentes

Este código es fruto de la investigación recogida en mi [post sobre cifrado cesar y afin](https://naibu3.github.io/Cifrado-Cesar-y-afin), donde se encuntran las fuentes consultadas y una explicación detallada. Además dejaré tanto el script como la versión mejorada con análisis de frecuencias en mi [github]() por si alguien quisiera descargarlo.


¡ TPLCZDJKZLVZDYFKAXXKXDWXYFDW !
