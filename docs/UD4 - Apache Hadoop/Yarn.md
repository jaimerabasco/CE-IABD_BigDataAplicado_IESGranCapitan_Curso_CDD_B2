# UD 6 - Apache Hadoop - Yarn

!!! Info "**Introducción**"

    Seguimos adentrándonos en lo que es Apache Hadoop. Ya hemos entendido que Apache Hadoop ofrece una capa de almacenamiento, que es HDFS, con la que podrá almacenar todos los datos. Hemos entendido cómo funciona HDFS y qué aspectos debe tener en cuenta para dimensionar correctamente su plataforma.
    El siguiente paso en su camino por entender Hadoop es conocer la capacidad que ofrece Hadoop para procesar todos los datos almacenados en HDFS. Es el turno de conocer YARN. YARN será la base sobre la que se ejecutarán todas las aplicaciones de procesamiento o análisis de datos, así que es un punto importante a conocer.

**YARN** es el acrónimo de **Yet Another Resource Negotiator**, es decir, según su acrónimo es un gestor de recursos.

En las primeras versiones de Hadoop, todo el procesamiento se realizaba con _MapReduce_, que es un framework de procesamiento distribuido que veremos en el siguiente apartado, y que, pese a que su funcionamiento era correcto, ya que era capazde ofrecer la capacidad de desarrollar aplicaciones complejas que procesaran un gran volumen de datos, tenía varios problemas:

- Restringía mucho el tipo de aplicaciones que los desarrolladores podían realizar, ya que había que ceñirse a las operaciones y forma de ejecución que MapReduce ofrecía, por lo que era difícil utilizar los datos de HDFS para otro tipo de usos como el procesamiento en tiempo real.
- MapReduce es un modelo de programación muy poco eficiente, lo que hace que los casos de uso que requieren respuestas rápidas no sean viables.
- La concurrencia en la ejecución de aplicaciones no estaba bien resuelta, por lo que cuando un usuario o aplicación lanzaba un trabajo MapReduce, se podría decir que el resto tenía que esperar a que terminara la tarea para poder lanzar nuevos trabajos.

<figure style="align: center; width:600px;">
    <img src="images/Figura6.1_Yarn_Yarn_en_Hadoop2.jpg">
    <figcaption>Figura 6.1_Yarn: Arquitectura YARN Hadoop </figcaption>
</figure>


Por este motivo, en la versión 2 de Hadoop se introdujo YARN. El objetivo de YARN era poder independizar el almacenamiento del procesamiento, abrir Hadoop a cualquier tipode aplicación que quiera trabajar con los datos de HDFS, y dar la posibilidad de quemúltiples usuarios puedan trabajar con la plataforma.

<figure style="align: center; width:600px;">
    <img src="images/Figura6.2_Yarn_Estructura_YARN_Hadoop.jpg">
    <figcaption>Figura 6.2_Yarn: Arquitectura YARN Hadoop </figcaption>
</figure>

## Contenedores

En YARN es importante conocer el concepto de **contenedor**, que es la unidad mínima de recursos de ejecución para las aplicaciones, y que representa una cantidad específica de memoria, núcleos de procesamiento (cores) y otros recursos (disco, red), para procesar
sus aplicaciones.

Todas las tareas de las aplicaciones YARN se ejecutan en contenedores. Cada trabajo puede contener múltiples tareas y cada una de las tareas se ejecuta en su propio contenedor. Por otro lado, al iniciar un trabajo, YARN puede asignar a cada tarea un conjunto de contenedores dependiendo de la demanda de la aplicación (al lanzar la tarea se le puede indicar el número de contenedores que necesita) y a la disponibilidad de los contenedores que hay en el clúster en ese momento (si hay menos contenedores disponibles de los solicitados, YARN se encargará de aplicar las reglas de prioridad para asignar contenedores que a lo mejor están siendo usados por otras aplicaciones).

Los contenedores se pueden configurar en cuanto al tamaño de memoria y la cantidad de elementos de procesamiento. La cantidad de tareas y, por lo tanto, la cantidad de aplicaciones de YARN que puede ejecutar en cualquier momento, está limitada por la cantidad de contenedores que tiene un clúster.

<figure style="align: center; width:600px;">
    <img src="images/Figura6.3_Yarn_Nodos_Servicios_YARN.jpg">
    <figcaption>Figura 6.3_Yarn: Nodos y Servicios YARN </figcaption>
</figure>

