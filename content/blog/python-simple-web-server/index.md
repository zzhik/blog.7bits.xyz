content/blog/python-simple-web-server/index.md---
title: "Tip: Un sencillo servidor web python para compartir archivos"
date: 2021-05-09T23:09:53-05:00
draft: true
publishdate: 2021-05-09T23:09:53-05:00
lastmod: 2021-05-09T23:09:53-05:00
tags: ["python", "tip"]
type: "post"
comments: false
---
Este es un sencillo tip para poder compartir archivos cuando nos encontramos en el mismo segmento de una red y no disponemos de otro medio para hacerlo. Cabe mencionar que usa una instalación predeterminada de python, por lo que de serie funciona tanto para equipos Linux y macos.<!--more-->

En más de una ocasión he tenido la necesidad de compartir archivos entre dos equipos que se ubican en la misma red inalambrica, como dispongo de un par de equipos mac, generalmente uso Airdrop para esta tarea y generalmente funciona sin problemas, sin embargo de vez en cuando algo pasa y de la nada dejan de reconocerse y esto generalmente pasa justo en un momento importante cuando no me puedo dar el lujo de estar apagando el bluetooth o el wifi hasta que se vuelvan a reconocer. En estas situaciones lo que me ha salvado es el módulo de servidor web que viene includo de serie con python.

Esta tarea es sumamente sencilla, simplemente inicio el servidor web en el directorio donde se encuentra el archivo que necesito compartir y en el otro equipo voy al navegador web, ingreso la dirección ip y listo puedo descargar el archivo sin problema.

Aquí la instrucción para ejecutar el servidor web en python2:

{{<highlight bash "linenos=table,linenostart=1">}}
$ python -m SimpleHTTPServer .
{{</highlight>}}

Equivalente en python 3:

{{<highlight bash "linenos=table,linenostart=1">}}
$ python -m http.server .
{{</highlight>}}

El servidor escuchará de manera predeterminada en el puerto 8000 a menos que se indique otra cosa.

No está demás mencionar que esto es algo que se debe hacer solo en casos puntuales y tan pronto como se comparte el archivo simplemente finalizar el servidor web con `CTRL+C`, de otra manera el contenido del sistema de archivos quedaría expuesto para que cualquiera que estuviera en la red pudiera descargar cualquier archivo.
