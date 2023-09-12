# UD 2 - Procesado y Presentación Datos Almacenados - MongoDB

El nombre **MongoDB** proviene de la palabra inglesa **"homongous"** que significa enorme. **MongoDB es, por tanto, un sistema de base de datos NoSQL, open source, orientado a documentos y escrito en lenguaje C++**. Este sistema de bases de datos está disponible no solo para múltiples plataformas y sistemas operativos (Windows, Linux, OS X) sino también como servicio empresarial en la nube y se puede integrar con otros servicios como, por ejemplo, Amazon Web Services (AWS).

## 1. Características

Tal y como se ha presentado anteriormente, MongoDB es un sistema de bases de datos documental u orientado a documentos. Esta orientación hace que los datos se almacenen de forma estructurada en forma de documentos, los cuales se acoplan sin problema en los tipos de datos utilizados por los lenguajes de programación. Además, esta concepción hace que una base de datos MongoDB disponga de un esquema dinámico y fácilmente modificable.

En este apartado se pretende poner de relieve **las principales características de MongoDB** que hacen que sea en la actualidad uno de los sistemas de bases de datos NoSQL más utilizados tanto en el ámbito académico como en el profesional. 

- **Alto rendimiento**: Gracias a la definición de los documentos y la creación de índices que hace que las lecturas y escrituras se realicen de forma más rápida.

- **Alta disponibilidad**: MongoDB dispone de servidores replicados con restablecimiento automático maestro.

- **Fácil escalabilidad**: Permitiendo distribuir colecciones de documentos entre diferentes máquinas de forma muy sencilla.

- **Indexación**: Similar al concepto de índice en una base de datos relacional, permite crear índices e índices secundarios para mejorar el rendimiento de las consultas.

- **Consultas ad-hoc**: Al igual que en una base de datos relacional, MongoDB da soporte a la búsqueda por campos, consultas de rangos, uso de expresiones regulares... A todo lo anterior, se añade la posibilidad de ejecutar y devolver una función JavaScript definida por el programador.

- **Replicación**: Siguiendo el modelo maestro-esclavo. El maestro puede ejecutar comandos de lectura y escritura mientras que el esclavo solo tiene acceso de lectura y la posibilidad de realizar copias de seguridad. En caso de que el maestro caiga, el esclavo puede elegir un nuevo maestro para mantener el servicio de replicación.

- **Balanceo de carga**: MongoDB es capaz de ejecutarse en múltiples servidores, pudiendo balancear la carga y/o duplicar los datos para mantener el correcto funcionamiento del sistema aunque se produzca un fallo hardware.

- **Almacenamiento de archivos**: Todo lo anterior, facilita que este sistema de base de datos pueda ser usado con un sistema de archivos GridFS con balanceo de carga y replicación.

- **Agregación**: Proporciona operadores de agregación y la posibilidad de utilizar funciones MapReduce para el procesamiento de datos por lotes.

- **Ejecución de Javscript del lado del servidor**: Es posible realizar consultas utilizando JavaScript, de forma que éstas son enviadas directamente a la base de datos para ser ejecutadas.

Todas estas características hacen que, en la actualidad, MongoDB sea uno de los principales sistemas de bases de datos NoSQL elegidos en multitud de aplicaciones como: **almacenamiento y registro de eventos, comercio electrónico, juegos, aplicaciones móviles, almacenes de datos, gestión de estadísticas en tiempo real y cualquier aplicación que requiera llevar a cabo analíticas sobre grandes volúmenes de datos**.

## 2. Conceptos básicos, utilidades y herramientas