Existe un nodo maestro, el **ResourceManager**, que coordina, asigna y controla la ejecución de todas las tareas, y nodos worker que disponen de un servicio **NodeManager**, que monitoriza el estado de ejecución de las tareas en el worker, así como el estado de
los recursos/contenedores en dicho nodo.

### ResourceManager

Este servicio sería el equivalente al Namenode en HDFS, ya que es el maestro que controla la ejecución de todas las tareas que están en ejecución, o las solicitudes de ejecución existentes.

Cuando un cliente quiere ejecutar una aplicación en YARN, se comunica con el **ResourceManager**, que será el encargado de asignarle los recursos en base a las políticas de prioridad asignadas y los recursos disponibles, distribuir la aplicación (el ejecutable) por los diferentes nodos worker que realizarán la ejecución, controlar la ejecución para detectar si ha habido una caída de una de las tareas, para relanzarla en otro nodo, y liberar los recursos una vez la ejecución haya finalizado.

El ResourceManager tiene dos componentes principales::

- El **ApplicationMaster**, que es el servicio que recibe las peticiones de ejecución por parte de los clientes, distribuye las aplicaciones por los nodos worker, asigna los recursos, coordina la ejecución de las tareas, monitoriza la ejecución, solventa los fallos en las ejecuciones, y libera los recursos una vez las tareas han finalizado.

- El **Scheduler**, que es el servicio que asigna prioridades y establece los **recursos/containers** que disfrutará cada aplicación. Este planificador no monitoriza el estado de ninguna aplicación ni les ofrece garantías de ejecución, ni recuperación por fallos de la aplicación o el hardware, sólo planifica. Este componente realiza su planificación a partir de los requisitos de recursos necesarios por las aplicaciones (CPU, memoria, disco y red).

<figure style="align: center; width:600px;">
    <img src="images/Figura6.4_Yarn_Arquitectura_Yarn.png">
    <figcaption>Figura 6.4_Yarn: Arquitectura YARN </figcaption>
</figure>

### Node Manager

El servicio NodeManager se ejecuta en cada nodo worker y proporciona los recursos computacionales necesarios para las aplicaciones en forma de **contenedores**. Implementa Heartbeats para mantener informado del estado al _Resource Manager_. Realiza las siguientes
funciones:

- Monitoriza y proporciona información sobre el consumo de recursos (CPU/memoria) por parte de los contenedores al _ResourceManager_.
- Envía mensajes para notificar al _ResourceManager_ su actividad (no está caído) así como la información sobre su estado a nivel de recursos.
- Supervisa el ciclo de vida de los contenedores de aplicaciones.
- Supervisa la ejecución de las distintas tareas en contenedores y termina aquellas tareas que se han quedado bloqueadas.
- Almacena un log (fichero en HDFS) con todas las operaciones que se realizan en el nodo.
- Lanza procesos _ApplicationMaster_, que coordinan los trabajos para cada aplicación.

!!! Info inline end

    Los **NodeManager**, al igual que los Datanodes en HDFS, son tolerantes a fallos, por lo que en caso de caída de alguno de ellos, el ResourceManager detectará que no funciona y redirigirá la ejecución de las aplicaciones al resto de nodos activos.

### AplicationMaster

Existe un proceso **ApplicationMaster** por aplicación. Este proceso se encarga de negociar con el _ResourceManager_ los recursos necesarios para la ejecución de las tareas de su aplicación.

El _ApplicationMaster_ se ejecuta en uno de los nodos worker, para garantizar la escalabilidad de YARN, ya que si se ejecutaran todos los _ApplicationMaster_ en el nodo maestro, junto con el _ResourceManager_, éste sería un cuello de botella para poder escalar
o poder lanzar un gran número de aplicaciones sobre el clúster.

Asimismo, a diferencia del _ResourceManager_ y los NodeManager, el _ApplicationMaster_ es específico para una aplicación por lo que, cuando la aplicación finaliza, el proceso _ApplicationMaster_ termina. En el caso de los servicios _ResourceManager_ y NodeManager, siempre se están ejecutando aunque no haya aplicaciones activas en el clúster. Cada vez que se inicia una nueva aplicación, _ResourceManager_ asigna un contenedor que ejecuta _ApplicationMaster_ en uno de los nodos del clúster.

## Funcionamiento

YARN, en concreto, el _ResourceManager_, es invocado por los clientes cuando quieren lanzar una aplicación en el clúster para su ejecución.

<figure style="align: center; width:600px;">
    <img src="images/Figura6.5_Yarn_Funcionamiento_YARN.jpg">
    <figcaption>Figura 6.5_Yarn: Funcionamiento YARN </figcaption>
