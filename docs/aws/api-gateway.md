# Amazon API Gateway

Es un servicio gestionado que hace sencillo para los desarrolladores publicar, mantener, monitorizar y securizar API.

## Características principales

* Los tipo de APIs que podemos configurar son:
    * REST
    * HTTP
    * WebSocket
* Las funcionalidades que ofrece el servicio son:
    * WebSocket (stateful).
    * HTTP y REST (stateless)
    * Autenticación flexible con servicios como:
        * IAM
        * Lambda authorized functions
        * Amazon Cognito
    * Despliegues de tipo `canary` para Rest API.
* El servicio puede integrar con:
    * Amazon CloudTrail
    * Amazon CloudWatch
    * AWS WAF
    * AWS X-RAY
* Es posible crear un `Mock integration` en una REST API.
* Para elegir entre una API REST o HTTP, tenemos [este](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-vs-rest.html) enlace donde nos explica las funcionalidades disponibles.

## Terminología y conceptos

* `API method`: Combinación de la ruta de un recurso con el verbo HTTP, por ejemplo:
    * `POST /incomes`
    * `GET /expenses`
* `API stage`: Referencia lógica a un ciclo de vida de la API. Es decir, el entorno (prod, staging, etc).
* `Developer portal`: Aplicación que permite a los clientes registrar, descubrir y suscribirse a la API, además de gestionar sus API keys y visitar las métricas de uso.

## HTTP API

* Es una RESTful API.
* HTTP API está diseñada con menos funcionalidades para tener un precio más bajo.
* Comparación entre REST API y HTTP API [aquí](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-vs-rest.html).
* Funcionalidades principales:
    * Es posible enviar las request a AWS Lambda o a cualquier HTTP endpoint.
    * Autenticación con OpenID Connect y OAuth 2.0.
    * [CORS](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-cors.html).
    * Despliegues automáticos.
* Para crear una API sencilla se necesitan configurar lo siguiente como mínimo:
    * Una ruta.
    * Una integración.
    * Un stage.
    * Un deployment.
* `Workflow de una request`:
    1. El cliente envía una API request.
    2. API Gateway determina a que `stage` debe enruta la petición.
        * Si se hace match con un stage, se enruta a éste la request, sino irá a `$default`, en caso de no existir, devuelve un `{"message":"Not Found"}` sin generar registros en CloudWatch Logs.
    3. Una vez que el stage fue seleccionado, API Gateway selecciona la route.
        * La selección se hace por el match más específico, usando las siguientes prioridades.
            1. Match completo de route y method.
            2. Greedy path variable.
            3. $default.
    * En caso de no haber match, se devuelve un `{"message":"Not Found"}` al cliente.
* `Routes`
    * Enruta las API requests a un backend.
    * Conta de dos partes:
        * HTTP method.
        * Es posible usar `ANY` para hacer match de todos los métodos no definidos en el resource.
        * Podemos usar `$default` para actuar como **catch-all** para todas las requests que no hagan match en otros routes.
        * Resource path.
        * Es posible usar variables. `GET /pets/{petID}`.
        * A **greedy path variable** catches all child resources of a route.
            * Debe añadirse un `+` al final de la variable: `{proxy+}`.
            * Debe estar siempre al final del path del resource.
    * API Gateway decodifica los request parameters de la URL antes de pasárselos a la integración del backend.
    * Por defecto, los query strings son enviados a la integración del backend si están incluidos en la request.
