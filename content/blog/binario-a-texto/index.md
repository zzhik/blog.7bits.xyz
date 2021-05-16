---
title: "Convertir una cadena de texto en formato binario a texto legible"
date: 2021-05-15T12:47:18-05:00
publishdate: 2018-10-07T11:17:14+02:00
lastmod: 2018-10-08T18:55:29+02:00
tags: ["python"]
type: "post"
comments: false
---
Hace unos días me encontré una publicación en facebook con un mensaje codificado como una cadena de texto en formato binario, aunque pude simplemente buscar en google un convertidor de binario a texto, decidí hacer un sencillo programa en python. El código es muy sencillo como se muestra a continuación:

{{<highlight python "linenos=table,linenostart=1">}}
str_bin = "01001100 01110101 01101001 01110011 00100000 01010011 01100001 01101110 01110100 01101001 01100001 01100111 01101111"
letters = []
for letter in str_bin.split(" "):
    letters.append(chr(int(letter, 2)))
print("".join(letters))
{{</highlight>}}

Esta solución, a parte del ciclo for, utiliza algunas funciones nativas de python, estas son:

* `str_bin.split(" ")`: La función `split()` aplica sobre una cadena de texto convirtiendola en una lista de elementos, para esto la cadena original se rompe por el caracter dado. En nuestro ejemplo se rompera utilzando un espacio, con lo que una cadena "uno dos" se convierte a una lista en la forma de ["uno", "dos"].
* `int(letter, 2)`: La función `int()` convierte una cadena de texto a un número, como primer parámetro se envía la cadena de texto y como segundo parámetro la base en la que se encuentra el número representado en la cadena, en nuestro ejemplo ya que la cadena incluye información en formato binario la base es 2.
* `chr(int)`: Esta función convierte un entero en su representación como letra.
* `"".join(lista)`: Esta función concatena una lista de cadenas utilizando un espacio vacío como elemento de unión. Por ejemplo para una lista como esta `lista = ["uno", "dos"]` aplicando esta función `"_".join(lista)` el resultado sería la siguiente cadena de texto `"uno_dos"`. 


Una segunda versión es la siguiente, la cual utiliza la característica conocida como `list comprehension`:

{{<highlight python "linenos=table,linenostart=1">}}
str_bin = "01001100 01110101 01101001 01110011 00100000 01010011 01100001 01101110 01110100 01101001 01100001 01100111 01101111"
str_items = str_bin.split(" ")
letters = [ chr(int(letter, 2)) for letter in str_items ]
print("".join(letters))
{{</highlight>}}


Esta versión con `list comprehension` nos permite iterar sobre cada elemento de una lista, aplicarle alguna operación y como resultado genera también una lista sin tener explicitamente que hacer un append. En otros lenguajes esta caracteristica sería el equivalente de una operación `map`.

Fin.