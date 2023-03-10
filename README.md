# Analisis-de-contenido-falso-en-RRSS-mediante-deteccion-y-estudio-de-usuarios-programados-bots-

Este proyecto se centra en la clasificación binaria de las cuentas de redes sociales (este caso práctico se centra en el estudio de Twitter en junio de 2022) en dos grupos: bots y personas. Para analizar su impacto en la sociedad, se contabilizará y analizará el número de bots participando en la difusión de tendencias (hashtags) populares, fake news o campañas publicitarias en las redes sociales. El propósito del proyecto es sensibilizar y concienciar a los usuarios de las redes sociales, especialmente a aquellos menos experimentados en la tecnología y aquellos más vulnerables a caer víctimas de la desinformación, sobre el impacto de los bots en la difusión de información o noticias falsas. De esta manera, el proyecto facilita al usuario tomar decisiones informadas en todo momento.

Este proyecto consiste en la simulación de un EDW de datos relacionados a noticias falsas y a perfiles de usuarios programados (Bots) en las redes sociales que puede ser accedido para realizar análisis sobre los datos a través de la formación de un Data Pipeline. Para llevar a cabo este proyecto, realizamos las siguientes fases:
1. Obtención de las fuentes de datos. Por un lado, buscamos fuentes de datos estáticos sobre los que trazar las bases de nuestro EDW. Estos datos son estructurados. Tenemos recopilaciones etiquetadas en CSVs de características de usuarios de Twitter (etiquetados entre Bots y No-Bots), y conjuntos de noticias falsas (CSVs) con titulares y tweets que las distribuyen. Por otro lado, usamos Twitter como fuente de datos en streaming semiestructurados para obtener datos sobre los usuarios de Twitter tanto accediendo por temáticas (hashtags) concretos como buscando los autores de determinados tweets.
2. Ingesta y Almacenamiento de los datos. Para los datos estáticos estructurados de los bots descargamos los ficheros CSV que los contienen y obtuvimos un esquema común que nos interesaba seguir. Una vez definido el esquema, creamos una base de datos relacional en MariaDB creando tablas de los datos relacionados con las características de los usuarios de Twitter para, después, insertar los datos de los CSVs en dichas tablas. Una vez almacenados los datos en las tablas en esta Base de Datos Relacional, se transfieren por medio de Sqoop a su almacenamiento permanente y distribuido sobre HDFS. En cuanto a los datos estáticos y estructurados de noticias falsas, por su naturaleza (contienen arrays de datos), decidimos almacenarlos en Hive sobre HDFS también. Para los datos en streaming semiestructurados optamos por usar Kafka para su ingesta de Twitter y almacenarlos en una base de datos no relacional en MongoDB para después transformarlos en formato JSON y almacenarlos en HDFS. Se crean colecciones separadas para cada hashtag o noticia falsa. Se deciden estudiar los hashtags de #iphone, #crypto y #capitol puesto que se trata de asuntos de actualidad que cubren las temáticas de política, publicidad y economía. Para seleccionar noticias falsas con una gran cantidad de participantes en su difusión, se realiza una búsqueda y conteo del array de usuarios que las tweetean y se eligen aquellas con mayor interacción. Se escogen tres para su análisis, las cuales son “Neon Nettle”, “Snapchat is shutting down” y “Georgia becomes first state to ban Muslim culture in historic move to restore Western values”.
3. Transformación de los datos. Optamos mayoritariamente por Apache Spark, en concreto, su interfaz de Pyspark para la transformación y tratamiento de los datos puesto que desde esta plataforma se puede acceder con facilidad a nuestra base de datos en HDFS. Se accede a las tablas de datos estáticos y estructurados para unificarlas y seleccionar las columnas y valores deseados y obtener un único Data Frame que contenga todos los datos de características de usuarios de Twitter. En cuanto a los datos de noticias falsas, se crea un Data Frame a partir de estos para después obtener un array de los números de identificación de los usuarios de Twitter que han distribuido dichas noticias falsas (se usan también Pandas y Numpy para esta última transformación). Se realizan distintas transformaciones sobre dicho Data Frame para poder obtener los datos en el formato idóneo para poder, con la API de Twitter, hacer la ingesta de los datos de los usuarios autores de dichos tweets. Por otro lado, los datos en streaming, una vez almacenados en sus respectivos archivos JSON, son transformados también en Data Frames para seleccionar solo aquella información que resulta necesaria y prepararlos para su posterior uso y aplicación.
4. Análisis Preliminar. Realizamos un análisis inicial de los datos estáticos por medio de librerías de visualización gráfica de Python con Pandas y Numpy (librerías de graficado matplotlib y seaborn).
5. Modelo de IA. Creamos nuestro modelo de clasificación binaria de los usuarios de Twitter entre bots y no bots. Para ello, optamos por Pyspark para crear tres posibles modelos de clasificación, un modelo de Regresión Logística, un modelo de Random Forest y un modelo de Gradient Boosting Tree. Para ponerlos en funcionamiento, usamos nuestra base de datos etiquetados (bot y no bot) de características de los usuarios de Twitter para crear dos conjuntos aleatorios, uno de entrenamiento y otro de test con una división de 80-20% de los datos respectivamente. Los modelos se basan en la diferenciación entre usuarios verificados o no para, después, crear una variable categórica que llamamos bot_binary que tiene valor verdadero cuando se detecta la palabra “bot” en el nombre o en la descripción de la cuenta. Luego, se introducen más parámetros que incorporan los datos de uso de los usuarios. De esta manera, pudimos probar y analizar los resultados de F1 (basado en las métricas de precisión y recall) y de área bajo la curva (ROC) de los tres modelos. Así, pudimos concluir que se obtenían mejores resultados con los modelos de Random Forest y Gradient Boosting Tree, existiendo una mínima diferencia entre estos últimos. Consecuentemente, seleccionamos el modelo que ofrecía mejores resultados en cuanto a la predicción de Bot o no cuando en la actualidad lo es, que es el Random Forest. Una vez seleccionado el modelo, pasamos a aplicarlo a los datos que obtuvimos en streaming de Twitter para conseguir predicciones de si los usuarios son o no bots. Dichos resultados se guardan en formato CSV para su posterior análisis. Para la obtención de estos se preparan los datos que se pasan al modelo 
6. Visualización y Análisis Final de los datos. Optamos por representar los resultados en Power Bi. Creamos dos dashboards, uno para los datos de las fakenews y otro para los hashtags. En cada uno, creamos diferentes diagramas de barras y de dispersión para visualizar los datos.



