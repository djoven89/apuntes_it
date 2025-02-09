# AWS - Lambda

Es un servicio serverless de computación, permite subir el código de la aplicación mediante funciones y luego invocarlas.

## Características principales

* Servicio serverless.
* AWS se encarga de gestionar los servidores que necesita la aplicación para funcionar.
* Es posible invocar manualmente una función desde:
        * Consola, petición HTTP, AWS CLI o AWS CDK.
* Las funciones pueden ser disparadas por otros servicios de AWS mediante eventos (JSON). Algunos de los servicios con los que se puede integrar son:
        * Amazon S3
        * Amazon SNS
        * Amazon API Gateway
* Los servicios Amazon SQS y Amazon Kineses no puede disparar directamente una función, sino que se usa la funcionalidad [event source mapping](https://docs.aws.amazon.com/lambda/latest/dg/invocation-eventsourcemapping.html).
* La función está compuesta por:
        * Lenguaje de programación y su versión.
        * Arquitectura.
        * CPU, RAM y almacenamiento efímero.
* Cuando una función es invocada, el entorno creado se queda congelador por unas horas por si hubiera que gestionar más eventos.
* Si una función requiere de librerías externas, éstas deberán de incluirse junto al código, habiendo distintas formas:
    1. [Archivo zip](https://docs.aws.amazon.com/lambda/latest/dg/configuration-function-zip.html).
    2. [Lambda Layers](https://docs.aws.amazon.com/lambda/latest/dg/chapter-layers.html).
    3. [Contenedor](https://docs.aws.amazon.com/lambda/latest/dg/images-create.html) de Docker.
* Podemos usar las funcionalidades [alias](https://docs.aws.amazon.com/lambda/latest/dg/configuration-aliases.html) y [version](https://docs.aws.amazon.com/lambda/latest/dg/configuration-versions.html) para invocar una versión de la función inmutable.
* Podemos verificar que el código ejecutado está correctamente firmado por una fuente confiable usando la funcionalidad [code signing](https://docs.aws.amazon.com/lambda/latest/dg/configuration-codesigning.html).
* Una función puede invocarse de forma síncrona o asíncrona.
* Es posible crear una [URL pública](https://docs.aws.amazon.com/lambda/latest/dg/urls-configuration.html) para invocar a la función.
* El servicio dispone de una quota que limite el número de invocaciones simultáneas (concurrency).
* Mediante [reserved concurrency](https://docs.aws.amazon.com/lambda/latest/dg/configuration-concurrency.html) podemos establecer límites a las invocaciones simultáneas de las funciones.
* Mediante [provisioned concurreny](https://docs.aws.amazon.com/lambda/latest/dg/provisioned-concurrency.html) podemos tener entornos de ejecución listos para responder a las invocaciones.
* El servicio [Application Auto Scaling](https://docs.aws.amazon.com/autoscaling/application/userguide/services-that-can-integrate-lambda.html) nos permite escalar bajo demanda o programado el incremento de entornos de ejecución.
* Usando [Lambda SnapStart](https://docs.aws.amazon.com/lambda/latest/dg/snapstart.html) podemos incrementar los tiempos de respuesta y escalado.
* Podemos usar [hooks](https://docs.aws.amazon.com/lambda/latest/dg/snapstart-runtime-hooks.html) cuando usamos Lambda SnapStart.
* Mediante el servicio Amazon Application Auto Scaling podemos configurar unas reglas para que el runtime de la función escale automáticamente según la carga.
* Podemos monitorizar las funciones con los servicios:
        * [CloudWatch metrics](https://docs.aws.amazon.com/lambda/latest/dg/monitoring-metrics.html).
        * [CloudWatch Logs](https://docs.aws.amazon.com/lambda/latest/dg/monitoring-cloudwatchlogs.html).
        * [AWS X-Ray](https://docs.aws.amazon.com/lambda/latest/dg/services-xray.html).
* Tiene un escalado 'scale-out' automático.
* Cada función es independiente (1 evento = 1 función).
* Un evento puede ejecutar X funciones de Lamba.
* Una función Lambda puede llamar a otra función Lambda.
* Lamba realiza las acciones de forma global (no depende de regiones o AZs).
* Hay unos pocos servicios los que pueden interactuar con Lamba.

## Consideraciones

1. Cuidado con crear bucles infinitos (lambda -> S3 -> Lambda -> S3)
2. Ser precisos con las dependencias que requiere la función (cargar un método en lugar de toda la librería).
3. Diseñar con detalle la fase de inicialización de la función, dado que el entorno se reusa aproximadamente durante 5 horas.
4. No usar global variables dado que si el entorno no fue eliminado por AWS, no volverán a inicializarse.
5. Usar Secrets Manager o Parameter Store para evitar nuevos despliegues en cambio de datos poco frecuentes.
6. Funciones sencillas, concretas y agnósticas.
7. Usar DynamoDB TTL para evitar que un evento se procese múltiples veces, es decir, que éste sea idempotente.
8. Analizar el sentido de la implementación de la función, quizás tenga más sentido que sea S3 quien haga la invocación en lugar de que sea Lambda.
9. Evitar tareas asíncronas, es decir, que requieran llamar a otros componentes para poder terminar su procesamiento, el motivo son costes, tiempo de ejecución, gestión de errores y escalado.
        * Usar Amazon SQS suele ser una opción recomendable.

## Detalle

* Servicio serverless.
* AWS se encarga de gestionar los servidores que necesita la aplicación para funcionar.
* Una función Lambda se puede invokar manualmente vía:
    * Consola, petición HTTP, AWS CLI o AWS CDK.
* También mediante un evento específico, al cual se le ha asociado un trigger de Lambda, la cual será invocada por el servicio de AWS, convirtiéndolo en un objeto JSON llamado `event`.
* Algunos otros servicios de AWS como Amazon Kinesis o Amazon SQS no pueden invocar a Lambda directamente, por lo que es necesario crear un `event source mapping`. Dicho evento hace que la función Lambda está continuamente haciendo 'polls' de los servicios mencionados para verificar si hay nuevos eventos a gestionar
* Algunos servicios de AWS invocan directamente una función Lambda usando `triggers`.
    * Estos servicios hacer un event push a Lambda, la cual invoca la función inmediatamente cuando el evento ocurre.
    * Este modo es apropiado para proceso en tiempo real o eventos discretos/aislados.
    * Algunos servicios que usan triggers para invocar a las funciones:
          * S3 (cuando un objeto es creado, eliminado o modificado en un bucket).
          * SNS (cuando un mensaje es publicado en un topic).
          * API Gateway (cuando una request es realizada a un endpoint concreto).
* Es posible usar la herramienta [AWS Lambda Power Tuning](https://github.com/alexcasalboni/aws-lambda-power-tuning) para medir el rendimiento de la función mediante distintos valores de la RAM.
* Al incrementar la RAM de la función, también se obtiene más CPU.
* El entorno de ejecución tiene un almacenamiento efímero en `/tmp/`, el cual perdura mientras el entorno esté levantado.
    * El mínimo son 512MB pero puede incrementarse hasta 10GB.
    * Toda la información almacena es cifrada en rest.
* Es posible conectar funciones Lambda a VPC de modo que éstas pueden usar servicios alojados en redes privadas.
* Es posible invocar una función Lambda desde un recurso de AWS a través de AWS PrivateLink sin necesidad de ir por redes públicas - Internet -.
* Podemos montar sistemas de archivos Amazon EFS en nuestras funciones Lambda.
* Podemos obtener información dinámicamente usando de varias maneras:
    * Variables de entorno: Se configurarán en la propia función.
    * Secrets Manager: Se obtienen los valores realizando llamadas a la API.
    * Parameter Store: Se obtienen los valores realizando llamadas a la API.
* El coste que tiene Lamba se basa en 2 partes:
    * Número de peticiones:
          * El primer millón son gratuitas.
    * Duración de la función:
          * Tiempo que dura la función y la RAM asignada a dicha función.

## Event source mapping

* Es un recurso de Lambda que leer desde un stream o un servicio de solas e invoca una función con un lote de registros.
* Los recursos se llaman **event pollers**, es decir que activamente están haciendo poll en busca de nuevos mensajes, los cuales acaban invocando la función Lambda.
* Los eventos se ejecutan al menos una vez, aunque es posible que se hagan más de una vez, por lo que es importante hacer las funciones idempotentes.
* Está diseñado para el procesamiento de una gran volumen de streaming data o mensajes de colas.
* Por defecto, se agrupan los registros en un simple payload, el cual es enviado a la función.
* Los condiciones para que se ejecute el batching:
  1. Por tiempo.
  2. Por tamaño del batch,
  3. Por tamaño en bytes de los batches.
* **TODO** https://docs.aws.amazon.com/lambda/latest/dg/invocation-eventsourcemapping.html

## Permisos
Hay dos tipos de permisos para las funciones:
* El que necesita la función para acceder a otros recursos de AWS (`execution role`).
    * Al crear una función, automáticamente se crea un rol con permisos de escritura en Amazon CloudWatch Logs.
* El que necesitan otros recursos o usuarios para invocar a la función.
    * El permiso de otros servicios servicios para usar la función es `lambda:InvokeFunction`.

* Una función tiene acceso al directorio `/tmp/`.

## Lifecycle

Las fases que conforman el `Lambda execution environment lifecycle` son:

1. `Init`.
    * Realiza 3 tareas en 10 segundos (iniciar las extensiones, iniciar el runtime y cargar la función).
    * Adicionalmente, puede ejecutar un hook `before-checkpoint` mediante Lambda SnapStart.
2. `Invoke`.
    * Aquí es cuando se ejecuta el código.
    * Se dispone de un timeout máximo de 360 segundos para que nuestra ejecución termine.
    * La duración reportada es la suma de la fase de init y el tiempo que tardó la función en concluir.
    * Si la función falla u obtiene un timeout, el entorno de ejecución es reiniciado.
* Cuando una función fue invocada, Lambda mantiene el entorno de ejecución activo (queda congelado) por unas horas por si se hubiera otro evento que gestionar.
    * También afecta a invocaciones de tipo **continuously**.
    * Las declaraciones fuera del método `handle` persisten, por ejemplo, la conexión establecida por una DB - es recomendable revisar si ya conexión ya existe antes de crear una nueva -.
    * Los datos cacheados almacenados en `/tmp/` persisten, por lo que es recomendable analizar si son válidos para las siguientes ejecuciones.
    * Hay que asegurar de no dejar procesos en segundo plano o callbacks comprobando esto mismo antes de arrojar un exit en el código.
* Hay dos tipos de 'starts' ![ver](https://docs.aws.amazon.com/images/lambda/latest/dg/images/perf-optimize-figure-1.png):
    * **Cold**: No hay entorno de ejecución activo, por lo que Lambda debe descargar el código e inicializar el entorno antes de inicializar el código y ejecutar el handle.
        * Este tipo de 'start' provoca una mayor latencia (100ms a 1s).
    * **Warn**: Cuando el entorno de ejecución está activo - congelado -, dado que no hay que inicializar el entorno.
* Antes de ejecutar el código del handle, Lambda hace una 'static initialization' del código, es decir, importa las dependencias y librerías, establece la configuración e inicializa la conexión con otros servicios (el código ejecutado antes de llamar a la función `lambda_handler`).
    * Si el entorno de ejecución sigue activo, no vuelve a inicializarse de nuevo.
    * Hay que tratar de ser precisos a la hora de realizar las llamadas a los servicios (en lugar de usar `aws-sdk` usar `aws-sdk/clients/dynamodb`).
    * Suele ser recomendable iniciar las conexiones a la DB para así ser reutilizadas en posteriores invocaciones.
    * Usar `variable scope` en lugar de `global variables`, dado que si el entorno se mantiene, estás continuarán teniendo su valor.

## Lambda Runtime

* Lambda soporte una serie de lenguajes de programación como Python o NodeJS, [aquí](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html#runtimes-supported) el detalle completo.
* Es posible usar [OS-only runtime](https://docs.aws.amazon.com/lambda/latest/dg/runtimes-provided.html) para usar otros lenguajes de programación.
* Cada major release tiene su runtime concreto, por ejemplo; Python 3.13.
* Para que una función use una versión de runtime superior, habrá que cambiar el runtime identifier.
* Para funciones definidas como contenedores, es necesario crear una nueva imagen.
* Si se usó un archivo .zip para desplegar un paquete, es necesario actualizar la configuración de la función.
* Un runtime tiene su ciclo de vida:
    * Release -> Deprecated date -> Block function create (30 días tras el deprecated date) -> Block function update (60 días tras el deprecated date).
* La política de deprecated runtime se produce cuando el runtime lleva al final de su LTS y ya no recibe actualizaciones de seguridad.
* Mediante Lambda versions y aliases es posible realizar la migración a una versión superior del runtime.
* Es posible seguir usando funciones que se ejecutan sobre runtimes deprecated, no obstante, éstas son vulnerables a nivel de bugs, seguridad y rendimiento.
* Se reciben notificaciones advirtiendo que nuestro runtime en uso quedará deprecated en 180 días.
* Por defecto para los runtimes gestionados, Lambda aplica las actualizaciones automáticamente, aunque es posible indicar que sólo se actualice el runtime cuando la función sea actualizada o manualmente, indicando el ARN del runtime.

## Alias y Versions

**Alias**

* Referencia a una versión concreta de una función.
* Es posible actualizar el alias.
* Es posible desplegar nuevas versiones mediante un Canary Deployment.

**Versions**

* La última versión de la función sin publicar se denomina `$LATEST`.
* Cada vez que se publica una nueva versión de la función se sobreescribe el código de `$LATEST`.
* `$LATEST` es mutable, es decir, podemos modificarla.
* Cuando una versión es publicada, se convierte en inmutable.
* Es posible referenciar una función usando Qualified o Unqualified ARN.
    * Qualified: Referencia a una versión de la función.
    * Unqualified: No referencia ninguna versión. Además, no es posible usar un alias con este tipo.

## Lambda layers

* https://docs.aws.amazon.com/lambda/latest/dg/chapter-layers.html
* Es un archivo .zip que contiene código o datos adicionales.
* Generalmente contienen dependencias de librerías, archivos de configuración o custom runtimes.
* Es posible usar la misma layer en múltiples funciones.
* El contenido de la layer se extrae en el directorio `/opt/` del entorno de ejecución.
* No es posible usar esta funcionalidad con funciones Dockerizadas.
* Una **layer version** es una snapshot inmutable de una versión concreta. Cuando creamos una layer, Lambda crea su versión concreta.
* Es posible vincular un máximo de 5 layers en una función.
* La función debe ser compatible con el runtime y la arquitectura.
* El tamaño total de las layers no puede superar los 250MB.

## Code signing

* Permite verificar que el código ejecutado está correctamente firmado por una fuente confiable.
* Lambda comprueba y verifica el código de cada deployment.
* No está disponible para funciones desplegadas con imágenes Docker.
* A través de [AWS Signer](https://docs.aws.amazon.com/signer/latest/developerguide/Welcome.html) se gestiona las firmas de los paquetes de las funciones y sus layers.
    * Servicio sin costes.
* Es necesario subir la función Lambda como **.zip** en un bucket de S3, dado que AWS Signed firmará dicho archivo y generará uno nuevo, siendo éste el usado por nuestra función.
* En AWS CloudTrail se registran los deployments exitosos o fallidos.
* Amazon Lambda realiza las siguientes pruebas de validación cuando desplegamos un cambio en nuestra función firmada:
    * **Integrity**: Valida que el paquete no ha sido modificado desde que fue firmado (revisa el hash).
    * **Expiry**: Valida que la firma no ha expirado.
    * **Mismatch**: Valida que la firma se haya hecho con un perfil permitido.
    * **Revocation**: Valida que la firma no haya sido revocada.
* Cuando una validación falla, podemos tomar dos acciones:
    * **Warn**: Advierte en CloudWatch y CloudTrail aunque la función es ejecutada.
    * **Enforce:** Advierte y bloquea la ejecución.

## Tipos de invocaciones

**Synchronous invocations**

* Esperamos a que la función procese el evento y devuelva una respuesta.
* En caso de error, se muestra el mensaje en la respuesta y los retries deberán de ser manuales.
* La respuesta contiene información adicional como la versión de la función ejecutada.

**Asynchronous invocations**

* Se envía el evento a Lambda, el cual es encolado y proceso posteriormente. Lambda nos devuelve inmediatamente un simple **HTTP 202**.
* Para que la invocación sea de este tipo, es necesario settear el parámetro `InvocationType` con valor `Event`.
* En caso de error, Lambda hace los retries automáticamente.
    * Por defecto, lo intento dos veces más, con ciertos minutos de margen entre los intentos.
* Podemos configurar la gestión de errores.
    * Tiempo máximo en segundos en los que un evento está encolado antes de descartarse.
    * Número máximo de intentos.
* Es posible enviar registros de la invocación a ciertos servicios de AWS (destinations), estos registros contienen información adicional de la requests así como su response.
    * Es posible establecer distintos destinations según si el procesamiento fue exitoso o fallido.
    * Podemos configurar una dead-letter queue (SQS o SNS) para enviar los eventos descartados, estos tendrán el contenido del evento pero no los detalles de la respuesta.
    * Como alternativa tenemos dead letter queues.

## Concurrency

* https://docs.aws.amazon.com/lambda/latest/dg/lambda-concurrency.html
* Número de requests **in-flight** que la función Lambda gestiona al mismo tiempo.
* Cada execution environment puede gestionar una única request (concurrency 1). Cuando la invocación termina, dicho entorno se reusa para gestionar una nueva petición.
* Cuando se reciben múltiples requests al mismo tiempo, se van creando nuevos execution enviroments.
* Lambda tiene una quota por defecto de 1.000 execution environments por región, es decir, que se pueden gestionar 1.000 invocaciones (concurrency 1.000) simultáneas.
* [Aquí](https://docs.aws.amazon.com/lambda/latest/dg/lambda-concurrency.html#calculating-concurrency) se muestra una fórmula para calcular el concurrency de una función Lambda.
* Cuando se llega al límite de la quota de concurrency, la función experimenta **throttling**, es decir, se rechazan las requests (429 status code).
* Hay dos tipos de concurrency:
    * **Reserved**
        * Reserva un número de concurrency para una función concreta, de modo que se evita que otra función acapare la creación de execution environments. Dicho de otro modo, es el número máximo de execution environments que puede usar una función.
        * No hay costes adicionales por usar esta configuración.
    * **Provisioned**
        * Pre-inicializa un número concreto de execution environments para una función.
        * Reduce la latencia del **cold start**.
        * El uso de esta funcionalidad tiene coste adicional.
        * A excepción de Java 11 o 17, no es posible usar la funcionalidad **Lambda SnapStart** junto con **Provisioned concurrency**.
        * Es posible usar **Reserved concurrency** también.
        * No es posible tener más **provisioned** que **Reserved** concurrency.
        * Sólo puede aplicarse a nivel de alias o versión.
        * Es posible usa el servicio Application Auto Scaling.
        * Hay dos maneras de configurar el servicio:
            * **Schedule scaling**
                * Para tráfico predecible.
                * Es posible configurar en base a una métrica de CloudWatch o a un fecha y hora concreta.
                * [Artículo](https://aws.amazon.com/blogs/compute/scheduling-aws-lambda-provisioned-concurrency-for-recurring-peak-usage/) donde se realiza la implementación.
                * [Doc](https://docs.aws.amazon.com/autoscaling/application/userguide/application-auto-scaling-scheduled-scaling.html) donde explica la funcionalidad a implementar.
            * **Target tracking**
                * Cuando no hya patrones de tráfico predecibles.
                * Crea y gestiona una serie de alertas en CloudWatch basadas en la política definida.
                * Cuando la alertas se activa, se Application Auto Scaling ajusta automáticamente el valor de provisioned concurrency.
                * [Breve explicación](https://docs.aws.amazon.com/lambda/latest/dg/provisioned-concurrency.html#managing-provisioned-concurency) de como implementarlo.
                * [Doc](https://docs.aws.amazon.com/autoscaling/application/userguide/application-auto-scaling-target-tracking.html) donde explica la funcionalidad a implementar.
          * Se requiere definir un valor inicial en el provisioned concurrency.
          * Ambos modos de configuración deberán ser aplicados vía CLI, dado que la configuración no está disponible vía consola.

## Lambda SnapStart

**TODO** https://docs.aws.amazon.com/lambda/latest/dg/snapstart.html

* La fase de Init sucede tras publicar una función. Lambda hace una snapshot de la memoria y el estado del disco tras la iniciación del entorno de ejecución, la cifra y cachea.
* Si tenemos before-checkpoint runtime hook, el código se ejecutará al final de la fase de Init.
* `Hooks`
    * https://docs.aws.amazon.com/lambda/latest/dg/snapstart-runtime-hooks.html

## Monitorización

**TODO** https://docs.aws.amazon.com/lambda/latest/dg/monitoring-cloudwatchlogs.html

**X-ray**

* Permite debugear lo que ocurre en una aplicación serverless.
