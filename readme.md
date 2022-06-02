# Backyard Bot

Bot de Slack que entrega notificaciones y presenta funcionalidades de Backayrd a los miembros de Unholster.
***
## Reporte Unholsterianos Ausentes

* Desde la plataforma Rex+ es posible extraer reportes relacianados a la gestión de los miembros de la empresa dentro de los reportes que se pueden cosultar a través del Generador de Consultas +, y de los cuales se nutre este bot son las consultas de vacaciones y de permisos administrativos.

* El proceso inicia con Rex+ y su sistema automatizado para enviar los reportes del generador por medio de un correo predeterminado y en donde además es posible programar este envío las veces que se requiera de este reporte. Integrando los servicios de AWS en este proyecto, se configura [SimpleEmailService](web ses), servicio que utiliza una dirección de correo electrónico, el cual recibirá este email proveniente de Rex+, y realiza la acción de derivar este correo en su totalidad a un bucket de [S3](web s3) que almacena todo el contenido en un formato MIME, una vez almacenado el correo se invoca a una función [Lambda](web lambda) que ejecuta un código desarrollado en Python capaz de extraer los datos del reporte tomando los nombres de los miembros de la empresa, y filtrando tanto las fechas de inicio y término de sus vacaciones o permisos administrativos, organizar estos datos en dos listas para luego generar un mensaje que será emitido en la plataforma de comunicación Slack. 

* La invocación de la lambda se debe a un desencadenador en este servicio, y consiste en que cada vez que un correo se almacene en un bucket, gatillará la ejecución del código que genera el mensaje de contiene a los Unhlsterianos  Ausentes, esto es considerado un evento dentro del servicio y es configurado directamente en la lambda especificando el script que se debiese ejecutar.
***
## Reporte Status de Facturación

* Existe un tablero en Trello, manejado por los altos cargos de la empresa en donde se maneja la facturación de los servicios que presta Unholster a sus clientes, tablero del cual se extrae la suficiente información para generar un mensaje a modo de recordatorio semanal mostrado en Slack para lo miembros encargados de cada factura.

* La extracción es posible gracias a un script desarrollado en Python que permite acceder y obtener datos específicos de cada tarjeta (que representa una factura) en cada lista que posee el tablero de facturación. La información que se requiere de cada tarjeta es el nombre del encargado, la fecha de vencimiento de cada tarjeta, y el detalle que considera tanto el valor de la factura y un pequeño resumen que posee tanto el servicio y a quién se le prestó dicho servicio. Esta información es organizada de la misma forma que se muestra en el tablero, a través de listas pero con una estructura que permita la visualización en la plataforma Slack.

* Este código está alojado dentro de la misma lambda que el anterior, y como se menciona, se utiliza un desencadenador contemplando un nuevo evento, esta vez programando un EventBridge en el servicio CloudWatch de Amazon Web Service. Este evento se configura a través de una expresión Cron, especificando la hora, el día y el mes, de esta manera indicamos que para este reporte, el código se ejecutará todos los Lunes de todos los meses a las 7 AM.
***
## Cómo cargar/actualizar el paquete de implementación de la lambda

Se deben considerar ambas situaciones, tanto la carga al momento de crear e iniciar la configuración de una nueva lambda, como también una atualización de los códigos e incluso nuevas dependencias que requiera al momento de ser ejecutados los scipts. 

> Es posible cargar los archivos de forma "manual" y es directamente en la lambda, en la esquina superior de donde se encuentra el código fuente en "Cargar desde" y selecciona "Archivo .zip". Es importante destacar el hecho que previo a esta acción, es necesario ubicar tanto códigos como las carpetas de las dependencias en una sola carpeta para luego comprimirlas en el forma zip.

> Una forma más automatizada para este procedimiento, es por medio de la terminal, y es que una vez conectados con AWS, se puede utilizar el comando ```aws lambda update-function-code --function-name <NOMBRE_LAMBDA> --zip-file fileb://<NOMBRE_PAQUETE>.zip```, de esta forma estamos cargando el zip con los archivos.
***