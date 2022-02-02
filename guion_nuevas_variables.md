# Sesión sobre nuevas variables a incorporar en la tarea final


> + **_Versión_**: 2020-2021
> + **_Asignatura (titulación)_**: Ciclo de gestión del dato: ecoinformática (máster conservación, gestión y restauración de la biodiversidad. UGR). Curso 2020-2021
> + **_Autor_**: Curro Bonet-García (fjbonet@uco.es)



## Objetivos del ejercicio

Esta actividad tiene los siguientes objetivos de aprendizaje:

+ Evocar el conocimiento adquirido sobre el problema planteado en la asignatura para identificar y proponer nuevas variables importantes.
+ Transferir el conocimiento adquirido sobre técnicas de análisis para pensar cómo preparar los datos relacionados con las nuevas variables identificadas.
+ Construir conocimiento de manera cooperativo en relación a las nuevas variables a incorporar al análisis.

## Contenido

La sesión se organiza en torno a la primera versión de flujo de trabajo que se elaboró al inicio de la asignatura. En él se observan algunas variables que se propusieron y que no se han abordado por falta de tiempo. La enumeración de estas variables da pie a que se inicie un debate muy fructífero en el que los alumnos hacen sus propuesta. El profesor conduce el debate y trata de destilar los consesos adoptados.

El siguiente vídeo reproduce el diálogo entablado entre los participantes.


<iframe width="560" height="315" src="https://www.youtube.com/embed/RN5kF4rFbA0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>



Como consecuencia de lo comentado más arriba, os paso la siguiente información por si os resulta útil:



### Incorporación de la variable "distancia a manchas de vegetación natural"

