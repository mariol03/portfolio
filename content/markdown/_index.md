+++
date = '2024-12-06T13:39:53+01:00'
draft = false
title = 'Markdown'
+++
# Markdown 
Markdown es un lenguaje de marcas, que permite generar documentos con texto enriquecido, usando una sintaxis sencilla.  Markdown es un lenguaje de marcas que se puede convertir perfectamente a un documento HTML. Su sintaxis consiste en un "carácter de control" seguido del contenido que va a tener este bloque de contenido.
## Sintaxis
1. **Listas.**

 Las listas de markdown contienen el carácter **\*** en el caso de listas no enumeradas y el número de la lista seguido de un punto. Ejemplos:  
``` [markdown]
1. Elemento 1
2. Elemento 2

* Primer elemente desordenado
* Segundo elemento desordenado
```
2. **Encabezados.** 

Los encabezados de markdown empiezan por el carácter **#** para tener más de un encabezado markdown usa el número de caracteres para el tipo de encabezado dentro de HTML. Ejemplos:  
``` [markdown]
# Encabezado 1. Equivalente a <h1>
## Encabezado 2. Equivalente a <h2>
```
3. **Saltos de línea.** 

Para realizar un salto de línea con markdown usamos dos espacios al final de la línea de texto. Ejemplos:   
``` [markdown]
Línea 1.  
Línea 2.
```
4. **Tablas.** 

Para crear una tabla con markdown podemos usar el carácter **|** para *dibujar* la barra vertical y el carácter **-** para realizar la separación entre el encabezado y el texto normal.  
| Primera Cabecera | Segunda Cabecera |
| ---------------- | ---------------- |
| Celda con contenido | Celda con contenido |
| Celda con contenido | Celda con contenido |

Por ejemplo esta tabla se define con el siguiente "código"
```
| Primera Cabecera | Segunda Cabecera |
| ---------------- | ---------------- |
| Celda con contenido | Celda con contenido |
| Celda con contenido | Celda con contenido |
```
5. **Enlaces**

Para usar enlaces en markdown como este: [Enlace](https://google.es) se pone el título del enlace entre corchetes y entre paréntesis la dirección URL. Ejemplo:
```
[Enlace](https://google.es)
```
6. **Imágenes**

Para añadir imágenes hay que poner el carácter **!** seguido de un texto alternativo entre corchetes y la dirección de este entre paréntesis.  
Ejemplo:
```
![Imagen](/image.png)
```

