---
title: Búsqueda rápida de puertos abiertos
published: true
---

# Búsqueda rápida de puertos abiertos con nmap

Este artículo pretende ser un pequeño apunte sobre un método para acelerar la búsqueda de puertos abiertos con nmap. Me baso en lo aprendido en el canal de S4vitar en youtube. Esto lo lograremos mediante el uso de una serie de parámetros:

```
nmap -p- --open -sS -min-rate 5000 -n -Pn <ip>
```