Para generar este mapa necesitamos contar con la distribución de los pinares de repoblación y con la de las formaciones vegetales que actuarán como donadoras de propágulos. Obtendremos ambos mapas a partir del mapa de usos y coberturas vegetales de Andalucía, generado por la REDIAM. En [este](https://github.com/aprendiendo-cosas/nuevas_variables_ecoinf_ugr/raw/2020--2021/geoinfo/MUCVA_25_multi_snevada.zip) enlace puedes descargar el mapa de usos de Sierra Nevada. Y [aquí](https://www.youtube.com/watch?v=RNQ7qwG5UDQ) tienes un video en el que te cuento cómo está estructurado. 

De manera resumida haremos lo siguiente: Crearemos un campo nuevo en el mapa de usos y coberturas anterior y asignaremos los valores 1 a todos los polígonos que tengan pinares, mientras que pondremos el valor 2 a todos los que sean considerados como fuentes de semillas. 

El algoritmo de QGIS que permite calcular el mapa de distancias necesita un raster como capa de entrada. Así que rasterizaremos el fichero de formas anterior. Una vez hecho esto podremos calcular la distancia entre todos los píxeles ocupados por pinares (1) y su mancha de vegetación natural más cercana (2).

Vamos con el paso a paso:

- **(1)** Crear un campo nuevo llamado _dist\_targe_ en la tabla de atributos del shapefile llamado _MUCVA\_multitemporal.shp_. Debe de ser un campo numérico.
- **(2)** Seleccionar los polígonos que tengan pinos. Para ello nos fijamos en el campo _DES\_UC07_, que contiene la descripción del uso del suelo en cada polígono en 2007. Veremos que hay una leyenda con muchos tipos. Buscamos aquellos que correspondan con bosques densos de coníferas. Y ejecutamos la siguiente consulta de selección con QGIS. Recuerda que hay que poner el nombre del cada campo cada vez que queremos seleccionar un tipo de uso. Y que el operador de unión es _OR_.

```r
"DES_UC07"  =  'FOR. ARBOL. DENSA: CONIFERAS' OR  
"DES_UC07"  =  'FOR. ARBOL. DENSA: QUERCINEAS+CONIFERAS' or 
 "DES_UC07" = 'MATORRAL DENSO ARBOLADO: CONIFERAS DENSAS' 

```

- **(2)** Ahora abrimos la tabla de atributos de la capa y hacemos que todos los registros seleccionados adquieran el valor de 1 en el campo _dist\_targe_. No olvides de guardar los cambios.
- **(3)** Repetimos la misma operación anterior, pero seleccionando los polígonos que tengan la palabra _Quercus_ o _quercínea_ en el campo _DES\_UC07_. 

```r
"DES_UC07" = 'FOR. ARBOL. DENSA: QUERCINEAS' or 
"DES_UC07"  ='MATORRAL DENSO' or  
"DES_UC07"  ='MATORRAL DENSO ARBOLADO: OTRAS FRONDOSAS' or
"DES_UC07"  = 'MATORRAL DENSO ARBOLADO: QUERCINEAS DENSAS' or
"DES_UC07"  = 'MATORRAL DENSO ARBOLADO: QUERCINEAS DISPERSAS' or
"DES_UC07"  =  'MATORRAL DISP. ARBOLADO: QUERCINEAS+CONIFERAS' or 
"DES_UC07"  = 'MATORRAL DISP. ARBOLADO: QUERCINEAS. DENSO' or
"DES_UC07"  = 'MATORRAL DISP. ARBOLADO: QUERCINEAS. DISPERSO'

```

- **(3)** Al igual que antes, haz que estos polígonos seleccionados tomen el valor 2 en el campo _dist\_targe_. Guarda los cambios.

- **(4)** Ahora rasterizamos el fichero de formas anterior con la opción _rasterizar_ de QGIS (menú raster -> conversion -> rasterizar). Aplicamos los siguientes valores a los parámetros necesarios:

  - _input layer_: _MUCVA\_25\_multi\_snevada.shp_
  - _field to use for a burn-in value_: _dist\_targe_
  - _output raster size units_: Georeferenced units
  - _Width/horizontal resolution_: 100m
  - _Height/horizontal resolution_: 100m
  - _output extent_: Selecciona la capa _TCD\_pinares\.tif_ para que QGIS copie de la misma la extensión del raster resultante. 
  - _output\_file_: _dist\_target.tif_. Recuerda guardar el archivo en un sito conocido por ti.

- **(5)** Creamos el mapa de distancia usando el algoritmo llamado _proximity raster_ (GDAL) en QGIS. Necesitamos añadir los siguientes parámetros:

  - _input\_layer_: _dist\_target.tif_
  - _band number_: 1
  - _A list of pixel values in the source image to be considered..._: Aquí debemos indicar los valores de nuestro raster inicial que son considerados como "fuentes" de semillas. En nuestro caso es el valor 2.
  - _distance units_: Georeferenced units.
  - _proximity map_ (mapa de salida): _distancia1.tif_

- **(6)** El mapa de distancias obtenido cubre toda la extensión de la zona de estudio. Pero a nosotros nos interesa conocer la distancia únicamente en los píxeles ocupados por pinares. Por eso, para borrar el resto, multiplicamos el mapa obtenido anteriormente (_distancia1.tif_) por el mapa que muestra la distribución de los pinares (_pinares\_repoblacion\.tif_). Puedes descargar dicho mapa [aquí](https://github.com/aprendiendo-cosas/nuevas_variables_ecoinf_ugr/raw/2020--2021/geoinfo/pinares_repoblacion.tif) aunque recuerda que deberás de recortarlo por tu zona de estudio. Para hacer esta operación usamos la calculadora de mapas. El raster resultante se llamará: _dist\_nat.tif_



### Radiación solar incidente

Vimos que esta variable es muy útil para analizar la distribución de la humedad del suelo y también para describir la microtopografía que es responsable de buena parte de lo que denominamos "microclima". Es fácil de calcular a partir de un modelo digital de elevaciones. [Aquí](https://github.com/aprendiendo-cosas/nuevas_variables_ecoinf_ugr/raw/2020--2021/geoinfo/mde_snev.tif.zip) hay un mde de Sierra Nevada. Busca cómo calcular la radiación total anual en tu herramienta favorita (QGIS, R, o phyton). 



### Mapas de clima (Temperatura y precipitación)

Esta variable también es importante porque nos permitirá distinguir cómo cambian en altura las condiciones macroclimáticas. Para acceder a esta información tendrás que aguzar tu ingenio y buscar en internet... Seguro que encuentras mapas útiles. Ánimo con ello ;)   



### Humedad potencial del suelo

El [índice de humedad](https://wikispaces.psu.edu/display/AnthSpace/Compound+Topographic+Index) (compound topographic index) se usa para inferir la capacidad que tiene un suelo de acumular agua en virtud de su posición topográfica en una ladera (en la parte alta o en la baja), y en relación a su altura relativa (está rodeado de píxeles más altos: cóncavo; está rodeado de píxeles más bajos: convexo). 

Este índice se puede calcular fácilmente con QGIS y con otras herramientas. Os paso [aquí](https://github.com/aprendiendo-cosas/nuevas_variables_ecoinf_ugr/raw/2020--2021/geoinfo/cti_pinares.tif) un mapa que muestra la distribución espacial de dicho índice en los pinares de repoblación de Sierra Nevada. 



### Densidad de los pinares

Aunque calculamos la densidad de los pinares de varias maneras al principio de la asignatura, os paso [aquí](https://github.com/aprendiendo-cosas/nuevas_variables_ecoinf_ugr/raw/2020--2021/geoinfo/TCD_pinares.tif) un mapa de densidad del estrato arbóreo (expresado en porcentaje) y calculado mediante teledetección. En [esta](https://land.copernicus.eu/pan-european/high-resolution-layers/forests/tree-cover-density/status-maps/2015) página tienes información sobre cómo se ha realizado.



### Distribución de los pinares de repoblación

Por si no lo tenéis controlado, [aquí](https://github.com/aprendiendo-cosas/nuevas_variables_ecoinf_ugr/raw/2020--2021/geoinfo/pinares_repoblacion.tif) va un mapa que muestra la distribución de los pinares en Sierra Nevada. 


### Regeneración de la encina en función de los usos del suelo en 1956

Esta variable es interesante y la comentamos al inicio de la asignatura. Pero no nos dio tiempo a ponerla en práctica. Si alguien tiene interés, [aquí](https://aprendiendo-cosas.github.io/peso_pasado_ecoinf_ugr/guion_peso_pasado.html) hay un guión que describe cómo incorporarla.