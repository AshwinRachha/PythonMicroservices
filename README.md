# PythonMicroservices

## API Gateway

service.py

This code defines an API for uploading and downloading audio files as MP3. It uses Flask, PyMongo and GridFS to interact with a MongoDB database.

The routes define the following functionalities:

/login: receives a POST request with JSON credentials and returns a token if they are valid.
/upload: receives a POST request with an audio file and uploads it to MongoDB as a video first using util.upload() function, then extracts audio and stores it on MongoDB as an MP3, finally sends a message to RabbitMQ to further process the audio.
/download: receives a GET request with a file ID (fid) as a query parameter, retrieves the file from GridFS, and returns it in the response after setting the proper headers for the MP3 to prompt a download.
Also, creates a connection to RabbitMQ to listen to messages sent to the MP3_QUEUE as defined in start() method implemented by util.py module.

The authentication is done by the validate.token() function, which uses the JWT tokens provided in the header of the HTTP requests.

The server runs on port 8080 and listens to all network interfaces (host="0.0.0.0").

### Auth Service

This code is a python application that allows a user to log in and then generates JSON Web Tokens (JWT) that can be used to authorize their access to specific endpoints.

Libraries Imported
The first line of the code imports several libraries, such as jwt, datetime, and os.

Flask Server Configuration
After that, we configure a server using Flask.

In the following lines, we have MySQL configuration, which gives the values for database details connecting to MySQL.

Copy
Insert
New

server.config["MYSQL_HOST"] = os.environ.get("MYSQL_HOST")
server.config["MYSQL_USER"] = os.environ.get("MYSQL_HOST")
server.config["MYSQL_PASSWORD"] = os.environ.get("MYSQL_HOST")
server.config["MYSQL_DB"] = os.environ.get("MYSQL_HOST")
server.config["MYSQL_PORT"] = os.environ.get("MYSQL_HOST")
We can see that environment variables are being used to set the various configuration parameters. This includes the host name, username, password, port and database name used in the connection string.

Endpoints
There are two different endpoints that are defined:

Login endpoint (/login) - It accepts POST requests with basic HTTP authorization header. If the email and password match with the data stored in the server, it will return a JWT token with user credentials, expiration date, and admin permission, i.e createJWT(). Otherwise, it will return "Invalid Credentials".

Validate endpoint(/validate) - It accepts POST requests with an Authorization header, containing a Bearer token generated during login. It checks whether the given bearer token is valid, i.e it's issued by the server and hasn't expired. If everything is ok, it returns OK. Otherwise, it returns "Not Authorized".

Finally, the app runs on the provided host and port.

if __name__ == "__main__": server.run(host="0.0.0.0", port = 5000)

Overall, this is a simple server application that demonstrates how authentication with JSON web tokens(JWT) works and how an endpoint could be authorized based on those tokens.

### Notification Service

notification.py

This code sets up a consumer for a RabbitMQ message queue to listen for messages, and when received it sends an email notification through the email module's notification function.

The pika module is used for setting up the connection to the RabbitMQ server. The send.email module is referenced to call the notification function which will actually send the email with the message body as the content.

The main function sets up a connection to the specified RabbitMQ server and Channel. The callback function is used to handle each received message. If there is any error sending the email, basic_nack is called to negatively acknowledge the message, whereas in other cases basic_ack is called to positively acknowledge the message.

Then channel.basic_consume() function consumes the messages from the queue and passes them to the specified callback function.

Finally, __name__ == "__main__": block calls the main function and starts consuming.

### Converter Service

The following code is a Python script that uses MongoDB's GridFS to store and retrieve videos and mp3s. It also uses RabbitMQ as a message broker to convert uploaded videos into mp3s.

This specific Python script listens to a queue named VIDEO_QUEUE in the RabbitMQ connection, which receives messages with information about videos that were uploaded. The script then converts the video into an mp3 format and saves it in the MP3S collection in the db_mp3s database.

The script first imports necessary libraries such as Pika for RabbitMQ integration, Sys and OS for Python-related system functions, and Time for processing. Additionally, the script relies on the MongoDB and GridFS drivers, importing these through MongoClient and gridfs.

After importing, the main() function establishes a MongoClient object client by passing the IP address of the MongoDB instance and credentials to access the instance (port number, IP address). Two connections are made to two separate databases, videos and mp3s using the already established client as a reference, and creating gridfs.GridFS objects for each one.

Then, the script creates a connection object and channel object utilizing Pika. After that, a callback function is defined that works as follows: when a message is received, the message body undergoes conversion from a video file into an mp3 file using the to_mp3.start() function imported previously. If the conversion is successful, the message will receive an acknowledgement or ch.basic_ack and be removed from the queue. Otherwise, the message remains within the queue without any acknowledgment support.

Finally, the script establishes a thread between the consuming JOB and listener in RabbitMQ via Broker, prints a message to confirm its readiness, and commences waiting for jobs until it closes.