- Selección y Descripción de los Datos:
Por un lado, buscamos fuentes de datos estáticos sobre los que trazar las bases de
nuestro EDW. Estos datos son estructurados. Tenemos recopilaciones etiquetadas en
CSVs de características de usuarios de Twitter (etiquetados entre Bots y No-Bots). Son
obtenidos de un repositorio de CSVs curado por la Universidad de Indiana
(https://botometer.osome.iu.edu/bot-repository/datasets.html) y de Kaggle
(https://www.kaggle.com/code/davidmartngutirrez/bots-accounts-eda/data).
Se tiene una serie de columnas que se compone de información de perfiles de usuarios
de Twitter. En concreto, se busca obtener el número de identificación y nombre de la
cuenta, la descripción, la localización, el número de seguidores, el número de seguidos,
el número de favoritos, un conjunto de parámetros de valor binario (verdadero o falso)
que son si la cuenta está verificada y si la cuenta tiene las propiedades por defecto sin
haber sido modificados, y, por último, el parámetro de tipo de cuenta que es la variable
principal que se usa para etiquetar los tipos de usuarios entre bots y no bots.
También disponemos de conjuntos de noticias falsas (CSVs) obtenidos de
https://github.com/KaiDMML/FakeNewsNet/tree/master/dataset que contienen los
titulares de dichas noticias, acompañadas de una url del periódico o la entidad que las
identifica como tales y de un array o lista de números de identificación de los tweets en
los que aparecen dichas noticias.
Por otro lado, usamos Twitter como fuente de datos en streaming semiestructurados
para obtener datos sobre los usuarios de Twitter. Buscamos obtener la información de
los usuarios que coincida con nuestros datos estáticos. Estos datos se obtienen tanto
accediendo por temáticas (hashtags) concretos como buscando los autores de
determinados tweets. Se deciden estudiar los hashtags de #iphone, #crypto y #capitol
puesto que se trata de asuntos de actualidad que cubren las temáticas de política,
publicidad y economía. Para seleccionar noticias falsas con una gran cantidad de
participantes en su difusión, se realiza una búsqueda y conteo del array de usuarios que
las tweetean y se eligen aquellas con mayor interacción. Se escogen tres para su
análisis, las cuales son “Neon Nettle”, “Snapchat is shutting down” y “Georgia becomes
first state to ban Muslim culture in historic move to restore Western values”.

- Transformación de los datos:
Optamos por Apache Spark, en concreto, su interfaz de Pyspark para la transformación y
tratamiento de los datos puesto que desde esta plataforma se puede acceder con
facilidad a nuestra base de datos en HDFS. Se accede a las tablas de datos estáticos y
estructurados para unificarlas y seleccionar las columnas y valores deseados y obtener
un único Data Frame que contenga todos los datos de características de usuarios de
Twitter.
En cuanto a los datos de noticias falsas, se accede crea un Data Frame a partir de estos
para después obtener un array de los números de identificación de los usuarios de
Twitter que han distribuido dichas noticias falsas.
Por otro lado, los datos en streaming, una vez almacenados en sus respectivos archivos
JSON, son transformados también en Data Frames para seleccionar solo aquella
información que resulta necesaria y prepararlos para su posterior uso y aplicación.

- Visualización y Query de los datos:
Por un lado, en cuanto a las visualizaciones del dashboard con los datos de hashtags,
podemos observar que los hashtags relacionados con criptomonedas presentan un
mayor número de bots que el resto de hashtags analizados. También se puede observar
la gran diferencia entre la relación de seguidos y seguidores de bots y humanos.
Por otro lado, las visualizaciones del dashboard de datos de noticias falsas muestran
porcentajes más bajos de participación de los Bots en noticias falsas en comparación a
los porcentajes de su presencia en los hashtags.
De los resultados obtenidos podemos obtener las siguientes conclusiones. El análisis de
hashtags muestra que hay altos porcentajes de perfiles programados participando en las
discusiones que se generan en estos hilos. Mientras que el análisis de noticias falsas
específicas presenta valores de participación más bajos, dando a entender que la
difusión de dicha desinformación es en gran parte debida a usuarios físicos.
Por tanto, a partir de este análisis de la presencia los Bots en la difusión de
desinformación y noticias falsas en Twitter se puede inducir que el impacto de los Bots
en las redes sociales tiene mayor peso en la divulgación de publicidad y en la
polarización de conversaciones sociales y/o políticas en comparación a su impacto en la
divulgación de noticias falsas o titulares particulares.
