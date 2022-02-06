# Modulo: Big Data Architecture
`Fecha maxima de entrega: 06 de Febrero 2022`

## Practica ##
### Enunciado: ###
Diseñar, especificar y desplegar (opcional) un data lake para el procesamiento de datos provenientes de fuentes de datos no estructurados extraídos mediante técnicas de scraping/crawling de sitios de dominio público.



### Definición de la estrategía del DAaaS: ###
Sistema de inteligencia de negocio, que centralizará toda aquella información importante para la organización y que aplicando el gobierno del dato los diferentes departamentos y sus empleados tendrán acceso a la información de manera segmentada.

El sistema proporcionará la información en diferentes formatos, reportes descargables, plataforma de visualización de datos en tiempo real y plataforma de inteligencia de negocio.



### Diagrama ###

*_Subir la imagen a GitHub_*



### Arquitectura DAaaS ###
La arquitectura diseñada para ser implementada en su totalidad sobre herrmientas cloud, de una manera eficiente, escalable y polivalente se compone de dos lineas de procesamiento totalmente diferenciadas:

* Una realizará procesamiento de datos por lotes.
* La otra realizará procesamiento de datos en real time.


#### Bloques Funcionales ####
El sistema planteado presenta tres bloques funcionales muy identificables:

* **Obtención de datos**: Es la primera parte del sistema y como su nombre indica, en ella realizamos la obtencion de los datos que vamos a utilizar. Los datos vienen de diferentes fuentes que podríamos resumir en los siguiente, herramientas utilizadas por los equipos de atección al cliente, archivos de uso diario, la base de datos del software de la compañia, las redes sociales o los archivos de log de los servidores que dan servicios.
* **Tratamiento de datos**: Es la segunda parte del sistema y es donde realizaremos las transformaciones, tratamientos y análisis necesarios sobre los datos obtenidos. 
* **Obtención de resultados**: A partir del análisis realizado obtendremos una serie de resultados que estarán disponibles en diferentes formatos, desde archivos CSV listos para descargas hasta herramientas de visualización.


#### Elementos de la arquitectura ####
El sistema contará con diferentes tecnologías y herrmientas:

* **Google Storage**: Será utilizado para el almacenamiento de datos.
* **Elasticsearch y Kibana**: Será utilizado para el almacenamiento y visualización de la información basada en texto. 
* **Flume + Kafka + Spark Streaming**: Será utilizado para la ingesta y el tratamiento de datos en realtime que vendran de las redes sociales y de los log de los servidores de producción.
* **Jobs ETL (JAVA-Spark-phyton)**: Serán utilizados para la ingesta de datos relacionados con los sistemas de producción que vamos analizar. Estos procesos se realizarán en su gran mayoría cuando los sistemas de producción no tienen una carga de trabajo elevadas.
* **Ecosistema Hadoop**: Será utilizado para el almacenamiento y procesamiento de los datos con el objetivo de generar los resultados deseados.
* **CloudSQL**: Será utilizado para el almacenamiento de resultados.



### DAaaS Operating Model Design and Rollout ###
En el diseño operativo vamos a describir cada uno de los procesos que van a ocurrir dentro de nuestra arquitectura y que la suma de todos ellos darán como resultado la información a explotar por parte de los integrantes de la organización.

**Parte de la arquitectura de procesamiento por lotes**

* Zona de ingesta de datos.
	* Dispondremos diferentes job (uno para cada herramienta de producción de la que queremos obtener datos) que se ejecutarán diariamente a la media noche, que se encargaran de traer la información necesaria de los sistemas de producción a nuestros sistema de data, para no interferir en la carga de los servidores. Estos jobs, serán ejecutados en un Dataproc que estará levantado mientras sucede la ingesta de esta información y que el resultado será gravado en Hive donde será procesado.
	* Dispondremos de un bucket en cloud storage configurado con una cloud function para que cuando se detecte un nuevo archivo en el se lance un job que realice el procesamiento del mismo. Previo al procesamiento deberá de comprobar que el entorno de dataproc donde estará alojado está levantado y de no ser así deberá realizarlo, puesto que este backet estará disponible 24/7 y no tiene porque recibir los archivos en un momento determinado.
