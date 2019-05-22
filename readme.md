*Introducción:*

Con las posibilidades que la nube entrega a personas y organizaciones de todos tipos en el mundo, la generación de los datos ha aumentado de forma exponencial. AWS te permite utiilzar bajo demanda el almacenamiento necesario para recolectar estos datos utilizando servicios como Amazon EBS (https://aws.amazon.com/es/ebs/) (volúmenes de almacenamiento), Amazon S3 (https://aws.amazon.com/es/s3/) (Almacenamiento tipo objeto), Amazon RDS (https://aws.amazon.com/es/rds/) (Bases de datos estructuradas) Amazon DynamoDB (https://aws.amazon.com/es/dynamodb/) (NoSQL Llave-Valor), Amazon Elastic Filesystem (https://aws.amazon.com/es/efs/) (EFS, Sistema administrado de archivos para linux), Amazon FSx para windows (https://aws.amazon.com/es/fsx/) (Sistema administrado de archivos para windows), entre muchos otros.

Aunque cada uno de estos servicios cuenta con APIs con las que se pueden programar y automatizar copias de seguridad, nuestros clientes nos solicitaban un servicio llave en mano que les permitiera cumplir con sus estándares empresariales de respaldo de información.

Primeramente explicaremos cómo son realizadas las copias de seguridad de volúmenes EBS en AWS:

Los snapshots son copias de seguridad incrementales, lo que significa que solo se guardan los bloques que han cambiado en el volumen después del snapshot más reciente. Esto disminuye el tiempo necesario para crearlo y ahorra costos de almacenamiento, ya que los datos no se duplican. Cuando se elimina un snapshot, solo se borran los datos que son únicos de dicho snapshot. Cada uno de ellos contiene toda la información necesaria para restaurar los datos (del momento en el que se tomó) en un volumen de EBS nuevo:

Fig 1
[Image: image.png]
En la imagen anterior (Fig 1):

* El Snapshot A ocupa 10GB de almacenamiento para respaldo

* El Snapshot B solo tiene las modificaciones de los 4GB y un apuntador a los 6GB que no han cambiado. El almacenamiento total utilizado es de 4GB.

* El Snapshot C sólo tiene 2 GB adicionales, por lo que cuenta con referencias a la información ya respaldada y solo utiliza 2GB.


*Cuerpo:*
Si bien es posible crear un snapshot utilizando APIs o la Consola de AWS, durante reInvent 2018 (https://www.youtube.com/watch?v=QDiXzFx2iMU) lanzamos un nuevo servicio llamado AWS Backup (https://aws.amazon.com/es/backup/). Este servicio está diseñado para ayudarlo a automatizar y administrar de forma centralizada sus copias de seguridad en una consola única. 

Usted puede crear planes de respaldos basados en políticas, monitorear el estado de los respaldos en curso y, en caso de ser necesario, restaurar estas copias de seguridad. 

Estos respaldos pueden ser realizados para volúmenes EBS, sistemas de archivos EFS, bases de datos RDS, tablas de DynamoDB y volúmenes de Storage Gateway. Todas estas copias de seguridad se guardarán automáticamente en S3. Si usted o su organización requiere cumplir con algún estándar o normativa que le solicite guardar datos por largos periodos de tiempo, es posible archivar respaldos en Amazon Glacier (https://aws.amazon.com/es/glacier/) para una larga duración al mejor costo.

Debido a que el servicio de AWS Backup incluye a AWS Storage Gateway, es posible utilizarlo para calendarizar respaldos de sus datos locales hacia la nube.

Esta es la primera vista que usted tendrá cuando visite el servicio en su consola de AWS: (Fig2)

(Fig2)
[Image: Screen Shot 2019-04-30 at 11.30.28.png]El primer paso es crear un plan de respaldos o seleccionar un plan de respaldos existente. Para fines de este ejemplo, crearemos un nuevo plan de respaldos desde el inicio (Fig3):
(Fig3)
[Image: Screen Shot 2019-04-30 at 11.42.22.png]Después, configuraremos una regla de respaldos, seleccionaremos la frecuencia con la que queremos se ejecute esa regla, el horario de inicio de la ventana para crear el respaldo y el tiempo que podría durar la ventana (Fig4):

(Fig4)
[Image: Screen Shot 2019-04-30 at 11.39.30.png]
Más adelante seleccionaremos el ciclo de vida necesario para nuestros respaldos. En este ejemplo, elegí que los respaldos sean colocados en almacenamiento frio después de un mes de haber sido creados y que sean eliminados después de un año de existencia (Fig5).

(Fig5)
[Image: Screen Shot 2019-04-30 at 11.41.01.png]Al final agregaremos una etiqueta para dar mejor seguimiento a los respaldos realizados por esta regla (Fig6):

(Fig6)
[Image: Screen Shot 2019-04-30 at 11.43.22.png]Y daremos clic en “Crear Plan”. Veremos el plan creado en nuestra consola (Fig7) :

(Fig7)
[Image: Screen Shot 2019-04-30 at 11.44.57.png]
Lo que falta es agregar recursos a esta regla de respaldo, entonces lo agregaremos al dar clic al “PlanDeRespaldos1” y después a “Asignar Recursos” (Fig8):

(Fig8)
[Image: Screen Shot 2019-04-30 at 11.49.03.png]Al asignar recursos, es posible seleccionarlos por ID específico o por etiquetas para que, cada recurso que concuerde con esta etiqueta será agregado de forma automática. En la siguiente imagen se pueden ver ambas opciones (Fig9):

(Fig9)
[Image: Screen Shot 2019-04-30 at 11.52.41.png]En ella seleccionamos un volúmen EBS conectado a una instancia EC2 (un Windows Server 2019 base) y también elegimos que todo lo que tenga la etiqueta Ambiente:Demo sea respaldado.

Aprovecharemos ahora para agregar otros recursos (una instancia RDS) (Fig10):

(Fig10)
[Image: Screen Shot 2019-04-30 at 11.56.22.png]Como puede observar, en este segundo grupo de recursos seleccionamos específicamente una instancia de base de datos MySQL que tengo en el servicio de RDS.

Al final tendremos dos grupos de recursos en el mismo plan de respaldos (Fig11):

(Fig11)
[Image: Screen Shot 2019-04-30 at 11.58.47.png]Lo que resta es esperar a que la regla sea ejecutada en la ventana de tiempo seleccionada (en este caso, seleccionamos 2:00am UTC y una ventana de las 2:00am a las 6:00 am UTC)

Una vez se haya ejecutado el plan de respaldos, en la consola principal de AWS Backup podrá revisar cuántos respaldos se han realizado en las últimas 24 horas (Fig12):

Fig12
[Image: Screen Shot 2019-05-09 at 12.57.55.png]Al dar clic en “Detalles de trabajos de copia de seguridad”, podremos revisar cuales recursos fueron respaldados con base en las reglas creadas anteriormente (Fig13):

Fig13
[Image: Screen Shot 2019-05-09 at 12.59.06.png]

Adicionalmente, en el apartado “Recursos protegidos” podemos revisar los puntos de restauración de cada uno de los recursos (Fig14):

Fig14
[Image: Screen Shot 2019-05-09 at 14.28.16.png]Por ejemplo, si se desea restaurar alguna de las copias de seguridad del volumen con terminación “de2”, simplemente damos clic al recurso y se listarán todas las copias de seguridad. Por default la consola muestra los últimos 5 respaldos: (Fig15)

Fig15
[Image: Screen Shot 2019-05-09 at 14.30.01.png]Para restaurar alguno de estos puntos en el tiempo simplemente se selecciona el punto de recuperación deseado para después dar clic al botón “Restaurar”.

Acto seguido, se abrirá un portal de restauración con las opciones disponibles para el tipo de recurso, en este caso un volúmen EBS (Fig16 y 17):

Fig16
[Image: Screen Shot 2019-05-09 at 14.32.35.png]Fig 17
[Image: Screen Shot 2019-05-09 at 14.32.51.png]

Al dar clic en “Restaurar copia de seguridad”, se creará un ID de trabajo de restauración y podremos ver el estatus en el que se encuentra aquí mismo (Fig 18):

Fig18
[Image: Screen Shot 2019-05-09 at 14.33.55.png]De estatus “Pendiente” pasará a “En ejecución” (Fig19):

Fig19
[Image: Screen Shot 2019-05-09 at 14.34.56.png]
y de “En ejecución” pasará a estatus “Completado” (Fig20):

Fig20
[Image: Screen Shot 2019-05-09 at 15.17.23.png]
En esta pantalla podremos ver el ID del recurso creado a partir de ese punto de recuperación, mismo que podrás buscar, en este caso, en los volúmenes EBS dentro del servicio de EC2 (Fig21):

Fig21
[Image: Screen Shot 2019-05-09 at 15.19.06.png]
Listo! hemos automatizado la creación de puntos de restauración que podemos utilizar para recuperación y cumplimiento dentro de nuestra organización.

**-------- *Notas adicionales: --------*

*Rastreo de actividad*
Debido a que en AWS todo funciona con base en APIs y todas las llamadas a estas APIs son grabadas, es posible revisar las acciones que AWS Backup toma para realizar las copias de seguridad de tus recursos. El servicio en donde podemos descubrir este historial de llamadas se llama Amazon CloudTrail.

Dentro del apartado de “Event History” podemos filtrar por ID de recurso y revisar el historial de llamadas de API en el que este recurso tuvo algo que ver. Por ejemplo en la siguiente imagen podemos ver que el servicio  “backup.amazonaws.com” (http://backup.amazonaws.com/) realizó una llamada de API para crear una copia de seguridad, siguiendo su calendario de respaldos automatizados (Fig22):

(Fig22)
[Image: Screen Shot 2019-05-08 at 10.25.40.png]
*Notificaciones *
Si usted lo desea, puede recibir notificaciones cada vez que una copia de seguridad sea creada ya sea de forma manual o por medio del servicio de AWS Backup.

Para hacerlo, es necesario utilizar en conjunto dos servicios de AWS. Amazon Simple Notification Service (https://aws.amazon.com/es/sns/) (Amazon SNS) y la característica de Eventos (https://docs.aws.amazon.com/es_es/AmazonCloudWatch/latest/events/WhatIsCloudWatchEvents.html) de Amazon CloudWatch (https://aws.amazon.com/es/cloudwatch/)

En primer lugar será necesario crear un Tema de mensajes dentro de Amazon SNS (Fig23, Fig24 y Fig25), para después generar una o varias suscripciones a este Tema. Durante el ejemplo, crearemos dos suscripciones. Una vía mensaje de texto SMS y otra vía correo electrónico.

Fig23
[Image: Screen Shot 2019-05-10 at 12.37.01.png]Fig24
[Image: Screen Shot 2019-05-10 at 12.37.47.png]Fig25
[Image: Screen Shot 2019-05-10 at 12.39.35.png]
El primer paso es dar clic en “Crear una suscripción” para despues seleccionar el método de entrega, en este caso será un mensaje de texto (Fig26):

Fig26
[Image: Screen Shot 2019-05-10 at 12.40.52.png]Repetimos el proceso para ahora crear una suscripción vía correo electrónico (Fig27):

Fig27
[Image: Screen Shot 2019-05-10 at 12.42.27.png]*Nota: Para las suscripciones vía correo electrónico, deberá aceptar la suscripción mediante una liga que el sistema envía de forma automática a la dirección de correo escrita (Fig 28 y 29)

Fig28:
[Image: Screen Shot 2019-05-10 at 12.44.22.png]Fig29:
[Image: Screen Shot 2019-05-10 at 12.44.49.png]Al final, las dos suscripciones pueden ser validadas dentro del Tema (Fig30):
[Image: Screen Shot 2019-05-10 at 12.47.59.png]

El segundo paso se debe realizar en la consola de Amazon Cloudwatch, en el apartado de Eventos. En él, crearemos una regla con los parámetros mostrados en la siguiente imagen (Fig31):

[Image: Screen Shot 2019-05-10 at 12.51.18.png]
Es importante mencionar que en este caso, las notificaciones serán enviadas vía el Tema de SNS creado unos pasos atrás y solamente se ejecutará cuando el Origen coincida, en este caso los snapshots creados a partir del volúmen EBS seleccionado en el campo “Orígenes específicos”.

*Precios*
Con AWS Backup, usted paga solo por la cantidad de almacenamiento de copias de seguridad que utilice y por la cantidad de datos que restaure al mes. No se aplica una tarifa mínima ni cargos de configuración.

-------

*Más información:*
https://aws.amazon.com/es/backup/
https://www.youtube.com/watch?v=QDiXzFx2iMU

*Referencias:*
https://aws.amazon.com/es/blogs/aws/aws-backup-automate-and-centrally-manage-your-backups/
https://docs.aws.amazon.com/es_es/AWSEC2/latest/UserGuide/EBSSnapshots.html
https://aws.amazon.com/es/backup/pricing/
