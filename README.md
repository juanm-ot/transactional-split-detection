

** Este repositorio ha sido marcado como un repositorio de archivos grandes (Git LFS)**

El objetivo de este producto de datos es **idear una solución para identificar transacciones que evidencian un comportamiento de Mala Práctica Transaccional**, es decir, un comportamiento donde se evidencia un uso de los canales mal intencionado.

En este caso, la orientación sera en la práctica de **Fraccionamiento Transaccional**, esta mala práctica consiste en fraccionar una transacción en un número mayor de transacciones con menor monto que agrupadas suman el valor de la transacción original. Estas transacciones se caracterizan por estar en una misma ventana de tiempo que suele ser 24 horas y tienen como origen o destino la misma cuenta o cliente

## Conocimiento del negocio

El analisis exploratorio de datos me permitio modelar el flujo de moneda en función del comportamiento entre las parejas transaccionales. 

[![flujo-moneda-grafico.png](https://i.postimg.cc/d1QGmvN2/flujo-moneda-grafico.png)](https://postimg.cc/QBPBjL3C)

Ademas, tener los siguientes insights: 

* El primer bimestre del año tiene un comportamiento transaccional que difiere de la normalidad presentada en el año. Tenemos el mayor numero de transacciones por dia y los menores valores para la mediana y el promedio de moneda por transaccion. Esta relacion entre variables son un fuerte indicativo del fraccionamiento transaccional
* Revisando la data en escala mensual, el mayor volumen transaccional se presenta en los ultimos meses. Hay una disminucion en en el segundo bimestre, pero el comportamiento se puede describir en una funcion lineal. Los rangos de minimos y maximos transaccionales por dia en cada mes se mantienen entre 18335 y 100215 transacciones por dia en los limites inferior y superior respectivamente. 
* El fraccionamiento transaccional no está influenciado por el tipo de transacción (crédito o débito) porque es independiente de la frecuencia y la cantidad de transacciones realizadas. Ademas una cuenta puede estar relacionada a credito y debito. 
* Solo el 24% de las cuentas han tenido flujo transaccional con un unico subsidiary y tienen > 10 dias transacciones por dia. Esto es un comportamiento atipico que debe revisarse


## Flujo de datos

[![flujo-datos.png](https://i.postimg.cc/W1btKQ3b/flujo-datos.png)](https://postimg.cc/N5Sg2CmW)

Los registros transaccionales deben actualizarse en tiempo real. El feature engineering permite sobre la ultima transaccion tenes estadisticas de todo el comportamiento transaccional en la ventana de tiempo, por esta razon los datos tienen que irse ingestando para actualizar estas metricas en la ventana de tiempo y poder ejecutar la marcacion del cliente para tomar las acciones necesarias en la app segun lo decida el negocio. 

En terminos teoricos, la data debe tomarse en tiempo real tambien para que el modelo detecte el cambio de patrones y capture las nuevas reglas asociadas a las malas practicas

## Modelo Isolate Forest

Es un algoritmo de detección de anomalias cuyo objetivo principal es identificar puntos que son inusuales o diferentes del resto del conjunto de datos a traves de la construcción de arboles de aislamiento. Tiene muy buen performance en terminos de procesamiento por eso es adecuado para conjuntos de datos con una gran cantidad de datos normales y solo unas pocas anomalias. 

* Pude generar un label_stage a partir de relaciones de los datos e identifique que el 88.9% de los registros estan excentos de fraccionamiento transaccional (visite el archivo feature_explorations para mas detalles). Esto carcateriza el set de datos con "pocas anomalias"
* El proceso de exploración de los modelos empezo con un clasico del aprendizaje no supervisado: DBSCAN. Este modelo fue descartado porque no tuvo buenas metricas de evaluación, tampoco fue funcional y la gran capacidad de procesamiento lo hace un algoritmo poco eficiente
* Itere de manera escalonada sobre el Isolate Forest, fui introduciendole data en cada entrenamiento y mostro buen performance,tambien fui ajustando sus parametros para lograr encontrar el umbral de los outliers (random state para mantener la reproducibilidad y contaminación para indicarle la base de outliers) Ademas de un procesamiento muy rapido. 

[![isolatef-model.png](https://i.postimg.cc/cHyTw6Vr/isolatef-model.png)](https://postimg.cc/Y4R1w22H)

Finalmente para el alcance del producto, con los resultados obtenidos tengo dos propuestas que dependeran de decisiones de negocio:

- A: Modelo que no permita que ninguna transaccion unitaria (solo una transaccion por pareja transaccional en la ventana de tiempo) aparezca como anomala / contaminación = 0.08%
- B: Modelo que no permita que ninguna pareja transaccional con mas de 5 transacciones al dia sea marcada como no anomala. contaminación = 2,4%

Bajo mi analisis  y las metricas de evaluación selecciono la opción A, ya que garantizo teoricamente la normalidad y para las transacciones que estan en el borde del umbral puedo proponer una regla programatica para enviarla a revision manual cuando se detecte sospechosa. 

** Los modelos generados se adjuntan en la carpeta con el nombre: 'modelo_optimo_more_5_transaction.joblib' y 'modelo_optimo_unitari_transactions.joblib'**


## Estructura repositorio

El diseño del producto esta modelado en archivos unitarios que definen cada etapa, siga el consecutivo de los .ipynb para la lectura del proyecto


```linux

.
├── README.md                          # Leeme
├── 01_process_pipeline.ipynb          # pre-procesamiento data.
├── 02_eda.ipynb                       # analisis exploratorio
├── 03_Dayfeature_engineering.ipynb    # feature engineering
├── 04_feature_explorations.ipynb      # analisis estadistico features
├── 05_model_dbscan_test.ipynb         # exploración modelo dbscan
├── 06_model_isolationf.ipynb          # exploración modelo isolation forest
│
├── propuesta_arquitectura             # propuesta arquitectura y recursos producto.
│   └── arquitectura_recursos.ipynb    
│   └── diagrama_arquitectura.png      # Consulte los diagramas para ver la arquitectura
│   └── diagrama_arquitectura.pdf
│
├── pruebas_concepto                   # carpeta auxiliar desarrollos experimentales
│   └── 24h_featured_data_0001.parquet
│   └── 03_24hfeature_engineering.ipynb
│
├── graficos                           # carpeta para cargar imagenes para los .ipynb
│   └── algún-archivo
│
├── sample_data_0006_part_00.parquet   # raw data
├── sample_data_0007_part_00.parquet
├── processing_data_0001_dummie.parquet  # output pre-procesamiento data
└── day_featured_data_0002.parquet       # output feture engineering

```

## Requerimientos

Recuerde si va a correr en el local, crear un ambiente virtual e instalar las dependencias necesarias desde el archivo requirements.txt

