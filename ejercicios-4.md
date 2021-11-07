# Sesión IV - Expresiones regulares: grep, sed y awk

Herramientas computacionales para bioinformática: UNIX, expresiones regulares y shell script

Edita esta plantilla en formato markdown [Guía aquí](https://guides.github.com/features/mastering-markdown/) como se pide en el guión. 
Cuando hayas acabado, haz un commit de tus cambios y súbelos al repositorio antes de la fecha de entrega señalada. 

======================================

**Añade por favor capturas de pantalla y el código de tus pipelines.**


## Ejercicio 1
Usando el fichero `aquella_voluntad.txt`, identifica usando grep:

1. El número de líneas que terminan por `o`. 
2. El número de líneas que terminan por `o` o por `a`. 
3. El número de líneas pares que terminan por `o` o por `a`
4. Todas las palabras que empiezan y acaban por `s` (ordenadas alfabéticamente)
5. Todas las palabras que no empiezan por t y acaban por `s`. (ordenadas por número de línea)
6. Todas las palabras que empiezan y acaban por la misma letra (volver a este punto al acabar toda la lección). 

### Respuesta ejercicio 1
**1.** 
Para buscar las líneas que terminan en "o" utilizaremos el metacaracter $:

`grep -E o$ aquella_voluntad.txt`

Podemos añadir el flag -c para saber el número de líneas que hay y que, en este caso, son 60.

![lineas terminadas en o](https://user-images.githubusercontent.com/92091175/140639718-01cd4c1c-2ad3-4b7f-9757-53f563722d0d.png)

**2.** Al igual que en el caso anterior, utilizaremos $, pero en este caso precedida de [oa]:

`grep -E [oa]$ aquella_voluntad.txt`

![líneas que terminan en oa](https://user-images.githubusercontent.com/92091175/140639876-d6a7e363-89ae-473f-af53-98c49f5e2f6c.png)

Vemos que el número de líneas asciende a 118.

**3.** 
En awk, NR es el numero de lineas en el fichero a procesar. Si utilizamos % estamos haciendo una operación módulo, es decir, obteniendo el resto de una división. Por tanto, con la orden `'NR%2==0'`, le estamos diciendo a awk que ejecute la acción solo si el número de línea dividido entre dos es igual a cero, esto es, estamos seleccionando las líneas pares. Con un pipe, ejecutamos el mismo filtro del apartado anterior.

`awk 'NR%2==0' aquella_voluntad.txt | grep -E [oa]$`

Añadimos `-c`para saber el número total de líneas: 57 (casi la mitad que las del ejercicio anterior, tiene sentido, ¿no?)

![Captura de pantalla de 2021-11-07 11-09-01](https://user-images.githubusercontent.com/92091175/140640745-fba9010f-2ccb-4086-b2ca-22bc9ea5a9c8.png)

**4.** Buscamos en la chuleta de expresiones regulares cómo delimitamos las palabras, y encontramos `\b`. Como sabemos que tiene que empezar y terminar por s, y que entre medias puede tener cualquier caracter o caracteres, utilizamos `'\bs[a-z]*s\b'`. Además, con el flag -o le decimos que nos saque la parte que coincide con el patrón, para luego crear un pipeline con `sort` y así sacar ordenadas todas las palabras que coincidan:

`grep -o -E '\bs[a-z]*s\b' aquella_voluntad.txt | sort`

![palabras que empiezan y terminan por s](https://user-images.githubusercontent.com/92091175/140641738-bb1cb224-775a-4f93-9988-3cf763f99217.png)

**5.** Seguimos utilizando `\b`, y añadimos `[^t]` para indicar que no queremos que nos busque palabras que comiencen por t. Con el flag `-n` nos muestra el número de línea en el que está cada palabra que nos encuentra de acuerdo con el criterio establecido. Así podemos comprobar que nos da el resultado ordenado por filas.

`grep -n -o -E -i '\b[^t][a-z]*s\b' aquella_voluntad.txt`

![palabras que no empiecen por t y acaben por s](https://user-images.githubusercontent.com/92091175/140642850-a79b4040-27a2-4bf2-82d3-cab38ac466df.png)


## Ejercicio 2
¿Cuántos gene_ids existen con varios ceros seguidos en los dos gtfs (Humano y Drosophila)?. ¿Cuáles son? ¿Cuántas veces aparece cada uno en el .gtf dado?
Explora el fichero de anotaciones para ver si existen otros gene_ids con muchos números seguidos iguales.

### Respuesta ejercicio 2


## Ejercicio 3

Crea un pipeline que convierta un fichero fasta con secuencias partidas en múltiples líneas en otro sin saltos de línea. 
Al final, para cada secuencia, imprimirá su nombre y el número de caracteres que tenga. 

### Respuesta ejercicio 3


## Ejercicio 4
En la sección 3.1., convertimos la cadena `chr1:3214482-3216968` a un formato tabular con `sed`. Sin embargo, existen otras maneras en las que podríamos haber obtenido el mismo resultado final. ¿Se te ocurren algunas? Recuerda que puedes usar el flag `g`, o puedes encadenar distintas llamadas a `sed` con tuberías si ves que meterlo todo en una única expresión regular se te antoja complicado. 

### Respuesta ejercicio 4