</figure>


La secuencia de ejecución de una aplicación es la siguiente:

!!! note inline end

    Como has visto tanto en YARN como en HDFS, en Hadoop se intenta que los nodos maestros hagan el menor número de operaciones posibles para cada tarea, con el objetivo de poder escalar. Si por cada tarea a ejecutar necesitaran realizar un gran número de tareas, la capacidad de escalar se vería muy reducida, ya que los nodos master son los únicos que pueden realizar este tipo de tareas, siendo el límite del sistema Hadoop la capacidad máxima que puede tener un nodo maestro.

1. El cliente se comunica con el _ResourceManager_ para solicitarle la ejecución de una aplicación. En la llamada, le envía el código/ejecutable de la aplicación, así como unos parámetros sobre los recursos necesarios para dicha ejecución.
2. El _ApplicationMaster_, tras chequear con el Scheduler la disponibilidad de recursos y su prioridad, pide al NodeManager de un nodo la creación de un container que ejecutará el _ApplicationMaster_ de la aplicación.
3. El NodeManager crea el conteneder y arranca su ejecución.
4. El _ResourceManager_ se comunica con el _ApplicationMaster_ para solicitarle los contenedores necesarios para la ejecución de la aplicación en caso necesario.
5. El _ApplicationMaster_ se comunica con los contenedores donde se está ejecutando distintas tareas para controlar su ejecución, y va notificando el status de la ejecución al _ResourceManager_.
6. El NodeManager, asimismo, envía información al _ResourceManager_ sobre el consumo de recursos y notificando que el nodo está activo.

