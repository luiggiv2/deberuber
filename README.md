# Prototipo simple utilizando servicios API Gateway y funciones Lambda de AWS

Se implementa una pequeña aplicación en la que se muestra un mapa que permita seleccionar dos puntos en el mapa y que al dar click en el botón de `Iniciar Movimiento` un tercer marcador recorra el trayecto entre ambas coordenadas o puntos seleccionados. Para ello emplearemos una función lambda y una API REST. 

**AWS Lambda** es un servicio de computación sin servidor que te permite ejecutar código en respuesta a eventos sin tener que aprovisionar o gestionar servidores. Lambda se encarga de ejecutarlo cuando sea necesario, escalando automáticamente los recursos para manejar la carga de trabajo.

**API REST** en términos más simples, es un conjunto de reglas y estándares que permiten que diferentes aplicaciones se comuniquen entre sí. Permiten que las aplicaciones y servicios interactúen con los recursos y servicios proporcionados por AWS a través de la web, usando protocolos estándar como HTTP. Estas API se implementan en la arquitectura de AWS para permitir la comunicación entre clientes (como aplicaciones móviles, navegadores, o servidores) y los servicios de AWS de manera eficiente y escalable.

## Arquitectura implementada

<img width="698" alt="Captura de pantalla 2024-12-13 a la(s) 21 03 20" src="https://github.com/user-attachments/assets/a08445d0-cb14-4c60-bafc-4e3f51946b80" />


## Como primer paso crearemos la función lambda

En este caso se ha creado la función lambda con python, a continuación se adjunta el código de la misma. 

<img width="606" alt="Captura de pantalla 2024-12-02 a la(s) 0 05 06" src="https://github.com/user-attachments/assets/89b50d0f-87ab-4ffb-b49d-c7bb22d9a3ef">

```python
import json

def lambda_handler(event, context):
    print("Evento recibido:", event)
    body = json.loads(event['body'])
    pointA = body['pointA']
    pointB = body['pointB']

    steps = 100
    lat_step = (pointB['lat'] - pointA['lat']) / steps
    lng_step = (pointB['lng'] - pointA['lng']) / steps

    route = [
        {'lat': pointA['lat'] + lat_step * i, 'lng': pointA['lng'] + lng_step * i}
        for i in range(steps + 1)
    ]

    return {
        'statusCode': 200,
        'headers': {'Content-Type': 'application/json'},
        'body': json.dumps({'route': route})
    }
```

Esencialmente crea una ruta directa entre dos puntos geográficos dados. Imagina que tienes un mapa y quieres trazar una línea recta entre dos ciudades. Este código calcula todos los puntos intermedios necesarios para dibujar esa línea.

`import json` este paso es fundamental para poder trabajar con datos en formato JSON, que es el formato estándar para transmitir datos en internet.
`def lambda_handler(event, context):` Eesta es la función principal que se ejecutará cuando se invoque la función Lambda. Los parámetros `event` y `context` proporcionan información sobre el evento que desencadenó la ejecución de la función y el contexto de ejecución, respectivamente.


```
body = json.loads(event['body'])
pointA = body['pointA']
pointB = body['pointB']
```

Aquí se extraen las coordenadas de los dos puntos (puntoA y pointB) del cuerpo de la solicitud. Se asume que los datos están en formato JSON.

```
steps = 100
lat_step = (pointB['lat'] - pointA['lat']) / steps
lng_step = (pointB['lng'] - pointA['lng']) / steps
```

Luego se define un número de pasos (100 en este caso) y se calcula el incremento en latitud y longitud por cada paso. Esto divide la línea recta en 100 segmentos iguales.

`
route = [
    {'lat': pointA['lat'] + lat_step * i, 'lng': pointA['lng'] + lng_step * i}
    for i in range(steps + 1)
]
`

Esta línea utiliza una comprensión de listas para crear una lista de puntos. Por cada paso, se calcula la latitud y longitud del nuevo punto y se agrega a la lista.

```
return {
    'statusCode': 200,
    'headers': {'Content-Type': 'application/json'},
    'body': json.dumps({'route': route})
}
```

Finalmente, se devuelve una respuesta HTTP con el código de estado 200 (éxito) y el cuerpo de la respuesta en formato JSON, que contiene la lista de puntos que conforman la ruta.

**¿Cómo funciona en la práctica?**

- **Solicitud** Cuando se invoca esta función Lambda, se le envía una solicitud JSON con las coordenadas de los dos puntos como entrada.
- **Procesamiento** La función calcula los puntos intermedios entre los dos puntos dados y crea una lista de estos puntos.
- **Respuesta** La función devuelve una lista de puntos que representa la ruta entre los dos puntos de entrada.
  
Si realizamos una prueba de la función lambda podemos validar que la misma funciona y se obtienen las coordenadas.

<img width="1244" alt="Captura de pantalla 2024-12-02 a la(s) 0 24 30" src="https://github.com/user-attachments/assets/4b78430e-d950-489d-a73c-e6f68ff9c124">

## Luego crearemos la API REST 

Una vez que se tiene creada la función lambda se procede a crear la API REST con método POST, cuando se utiliza el método POST en una API REST asociada a una función Lambda, se está enviando datos a esa función.

<img width="1260" alt="Captura de pantalla 2024-12-02 a la(s) 0 35 00" src="https://github.com/user-attachments/assets/5630d571-63e7-42a0-9e30-89ccc6805ed8"> <br/>

**Ventajas de usar API REST con Lambda**

- **Escalabilidad** lambda se escala automáticamente para manejar grandes cargas de trabajo.
- **Desarrollo rápido** Se puedes crear APIs rápidamente sin preocuparse por la infraestructura.
- **Integración** se integra fácilmente con otros servicios de AWS.

Una vez que se encuentra creada la API, se procede a realizar una probación de la misma, para ello lo hacemos mediante **Postman** que es una herramienta de desarrollo de software que facilita la creación, prueba y documentación de APIs.

<img width="1034" alt="Captura de pantalla 2024-12-02 a la(s) 0 45 56" src="https://github.com/user-attachments/assets/3509ba31-97f3-4ad6-be1e-99e5b8da5e7c"> <br/>

Cuando se tiene creada la API y se ha implementado la misma se obtiene la URL de Invocación.

`https://qnrykbwrvk.execute-api.us-east-1.amazonaws.com/uberdeber/move`

Esta URL la copiamos en el archivo `index.html` adjunto en el presente repositorio.

<img width="878" alt="Captura de pantalla 2024-12-02 a la(s) 0 54 50" src="https://github.com/user-attachments/assets/852c990c-0892-48c2-85a0-b83594340932"> <br/>

Dentro del archivo `index.html` se encuentra un código JavaScript, en conjunto con la biblioteca Leaflet, permite visualizar un mapa interactivo y simular el movimiento de un marcador a lo largo de una ruta definida. El código permite a los usuarios seleccionar dos puntos en un mapa y luego simula el movimiento de un marcador a lo largo de una ruta calculada entre esos dos puntos. La ruta se obtiene haciendo una solicitud a una API externa. El movimiento se logra utilizando la función `setTimeout` para actualizar la posición del marcador en intervalos regulares.

