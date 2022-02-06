# Modulo: Big Data Architecture
`Fecha maxima de entrega: 06 de Febrero 2022`

## Practica ##
### Enunciado: ###
Diseñar, especificar y desplegar (opcional) un data lake para el procesamiento de datos provenientes de fuentes de datos no estructurados extraídos mediante técnicas de scraping/crawling de sitios de dominio público.




### Definición de la estrategia del DAaaS: ###
Sistema de inteligencia de negocio, que centralizará toda aquella información importante para la organización y que aplicando el gobierno del dato los diferentes departamentos y sus empleados tendrán acceso a la información de manera segmentada.

El sistema proporcionará la información en diferentes formatos, reportes descargables, plataforma de visualización de datos en tiempo real y plataforma de inteligencia de negocio.




### Diagrama: ###

![PracticaArquitecturaBigData-v3](https://user-images.githubusercontent.com/2152086/152693994-af039b83-01b9-4514-ab56-686d45acef7d.jpg)




### Arquitectura DAaaS: ###
La arquitectura diseñada para ser implementada en su totalidad sobre herramientas cloud, de una manera eficiente, escalable y polivalente se compone de dos líneas de procesamiento totalmente diferenciadas:

* Una realizará procesamiento de datos por lotes.
* La otra realizará procesamiento de datos en real time.


#### Bloques Funcionales. ####
El sistema planteado presenta tres bloques funcionales muy identificables:

* **Obtención de datos** (Zona de ingesta): Es la primera parte del sistema y como su nombre indica, en ella realizamos la obtencion de los datos que vamos a utilizar. Los datos vienen de diferentes fuentes que podríamos resumir en las siguientes: herramientas utilizadas por los equipos de atención al cliente, archivos de uso diario, la base de datos del software de la compañía, las redes sociales o los archivos de log de los servidores que dan servicios.
* **Tratamiento de datos** (Zona de procesamiento): Es la segunda parte del sistema y es donde realizaremos las transformaciones, tratamientos y análisis necesarios sobre los datos obtenidos. 
* **Obtención de resultados** (Zona de almacenamiento): A partir del análisis realizado obtendremos una serie de resultados que estarán disponibles en diferentes formatos, desde archivos CSV listos para descargas hasta herramientas de visualización.



#### Elementos de la arquitectura. ####
El sistema contará con diferentes tecnologías y herramientas:

* **Google Storage**: Es un servicio de almacenamiento de archivos en línea para almacenar y acceder a datos en la infraestructura de Google Cloud Platform. El servicio combina el rendimiento y la escalabilidad de la nube de Google con las capacidades avanzadas de seguridad y uso compartido.
* **Elasticsearch y Kibana**: O Elastic Stack. Elasticsearch es un servidor de búsqueda basado en Lucene. Provee un motor de búsqueda de texto completo y distribuido. Kibana proporciona capacidades de visualización a la información indexada en Elasticsearch. Será utilizado para el almacenamiento y visualización de la información basada en texto. 
* **Flume**: Es un servicio distribuido, fiable y altamente disponible que se utiliza para recopilar, agregar y mover eficientemente grandes cantidades de datos.
* **Kafka**: Es un proyecto tiene como objetivo proporcionar una plataforma unificada, de alto rendimiento y de baja latencia para la manipulación en tiempo real de fuentes de datos. Puede verse como una cola de mensajes bajo el patrón publicación-suscripción.
* **Hadoop**: Es un entorno de trabajo para programar aplicaciones distribuidas que manejen grandes volúmenes de datos.
* **Hive**: Es una infraestructura de almacenamiento de datos construida sobre Hadoop para proporcionar agrupación, consulta y análisis de datos. Hive utiliza la tecnología HDFS de Hadoop para poder realizar el almacenamiento de manera distribuida.
* **Spark**: Se puede considerar un sistema de computación en clúster de propósito general y orientado a la velocidad. Es compatible con desarrollos en Java, Scala, Python y R.
* **Spark Streaming**: Utiliza la capacidad de programación rápida de Spark para realizar análisis de transmisión. Ingiere datos en mini lotes y realiza transformaciones RDD en esos mini lotes de datos.
* **Jobs ETL**: Pequeñas aplicaciones que pueden ser desarrolladas en JAVA, Spork o Phyton que serán utilizados para la ingesta de datos relacionados con los sistemas de producción que vamos a analizar. Estos procesos se realizarán en su gran mayoría cuando los sistemas de producción no tienen una carga de trabajo elevadas.
* **CloudSQL**: Es un producto de almacenamiento de GCP que ofrece MySQL o PostgreSQL completamente administrado e integrado dentro de la infraestructura de Google Cloud.




### DAaaS Operating Model Design and Rollout: ###
En el diseño operativo vamos a describir cada uno de los procesos que van a ocurrir dentro de nuestra arquitectura y que la suma de todos ellos darán como resultado la información a explotar por parte de los integrantes de la organización.

**Parte de la arquitectura de procesamiento por lotes:**

* Zona de ingesta de datos.
	* Dispondremos diferentes jobs (uno para cada herramienta de producción de la que queremos obtener datos) que se ejecutarán diariamente a medianoche y que se encargarán de traer la información necesaria de los sistemas de producción a nuestros sistemas de data, para no interferir en la carga de los servidores. Estos jobs, serán ejecutados en un Dataproc que estará levantado mientras sucede la ingesta de esta información y que el resultado será gravado en Hive donde será procesado.
	* Dispondremos de un bucket en cloud storage configurado con una cloud function para que cuando se detecte un nuevo archivo en el se lance un job que realice el procesamiento del mismo. Previo al procesamiento deberá comprobar que el entorno de dataproc donde estará alojado está levantado y de no ser así deberá realizarlo, puesto que este backet estará disponible 24/7 y no tiene porque recibir los archivos en un momento determinado.
* Zona de procesamiento de datos.
	* Para la zona de procesamiento utilizaremos dataproc, donde instalaremos hadoop que nos permitirá realizar el procesamiento de datos de manera distribuida. Este entorno estará disponible mientras sea necesario (procesos nocturnos de ingesta y necesidades del backet). Además, tendremos instalado un cluster con un nodo maestro y uno o múltiples nodos esclavos en función de la demanda, lo que nos dará la posibilidad de crecer de manera horizontal.
	* El almacenamiento de datos estará bajo la infraestructura de almacenamiento Hive que está construida sobre Hadoop y soporta el análisis de grandes conjuntos de datos almacenados bajo HDFS de Hadoop y en sistemas compatibles como Amazon S3 y Google Storage. Dándonos gran versatilidad. 
	* Como "cerebro" del procesamiento de datos utilizaremos Spark haciendo uso del "músculo" de Hadoop (Hadoop YARN, HDFS ...) ya que trabaja en memoria RAM y es más rápido que Map Reduce de hadoop, entre otras ventajas. Spark resulta más fácil de programar ya que no se debe seguir una metodología concreta para la utilización de Map Reduce y es compatible con diferentes lenguajes de programacion como pueden ser Java, Scala, Python y R.
* Zona de almacenamiento de datos.
	* Crearemos un Data Warehouse, disponible 24/7, para el almacenamiento de los resultados obtenidos en la zona de procesamiento. EL objetivo principal de este DW es tener la información estructurada de tal manera que permita, de forma sencilla, ser consultada por los integrantes de la organización. 
	* Dispondremos de diferentes Data Marts, obtenidos a partir del DW y con procesos sencillos de ETL, para dar acceso a determinados datos de una manera mas específica y con controles de acceso mas restringidos.
	* Crearemos un almacén de reportes en Cloud Storage, donde se depositarán reportes en formatos listos para el consumo por parte de aplicaciones o usuarios. Estos reportes tienen dos orígenes diferentes, o vienen desde la parte de procesamiento o son generados bajo demanda desde el DW.
* Zona de visualización y consumo de datos.
	* Para generar informes y dashboard utilizaremos Google Data Studio, herramienta del ecosistema google.
	* Para el consumo de reportes puntuales de manera offline, utilizaremos cualquier herramienta capaz de abrir archivos de tipo csv, de tipo pdf que será los formatos que utilizaremos para la generación de este tipo de informes.
	* También será posible utilizar aplicaciones que consuman los datos en formato json que será otro de los formatos en los que dejaremos la información.


**Parte de la arquitectura de procesamiento en real time.**

* Zona de ingesta de datos.
	* 	Dispondremos de algunos jobs que enviarán información a Kafka en función de las necesidades definidas.
		*  El primer tipo de trabajos estarán orientados a escuchar las diferentes redes sociales que desde la organización queremos escuchar. La idea es localizar, en función de algunas palabras definidas, la información que se publica y guardarla en nuestro sistema.
		*  EL segundo tipo de trabajo estará orientado a escuchar los sistemas de log de los servidores de la organización, con el objetivo de recopilar lo que está ocurriendo en el sistema para su posterior explotación.
* Zona de procesamiento de datos.
	* 	Siguiendo con la filosofía aplicada en el procesamiento por lotes utilizaremos Spark para el procesamiento de la información. Para ello, Spark Streaming estará escuchando Kafka y cuando disponga de datos realizará el procesamiento necesario para terminar almacenando la información en una base de datos de tipo Elasticsearch que despliega todo su potencial en las búsquedas en tiempo real sobre texto.
* Zona de almacenamiento de datos.
	* 	Para almacenamiento de datos, como hemos comentado anteriormente, utilizaremos Elasticsearch, en la que se puede buscar todo tipo de documentos. Las busquedas son escalables y casi en tiempo real. 
* Zona de visualización.
	* Para la visualización en real time utilizaremos Kibana un software optimizado para la utilización de datos almacenados en Elasticsearch.