YARN soporta la reserva de recursos mediante el [Reservation System](https://hadoop.apache.org/docs/stable/hadoop-yarn/hadoop-yarn-site/ReservationSystem.html), un componente que permite a los usuarios especificar un perfil de recurso y restricciones temporales (deadlines) y posteriormente reservar recursos para asegurar la ejecución predecibles de las tareas importantes. Este sistema registra los recursos a lo largo del tiempo, realiza control de admisión para las reservas, e informa dinámicamente al planificador para asegurarse que se produce la reserva.

Para conseguir una alta escalabilidad (del orden de miles de nodos), YARN ofrece el concepto de [YARN Federation](https://hadoop.apache.org/docs/stable/hadoop-yarn/hadoop-yarn-site/Federation.html). Esta funcionalidad permite conectar varios clústeres YARN y hacerlos visibles como un clúster único. De esta forma puede ejecutar trabajos muy pesados y distribuidos.

## Configuración

Para configurar YARN, primero editaremos el archivo `yarn-site.xml` para indicar quien va a ser el nodo maestro, así como el manager y la gestión para hacer el MapReduce:

!!! tip inline end

    Recuerda que los archivos de configuración se encuentran dentro de la carpeta `$HADOOP_HOME/etc/hadoop`.

```xml title="yarn-site.xml"
<configuration>
    <property>
        <name>yarn.webapp.ui2.enable</name>
        <value>true</value>
    </property>  
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>bda-iesgrancapitan</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services.mapreduce_shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>
    <property>
        <name>yarn.log-aggregation-enable</name>
        <value>true</value>
    </property>   
</configuration>
```

**Mención especial tiene la configuración del parámetro `yarn.application.classpath`. Si no tenemos bien configurado este parámetro, no funcionará MapReducev2 ejecutado sobre Yarn, ya que no encontrará las librerías necesarías para su correcta ejecución.**

Para obtener la ruta correcta del classpath de Hadoop ejecutamos la siguiente instrucción

```
echo `hadoop classpath`
```

Y es esta salida la que tenemos que poner como valor de la propiedad. En mi caso:

```
home/hadoop/hadoop-3.3.4/etc/hadoop:/home/hadoop/hadoop-3.3.4/share/hadoop/common/lib/*:/home/hadoop/hadoop-3.3.4/share/hadoop/common/*:/home/hadoop/hadoop-3.3.4/share/hadoop/hdfs:/home/hadoop/hadoop-3.3.4/share/hadoop/hdfs/lib/*:/home/hadoop/hadoop-3.3.4/share/hadoop/hdfs/*:/home/hadoop/hadoop-3.3.4/share/hadoop/mapreduce/*:/home/hadoop/hadoop-3.3.4/share/hadoop/yarn:/home/hadoop/hadoop-3.3.4/share/hadoop/yarn/lib/*:/home/hadoop/hadoop-3.3.4/share/hadoop/yarn/*
```
También hay que tener en cuenta, que si te conectas desde otra máquina, hay que dar permiso de acceso desde fuera de la máquina con la siguiente propiedad:

```xml title="yarn-site.xml"
    <property>
        <name>yarn.resourcemanager.webapp.address</name>
        <value>0.0.0.0:8088</value>
    </property> 
```

Por tanto, la configuración final del archivo de configuración `yarn-site.xml` sería: 

```xml title="yarn-site.xml"
<configuration>
    <property>
        <name>yarn.webapp.ui2.enable</name>
        <value>true</value>
    </property>  
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>bda-iesgrancapitan</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services.mapreduce_shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>
    <property>
        <name>yarn.log-aggregation-enable</name>
        <value>true</value>
    </property>

    <property>
        <name>yarn.application.classpath</name>
        <value>/home/hadoop/hadoop-3.3.4/etc/hadoop:/home/hadoop/hadoop-3.3.4/share/hadoop/common/lib/*:/home/hadoop/hadoop-3.3.4/share/hadoop/common/*:/home/hadoop/hadoop-3.3.4/share/hadoop/hdfs:/home/hadoop/hadoop-3.3.4/share/hadoop/hdfs/lib/*:/home/hadoop/hadoop-3.3.4/share/hadoop/hdfs/*:/home/hadoop/hadoop-3.3.4/share/hadoop/mapreduce/*:/home/hadoop/hadoop-3.3.4/share/hadoop/yarn:/home/hadoop/hadoop-3.3.4/share/hadoop/yarn/lib/*:/home/hadoop/hadoop-3.3.4/share/hadoop/yarn/*</value>
    </property>
    
    <property>
        <name>yarn.resourcemanager.webapp.address</name>
        <value>0.0.0.0:8088</value>
    </property>      

</configuration>
```

Y finalmente el archivo `mapred-site.xml` para indicar que utilice YARN como framework MapReduce:

```xml title="mapred-site.xml"
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```
_Si prefieres usar la versión anterior y ejecutar MapReduce sobre HDFS, elimina esta propiedad_


Levantamos YARN

```
hadoop@hadoop-VirtualBox:~/hadoop-3.3.4/sbin$ start-yarn.sh 
Starting resourcemanager
Starting nodemanagers
```

Al habilitar en la configuración la webui de Yarn, podemos acceder a su API Web en el puerto `8088`. Podemos acceder a 2 versiones de WebUI.

1. Versión antigua de Hadoop Yarn. En mi caso

```
http://bda-iesgrancapitan:8088/cluster
```

<figure style="align: center; width:600px;">
    <img src="images/Figura6.6_Yarn_WebUI_YARN_Antigua.jpg">
    <figcaption>Figura 6.6_Yarn: WebUI YARN Antigua</figcaption>
</figure>

2. Versión actual de Hadoop Yarn. En mi caso:

```
http://bda-iesgrancapitan:8088/ui2 
```
<figure style="align: center; width:600px;">
    <img src="images/Figura6.7_Yarn_WebUI_YARN_Nueva.jpg">
    <figcaption>Figura 6.7_Yarn: WebUI YARN Nueva</figcaption>
</figure>

Ya podemos acceder a la interfaz de YARN. Además de en el log, podemos observar aquí todos los trabajos que se van realizando. Lo comprobaremos cuando lancemos alguna aplicación MapReduce en la siguiente parte del tema

[figura6.1_Yarn]: images/Figura6.1_Yarn_Yarn_en_Hadoop2.jpg "Figura6.1_Yarn: Yarn en Hadoop2"

[figura6.2_Yarn]: images/Figura6.2_Yarn_Estructura_YARN_Hadoop.jpg "Figura6.2_Yarn: Estructura YARN Hadoop"

[figura6.3_Yarn]: images/Figura6.3_Yarn_Nodos_Servicios_YARN.jpg "Figura6.3_Yarn: Nodos y Servicios YARN"

[figura6.4_Yarn]: images/Figura6.4_Yarn_Arquitectura_Yarn.png "Figura6.4_Yarn: Arquitectura YARN Hadoop"

[figura6.5_Yarn]: images/Figura6.5_Yarn_Funcionamiento_YARN.jpg "Figura6.5_Yarn: Funcionamiento YARN Hadoop"

[figura6.6_Yarn]: images/Figura6.6_Yarn_WebUI_YARN.jpg "Figura6.6_Yarn: WebUI YARN Antigua"

[figura6.7_Yarn]: images/Figura6.7_Yarn_WebUI_YARN_Nueva.jpg "Figura6.7_Yarn: WebUI YARN Nueva"