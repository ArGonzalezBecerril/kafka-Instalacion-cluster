Instalación y Configuración de Kafka & Zookeeper.

[![N|Solid](https://image.ibb.co/jOZM45/logo.png)](https://nodesource.com/products/nsolid)


A continuación se mostraran los paso para descargar y configurar un cluster kafka con 10 brokers y un master denominado server-dell, la estructura del cluster quedara definida de la siguiente manera.

[![N|Solid](https://image.ibb.co/mGj2u5/arquiecturakafkacluster.png)](https://nodesource.com/products/nsolid)

Donde :
- Cada slave representa un broker de kafka.
- Solo existira una instancia de zookeeper ejecutandose, especificamente en server-dell
- Server-dell no sera un broker, solo se usara para administrar el cluster.
- Adicionalmente se instalara kafka-manager para administrar el cluster kafka.
- En todos los slaves existe un usuario llamado eglobal, incluido server-dell ademas tiene un directorio /home/eglobal/Software/NewUtils/ para depositar software.


# Descargar las fuentes de kafka!
.
> Iniciare desde una terminal en mi equipo arturo@arturo y me conectare a server-dell y a los slaves mediante ssh(Para esto nuestro equipo debe tener registrados a todos los slaves en /etc/hosts, incluido server-dell y lógicamente debemos estar conectados mediante cable físico a la red del cluster.
.
```sh
arturo@arturo$ pwd
 /home/usuario/Descargas
arturo@arturo$ wget -c https://www.apache.org/dyn/closer.cgi?path=/kafka/0.11.0.0/kafka-0.11.0.0-src.tgz
arturo@arturo$ ls -llrth
rwx------ 14 arturo arturo 4.0K aug 17  2017 kafka-0.11.0.0-src.tgz
#Copiar el paquete kafka a: server-dell,slave9,slave10,slave11,slave12,slave13,slave14,slave15,slave16, slave17,slave18
arturo@arturo$ scp kafka-0.11.0.0-src.tgz  eglobal@server-dell:/home/eglobal/Software/NewUtils/
arturo@arturo$ scp kafka-0.11.0.0-src.tgz  eglobal@slave9:/home/eglobal/Software/NewUtils/
arturo@arturo$ scp kafka-0.11.0.0-src.tgz  eglobal@slave10:/home/eglobal/Software/NewUtils/
arturo@arturo$ scp kafka-0.11.0.0-src.tgz  eglobal@slave11:/home/eglobal/Software/NewUtils/
arturo@arturo$ scp kafka-0.11.0.0-src.tgz  eglobal@slave12:/home/eglobal/Software/NewUtils/
arturo@arturo$ scp kafka-0.11.0.0-src.tgz  eglobal@slave13:/home/eglobal/Software/NewUtils/
arturo@arturo$ scp kafka-0.11.0.0-src.tgz  eglobal@slave14:/home/eglobal/Software/NewUtils/
arturo@arturo$ scp kafka-0.11.0.0-src.tgz  eglobal@slave15:/home/eglobal/Software/NewUtils/
arturo@arturo$ scp kafka-0.11.0.0-src.tgz  eglobal@slave16:/home/eglobal/Software/NewUtils/
arturo@arturo$ scp kafka-0.11.0.0-src.tgz  eglobal@slave17:/home/eglobal/Software/NewUtils/
arturo@arturo$ scp kafka-0.11.0.0-src.tgz  eglobal@slave18:/home/eglobal/Software/NewUtils/
```
# Crear el directorio kafka y asignar permisos

Para todos los slaves se tiene que crear una carpeta en /opt/kafka, esto para hacer mas transparente la instalación de kafka, de esta manera los pasos siguientes se repiten en todos los nodos y en server-dell tambien.

Cabe destacar que el paquete kafka lo moví al directorio /home/eglobal/Software/NewUtils/ ya que en todos los slaves y en server-dell existe este directorio.

```sh
arturo@arturo$ ssh eglobal@slave9
 ..coneccted
eglobal@slave9$ cd /home/eglobal/Software/NewUtils/
#Elevamos nuestros privilegios para crear la carpeta kafka
eglobal@slave9$ su
password from eglobal:*****
root@slave9$ mkdir /opt/kafka
#Cambiamos el propietario de esta carpeta a eglobal, ya que sera el usuario encargado de ejecutar kafka
root@slave9$ chown -R eglobal:eglobal /opt/kafka
root@slave9$ exit
#Movemos el tar.gz a /opt/kafka
eglobal@slave9$ mv /home/eglobal/Software/NewUtils/kafka-0.11.0.0-src.tgz /opt/kafka/
eglobal@slave9$ cd /opt/kafka
eglobal@slave9$ tar -xvf kafka-0.11.0.0-src.tgz
eglobal@slave9$ cd kafka-0.11.0.0/
eglobal@slave9$ ls -lrht
-rw-r--r-- 1 eglobal eglobal  336 may 17  2016 NOTICE
-rw-r--r-- 1 eglobal eglobal  29K may 17  2016 LICENSE
drwxr-xr-x 2 eglobal eglobal 4.0K may 17  2016 site-docs
drwxr-xr-x 2 eglobal eglobal 4.0K ago  1 16:24 libs
drwxr-xr-x 3 eglobal eglobal 4.0K ago 17 14:16 bin
drwxr-xr-x 2 eglobal eglobal 4.0K ago 22 12:22 config
drwxr-xr-x 2 eglobal eglobal  28K ago 22 13:05 logs
eglobal@slave9$ cd..
eglobal@slave9$ pwd
/opt/kafka/
#Mover todas las subcarpetas de kafka-0.11.0.0 a /opt/kafka para hacer mas descriptiva la instalación
eglobal@slave9$ mv kafka-0.11.0.0/* .
#Eliminar la carpeta la cual quedo vacia.
eglobal@slave9$ rmdir kafka-0.11.0.0/
###########Repetir estos mismos pasos para todos los slaves y tambien aplicar a server-dell##########
```

# Configurar kafka en todos los slaves
En este paso se configura los siguientes archivos :
   - zookeeper.properties
   - server.properties
   
En este apartado solo vamos a configurar todos los slaves, el server-dell lo dejaremos al final ya que cambian algunas configuraciones. La configuración para cada slave es similar salvo algunas lineas que cambian.

```sh
eglobal@slave9$ pwd 
/opt/kafka
eglobal@slave9$ ls -lrht
-rw-r--r-- 1 eglobal eglobal  336 may 17  2016 NOTICE
-rw-r--r-- 1 eglobal eglobal  29K may 17  2016 LICENSE
drwxr-xr-x 2 eglobal eglobal 4.0K may 17  2016 site-docs
drwxr-xr-x 2 eglobal eglobal 4.0K ago  1 16:24 libs
drwxr-xr-x 3 eglobal eglobal 4.0K ago 17 14:16 bin
drwxr-xr-x 2 eglobal eglobal 4.0K ago 22 12:22 config
drwxr-xr-x 2 eglobal eglobal  28K ago 22 13:05 logs
eglobal@slave9$ cd config/
eglobal@slave9$ ls -rlht
-rw-r--r-- 1 eglobal eglobal 1.1K may 17  2016 tools-log4j.properties
-rw-r--r-- 1 eglobal eglobal 1.9K may 17  2016 producer.properties
-rw-r--r-- 1 eglobal eglobal 4.3K may 17  2016 log4j.properties
-rw-r--r-- 1 eglobal eglobal 1.2K may 17  2016 consumer.properties
-rw-r--r-- 1 eglobal eglobal 2.1K may 17  2016 connect-standalone.properties
-rw-r--r-- 1 eglobal eglobal 1.1K may 17  2016 connect-log4j.properties
-rw-r--r-- 1 eglobal eglobal  881 may 17  2016 connect-file-source.properties
-rw-r--r-- 1 eglobal eglobal  883 may 17  2016 connect-file-sink.properties
-rw-r--r-- 1 eglobal eglobal 2.7K may 17  2016 connect-distributed.properties
-rw-r--r-- 1 eglobal eglobal  909 may 17  2016 connect-console-source.properties
-rw-r--r-- 1 eglobal eglobal  906 may 17  2016 connect-console-sink.properties
-rw-r--r-- 1 eglobal eglobal 1023 ago 14 17:22 zookeeper.properties
-rw-r--r-- 1 eglobal eglobal 5.3K ago 17 16:10 server.properties

eglobal@slave9$ vim server.properties
#Asignar un identificador unico a cada nodo del cluster
broker.id=9 #Cada id estara relacionado con el numero de slave p.e slave10--broker.id=10
log.dirs=/mnt/hd04/kafka #Asegurarse de que todos los slaves apunten a este directorio ya que es el disco donde estaran los logs 
##Establecer el numero de particiones
num.partitions=4

replication.factor=2
num.replica.fetchers=3

offsets.topic.replication.factor=4
num.standby.replicas=2
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1
# The minimum age of a log file to be eligible for deletion due to age
log.retention.hours=1
#Servidor zookeper. Todos los slaves apuntaran a server-dell
zookeeper.connect=server-dell:2181
# Timeout in ms for connecting to zookeeper
zookeeper.connection.timeout.ms=6000

delete.topic.enable=true
port=9092
#Asegurarse que esta configuracion este en todos los slaves, cabe destacar que el unico parametro que estara cambiando es el broker.id.para cada slave. Para los demas parametros usar las mismas configuraciones.
```

### Configurar kafka en server-dell

> Importante, esta configuración solo aplica para server dell.

```sh
#Conectamos a server dell mediante ssh
arturo@arturo$ ssh eglobal@server-dell
DELL:eglobal~$ cd /opt/kafka
#Primero agregaremos al configuración a zookeeper.properties.
DELL:eglobal~$ ls -lrht
-rw-r--r-- 1 eglobal eglobal  336 may 17  2016 NOTICE
-rw-r--r-- 1 eglobal eglobal  29K may 17  2016 LICENSE
drwxr-xr-x 2 eglobal eglobal 4.0K may 17  2016 site-docs
drwxr-xr-x 2 eglobal eglobal 4.0K ago  1 16:24 libs
drwxr-xr-x 3 eglobal eglobal 4.0K ago 17 14:16 bin
drwxr-xr-x 2 eglobal eglobal 4.0K ago 22 12:22 config
drwxr-xr-x 2 eglobal eglobal  28K ago 22 13:05 logs
DELL:eglobal~$  cd config 
DELL:eglobal~$  ls -lrht
-rw-r--r-- 1 eglobal eglobal 1.1K may 17  2016 tools-log4j.properties
-rw-r--r-- 1 eglobal eglobal 1.9K may 17  2016 producer.properties
-rw-r--r-- 1 eglobal eglobal 4.3K may 17  2016 log4j.properties
-rw-r--r-- 1 eglobal eglobal 1.2K may 17  2016 consumer.properties
-rw-r--r-- 1 eglobal eglobal 2.1K may 17  2016 connect-standalone.properties
-rw-r--r-- 1 eglobal eglobal 1.1K may 17  2016 connect-log4j.properties
-rw-r--r-- 1 eglobal eglobal  881 may 17  2016 connect-file-source.properties
-rw-r--r-- 1 eglobal eglobal  883 may 17  2016 connect-file-sink.properties
-rw-r--r-- 1 eglobal eglobal 2.7K may 17  2016 connect-distributed.properties
-rw-r--r-- 1 eglobal eglobal  909 may 17  2016 connect-console-source.properties
-rw-r--r-- 1 eglobal eglobal  906 may 17  2016 connect-console-sink.properties
-rw-r--r-- 1 eglobal eglobal 1023 ago 14 17:22 zookeeper.properties
-rw-r--r-- 1 eglobal eglobal 5.3K ago 17 16:10 server.properties
DELL:eglobal~$ vim zookeeper.properties #Editamos el archivo y agregamos lo siguiente.
1 #Disco donde zookeeper almacenara sus logs.
2 dataDir=/mnt/hd02/zookeeper
3 #Puerto al cual los clientes se van a conectar.
4 clientPort=2181
5 # disable the per-ip limit on the number of connections since this is a non-production config
6 maxClientCnxns=0
7 server.1=10.10.10.104:2888:3888
#Para salir de vim  Tecla Esc + : y escribimos wq(Significa write and quit)
DELL:eglobal~$ vim server.properties #Editamos el archivo de configuracion de kafka.
1  broker.id=0#id unico, puesto que server-dell sera el master se le agrega el id 0.
  num.network.threads=3
2  # The number of threads that the server uses for processing requests, which may include disk I/O
3  num.io.threads=8
4  # The send buffer (SO_SNDBUF) used by the socket server
5  socket.send.buffer.bytes=102400
6  # The receive buffer (SO_RCVBUF) used by the socket server
7  socket.receive.buffer.bytes=102400
8  # The maximum size of a request that the socket server will accept (protection against OOM)
9  socket.request.max.bytes=104857600
10 log.dirs=/mnt/hd02/kafka
11 delete.topic.enable=true
12 confluent.support.customer.id=anonymous
13 port=9092
14 advertised.host.name=10.10.10.104
15 delete.topic.enable=true
16 confluent.support.customer.id=anonymous
17 replication.factor=2
18 num.replica.fetchers=3
19 offsets.topic.replication.factor=4
20 num.standby.replicas=2
21 transaction.state.log.replication.factor=1
22 transaction.state.log.min.isr=1
17 port=9092
18 advertised.host.name=10.10.10.104
#La demas configuracion es identica a la configuracion de los slaves, 
#Para salir de vim  Tecla Esc + : y escribimos wq(Significa write and quit)
```
Una vez configurado kafka en server-dell la instalación ha terminado, ahora solo es nesesario lanzar el proceso de la siguiente manera:
### Configurar puerto Jmx
Antes de Iniciar los servicios de kafka y zookeeper debemos habilitar el puerto Jxm de todos los servicios de kafka para que kafka manager pueda mostrarnos las métricas de cada broker.

```sh
#Este paso se realiza en todos los slaves excepto en server-dell ya que ahi no existe ningun servicio kafka corriendo.
arturo@arturo$ ssh eglobal@slaveXX
 password:******
 #Editamos el script kafka-server.start.sh, y añadimos la siguiente linea
eglobal@slaveXX$ cd /opt/kafka/bin
eglobal@slaveXX$ vim kafka-server.start.sh
#Agregar esta linea al script
EXTRA_ARGS="-name kafkaServer -loggc -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=9999 -Dcom.sun.management.jmxremote.local.only=false -Dcom.sun.management.jmxremot    e.authenticate=false -Dcom.sun.management.jmxremote.ssl=false"
#Esc + :wq para salir de vim
```


### Iniciar el servicio Zookeeper en server-dell e iniciar los servicios kafka en todos los Slaves.
```sh
#1- En server-dell levantar zookeeper
DELL:eglobal~$ ./opt/kafka/bin/zookeeper-server-start.sh /opt/kafka/conf/zookeeper.properties &
```
### Lanzar el siguiente comando en todos los slaves.
```sh
#1- En cada slave se tiene que iniciar el servicio kafka, y todos estos servicios apuntan al zookeeper de server-dell
eglobal@slaveXX$ ./opt/kafka/bin/kafka-server-start.sh /opt/kafka/conf/server.properties &
```

### Instalacion de kafka-manager
> Kafka manager es una herramienta para gestionar un cluster kafka, ademas de ello podemos visualizar las metricas de kafka(Brokers,Topics...)

Kafka-manager se instalara en server-dell ya que desde ahi se levantara un puerto para una salida a una interfaz web, basta con tener registrados todos los slaves en /etc/hosts de nuestro equipo y estar conectado a la red del cluster.
A continuación muestro como descargar el proyecto, compilar y generar los binario para poder ejecutar la herramienta.
```sh
#Clonamos  el proyecto en github
arturo@arturo$ git clone https://github.com/yahoo/kafka-manager.git
arturo@arturo$ ls -lrht
-rw-r--r-- 1 eglobal eglobal 1.1K may 17  2016 kafka-manager
#Es nesesario tener instalado sbt, para compilar el proyecto y generar los binarios.
arturo@arturo$  cd kafka-manager/
arturo@arturo$ ls -lrht
-rw-r--r-- 1 eglobal eglobal 11307 ago 18 13:38 LICENCE
-rw-r--r-- 1 eglobal eglobal  3133 ago 18 13:38 build.sbt
drwxr-xr-x 9 eglobal eglobal  4096 ago 18 13:38 app
drwxr-xr-x 2 eglobal eglobal  4096 ago 18 13:38 img
drwxr-xr-x 4 eglobal eglobal  4096 ago 18 13:38 test
drwxr-xr-x 4 eglobal eglobal  4096 ago 18 13:38 src
-rwxr-xr-x 1 eglobal eglobal 19460 ago 18 13:38 sbt
drwxr-xr-x 5 eglobal eglobal  4096 ago 18 13:38 public
drwxr-xr-x 4 eglobal eglobal  4096 ago 18 13:45 project
drwxr-xr-x 7 eglobal eglobal  4096 ago 18 18:11 target
drwxr-xr-x 2 eglobal eglobal  4096 ago 21 10:18 conf
-rw-r--r-- 1 eglobal eglobal  6646 ago 21 10:51 README.md
arturo@arturo$ ./sbt clean dist
#Aqui genera el target/universal/kafka.zip
#Mover el zip a server-dell y ejecutarlo. Por defecto inicia en el puerto 9000
arturo@arturo$ scp /target/universal/kafka.zip eglobal@server-dell:/home/eglobal/Software/NewUtils/
arturo@arturo$ ssh eglobal@server-dell
password :******
DELL:eglobal~$ cd /home/eglobal/Software/NewUtils/
#Descomprimir el zip
Dell:eglobal~$ unzip kafka-manager.zip
Dell:eglobal~$ cd kafka-manager/conf
#Modificamos el fichero para apuntar al zookeeper de server-dell.
Dell:eglobal~$ vim application.conf 
#Modificamos la siguiente linea quedando asi.
kafka-manager.zkhosts="server-dell:2181"
#Guardamos Esq + :wq y salimos de vim
DELL:eglobal~$cd ../ && cd bin/
DELL:eglobal~$  ./kafka-manager -Dhttp.port=8180
```

[![N|Solid](https://image.ibb.co/jCnHcQ/kafkamanager.png)](https://nodesource.com/products/nsolid)


License
----
2017-Agosto-20 Eglobal
