Claro! Agregar manejo de errores y seguridad es crucial para hacer tu aplicación más robusta y segura. A continuación, te mostraré cómo puedes hacerlo.

Manejo de Errores
Primero, vamos a agregar manejo de errores tanto en el productor como en el consumidor.

Productor con Manejo de Errores
python
import pika
import json

def send_message(message):
    try:
        connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
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
Consumidor con Manejo de Errores
python
import pika
import json
from pymongo import MongoClient, errors

def get_database():
    try:
        client = MongoClient('mongodb://localhost:27017/', serverSelectionTimeoutMS=5000)
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
        connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
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
Seguridad
Para agregar seguridad, vamos a configurar autenticación para MongoDB y RabbitMQ.

Configuración de Seguridad para MongoDB
Crear un Usuario en MongoDB:

Abre la consola de MongoDB y ejecuta los siguientes comandos para crear un usuario:

bash
use admin
db.createUser({
  user: "myUserAdmin",
  pwd: "abc123",
  roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
})
Habilitar Autenticación:

Edita el archivo de configuración de MongoDB (mongod.conf) para habilitar la autenticación:

yaml
security:
  authorization: "enabled"
Reinicia MongoDB:

Reinicia el servicio de MongoDB para aplicar los cambios.

Actualizar la Conexión en el Código:

Actualiza la función get_database para incluir las credenciales de autenticación:

python
def get_database():
    try:
        client = MongoClient('mongodb://myUserAdmin:abc123@localhost:27017/', serverSelectionTimeoutMS=5000)
        client.server_info()  # Trigger exception if cannot connect to db
        return client['my_database']
    except errors.ServerSelectionTimeoutError as err:
        print(f"Error connecting to MongoDB: {err}")
        return None
Configuración de Seguridad para RabbitMQ
Crear un Usuario en RabbitMQ:

Abre la consola de RabbitMQ y ejecuta los siguientes comandos para crear un usuario:

bash
rabbitmqctl add_user myuser mypassword
rabbitmqctl set_user_tags myuser administrator
rabbitmqctl set_permissions -p / myuser ".*" ".*" ".*"
Actualizar la Conexión en el Código:

Actualiza las funciones send_message y consume_messages para incluir las credenciales de autenticación:

python
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
Resumen
Con estos cambios, has agregado manejo de errores y configurado autenticación para MongoDB y RabbitMQ. Esto hará que tu aplicación sea más robusta y segura. Asegúrate de probar estos cambios en tu entorno para verificar que todo funciona correctamente.