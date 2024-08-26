# python_api

RabbitMQ y MongoDB con Manejo de Errores y Seguridad


Esta aplicación demuestra cómo integrar RabbitMQ y MongoDB en una aplicación Python, con manejo de errores y configuraciones de seguridad.



Requisitos
Python 3.x
RabbitMQ
MongoDB
Instalación
Clonar el Repositorio

git clone https://github.com/tu-usuario/tu-repositorio.git
cd tu-repositorio
Instalar Dependencias

pip install pika pymongo
Configuración de Seguridad
MongoDB
Crear un Usuario en MongoDB

Abre la consola de MongoDB y ejecuta los siguientes comandos:

use admin
db.createUser({
  user: "myUserAdmin",
  pwd: "abc123",
  roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
})
Habilitar Autenticación

Edita el archivo de configuración de MongoDB (mongod.conf) para habilitar la autenticación:

security:
  authorization: "enabled"
Reinicia MongoDB

sudo systemctl restart mongod
RabbitMQ
Crear un Usuario en RabbitMQ

Abre la consola de RabbitMQ y ejecuta los siguientes comandos:

rabbitmqctl add_user myuser mypassword
rabbitmqctl set_user_tags myuser administrator
rabbitmqctl set_permissions -p / myuser ".*" ".*" ".*"
Uso
Productor
El productor envía mensajes a RabbitMQ. Aquí está el código con manejo de errores y autenticación:

import pika
import json

def send_message(message):
    try:
        credentials = pika.PlainCredentials('myuser', 'mypassword')
        connection = pika.BlockingConnection(pika.ConnectionParameters('localhost', 5672, '/', credentials))
        channel = connection.channel()

        channel.queue_declare(queue='my_queue')

        channel.basic_publish(exchange='',
                              routing_key='my_queue',
                              body=json.dumps(message))
        print(f" [x] Sent {message}")
    except pika.exceptions.AMQPConnectionError as e:
        print(f"Error connecting to RabbitMQ: {e}")
    except Exception as e:
        print(f"An error occurred: {e}")
    finally:
        if 'connection' in locals() and connection.is_open:
            connection.close()

# Ejemplo de uso
message = {"name": "John Doe", "email": "john.doe@example.com"}
send_message(message)
Consumidor
El consumidor recibe mensajes de RabbitMQ y los guarda en MongoDB. Aquí está el código con manejo de errores y autenticación:

python
import pika
import json
from pymongo import MongoClient, errors

def get_database():
    try:
        client = MongoClient('mongodb://myUserAdmin:abc123@localhost:27017/', serverSelectionTimeoutMS=5000)
        client.server_info()  # Trigger exception if cannot connect to db
        return client['my_database']
    except errors.ServerSelectionTimeoutError as err:
        print(f"Error connecting to MongoDB: {err}")
        return None

db = get_database()

def callback(ch, method, properties, body):
    try:
        message = json.loads(body)
        print(f" [x] Received {message}")
        if db:
            db.my_collection.insert_one(message)
            print(" [x] Message saved to MongoDB")
        else:
            print(" [x] Could not save message to MongoDB")
    except json.JSONDecodeError as e:
        print(f"Error decoding JSON: {e}")
    except Exception as e:
        print(f"An error occurred: {e}")

def consume_messages():
    try:
        credentials = pika.PlainCredentials('myuser', 'mypassword')
        connection = pika.BlockingConnection(pika.ConnectionParameters('localhost', 5672, '/', credentials))
        channel = connection.channel()

        channel.queue_declare(queue='my_queue')

        channel.basic_consume(queue='my_queue',
                              on_message_callback=callback,
                              auto_ack=True)

        print(' [*] Waiting for messages. To exit press interrupt the kernel')
        channel.start_consuming()
    except pika.exceptions.AMQPConnectionError as e:
        print(f"Error connecting to RabbitMQ: {e}")
    except Exception as e:
        print(f"An error occurred: {e}")
    finally:
        if 'connection' in locals() and connection.is_open:
            connection.close()

# Ejemplo de uso
consume_messages()
Verificación
Para verificar que los mensajes se han almacenado correctamente en MongoDB, puedes usar la consola de MongoDB o una herramienta como MongoDB Compass.

Usando la Consola de MongoDB
Abre una nueva terminal y ejecuta el shell de MongoDB:

bash
mongo
Conéctate a la base de datos que estás utilizando (my_database en este caso):

bash
use my_database
Consulta la colección para ver los documentos almacenados:

bash
db.my_collection.find().pretty()
Notas Adicionales
Interrumpir el Consumidor: Cuando quieras detener el consumidor, puedes interrumpir el kernel en Jupyter Notebook (Kernel -> Interrupt).
Manejo de Errores: El código incluye manejo de errores para conexiones a RabbitMQ y MongoDB, así como para la decodificación de JSON.
Seguridad: La aplicación está configurada para usar autenticación tanto en MongoDB como en RabbitMQ.
Contribuciones
Las contribuciones son bienvenidas. Por favor, abre un issue o un pull request para discutir cualquier cambio que te gustaría hacer.

Licencia
Este proyecto está licenciado bajo la Licencia MIT. Consulta el archivo LICENSE para más detalles.
