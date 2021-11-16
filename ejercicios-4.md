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

**5.** Seguimos utilizando `\b`, y añadimos `[^t]` para indicar que no queremos que nos busque palabras que comiencen por t. Con el flag `-n` nos muestra el número de línea en el que está cada palabra que nos encuentra de acuerdo con el criterio establecido. Así podemos comprobar que nos da el resultado ordenado por filas. Adicionalmente, le agregamos a la expresión regular `\s` para que tenga en cuenta el espacio antes de la palabra y comience a buscar a partir de la primera letra.

`grep -n -o -E -i '\s\b[^t][a-z]*s\b' aquella_voluntad.txt`


![grep](https://user-images.githubusercontent.com/92113066/140740660-c345cdbe-5b6c-4efe-9092-bea3073732fe.png)

**6.** Para encontrar las palabras que comienzan y terminan por la misma letra empleamos el comando 

`grep -E -o -i '\b(^[a-z])[a-z]+\1\b' aquella_voluntad.txt` donde con `\1` hacemos referencia al grupo de captura `(^[a-z])` que indica la letra de inicio de la palabra.

![grep3](https://user-images.githubusercontent.com/92113066/141787111-efba0c8c-f4b6-4a6a-835f-713de46da2bb.png)



## Ejercicio 2
¿Cuántos gene_ids existen con varios ceros seguidos en los dos gtfs (Humano y Drosophila)?. ¿Cuáles son? ¿Cuántas veces aparece cada uno en el .gtf dado?
Explora el fichero de anotaciones para ver si existen otros gene_ids con muchos números seguidos iguales.

### Respuesta ejercicio 2

Para averiguar el número de `gene_id` con varios ceros seguidos en el fichero de Drosophila, hemos utilizado el siguiente pipeline:

`grep -v "^#" Drosophila_melanogaster.BDGP6.28.102.gtf  | sed -E -n 's/.*gene_id "([^"]+)".*/\1/p' | grep -E ".*[0]{3,}" | wc -l`

Con esto, le estamos diciendo a la shell que nos quite con `grep -v` los metadatos; posteriormente, con  `sed` hacemos que nos saque por pantalla únicamente las referencias de todos los `gene_id` y finalmente, con `grep` seleccionamos todas las referencias que contengan varios ceros (en nuestro caso, hemos entendido varios como 3 o más). Comprobamos con `head` que obtenemos lo que nos interesa y ejecutamos `wc -l` para responder a la primera pregunta: existen 56351 gene_id en el archivo Drosophila con 3 o más 0 seguidos.
Para ver cuántas veces aparece cada uno, ejecutaremos `sort` y `uniq -c`:

![geneid drosophila](https://user-images.githubusercontent.com/92091175/141675420-df088caa-6eee-4fae-b5bd-e40a1f7c6f31.png)

Si volvemos a ejecutar `sort -nr`, podremos ver de un vistazo las referencias que aparecen más frecuentemente:

`grep -v "^#" Drosophila_melanogaster.BDGP6.28.102.gtf  | sed -E -n 's/.*gene_id "([^"]+)".*/\1/p' | grep -E ".*[0]{3,}" | sort | uniq -c | sort -nr | head -n10`

![frecuencias geneid drosophila](https://user-images.githubusercontent.com/92091175/141675581-d56fd42f-7e64-4220-91a0-4c21d2bb0ee8.png)

Por otra parte, para buscar referencias de gene id con otros números repetidos varias veces, podemos añadir a nuestro pipeline grupos de captura:

`grep -v "^#" Drosophila_melanogaster.BDGP6.28.102.gtf | sed -E -n 's/.*gene_id "([^"]+)".*/\1/p' | grep -E ".*([1-9])\1{3,}" | sort | uniq -c | sort -nr | head -n10`

![otros numeros repetidos](https://user-images.githubusercontent.com/92091175/141968265-bf9ba26f-7629-4d97-b104-7fa0fe29fb73.png)


Ahora, con el archivo comprimido de Homo sapiens, utilizaremos el mismo pipeline pero con una modificación: usaremos `zgrep`:

`zgrep -v "^#" Homo_sapiens.GRCh38.102.gtf.gz | sed -E -n 's/.*gene_id "([^"]+)".*/\1/p' | grep -E ".*[0]{3,}" | sort | uniq -c | sort -nr | head`


![geneid homo sapiens](https://user-images.githubusercontent.com/92091175/141676645-3ae55f9f-8d45-4533-9fa3-9a06e7ab48a5.png)

Para variar, esta vez sacaremos el número de repeticiones añadiendo a `grep` el flag `-c`:

![numero de referencias con ceros repetidos](https://user-images.githubusercontent.com/92091175/141970935-10ef682e-fe1f-4180-bacb-db51dcea362e.png)


Lo mismo para buscar referencias con otros números repetidos:

`zgrep -v "^#" Homo_sapiens.GRCh38.102.gtf.gz | sed -E -n 's/.*gene_id "([^"]+)".*/\1/p' | grep -E ".*([1-9])\1{2,}" | sort | uniq -c | sort -nr | head -n10`

![secuencias repetidas en homo sapiens](https://user-images.githubusercontent.com/92091175/141969966-dc292714-b8fa-4ac2-90b6-db8bd831f7a4.png)




## Ejercicio 3

Crea un pipeline que convierta un fichero fasta con secuencias partidas en múltiples líneas en otro sin saltos de línea. 
Al final, para cada secuencia, imprimirá su nombre y el número de caracteres que tenga. 

### Respuesta ejercicio 3

En este caso trabajamos con el fichero fasta `covid_samples.fasta`. Primero para convertir este fichero sin saltos de linea empleamos la orden `cat covid-samples.fasta | tr -d '\n'`, con el flag `-d` indicamos que nos elimine los saltos de linea.

## Ejercicio 4
En la sección 3.1., convertimos la cadena `chr1:3214482-3216968` a un formato tabular con `sed`. Sin embargo, existen otras maneras en las que podríamos haber obtenido el mismo resultado final. ¿Se te ocurren algunas? Recuerda que puedes usar el flag `g`, o puedes encadenar distintas llamadas a `sed` con tuberías si ves que meterlo todo en una única expresión regular se te antoja complicado. 

### Respuesta ejercicio 4

Como vimos en clase, con `sed` podemos hacer que nos sustituya los dos puntos y guiones por tabuladores con la orden: 

`echo "chr1:3214482-3216968" | sed -E 's/[:-]/\t/g'`

![formato tabular](https://user-images.githubusercontent.com/92091175/141677420-423efa8f-583e-435e-972f-0f5ff5db6508.png)