* `Access control`
    * Hay tres mecanismos para controlar y gestionar el acceso a la HTTP API:
        * **Lambda authorizers**
        * Cuando un cliente hace una API requests, API Gateway invoca una función Lambda. Dependiendo de la respuesta, el cliente podrá o no acceder a la API.
        * En caso de no usar la consola de AWS; habrá que indicar la versión del payload soportada (1.0 ó 2.0).
        * Opcionalmente, se pueden establecer [identity sources](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-lambda-authorizer.html#http-api-lambda-authorizer.identity-sources) para la autorización de Lambda, de modo que si el cliente no los proporciona en la requests, API Gateway le devolverá un **401** sin llegar siquiera a invocar la función.
            * Headers.
            * Query string.
            * Context.
            * Stage.
        * Es posible cachear la autorización.
    * **JWT authorizers**
        * Es posible usar JWT como parte de OIDC y OAuth 2.0 para restringir el acceso a la API.
        * Cuando un cliente hace una API request, API Gateway valida el JWT enviado por el cliente. y determina si puede acceder a la API basándose en la validación del token y opcionalmente, en el scope del token.
        * API Gateway enviará el **claim** en el token a la integración de la ruta en cuestión.
    * **Standard AWS IAM roles and policies**
        * El cliente deberá firmar la request con [SigV4](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_sigv-create-signed-request.html).
        * Será necesario que se disponga del permiso `execute-api` en la ruta para que API Gateway se invoque.
  * `Integrations`
    * Permite conectar un route a recursos backend.
    * Soporta:
      * **Lambda proxy**
        * En caso de no usar la consola de AWS; habrá que indicar la versión del payload soportada (1.0 ó 2.0).
      * **AWS service**
        * Es posible integrar uno de [estos servicios](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-develop-integrations-aws-services-reference.html) de AWS usando first-class integrations.
      * **HTTP proxy integrations**
        * Permite establecer un endpoint HTTP público.
        * API Gateway envía toda la request y la respuesta.
      * **Private integrations**
        * Se integra con recursos privados de una VPC, que son:
          * ELB
          * ECS
          * AWS Cloud Map service.
        * Es necesario crear un [VPC link](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-vpc-links.html) para usar esta funcionalidad.
  * Mediante **parameter mapping** podemos modificar la request ya sea por parte del clientes antes de enviarla a la integración o la respuesta procedente de la integración antes de enviársela al cliente.
    * **Headers**, **Query strings** y el **path**, son los parámetros que podemos modificar antes de enviar a la integración.
    * **Headers** y **Status code** son los parámetros que podemos modificar de la respuesta de la integración antes de enviarla al cliente.
    * [Estas headers](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-parameter-mapping.html#http-api-mapping-reserved-headers) están reservadas, por lo que no podremos modificarlas.
* Podemos definir nuestra HTTP API mediante la importación de un archivo OpenAPI 3.0.
  * Podemos migrar una API REST a una HTTP API mediante la exportación de la API REST como OpenAPI 3.0.
  * Por defecto, es posible crear una API aún con errores en la estructura del archivo. Es posible modificar este valor.
* `Stage`
    * Referencia lógica a un ciclo de estado (dev, prod, v1, v2).
    * Podemos usar stages o custom domain names para publicar la API.
    * Es posible configurar integraciones y configuraciones distinta a cada stage.
    * Podemos definir variables a un stage, las cuales actuarían como variables de entorno.
        * No deben contener información sensible como credenciales.
    * Si queremos usar un dominio custom, tendremos que usar **API mapping** para conectar la API stage con el dominio.
        * Es posible usar el mismo map para HTTP y REST APIs stages de un mismo dominio custom.
        * Si queremos que los clientes sólo puedan usar la API bajo el dominio custom, habrá que deshabilitar el endpoint [execute-api](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-disable-default-endpoint.html).
    * Para enviar información sensible a las integraciones, usar [AWS Lambda authorized](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-lambda-authorizer.html#http-api-lambda-authorizer.payload-format-response) (usar el output de la función Lambda para enviar la información sensible a las integraciones).
    * En caso de necesitar establecer cookies con información sensible, es recomendable usar cookies con el prefijoo `__Host-` tal y como se comenta [aquí](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-publish.html).
* Podemos configurar la API con **throttling** para protegernos de recibir demasiadas requests.
    * API Gateway isa el algoritmo **token bucket**, se examina el rate y el burst of requests contra todas las APIs de una **región**.
    * Cuando las solicitudes superan los límites, el cliente recibe un **429 Too Many Requests**.
    * Es posible configurarlo a nivel de API stage o a nivel de routes.
    * A nivel de cuenta hay una quota que limita esta funcionalidad para protegernos de posibles ataques.
    * Los valores son por segundo y son dos conceptos:
        * **Burst limit**
        * Tamaño del bucket.
        * Número de peticiones concurrentes por segundo.
        * **Rate limit**
        * Número de peticiones por segundo.
* Podemos configurar [mutual TLS authentication](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-mutual-tls.html), es decir, el cliente debe presentar un certificado X.509 para verificar el acceso a la API.
    * Se usa para IoT y aplicaciones business-to-business.
* Podemos usar Amazon CloudWatch metrics y Logs para monitorizar la API.
    * Es posible que [estos casos](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-monitor.html) no sean registrados en Amazon CloudWatch.
    * Por defecto, las métricas se envían cada minuto.
    * Las métricas se almacenan durante 15 meses.
    * [Estas](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-metrics.html) son las métricas disponibles.
    * Es posible personalizar el access log con [estas variables](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-logging-variables.html).

## REST API

* Colección de recursos y métodos que se integran con un backend de tipo:
    * HTTP endpoint.
    * Lambda functions.
    * Otros servicios de AWS.
* El servicio interactúa con los backend a través de `integration requests` y `integration responses`.
* El frontend es encapsulado en `method requests` y `method responses`.
* Permite generar la documentación de la API con `API Gateway extensions to OpenAPI`.
* El módulo que usa este tipo de API es request/response, es decir, el cliente envía una petición a un servicio y el servicio response de forma sincronizada.
* Construimos una REST API como una colección de entidades programables llamadas `API Gateway Resources`.
* Cada entidad `Resource` tiene uno o más `Methods resources`.
    * Un `Method resource` es una petición entrante de un cliente que es expresada con parámetros y un body.
* Para integrar un `method resource` con un backend endpoint (llamado `Integration endpoint`), creamos un `Integration resource`.
    * Permite enrutar la petición del cliente a un integration endpoint concreto.  -
* Es posible transformar el parámetro o el body recibido para cumplir con los requisitos del backend.
* Para las respuestas, se crea un `Method response`, que representa la respuesta recibida del cliente.
* Para representar la respuesta del backend se crea un `Integration Response`.
* Es posible transformar los datos de la respuesta del backend antes de enviarlos al cliente.
* Para controlar quien puede consultar la API, podemos usar:
    * IAM.
    * Lambda authorized.
    * Amazon Cognito user pool.
* Para medir y limitar el uso de la API, podemos usar `usage plans`.
* El `API endpoint type` (hostname), puede ser de varios tipos:
  * `Edge-optimized`
        * Default.
        * Enruta las peticiones al POP de CloudFront más cercano a la ubicación geográfica del cliente.
        * El nombre de los HTTP headers se establecen en mayúsculas.
        * Los custom domain name se aplican a todas las regiones.
    * `Regional`
        * El uso es para clientes que están en la misma región (EC2 como cliente y API gateway en la misma región). También cuando tenemos pocos clientes pero con gran demanda.
        * Los custom domain name son específicos de una región.
        * Es posible integrar los custom domain con Amazon Route 53 para habilitar la funcionalidad [latency-based routing](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html#routing-policy-latency).
    * `Private`
        * Sólo puede accederse al API endpoint desde una VPC.
* Es posible cambiar el tipo de API endpoint, [aquí](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-api-migration.html) se explican las combinaciones.
* Al configurar el `API method` indicamos qué debería o debe enviar un cliente en su respuesta al backend, además, definimos las respuestas al cliente.
    * Es una petición HTTP, con su verbo HTTP, el path al recurso, headers y query strings.
    * Deberíamos configurar un payload cuando usamos `POST`, `PUT` o `PATCH`.
* Como `Method request` (input de la request), es posible elegir entre parámetros o payload.
* Como `Method response` (output), determinamos el STATUS code, headers y el body antes de enviarlo al cliente.
* Si no tenemos un proxy integrado, deberemos configurar los `method responses`.
    * Típicamente, las respuestas son 2XX, 4XX o 5XX.
    * Es común establecer 500 como respuesta por defecto para excepciones.

https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-method-settings-method-request.html

### Deployments

* https://docs.aws.amazon.com/apigateway/latest/developerguide/rest-api-publish.html

### Monitoring

## WebSocket APIs

* Tanto el cliente como el servidor pueden enviarse mensajes simultáneamente.
* El uso típico de este tipo de APIs es:
    * Chats.
    * Dashboards a tiempo real.
    * Alertas y notificaciones a tiempo real.
* El servicio permite:
    * Monitorización y limitación de conexiones y mensajes
    * Traceo de mensajes hacía los backends con AWS X-RAY.
    * Fácil integración con endpoints HTTP/HTTPS
