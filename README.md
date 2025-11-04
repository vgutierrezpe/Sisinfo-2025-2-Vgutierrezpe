# AWS Lambda Functions

Las *funciones Lambda* son una de las herramientas más conocidas del modelo serverless (sin servidor).  
Esto significa que puedes ejecutar tu código *sin tener que preocuparte por servidores, infraestructura o mantenimiento*.  
AWS se encarga de todo: desde los contenedores hasta la ejecución y el escalado.

## Concepto general

- *AWS Lambda* ejecuta tu código dentro de *contenedores administrados automáticamente*.  
- Cada vez que ocurre un evento (por ejemplo, subir un archivo o recibir una solicitud HTTP), Lambda *crea un contenedor temporal* para ejecutar tu función.  
- Cuando termina, ese contenedor se destruye o se reutiliza según sea necesario.  
- Si se requieren más ejecuciones simultáneas, *AWS lanza más contenedores automáticamente*. Basicamente tú solo te enfocas en el código; AWS se encarga del resto.

## Proceso de funcionamiento

1. Escribes el código de tu función (en Python, Node.js, Java, Go, etc.).  
2. Subes la función a *AWS Lambda* desde la consola o CLI.  
3. Defines un *evento de activación* (trigger) como:  
   - Subida de archivo a un bucket S3.  
   - Solicitud HTTP a través de API Gateway.  
   - Mensaje en una cola SQS o evento de DynamoDB.  
4. AWS crea un *contenedor* con todo lo necesario para ejecutar tu función.  
5. Cuando la ejecución termina, el contenedor se destruye o queda en espera para una nueva llamada.  

> AWS recomienda que las funciones sean *idempotentes*: si se ejecutan más de una vez, deben producir el mismo resultado sin causar errores ni duplicaciones.

---

## Costo

El costo se calcula en función de:

1. *La memoria asignada* a la función (de 128 MB a 10 GB).  
2. *El número de ejecuciones*.  
3. *La duración de la ejecución*, medida en milisegundos.  

> Pagas únicamente por lo que usas, no por el tiempo que el servidor está encendido.

### Ejemplo de costo

Si tienes una función que:
- Usa *512 MB de memoria*,  
- Dura *1 segundo por ejecución*,  
- Se ejecuta *100,000 veces al mes*,  

Solo pagarías unos pocos dólares mensuales.

## Casos de uso comunes

Las funciones Lambda son muy útiles para automatizar tareas pequeñas, responder a eventos o conectar servicios.

### Procesamiento de archivos
Ejemplo:  
Cuando se sube una imagen a *Amazon S3*, una función Lambda se activa automáticamente para:
- Redimensionar la imagen.  
- Cambiar el formato.  
- Guardarla en otro bucket optimizado.

### Integraciones con servicios de terceros
Ejemplo:  
Cuando un usuario se registra en tu aplicación, Lambda puede:
- Enviar sus datos a una API externa.  
- Crear un registro en una base de datos.  
- Disparar una notificación en Slack.

### Streaming de datos
Ejemplo:  
Procesar información en tiempo real desde *Amazon Kinesis* o *DynamoDB Streams*.  
Ideal para analítica en vivo, dashboards o monitoreo de sensores IoT.

## Limitaciones

- *Tiempo máximo de ejecución:* 15 minutos por invocación.  
- *Tamaño del archivo comprimido:* 50 MB (desde consola) o 250 MB (desde S3).  
- *Requiere funciones bien enfocadas:* cada Lambda debe resolver *una única tarea específica*.  
- *Cold start:* si la función no se ejecuta con frecuencia, el contenedor puede tardar un poco más en inicializarse.


## Ejemplo práctico

### Escenario: procesar imágenes subidas a S3

1. El usuario sube una imagen a un bucket S3.  
2. Ese evento activa una *función Lambda* llamada resizeImage().  
3. La función descarga la imagen, la redimensiona y la guarda en otro bucket.  
4. AWS crea un contenedor para esta tarea, ejecuta el código y luego destruye el contenedor.

*Código básico (Python):*

```python
import boto3
from PIL import Image
import io

def lambda_handler(event, context):
    s3 = boto3.client('s3')
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']

    # Descargar imagen
    image_obj = s3.get_object(Bucket=bucket, Key=key)
    image = Image.open(image_obj['Body'])

    # Redimensionar
    image = image.resize((300, 300))
    buffer = io.BytesIO()
    image.save(buffer, 'JPEG')
    buffer.seek(0)

    # Subir imagen redimensionada
    s3.put_object(Bucket='imagenes-optim', Key=key, Body=buffer)
    return {'statusCode': 200, 'body': 'Imagen redimensionada correctamente'}
