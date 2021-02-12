# spark-course
1. [Primeros pasos](#schema)
2. [Primera ejecución del contador](#schema2)
3. [Key/ Value RDD'S](#schema3)
4. [Filtering RDD'S](#schema4)
10. [Enlaces ](#schema10)

<hr>

<a name="schema"></a>

# 1. Primeros pasos
1º  Crear entorno con Conda para trabajar con pyspark:

          
 ~~~ python
conda create --name spark-course python=3.8
~~~
spark-course = nombre del entorno
python = 3.8, versióm de python para el entrono.

2ª Activar el entorno
~~~ python
conda activate spark-course
~~~

2º Instalar pyspark  y openjdk
~~~ python
conda install pyspark openjdk

~~~

3º Comprobamos que se ha instalado correctamente entrando el promt de spark
~~~ 
(spark-course) ➜  ~ pyspark
~~~ 
![spark](./image/001.png)

<a name="schema2"></a>

# 2. Primera ejecución del contador
Hemos tenido que cambiar la ruta de los datos. En el archivo original aparecía así :
~~~ python
lines = sc.textFile("file:///SparkCourse/ml-100k/u.data")
~~~
Y nosotros hemos tenido que ponerlo así:
~~~ python
lines = sc.textFile("./ml-100k/u.data")
~~~
Simplemente porque nuestra ruta hacia los datos es distinta.

~~~ python
(spark-course) ➜  ~ spark-submit ratings.counter.py
~~~

<a name="schema3"></a>

# 3. Key / Value RDD'S

Vamos analizar la media de amigos por edad y para eso tenemos que ejecutar el archivo friends-by-age.py

1º cargamos el archivo fakefriend.csv
~~~ python
lines = sc.textFile("./data/fakefriends.csv")
~~~

2º Creamos RDD pero haciendo un map a los datos con la función parseline

~~~ python

rdd = lines.map(parseLine)

def parseLine(line):
    fields = line.split(',')
    age = int(fields[2])
    numFriends = int(fields[3])
    return (age, numFriends)
~~~ 
3º Creamos una variable con los totales por la edad `totalsByAge`, para ello usamos las acciones mapValues(func) y reduceByKey(func)

*mapValues(func)* crea un nuevo RDD de pares clave/valor, resultado de aplicar únicamente sobre los valores la función func, que recibe un solo parámetro.
*reduceByKey(func)* devuelve un RDD de pares clave/valor, donde cada clave única se corresponde con las diferentes claves del RDD original, y el valor es el resultado de aplicar una operación reduce sobre los valores correspondientes a una misma clave.

~~~ python
totalsByAge = rdd.mapValues(lambda x: (x, 1)).reduceByKey(lambda x, y: (x[0] + y[0], x[1] + y[1]))
~~~
4º Obtenemos las medias de las edades.

~~~python
averagesByAge = totalsByAge.mapValues(lambda x: round(x[0] / x[1],2))
results = averagesByAge.collect()
for result in results:
    print(result)
~~~

![result](./image/002.png)


<a name="schema4"></a>

# 4.Filtering RDD'S
Vamos a obtener la temperatura mínima obsevada por cada estación metereológica.


1º cargamos el archivo fakefriend.csv
~~~ python
lines = sc.textFile("./data/1800.csv")
~~~

2º Creamos RDD pero haciendo un map a los datos con la función parseline

~~~ python

rdd = lines.map(parseLine)

def parseLine(line):
    fields = line.split(',')
    stationID = fields[0]
    entryType = fields[2]
    temperature = float(fields[3]) * 0.1 * (9.0 / 5.0) + 32.0
    return (stationID, entryType, temperature)

~~~ 

3º Creamos una variable`minTemps` dondes solo vamos a guardar los valores que estén filtrados por el valor  `TMIN`
~~~ python
minTemps = parsedLines.filter(lambda x: "TMIN" in x[1])
~~~
*filter(func)* retorna un nuevo RDD que solo contiene los elementos del RDD original que satisfacen el predicado especificado en la función func (que retornará True o False)

4º Obtenemos `stationTemps` que cogemos sólo los valores de la posición 0 y 2 del la varible `minTemps` y reducimos los valores por lo que tengan el valor mínimo y por último imprimimos los resultados

~~~ python
stationTemps = minTemps.map(lambda x: (x[0], x[2]))
minTemps = stationTemps.reduceByKey(lambda x, y: min(x,y))
results = minTemps.collect()

for result in results:
    print(result[0] + "\t{:.2f}F".format(result[1]))
~~~

![result](./image/003.png)



Como ejericio hemos creado obtenido el valor máximo.

Sólo hemos cambiado estas líneas
~~~ python
maxTemps = parsedLines.filter(lambda x: "TMAX" in x[1])

stationTemps = maxTemps.map(lambda x: (x[0], x[2]))
maxTemps = stationTemps.reduceByKey(lambda x, y: max(x,y))
results = maxTemps.collect()

~~~
![result](./image/004.png)























<hr>

<a name="schema"></a>

# Enlaces que hemos usado
[Digitalbooks](http://reader.digitalbooks.pro/content/preview/books/41061/book/OEBPS/Text/capitulo_3.html)

[Spark doc](http://spark.apache.org/docs/latest/api/python/pyspark.html?highlight=mapvalues)