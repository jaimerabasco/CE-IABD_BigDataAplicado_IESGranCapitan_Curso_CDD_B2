# UD 4 - Apache Hadoop - Cluster

En este recurso vamos a explicar como se instala y configura un cluster con Apache Hadoop

## 1. Prerequisitos

Debemos tener instalado VirtualBox. 

### 1.1 Configurar Red NAT

Para crear nuestro cluster, vamos a configurar un **red NAT** para que los nodos tenga conexión entre ellos y salida a Internet a través del Host

Para ello, explicamos con una imagen como funciona VirtualBox en este tipo de configuración de red

<figure style="align: center;">
    <img src="images/Figura4.1_ClusterHadoop_RedNAT_Virtualbox.webp">
    <figcaption>Figura 1 Cluster Hadoop: Red NAT Virtualbox - Fuente: medium.com/@sidlors</figcaption>
</figure>

Puedes observar que podemos configurar nuestra propia subred, dentro de las cuales, hay 2 ips que VirtualBox asigna estáticas dentro de la red: la **puerta de enlace**(primera de la red) y el **DHCP** (tercera de la red). Para más información, consulta la [documentación oficial de VirtualBox](https://www.virtualbox.org/manual/UserManual.html#network_nat_service)

Teniendo en cuenta esto, vamos a configurar nuestra propia subred, que será la `192.168.11.0/24`

1. Abrimos la configuración de VirtualBox para crear una nueva red NAT en `Preferencias -> Redes -> Redes NAT -> Agregar nueva red NAT`

2. Creamos una nueva Red NAT llamada `BDA`. Usaremos la red `192.168.11.0/24` con DHCP Deshabilitado. Puedes elegir cualquier otra si quieres.

<figure style="align: center;">
    <img src="images/Figura4.2_ClusterHadoop_RedNAT.jpg">
    <figcaption>Figura 2 Cluster Hadoop: Red NAT</figcaption>
</figure>

3. Una vez configurada la Red NAT, podemos empezar a instalar y configurar el cluster.

### 1.2 Configuración de las máquinas

Vamos a crear una primera máquina que después clonaremos y cambiaremos las configuraciones necesarias. Las máquinas tendrán la siguiente configuración (***siempre que sea posible***):

- **Nombre**: master (las otras dos máquinas se llamarán nodo1, nodo2 y nodo3)
- **RAM**: 4GB (_yo por ejemplo, no puedo darle más de 3GB_)
- **Núcleos**: 2
- **Disco duro**: 50GB
- **Interfaz de red**: Red NAT ("BDA"): 192.0.11.10 (las s de las otras 3 máquinas serán 192.0.11.11, 192.0.11.12 y 192.0.11.13)
- **Sistema operativo**: Ubuntu server 22.04
- **Usuario**: hadoop

### 1.3 Configuración de red 

Habiendo entendido correctamente lo explicado en los puntos anteriores,podemos configurar la interfaz de red de forma manual, con la siguiente configuración:

- **Subred**: `192.168.11.0/24`
- **Direción Ip**: `192.168.11.10`
- **Puerta de enlace**: `192.168.11.1` 
- **DNS**: `8.8.8.8,1.1.1.1`

<figure style="align: center;">
    <img src="images/Figura4.3_ClusterHadoop_InterfazRedMaster.jpg">
    <figcaption>Figura 3 Cluster Hadoop: Interfaz de Red Nodo Master</figcaption>
</figure>


## 2. Nodo Master

Creamos en VirtualBox el nodo `master` configurando el Interfaz de red como Red NAT y elegimos la que acabamos de crear `BDA`

### 2.1 Instalación

1. **Java™** debe ser instalado. Las versiones de Java recomendadas se encuentran descritas en [HadoopJavaVersions](https://cwiki.apache.org/confluence/display/HADOOP/Hadoop+Java+Versions).

```bash
sudo apt-get install openjdk-8-jdk
/usr/bin/java -version
```

2. **ssh** debe estar instalado y sshd debe estar ejecutándose para usar las secuencias de comandos de Hadoop que administran los demonios ssh remotos de Hadoop, ya que vamos a usar las secuencias de comandos de inicio y detección opcionales.

```bash
sudo apt-get install ssh
```

3. Abre una terminal

4. Para obtener la distribución de Apache Hadoop, descarga la versión estable más reciente desde [Apache Download Mirrors](https://www.apache.org/dyn/closer.cgi/hadoop/common/)

```bash
wget https://dlcdn.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz
```

5. Una vez descargado, desempaquetamos el archivo descargado con el comando tar y entra dentro de la carpeta:

```bash
sudo tar -xzf hadoop-3.3.6.tar.gz -C /opt
cd /opt/hadoop-3.3.6
```

6. Edita el siguiente archivo `etc/hadoop/hadoop-env.sh` para definir la variable de entorno de Java y añádela.

```bash
# Technically, the only required environment variable is JAVA_HOME.
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/
```

7. Para poder usar los comandos de HDFS en cualquier lugar del sistema, sin tener que hacerlo desde el directorio de Hadoop (por ejemplo `/opt/hadoop-3.3.6/bin`), creamos las variables de entorno y añadimos al PATH. Para ello abrimos el archivo `~/.bashrc` y añadimos al final el siguiente código y ejecuta el comando `source ~/.bashrc`

```bash title="~/.bashrc"
export HADOOP_HOME=/opt/hadoop-3.3.6
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export HADOOP_YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"
```

8. Ejecuta el siguiente comando. Si no da error, podemos continuar

```bash
hadoop version
```

Nos debe salir la versión de hadoop

```bash
Hadoop 3.3.6
Source code repository https://github.com/apache/hadoop.git -r 1be78238728da9266a4f88195058f08fd012bf9c
Compiled by ubuntu on 2023-06-18T08:22Z
Compiled on platform linux-x86_64
Compiled with protoc 3.7.1
From source with checksum 5652179ad55f76cb287d9c633bb53bbd
This command was run using /opt/hadoop-3.3.6/share/hadoop/common/hadoop-common-3.3.6.jar
```

### 7.2 Configuración (Pseudo-Distributed Operation)

Hadoop se puede ejecutar en un solo nodo en un modo pseudo-distributed donde cada demonio de Hadoop se ejecuta en un proceso Java separado.

Los archivos que vamos a revisar a continuación se encuentran dentro de la carpeta `$HADOOP_HOME/etc/hadoop`.

1. El archivo que contiene la configuración general del clúster es el archivo `core-site.xml`. En él se configura cual será el sistema de ficheros, que normalmente será hdfs, indicando el dominio del nodo que será el maestro de datos (namenode) de la arquitectura. Podéis sustituir el nombre del dominio `bda-iesgrancapitan` por el que queráis

```xml title="core-site.xml"
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://bda-iesgrancapitan:9000</value>
    </property>
</configuration>
```

2. El siguiente paso es configurar el archivo `hdfs-site.xml` donde se indica tanto el factor de replica como la ruta donde se almacenan tanto los metadatos (namenode) como los datos en sí (datanode):

```xml title="hdfs-site.xml"
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```

3. **Opcional:** Si quieres especificar la ruta donde se almacenan los metadatos(namenode) y los datos(datanode) donde el propio hadoop los configura por defecto puedes hacerlo cambiando dichos parámetros correspondientes. Todos lo parámetros por defecto susceptibles de cambio se encuentran en este [recurso](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml)

!!! note inline end

    Si tuviésemos un clúster, en el nodo maestro sólo configuraríamos la ruta del namenode y en cada uno de los nodos esclavos, únicamente la ruta del datanode.

```xml title="hdfs-site.xml"
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>

    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/opt/hadoop/hadoop_data/hdfs/namenode</value>
    </property>

    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/opt/hadoop/hadoop_data/hdfs/datanode</value>
    </property>
</configuration>
```

4. Crea los directorios de `hadoop-data` configurados anteriormente en `hdfs-site.xml` para cuando ejecutemos **hadoop** y configura los permisos oportunos.

```bash
sudo mkdir -p /opt/hadoop
sudo chown -R hadoop:hadoop /opt/hadoop
```

5. Comprobamos que podemos entrar por ssh al localhost sin un passphrase:

```bash
ssh localhost
exit //Si hemos podido acceder
```

6. Si no puedes entrar por ssh al localhost sin un passphrase, ejecuta los siguientes comandos:

```bash
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys
```

7. Añade a `/etc/hosts`el nombre de tu dominio indicado en `core-site.xml` para que no te de error de resolución de nombres. En mi caso añado la siguiente linea y reinicia el servicio:

```bash
127.0.0.1   bda-iesgrancapitan
```

### 7.3 Ejecución

1. Ejecuta el siguiente comando

```bash
hdfs namenode -format
```

2. Debería darte una salida como la siguiente

```bash hl_lines="7 11"
WARNING: /opt/hadoop-3.3.6/logs does not exist. Creating.
2023-11-27 11:51:42,564 INFO namenode.NameNode: STARTUP_MSG: 
/************************************************************
STARTUP_MSG: Starting NameNode
STARTUP_MSG:   host = hadoop-VirtualBox/127.0.1.1
STARTUP_MSG:   args = [-format]
STARTUP_MSG:   version = 3.3.6
......
......
2023-11-27 11:51:44,678 INFO namenode.FSImage: Allocated new BlockPoolId: BP-693156123-127.0.1.1-1701082304668
2023-11-27 11:51:45,014 INFO common.Storage: Storage directory /opt/hadoop/hadoop_data/hdfs/namenode has been successfully formatted.
2023-11-27 11:51:45,164 INFO namenode.FSImageFormatProtobuf: Saving image file /opt/hadoop/hadoop_data/hdfs/namenode/current/fsimage.ckpt_0000000000000000000 using no compression
2023-11-27 11:51:45,253 INFO namenode.FSImageFormatProtobuf: Image file /opt/hadoop/hadoop_data/hdfs/namenode/current/fsimage.ckpt_0000000000000000000 of size 401 bytes saved in 0 seconds .
2023-11-27 11:51:45,386 INFO namenode.NNStorageRetentionManager: Going to retain 1 images with txid >= 0
2023-11-27 11:51:45,430 INFO namenode.FSNamesystem: Stopping services started for active state
2023-11-27 11:51:45,431 INFO namenode.FSNamesystem: Stopping services started for standby state
2023-11-27 11:51:45,452 INFO namenode.FSImage: FSImageSaver clean checkpoint: txid=0 when meet shutdown.
2023-11-27 11:51:45,452 INFO namenode.NameNode: SHUTDOWN_MSG
```

3. Iniciando el demonio Namenode y Datanode

```bash
start-dfs.sh
```

4. Debería darte una salida como la siguiente

```bash
hadoop@hadoop-VirtualBox:/opt/hadoop-3.3.6$ start-dfs.sh 
Starting namenodes on [bda-iesgrancapitan]
Starting datanodes
Starting secondary namenodes [hadoop-VirtualBox]
hadoop@hadoop-VirtualBox:/opt/hadoop-3.3.6$ jps
16401 Jps
15970 NameNode
16085 DataNode
16283 SecondaryNameNode
```

5. Accede desde el navegador a `http://bda-iesgrancapitan:9870/` para acceder al interfaz web de HDFS


<figure style="align: center;">
    <img src="images/Figura4.1_InstalandoHDFS_InterfazWeb.jpg">
    <figcaption>Figura 1 Instalando HDFS: Interfaz Web</figcaption>
</figure>


### 7.4 Usando HDFS

Vamos a investigar cuál es el funcionamiento interno de HDFS estudiado en teoría.

Para ello vamos a añadir a HDFS un [fichero de gran volumen](https://files.grouplens.org/datasets/tag-genome-2021/). Accede al enlace y descarga el archivo [genome_2021.zip](https://files.grouplens.org/datasets/tag-genome-2021/genome_2021.zip)

1. Descargamos el archivo en el sistema de archivos local

```bash
wget https://files.grouplens.org/datasets/tag-genome-2021/genome_2021.zip
```
2. Vamos a observar la salida de logs en cada uno de los siguientes pasos, que nos va a servir para afianzar como como funciona HDFS. Observamos el log del namenode. En mi caso:

```bash
tail -f $HADOOP_HOME/logs/hadoop-hadoop-namenode-hadoop-VirtualBox.log
```

3. Lo añadimos a HDFS

```bash
hdfs dfs -copyFromLocal genome_2021.zip /
```

La salida del log nos indica la división en bloques y la adición de la transacción en el EditLog ()

```bash hl_lines="1 6 30 36"
2023-11-27 11:58:19,590 INFO org.apache.hadoop.hdfs.server.namenode.FileJournalManager: Finalizing edits file /opt/hadoop/hadoop_data/hdfs/namenode/current/edits_inprogress_0000000000000000001 -> /opt/hadoop/had
oop_data/hdfs/namenode/current/edits_0000000000000000001-0000000000000000002
2023-11-27 11:58:19,608 INFO org.apache.hadoop.hdfs.server.namenode.FSEditLog: Starting log segment at 3
2023-11-27 11:58:20,224 INFO org.apache.hadoop.hdfs.server.namenode.TransferFsImage: Sending fileName: /opt/hadoop/hadoop_data/hdfs/namenode/current/fsimage_0000000000000000000, fileSize: 401. Sent total: 401 by
tes. Size of last segment intended to send: -1 bytes.
2023-11-27 11:58:20,496 INFO org.apache.hadoop.hdfs.server.namenode.TransferFsImage: Sending fileName: /opt/hadoop/hadoop_data/hdfs/namenode/current/edits_0000000000000000001-0000000000000000002, fileSize: 42. S
ent total: 42 bytes. Size of last segment intended to send: -1 bytes.
2023-11-27 11:58:21,070 INFO org.apache.hadoop.hdfs.server.namenode.ImageServlet: Rejecting a fsimage due to small time delta and txnid delta. Time since previous checkpoint is 395 expecting at least 2700 txnid 
delta since previous checkpoint is 2 expecting at least 1000000
2023-11-27 12:37:53,007 INFO org.apache.hadoop.hdfs.server.namenode.FSEditLog: Number of transactions: 2 Total time for transactions(ms): 83 Number of transactions batched in Syncs: 0 Number of syncs: 2 SyncTime
s(ms): 217 
2023-11-27 12:37:53,265 INFO org.apache.hadoop.hdfs.StateChange: BLOCK* allocate blk_1073741825_1001, replicas=127.0.0.1:9866 for /genome_2021.zip._COPYING_
2023-11-27 12:37:55,015 INFO org.apache.hadoop.hdfs.StateChange: BLOCK* allocate blk_1073741826_1002, replicas=127.0.0.1:9866 for /genome_2021.zip._COPYING_
2023-11-27 12:37:58,097 INFO org.apache.hadoop.hdfs.StateChange: BLOCK* allocate blk_1073741827_1003, replicas=127.0.0.1:9866 for /genome_2021.zip._COPYING_
2023-11-27 12:38:00,850 INFO org.apache.hadoop.hdfs.StateChange: BLOCK* allocate blk_1073741828_1004, replicas=127.0.0.1:9866 for /genome_2021.zip._COPYING_
2023-11-27 12:38:03,102 INFO org.apache.hadoop.hdfs.StateChange: BLOCK* allocate blk_1073741829_1005, replicas=127.0.0.1:9866 for /genome_2021.zip._COPYING_
2023-11-27 12:38:07,189 INFO org.apache.hadoop.hdfs.StateChange: BLOCK* allocate blk_1073741830_1006, replicas=127.0.0.1:9866 for /genome_2021.zip._COPYING_
2023-11-27 12:38:09,171 INFO org.apache.hadoop.hdfs.StateChange: BLOCK* allocate blk_1073741831_1007, replicas=127.0.0.1:9866 for /genome_2021.zip._COPYING_
2023-11-27 12:38:13,229 INFO org.apache.hadoop.hdfs.StateChange: BLOCK* allocate blk_1073741832_1008, replicas=127.0.0.1:9866 for /genome_2021.zip._COPYING_
2023-11-27 12:38:16,796 INFO org.apache.hadoop.hdfs.StateChange: BLOCK* allocate blk_1073741833_1009, replicas=127.0.0.1:9866 for /genome_2021.zip._COPYING_
2023-11-27 12:38:18,812 INFO org.apache.hadoop.hdfs.StateChange: BLOCK* allocate blk_1073741834_1010, replicas=127.0.0.1:9866 for /genome_2021.zip._COPYING_
2023-11-27 12:38:22,466 INFO org.apache.hadoop.hdfs.StateChange: BLOCK* allocate blk_1073741835_1011, replicas=127.0.0.1:9866 for /genome_2021.zip._COPYING_
2023-11-27 12:38:26,859 INFO org.apache.hadoop.hdfs.StateChange: BLOCK* allocate blk_1073741836_1012, replicas=127.0.0.1:9866 for /genome_2021.zip._COPYING_
2023-11-27 12:38:28,264 INFO org.apache.hadoop.hdfs.StateChange: BLOCK* allocate blk_1073741837_1013, replicas=127.0.0.1:9866 for /genome_2021.zip._COPYING_
2023-11-27 12:38:31,502 INFO org.apache.hadoop.hdfs.StateChange: BLOCK* allocate blk_1073741838_1014, replicas=127.0.0.1:9866 for /genome_2021.zip._COPYING_
2023-11-27 12:38:33,935 INFO org.apache.hadoop.hdfs.StateChange: BLOCK* allocate blk_1073741839_1015, replicas=127.0.0.1:9866 for /genome_2021.zip._COPYING_
2023-11-27 12:38:36,202 INFO org.apache.hadoop.hdfs.StateChange: DIR* completeFile: /genome_2021.zip._COPYING_ is closed by DFSClient_NONMAPREDUCE_-294249091_1
2023-11-27 12:43:41,454 INFO org.apache.hadoop.http.HttpServer2: Process Thread Dump: jsp requested
2023-11-27 12:58:22,096 INFO org.apache.hadoop.hdfs.server.namenode.FSNamesystem: Roll Edit Log from 127.0.0.1
2023-11-27 12:58:22,096 INFO org.apache.hadoop.hdfs.server.namenode.FSEditLog: Rolling edit logs
2023-11-27 12:58:22,096 INFO org.apache.hadoop.hdfs.server.namenode.FSEditLog: Ending log segment 3, 51
2023-11-27 12:58:22,096 INFO org.apache.hadoop.hdfs.server.namenode.FSEditLog: Number of transactions: 50 Total time for transactions(ms): 83 Number of transactions batched in Syncs: 27 Number of syncs: 23 SyncTimes(ms): 18260 
2023-11-27 12:58:22,164 INFO org.apache.hadoop.hdfs.server.namenode.FSEditLog: Number of transactions: 50 Total time for transactions(ms): 83 Number of transactions batched in Syncs: 27 Number of syncs: 24 SyncTimes(ms): 18327 
2023-11-27 12:58:22,164 INFO org.apache.hadoop.hdfs.server.namenode.FileJournalManager: Finalizing edits file /opt/hadoop/hadoop_data/hdfs/namenode/current/edits_inprogress_0000000000000000003 -> /opt/hadoop/hadoop_data/hdfs/namenode/current/edits_0000000000000000003-0000000000000000052
2023-11-27 12:58:22,178 INFO org.apache.hadoop.hdfs.server.namenode.FSEditLog: Starting log segment at 53
2023-11-27 12:58:22,546 INFO org.apache.hadoop.hdfs.server.namenode.TransferFsImage: Sending fileName: /opt/hadoop/hadoop_data/hdfs/namenode/current/fsimage_0000000000000000000, fileSize: 401. Sent total: 401 bytes. Size of last segment intended to send: -1 bytes.
2023-11-27 12:58:22,796 INFO org.apache.hadoop.hdfs.server.namenode.TransferFsImage: Sending fileName: /opt/hadoop/hadoop_data/hdfs/namenode/current/edits_0000000000000000003-0000000000000000052, fileSize: 2741. Sent total: 2741 bytes. Size of last segment intended to send: -1 bytes.
2023-11-27 12:58:23,463 INFO org.apache.hadoop.hdfs.server.common.Util: Combined time for file download and fsync to all disks took 0,11s. The file download took 0,00s at 0,00 KB/s. Synchronous (fsync) write to disk of /opt/hadoop/hadoop_data/hdfs/namenode/current/fsimage.ckpt_0000000000000000052 took 0,11s.
2023-11-27 12:58:23,466 INFO org.apache.hadoop.hdfs.server.namenode.TransferFsImage: Downloaded file fsimage.ckpt_0000000000000000052 size 740 bytes.
2023-11-27 12:58:23,580 INFO org.apache.hadoop.hdfs.server.namenode.NNStorageRetentionManager: Going to retain 2 images with txid >= 0
```

<figure style="align: center;">
    <img src="images/Figura4.2_InstalandoHDFS_SecondaryNamenode_to_Namenode.jpg">
    <figcaption>Figura 2 Instalando HDFS: SecondaryNamenode y Namenode</figcaption>
</figure>

4. Como puedes observar en el log, se generan un conjunto de ficheros en la carpeta `current`, que continen un conjunto de ficheros cuyos prefijos son:
   
   - _edits_000NNN_: histórico de cambios que se van produciendo.
   - _edits_inprogress_NNN_: cambios actuales en memoria que no se han persistido.
   - _fsimagen_000NNN_: snapshot en el tiempo del sistema de ficheros.
   
5. Si accedes a la carpeta HDFS `/opt/hadoop/hadoop_data/hdfs/namenode/current` desde nuestro sistema de archivos, puedes observarlos también

```bash
hadoop@hadoop-VirtualBox:/opt/hadoop/hadoop_data/hdfs/namenode/current$ ls
edits_0000000000000000001-0000000000000000002
edits_0000000000000000003-0000000000000000052
edits_inprogress_0000000000000000053
fsimage_0000000000000000000
fsimage_0000000000000000000.md5
fsimage_0000000000000000052
fsimage_0000000000000000052.md5
seen_txid
VERSION
```

6. Por otro lado, si accedemos a la carpeta HDFS `/opt/hadoop/hadoop_data/hdfs/datanode` desde nuestro sistema de archivos, y entramos dentro de su subdirectorio creado después de la transacción, también podemos observar la generación de los diferentes bloques

```bash
hadoop@hadoop-VirtualBox:/opt/hadoop/hadoop_data/hdfs/datanode/current/BP-693156123-127.0.1.1-1701082304668/current/finalized/subdir0/subdir0$ ls -l 
total 1897632
-rw-rw-r-- 1 hadoop hadoop 134217728 nov 27 12:37 blk_1073741825
-rw-rw-r-- 1 hadoop hadoop   1048583 nov 27 12:37 blk_1073741825_1001.meta
-rw-rw-r-- 1 hadoop hadoop 134217728 nov 27 12:37 blk_1073741826
-rw-rw-r-- 1 hadoop hadoop   1048583 nov 27 12:37 blk_1073741826_1002.meta
-rw-rw-r-- 1 hadoop hadoop 134217728 nov 27 12:38 blk_1073741827
-rw-rw-r-- 1 hadoop hadoop   1048583 nov 27 12:38 blk_1073741827_1003.meta
-rw-rw-r-- 1 hadoop hadoop 134217728 nov 27 12:38 blk_1073741828
-rw-rw-r-- 1 hadoop hadoop   1048583 nov 27 12:38 blk_1073741828_1004.meta
-rw-rw-r-- 1 hadoop hadoop 134217728 nov 27 12:38 blk_1073741829
-rw-rw-r-- 1 hadoop hadoop   1048583 nov 27 12:38 blk_1073741829_1005.meta
-rw-rw-r-- 1 hadoop hadoop 134217728 nov 27 12:38 blk_1073741830
-rw-rw-r-- 1 hadoop hadoop   1048583 nov 27 12:38 blk_1073741830_1006.meta
-rw-rw-r-- 1 hadoop hadoop 134217728 nov 27 12:38 blk_1073741831
-rw-rw-r-- 1 hadoop hadoop   1048583 nov 27 12:38 blk_1073741831_1007.meta
-rw-rw-r-- 1 hadoop hadoop 134217728 nov 27 12:38 blk_1073741832
-rw-rw-r-- 1 hadoop hadoop   1048583 nov 27 12:38 blk_1073741832_1008.meta
-rw-rw-r-- 1 hadoop hadoop 134217728 nov 27 12:38 blk_1073741833
-rw-rw-r-- 1 hadoop hadoop   1048583 nov 27 12:38 blk_1073741833_1009.meta
-rw-rw-r-- 1 hadoop hadoop 134217728 nov 27 12:38 blk_1073741834
-rw-rw-r-- 1 hadoop hadoop   1048583 nov 27 12:38 blk_1073741834_1010.meta
-rw-rw-r-- 1 hadoop hadoop 134217728 nov 27 12:38 blk_1073741835
-rw-rw-r-- 1 hadoop hadoop   1048583 nov 27 12:38 blk_1073741835_1011.meta
-rw-rw-r-- 1 hadoop hadoop 134217728 nov 27 12:38 blk_1073741836
-rw-rw-r-- 1 hadoop hadoop   1048583 nov 27 12:38 blk_1073741836_1012.meta
-rw-rw-r-- 1 hadoop hadoop 134217728 nov 27 12:38 blk_1073741837
-rw-rw-r-- 1 hadoop hadoop   1048583 nov 27 12:38 blk_1073741837_1013.meta
-rw-rw-r-- 1 hadoop hadoop 134217728 nov 27 12:38 blk_1073741838
-rw-rw-r-- 1 hadoop hadoop   1048583 nov 27 12:38 blk_1073741838_1014.meta
-rw-rw-r-- 1 hadoop hadoop  48980391 nov 27 12:38 blk_1073741839
-rw-rw-r-- 1 hadoop hadoop    382667 nov 27 12:38 blk_1073741839_1015.meta
```

7. Comprobamos toda esta información y mucha más adicional a través de la interfaz web de HDFS `http://bda-iesgrancapitan:9870/` (que es mayor que la que vimos con Cloudera, cuya versión de Hadoop es inferior)

### 7.5 Administración

HDFS también permite administración desde linea de comandos. El más usado es la opción `hdfs dfsadmin`

Puedes ver todas las opciones en la [documentación oficial](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HDFSCommands.html#dfsadmin).

```bash
hadoop@hadoop-VirtualBox:~$ hdfs dfsadmin
Usage: hdfs dfsadmin
Note: Administrative commands can only be run as the HDFS superuser.
	[-report [-live] [-dead] [-decommissioning] [-enteringmaintenance] [-inmaintenance] [-slownodes]]
	[-safemode <enter | leave | get | wait | forceExit>]
	[-saveNamespace [-beforeShutdown]]
	[-rollEdits]
	[-restoreFailedStorage true|false|check]
	[-refreshNodes]
	[-setQuota <quota> <dirname>...<dirname>]
	[-clrQuota <dirname>...<dirname>]
	[-setSpaceQuota <quota> [-storageType <storagetype>] <dirname>...<dirname>]
	[-clrSpaceQuota [-storageType <storagetype>] <dirname>...<dirname>]
	[-finalizeUpgrade]
	[-rollingUpgrade [<query|prepare|finalize>]]
	[-upgrade <query | finalize>]
	[-refreshServiceAcl]
	[-refreshUserToGroupsMappings]
	[-refreshSuperUserGroupsConfiguration]
	[-refreshCallQueue]
	[-refresh <host:ipc_port> <key> [arg1..argn]
	[-reconfig <namenode|datanode> <host:ipc_port|livenodes> <start|status|properties>]
	[-printTopology]
	[-refreshNamenodes datanode_host:ipc_port]
	[-getVolumeReport datanode_host:ipc_port]
	[-deleteBlockPool datanode_host:ipc_port blockpoolId [force]]
	[-setBalancerBandwidth <bandwidth in bytes per second>]
	[-getBalancerBandwidth <datanode_host:ipc_port>]
	[-fetchImage <local directory>]
	[-allowSnapshot <snapshotDir>]
	[-disallowSnapshot <snapshotDir>]
	[-shutdownDatanode <datanode_host:ipc_port> [upgrade]]
	[-evictWriters <datanode_host:ipc_port>]
	[-getDatanodeInfo <datanode_host:ipc_port>]
	[-metasave filename]
	[-triggerBlockReport [-incremental] <datanode_host:ipc_port> [-namenode <namenode_host:ipc_port>]]
	[-listOpenFiles [-blockingDecommission] [-path <path>]]
	[-help [cmd]]

Generic options supported are:
-conf <configuration file>        specify an application configuration file
-D <property=value>               define a value for a given property
-fs <file:///|hdfs://namenode:port> specify default filesystem URL to use, overrides 'fs.defaultFS' property from configurations.
-jt <local|resourcemanager:port>  specify a ResourceManager
-files <file1,...>                specify a comma-separated list of files to be copied to the map reduce cluster
-libjars <jar1,...>               specify a comma-separated list of jar files to be included in the classpath
-archives <archive1,...>          specify a comma-separated list of archives to be unarchived on the compute machines

The general command line syntax is:
command [genericOptions] [commandOptions]
```

Vamos a probar algunas de ellas:

- `hdfs dfsadmin -report`: Realiza un resumen del sistema HDFS, donde podemos comprobar el estado de los diferentes nodos. Es similar al que aparece en el interfaz web.
- `hdfs dfsadmin -listOpenFiles`: Comprueba si hay algún fichero abierto.
- `hdfs dfsadmin -printTopology`: Muestra la topología, identificando los nodos que tenemos y al rack al que pertenece cada nodo.
- `hdfs dfsadmin -safemode enter`: Pone el sistema en modo seguro, el cual evita la modificación de los recursos del sistema de archivos.
- `hdfs dfsadmin -safemode leave`: Sale del modo seguro.

Otro ejemplo:

- `hdfs fsck`: Comprueba el estado del sistema de ficheros. Si queremos comprobar el estado de un determinado directorio, lo indicamos mediante un segundo parámetro: `hdfs fsck /`

```bash
hadoop@hadoop-VirtualBox:~$ hdfs fsck /
onnecting to namenode via http://bda-iesgrancapitan:9870/fsck?ugi=hadoop&path=%2F
FSCK started by hadoop (auth:SIMPLE) from /127.0.0.1 for path / at Mon Nov 27 13:14:05 CET 2023


Status: HEALTHY
 Number of data-nodes:	1
 Number of racks:		1
 Total dirs:			1
 Total symlinks:		0

Replicated Blocks:
 Total size:	1928028583 B
 Total files:	1
 Total blocks (validated):	15 (avg. block size 128535238 B)
 Minimally replicated blocks:	15 (100.0 %)
 Over-replicated blocks:	0 (0.0 %)
 Under-replicated blocks:	0 (0.0 %)
 Mis-replicated blocks:		0 (0.0 %)
 Default replication factor:	1
 Average block replication:	1.0
 Missing blocks:		0
 Corrupt blocks:		0
 Missing replicas:		0 (0.0 %)
 Blocks queued for replication:	0

Erasure Coded Block Groups:
 Total size:	0 B
 Total files:	0
 Total block groups (validated):	0
 Minimally erasure-coded block groups:	0
 Over-erasure-coded block groups:	0
 Under-erasure-coded block groups:	0
 Unsatisfactory placement block groups:	0
 Average block group size:	0.0
 Missing block groups:		0
 Corrupt block groups:		0
 Missing internal blocks:	0
 Blocks queued for replication:	0
FSCK ended at Mon Nov 27 13:14:06 CET 2023 in 13 milliseconds


The filesystem under path '/' is HEALTHY
```

También existen otros comandos interesantes como: `balancer`, `cacheadmin`, `datanode`, `namenode`,... 

Puedes consultar la lista completa en la [documentación oficial](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HDFSCommands.html#Administration_Commands)


### 7.6 Snapshots

Mediante ***Snapshots*** podemos guardar la instantánea de como se encuentra todo nuestros datos dentro del sistema de ficheros, que puede servir como copia de seguridad, para un futuro backup.

Vamos a realizar un ejemplo. Creamos un directorio dentro de nuestro HDFS y copiamos nuestro fichero de genoma 2021 dentro de él:

```bash
hdfs dfs -mkdir /bda
hdfs dfs -cp /genome_2021.zip /bda
hdfs dfs -ls /bda
```
Activamos el uso de snapshot en el directorio que queramos obtener una instantánea:

```bash
hdfs dfsadmin -allowSnapshot /bda
```

Procedemos a crear una instantánea indicando la carpeta y el nombre que va a tener

```bash
hdfs dfs -createSnapshot /bda snapshot_bda_1
```

Se crea una carpeta oculta dentro de la carpeta que contendrá la información `/bda/.snapshot/snapshot_bda_1`

Puedes verlo también desde la interfaz web de HDFS en su apartado de Snapshot

<figure style="align: center;">
    <img src="images/Figura4.3_InstalandoHDFS_Snapshot.jpg">
    <figcaption>Figura 3 Instalando HDFS: Snapshot</figcaption>
</figure>

Vamos a borrar el archivo que hemos copiado y comprobamos

```bash
hdfs dfs -rm /bda/genome_2021.zip
//Deleted /bda/genome_2021.zip
hdfs dfs -ls /bda/
```
Para recuperar el fichero usamos el snapshot creado anteriormente

```bash
hdfs dfs -cp /bda/.snapshot/snapshot_bda_1/genome_2021.zip /bda/genome_2021.zip
```

Para comprobar los directorios que actualmente soportan snapshot hacemos un ls de los mismos con su comando correspondiente:

```bash
hdfs lsSnapshottableDir
```

Por último, para borrar un snapshot:

```bash
hdfs dfs -deleteSnapshot /bda snapshot_bda_1
```

Y si queremos desabilitarlo los snapshot:

```bash
hdfs dfsadmin -disallowSnapshot /bda
```

### 7.7 Navegación WebUI HDFS

Desde el apartado `Browser Directory` del Web IU `http://bda-iesgrancapitan:9870/explorer.html` podemos acceder al sistema de ficheros y su contenido de HDFS, incluidos los bloques.

<figure style="align: center;">
    <img src="images/Figura4.4_InstalandoHDFS_Navegacion_WebUI.jpg">
    <figcaption>Figura 4 Instalando HDFS: Navegación WebUI</figcaption>
</figure>


### 7.8 Permisos WebUI HDFS

Como hemos comentado en el punto anterior, podemos acceder al sistema de ficheros y su contenido de HDFS. Pero si intentamos borrar algún contenido nos salta un error de permisos: `Permission denied: user=dr.who, access=WRITE, inode="/bda":hadoop:supergroup:drwxr-xr-x`. Esto es debido a que, por defecto, los recursos vía web se realizan desde el usuario `dr.who`

<figure style="align: center;">
    <img src="images/Figura4.5_InstalandoHDFS_WebUI_Permisos.jpg">
    <figcaption>Figura 5 Instalando HDFS: Permisos WebUI</figcaption>
</figure>

Para poder tener permisos para ello podemos modificar los permisos:

```bash
hdfs dfs -mkdir /bda/prueba_permisos
hdfs dfs -chmod 777 /bda/prueba_permisos
hdfs dfs -cp /bda/genome_2021.zip /bda/prueba_permisos/
//Ya podríamos borrar cualquier archivo dentro del directorio pruebas_permisos desde la WebUI
```

Otra posibilidad es modificar el archivo de configuraciónb `core-site.xml` y añadir la propiedad para modificar el usuario estático, en mi caso, el usuario `hadoop`

```xml title="core-site.xml"
<property>
    <name>hadoop.http.staticuser.user</name>
    <value>hadoop</value>
</property>
```

### 7.9 Acceso a HDFS a través de Python

Para ello, usaremos la libreria [HdfsCLI](https://pypi.org/project/hdfs/). La instalamos mediante `pip`

```bash
pip install hdfs
```

Para nuestro ejemplo, vamos a descargar un ejemplo con formato csv y añadirlo a nuestro HDFS

```bash
wget https://www.ine.es/jaxi/files/tpx/csv_bdsc/53938.csv
hdfs dfs -mkdir /bda/python
hdfs dfs -copyFromLocal 53938.csv  /bda/python/
hdfs dfs -ls /bda/python
```

[Teniendo como referencia la documentación](https://hdfscli.readthedocs.io/en/latest/quickstart.html#python-bindings), vamos a conectarnos a HDFS y copiar un archivo

Creamos un fichero python con el siguiente código:

```python
from hdfs import InsecureClient

# Datos de conexión
HDFS_HOSTNAME = 'bda-iesgrancapitan'
HDFS_PORT = 9870
HDFS_CONNECTION = f'http://{HDFS_HOSTNAME}:{HDFS_PORT}'

# En nuestro caso, al no usar Kerberos, creamos una conexión no segura
hdfs_client = InsecureClient(HDFS_CONNECTION)

#Lectura
# Leemos el fichero de '53938.csv' que tenemos en HDFS
fichero = '/bda/python/53938.csv'
with hdfs_client.read(fichero) as reader:
    texto = reader.read()

print(texto)

#Escritura
# Escribimos los elementos de la lista en formato csv
datos="dni,nombre,apellidos,direccion,cp\n"
lista = [['123', 'Nombre1', 'Apellidos1', 'Mikasa1', '14000'],
         ['456', 'Nombre4', 'Apellidos4', 'Mikasa4', '41000'],
         ['789', 'Nombre7', 'Apellidos7', 'Mikasa7', '19000']]
for i in range(0, len(lista), 1):
    for j in range(0, len(lista[i]), 1):
        if(j<len(lista[i])-1):
            datos+=f'{lista[i][j]},'
        else:
            datos+=f'{lista[i][j]}\n'
hdfs_client.write("/bda/python/datos.csv", datos)
```

Ejecutamos el fichero

```bash
python3 prueba_hdfs_Python.py
```

Comprobamos la cración del nuevo fichero

```bash
hadoop@hadoop-VirtualBox:~$ hdfs dfs -ls /bda/python/
Found 2 items
-rw-r--r--   1 hadoop supergroup      60116 2023-11-27 13:55 /bda/python/53938.csv
-rw-r--r--   1 hadoop supergroup        145 2023-11-27 13:56 /bda/python/datos.csv
```

Adicionalmente, esta librería te da [funcionalidad opcional](https://pypi.org/project/hdfs/) para `avro`, `dataframe` (con Pandas) y `Kerberos`. Estos casos son los más habituales en el mundo real.