* Zona de procesamiento de datos.
	* Para la zona de procesamiento utilizaremos dataproc, donde instalaremos hadoop que nos permitirá realizar el procesamiento de datos de manera distribuida. Este entorno que estará disponible mientras sea necesario (procesos nocturnos de ingesta y necesidades del backet) tendremos instalado un cluster con un nodo maestro y uno o multiples nodos esclavos en función de la demanda, esto nos dará la posibilidad de crecer de manera horizontal.
	* El almacenamiento de datos estará bajo la infraestructura de almacenamiento Hive que esta construida sobre Hadoop y soporta el análisis de grandes conjuntos de datos almacenados bajo HDFS de Hadoop y en sistemas compatibles como Amazon S3 y Google Storage. Dandonos gran versatilidad. 
	* Como "cerebro" del procesamiento de datos utilizaremos Spark haciendo uso del "musculo" de Hadoop (Hadoop YARN, HDFS ...) ya que trabaja en memoria RAM y es mas rapido que Map Reduce de hadoop entre otras ventajas. Spark resulta mas facil de programar ya que no se debe de seguri una metodología concreta para la utilizacion de Map Reduce, y es compatible con diferentes lenguajes de programacion com pueden ser Java, Scala, Python y R.
* Zona de almacenamiento de datos.
	* Crearemos un Data Warehouse, disponible 24/7, para el almacenamiento de los resultados obtenidos en la zona de procesamiento. EL objetivo principal de este DW es tener la información estructurada de tal manera que permita de una manera fácil ser consultada por los integrantes de la organización. 
	* Dispondremos de diferentes Data Marts, obtenidos a partir del DW y con procesos sencillos de ETL, para dar acceso a determinados datos de una manera mas específica y con controles de acceso mas restringidos.
	* Crearemos un almacen de reportes en Cloud Storage, donde se depositarán reportes en formatos listos para el consumo por parte de aplicaciones o usuarios. Estos reportes tienen dos origens diferentes, o vienen desde la parte de procesamiento o son generados bajo demanda desde el DW.
* Zona de visualización y consumo de datos.
	* Para generar informes y dashboard utilizaremos Google Data Studio, herramienta del ecosistema google.
	* Para el consumo de reportes puntuales de manera offline, utilizaremos cualquier herramienta capaz de abrir archivos de tipo csv, de tipo pdf que será los formatos que utilizaremos para la generaciónd de este tipo de informes.
	* Tambien será posible utilizar aplicaciones que consuman los datos en formato json que será otro de los formatos en los dejaremos la información.


**Parte de la arquitectura de procesamiento en real time**

* Zona de ingesta de datos.
	* 	Dispondremos de algunos job que enviaran información a Kafka en función de las necesidades definidas.
		*  El primer tipo de trabajos estarán orientados a escuchar las diferentes redes sociales que desde la organización queremos escuchar. La idea es localizar en función de aglunas palabras definidas la información que se publica y guardarla en nuestro sistema.
		*  EL segundo tipo de trabajo estará orientado a escuchar los sistemas de log de los servidores de la organización, con el objetivo de recopilar lo que está ocurriendo en el sistema para su posterior explotación.
* Zona de procesamiento de datos.
	* 	Siguiendo con la filosofía aplicada en el procesamiento por lotes utilizaremos Spark para el procesamiento de la información. Para ello Spark Streaming estará escuchando Kafka y cuando disponga de datos realizará el procesamiento necesario para terminar almacenando la información en una base de datos de tipo Elasticsearch que despliega todo su potencial en las busquedas en tiempo real sobre texto.
* Zona de almacenamiento de datos.
	* 	Para almacenamiento de datos, como hemos comentado anteriormente, utilizaremos Elasticsearch, en la que se puede buscar todo tipo de documentos. Las busquedas son escalables y casi en tiempo real. 
* Zona de visualización.
	* Para la visualización en real time utilizaremos Kibana un software optimizado para la utilización de datos almacenados en Elasticsearch.

