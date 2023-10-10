# Big Data APlicado
## UD 3 - Gestión de Soluciones III
### Ejemplo 1 MongoDB

1. Instalar MongoDB. Para ello puedes optar por diferentes opciones
    - Instalarla en tu máquina local
    - Usar Atlas MongoDB: https://www.mongodb.com/docs/atlas/getting-started/
    - Crear un contendor docker. Yo voy a usar este caso

2. Instalar MondoDB en Docker
   1. Vamos a la [imagen oficial de mondodb en docker](https://hub.docker.com/_/mongo)
   2. Seguimos los pasos, que tienen este formato
   
   ```
    docker run -d --network some-network --name some-mongo \
	-e MONGO_INITDB_ROOT_USERNAME=mongoadmin \
	-e MONGO_INITDB_ROOT_PASSWORD=secret \
	mongo
    ```

    En mi caso voy a añadr un bind mount para poder luego importar un fichero para el ejemplo. Por tanto sería:

    ```
    docker run -d --name mongo-bda1 -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=admin --mount type=bind,src=/home/jaime/BigData/BDA/,dst=/home/bda mongo
    ```

3. Abre una terminal en el contenedor

    ```
    docker exec -it mongodb-bda1 bash
    ```

4. Entrar a la consola de mongo y autorizarnos

    ```
    mongosh
    use admin
    db.auth("admin","admin")
    ```

5. Ver las bases de datos que hay
   
    ```
    show collections
    ```

6. Ver las colecciones de una base de datos

    ```
    show dbs
    ```

7. Crear y/o usar una Base de datos (_use nombre_bd_)

    ```
    use db_ejemplo1
    ```
8. Probamos los ejemplos que aparecen en el temario

9.  Probamos otro ejemplo. Vamos a [importar](https://www.mongodb.com/docs/database-tools/mongoimport/) un archivo fuente. Para ello nos abrimos otra terminal en la máquina y ejecutamos el siguiente comando

    ```
    mongoimport --db=db_ej1_restaurantes --collection=restaurantes --file=home/bda/UD3/Ejemplo1/restaurantes1.csv --authenticationDatabase=admin --username=admin --password=admin
    ```

10. Comprobamos la carga

    ```
    show dbs
    db.restaurantes.find()
    ```

11. Crear una consultar para encontrar qué restaurantes no tienen dirección (Todas tienen dirección)

    ```
    db.restaurantes.find({address:{$exists:false}})
    ```

12. Crear una consulta para encontrar aquellos restaurantes de cocina italiana que se encuentren en la zona geográfica con código postal 10075

    ```
    db.restaurantes.find({$and:[{"cuisine":"Italian"},{"address.zipcode":"10075"}]})
    ```
13. Encontrar aquellos restaurantes que tengan grado A, puntuación 11 y fecha 2014-10-01T00:00:00Z

    ```
    db.restaurantes.find({grades:{"date":ISODate("2014-10-01T00:00:00Z"),"grade":"A","score":11}})
    ```

14. Contabiliza cuántos restaurantes tienen una puntuación menor o igual a 5

    ```
    db.restaurantes.find({"grades.score":{$lt:5}}).count()
    ```

15. Obtener los nombres del segundo y el tercer restaurante de cocina italiana ordenados por nombre

    ```
    db.restaurantes.find({"cuisine":"Italian"}).sort({name:1}).limit(2).skip(1)
    ```

16. Añadir una valoración al restaurante 41156888

    ```
    db.restaurantes.updateOne({"restaurant_id":"41156888"},{$push:{grades: {"date":ISODate("2016-01-02T00:00:00.000Z"),"grade":"A","score":14}}})
    ```