***[Getting Started MongoDB](https://www.mongodb.com/docs/manual/tutorial/getting-started/)***

Aunque MongoDB es un sistema de bases de datos NoSQL con todo lo que ello implica, también ofrece toda la funcionalidad de la que disponen las bases de datos relacionales. Sin embargo, **la estructura de una base de datos MongoDB difiere de la de una base de datos relacional**.

En un servidor MongoDB es posible crear tantas bases de datos como se desee. El concepto de **base de datos** es en este caso equivalente al de **base de dato**s en los sistemas relacionales. Una vez creada, la base de datos estará compuesta por una o más **colecciones**. El término colección en el ámbito NoSQL es el equivalente al concepto de **tabla** en los sistemas relacionales. Cada colección, por tanto, estará formada por un conjunto de **documentos** (o incluso ninguno, en cuyo caso la colección estaría vacía). El concepto de documento en NoSQL se corresponde con el concepto de **registro** en una base de datos relacional. Un documento estará compuesto por una serie de **campos**, al igual que lo están los registros de una base de datos relacional. En el ámbito NoSQL, y más concretamente en MongoDB, cada documento viene dado por un archivo **JSON** (http://www.json.org/) en el cual, siguiendo una estructura clave-valor se especifican las características de cada documento. El listado 3.11 muestra un ejemplo de documento que define un barco.

```json
{
    name: 'USS Enterprise-D',
    operator: 'Starfleet',
    type: 'Explorer',
    class: 'Galaxy',
    crew: 750,
    codes: [10,11,12]
}
```
_Listado 3.11: Definición json de un documento barco_

Para la administración del sistema de bases de datos, MongoDB pone a disposición de los usuarios las siguientes utilidades:

- **mongo**: Se trata del shell interactivo de MongoDB que permite insertar, eliminar, actualizar datos y realizar consultas, además de replicar la información, apagar servidores y ejecutar código JavaScript.

- **mongostat**: Herramienta de línea de comandos que muestra las estadísticas de una instancia en ejecución de MongoDB

- **mongotop**: Herramienta de línea de comandos que muestra la cantidad de tiempo empleado en la lectura y escritura de datos por parte de la instancia en ejecución

- **mongosniff**: Herramienta de línea de comandos que permite hacer un rastreo del tráfico de la red que va desde y hacia MongoDB.

- **mongoimport/mongoexport**: Herramienta de línea de comandos para importar y exportar contenido en o desde .json, .csv o .tsv entre otros.

- **mongodump/mongorestore**: Herramienta de línea de comandos para la creación de una exportación binaria del contenido de la base de datos.

Aunque MongoDB tiene una interfaz administrativa accesible desde http://localhost:28017/ (siempre que se haya iniciado con la opción mongod − −rest, existen algunas herramientas gráficas para la administración y uso de este sistema de base de datos como **UMongo** (https://github.com/agirbal/umongo) , el cual es una aplicación open source de sobremesa para navegar y administrar un clúster MongoDB o **Robomongo** (https://robomongo.org), que también es una herramienta gráfica open source que incluye una terminal de comandos completamente compatible con el shell de mongo.

_[ver presentación BDA3.5](https://moodle.iesgrancapitan.org/pluginfile.php/58312/mod_folder/content/0/BDA_UD3_05.pdf)_

## 3. Operaciones CRUD

***[MongoDB CRUD Operations](https://www.mongodb.com/docs/manual/crud/#mongodb-crud-operations)***

Cualquier sistema de datos permite definir operaciones básicas como son la creación de tablas e inserción de registros, lectura de datos, actualización y eliminación de registros. Estas operaciones básicas se conocen con el nombre de **CRUD (Create, Read, Update, Delete)**. A continuación, se muestran distintos ejemplos para ilustrar la implementación de estas operaciones en MongoDB.

Una vez iniciada una sesión del Shell de Mongo, es posible crear una base de datos utilizando el siguiente comando.

```
test>use DB_Ejemplo
switched to db DB_Ejemplo
```

Para crear una colección de documentos "Barco" dentro de la base de datos
DB_Ejemplo se puede escribir.
```
DB_Ejemplo>db.createCollection("ships")
{ ok: 1 }
```

Para visualizar las bases de datos almacenadas en el servidor se puede utilizar el comando _show dbs_ mientras que para obtener un listado de las colecciones de la base de datos sobre la que se está trabajando, es posible utilizar el comando _show collections_. Por otra parte, para insertar un documento dentro de la colección, como el mostrado en el listado 3.11, se ejecuta el siguiente comando.

```
db.ships.insertOne({name:'USS-Enterprise-D',operator:'Starfleet',type:'Explorer',class:'Galaxy',crew:750,codes:[10,11,12]})
```
_[ver presentación BDA3.6](https://moodle.iesgrancapitan.org/pluginfile.php/58312/mod_folder/content/0/BDA_UD3_06.pdf)_

Para **actualizar** el nombre del barco "USS-Enterprise-D" por "USS Something" es posible escribir el comando.

***[MongoDB Update Operations](https://www.mongodb.com/docs/manual/crud/#update-operations)***

```
db.ships.updateOne({name : {$eq: 'USS-Enterprise-D'}}, {$set : {name: 'USS Something'}})
```

Mientras que si se pretenden establecer o cambiar varios atributos de un mismo documento, como por ejemplo el operador y la clase, se puede utilizar el comando update de la siguiente forma.

```
db.ships.updateOne({name : {$eq: 'USS Something'}}, {$set : {name: 'USS Prometheus', class: 'Prometheus'}})
```

Finalmente, la eliminación de algún atributo de un documento también es una operación de actualización que puede realizarse de la siguiente forma.

```
db.ships.updateOne({name : {$eq: 'USS Prometheus'}}, {$unset : {operator: 1}})
```

***[MongoDB Delete Operations](https://www.mongodb.com/docs/manual/crud/#delete-operations)***

La **eliminación** de documentos de una colección puede realizarse de forma directa o bien utilizando expresiones regulares. A continuación se muestran dos comandos para ilustrar ambas formas de eliminación. El segundo de ellos elimina aquellos documentos que comienzan por "a".

```
db.ships.deleteOne({name : 'USS Prometheus'})
db.ships.deleteOne({name:{$regex:'^A*'}})
```

Para **leer y mostrar** documentos, se utiliza el comando find(). A continuación se muestra un ejemplo en el que, el primer comando muestra un documento al azar de los existentes, el segundo muestra todos los documentos y lo hace de forma indexada en lugar de como texto seguido, el tercero muestra solo los nombres de los barcos y el último, encuentra un documento cuyo nombre sea "USS Defiant".

***[MongoDB Read Operations](https://www.mongodb.com/docs/manual/crud/#read-operations)***

```
db.ships.findOne()
db.ships.find().pretty()
db.ships.find({}, {name:true})
db.ships.findOne({'name':'USS Defiant'})
```
_[ver presentación BDA3.7](https://moodle.iesgrancapitan.org/pluginfile.php/58312/mod_folder/content/0/BDA_UD3_07.pdf)_

## 4. Consultas y Agregación

Al igual que cualquier lenguaje de manipulación de datos, MongoDB dispone de las herramientas y operadores necesarios para realizar consultas sobre los documentos almacenados en las colecciones. A continuación, se muestran ejemplos de uso de los principales operadores.

Los operadores relaciones mayor que, menor que, mayor o igual que y menor o igual que se corresponden, respectivamente, con los operadores $gt, $lt, $gte, $lte. Además, también es posible utilizar el operador $regex para recuperar elementos que cumplan una expresión regular. A continuación, se muestran dos consultas que recuperan aquellos barcos que permitan subir a bordo a más de cien pasajeros y otra consulta para recuperar aquellos que dejen subir tan solo a 1" pasajeros o menos.

```
db.ships.find({crew:{$gt:100}})
db.ships.find({crew:{$lte:100}})
```

El operador \$exists permite devolver aquellos documentos para los que existe o no un determinado atributo. El siguiente comando permite encontrar aquellos barcos para los cuales existe el campo "class".

```
db.ships.find({class:{$exists:true}})
```

***[MongoDB Aggregation Operations](https://www.mongodb.com/docs/manual/aggregation/#aggregation-operations)***

Las funciones de **agregación** son especialmente útiles en los lenguajes de consulta y manipulación de datos. De esta forma, $sum permite agregar mediante la operación suma una serie de valores, $avg permite obtener la media, $mnin y $max encontrar el valor máximo y mínimo de un atributo para el conjunto de elementos, $push introduce en un array los resultados de la consulta que se ha realizado, $addToSet es similar al anterior solo que sin incluir valores duplicados y, por último, $first y $last permiten obtener el primer y último documento en una consulta. A continuación, se muestra un ejemplo del uso de cada uno de estos operadores de agregación.

```
db.ships.aggregate([{$group:{_id:"$operator",num_ships:{$sum:"$crew"}}}])
db.ships.aggregate([{$group : {_id : "$operator", num_ships : {$avg : "$crew"}}}])
db.ships.aggregate([{$group : {_id : "$operator", num_ships : {$min : "$crew"}}}])
db.ships.aggregate([{$group : {_id : "$operator", classes : {$push: "$class"}}}])
db.ships.aggregate([{$group : {_id : "$operator", classes : {$addToSet : "$class"}}}])
db.ships.aggregate([{$group:{_id:"$operator",last_class:{$last: "$class"}}}])
```

Finalmente, MongoDB pone a disposición de los usuarios multitud de funciones útiles en la realización de consultas. La tabla 3.4 muestra algunas de las más utilizadas.

| **Función** | **Significado** |
| -- | -- |
| $project | Cambia el conjunto de documentos modificando sus claves y valores |
| $match | Operación de filtrado para reducir el número de elementos recuperados |
| $group | Operador de agregación para agrupar resultados |
| $sort | Ordena los documentos |
| $skip | Recupera los documentos a partir de un numero especificado por el usuario |
| $limit | Limita los resultados de la consulta al valor pasado como parámetro a la función |
| $unwind | Utilizado como equivalente al join de SQL |

_Tabla 3.4: Funciones útiles en MongoDB_

_[ver presentación BDA3.8](https://moodle.iesgrancapitan.org/pluginfile.php/58312/mod_folder/content/0/BDA_UD3_08.pdf)_



