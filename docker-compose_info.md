Docker Compose YAML — Complete Explanation with Real-World Scenarios
This docker-compose.yaml file deploys a book management application containing three services:
1. Frontend — the user interface.
2. Backend — application logic and APIs.
3. MySQL — database for storing book information.
Docker Compose builds the images, creates the containers, connects them through a network, configures database credentials, and stores database files in a persistent volume.
1. Understand the architecture
User's browser

http://EC2-PUBLIC-IP:80

Host port 80 → container port 80

Docker network: appnet
Frontend

frontend-app

Container port 80


Backend

backend-app

Container port 80



DB_HOST=mysql · MySQL port 3306

MySQL container

mysql-db · database booksapp

Persistent volume: mysql_data



Real-world scenario: A user opens the website and adds a book. The frontend sends a request to the backend, and the backend writes the book details into MySQL. The database volume keeps the stored data available even if the database container is replaced.

2. version: "3.9"
version: "3.9"


- Historically, this specified the Docker Compose file format version.
- In modern Docker Compose, the version field is obsolete and is retained only for backward compatibility. Compose validates the file using its current schema.
Scenario: You are learning from an older tutorial that uses Compose version 3.9. The file can generally be used with modern Compose, but you do not need this line in a new Compose file.

You can omit it in modern Docker Compose.
3. services:
services:


A service defines an application component that Compose manages. Each service usually becomes one or more containers.

Your file has three services:
Service	Responsibility
frontend	Displays the website
backend	Handles API requests and business logic
mysql	Stores application data

Scenario: If you run docker compose up -d, Compose reads these definitions and starts the services and their dependencies.
4. Frontend service
A. Build configuration
frontend:
  build:
    context: ./client
    dockerfile: Dockerfile


- frontend is the Compose service name.
- build tells Compose to build an image locally.
- context: ./client specifies the directory containing the frontend build files.
- dockerfile: Dockerfile specifies the Dockerfile to use within that context.
Scenario: Your repository contains client/Dockerfile, which explains how to package the frontend into a Docker image. Compose builds that image before starting the frontend container.

Example project layout:
project/
├── docker-compose.yaml
├── client/
│   └── Dockerfile
├── backend/
│   └── Dockerfile
└── mysql/
    └── Dockerfile


This is the expected layout for your YAML; verify that these files actually exist.
B. Container name
container_name: frontend-app


Assigns the container the explicit name frontend-app.

Scenario: You want to identify the frontend easily when running:
docker ps
docker logs frontend-app
docker restart frontend-app


Without container_name, Compose generates a name automatically. Explicit names can be convenient, but they can also prevent scaling the service to multiple replicas under the same fixed name.
C. Port mapping
ports:
  - "80:80"


The format is:

HOST_PORT:CONTAINER_PORT
- First 80 — port on the EC2 host.
- Second 80 — port inside the frontend container.
Scenario: A user visits http://YOUR-EC2-PUBLIC-IP. The request reaches EC2 port 80 and is forwarded to port 80 inside the frontend container.

The frontend web server must actually listen on container port 80. The EC2 security group must also allow inbound port 80.
D. Network
networks:
  - appnet


Connects the frontend container to the Docker network named appnet.

Scenario: The frontend and backend need network connectivity to communicate. A shared Docker network lets containers resolve other services by their Compose service names.
E. Dependency
depends_on:
  - backend


Tells Compose to start the backend before starting the frontend.

Important: This does not guarantee the backend is ready to handle requests. It controls startup order, not application health, unless suitable health checks and conditions are configured.

Scenario: The frontend needs the backend API. Starting the backend first can help with startup ordering, but the frontend should still handle temporary API unavailability.

5. Backend service
The backend processes requests from the frontend and communicates with the database.
A. Build the backend image
backend:
  build:
    context: ./backend
    dockerfile: Dockerfile


- context: ./backend — uses the backend directory as the build context.
- dockerfile: Dockerfile — reads the backend's Dockerfile.
Scenario: The backend may contain Node.js code, API routes, authentication logic and database queries. The Dockerfile installs dependencies and packages that code into an image.
B. Container name
container_name: backend-app


Gives the backend container a fixed name.

Useful for checking its status and logs:
docker logs backend-app
docker restart backend-app


C. Port mapping
ports:
  - "84:80"


This maps EC2 host port 84 to backend container port 80.
Port	Purpose
84	Port exposed on the EC2 host
80	Port inside the backend container

Scenario: If you visit http://YOUR-EC2-PUBLIC-IP:84, the request can reach the backend directly, provided the EC2 security group allows port 84.

For a typical deployment, direct public access to the backend may not be necessary. If the frontend uses a reverse proxy, the backend port can instead be kept internal by removing the ports mapping. If the browser calls the backend API directly, you need an appropriate public endpoint and CORS configuration.
D. Environment variables
environment:
  DB_HOST: mysql
  DB_USER: admin
  DB_PASSWORD: Devops123
  PORT: 3306
  DB_NAME: booksapp
  APP_PORT: 80
  NODE_ENV: development
  JWT_SECRET: sdfghjkldfcgvhjnk
  JWT_EXPIRES_IN: 7d


Environment variables provide configuration to the backend without embedding every setting in the application source code.
Variable	Meaning and scenario
DB_HOST: mysql	Database hostname. Docker Compose DNS resolves mysql to the MySQL service on the shared network.
DB_USER: admin	Username the backend uses to connect to MySQL.
DB_PASSWORD	Password for that database user. It must match the configured MySQL user's password.
PORT: 3306	Potentially problematic. Port 3306 is the standard MySQL port, not the usual HTTP port for a Node.js backend. Check how the code uses this variable.
DB_NAME: booksapp	Database the backend should use.
APP_PORT: 80	Likely the application listening port, if the code reads APP_PORT.
NODE_ENV: development	Marks the Node.js runtime as development. Use production for a production deployment when the application supports it.
JWT_SECRET	Secret used to sign or verify JSON Web Tokens, if the backend uses JWT authentication.
JWT_EXPIRES_IN: 7d	Likely configures JWT expiry to seven days, depending on the authentication library.

Important: Check PORT: 3306
The meaning of an environment variable depends on the backend code. If the application uses PORT for its HTTP listener, setting it to 3306 is likely incorrect.

For example, a backend may contain:
const port = process.env.APP_PORT || 80;
app.listen(port);


In that case, APP_PORT: 80 controls the HTTP listener and PORT may be ignored. But if it uses process.env.PORT, then PORT: 3306 could cause the application to listen on the wrong port.

What to do: Inspect the backend code and its Dockerfile before changing anything. The backend's listening port must agree with the container port in the mapping "84:80" if the service is intended to listen on port 80.

Also, never use the example JWT secret or hardcoded database password in a public or production deployment. Use strong secrets supplied through an appropriate secrets mechanism.
E. Backend network
networks:
  - appnet


Connects the backend to the same network as the frontend and MySQL.

Scenario: The backend can connect to MySQL using mysql:3306 without needing the database container's IP address.
F. Wait for MySQL health
depends_on:
  mysql:
    condition: service_healthy


This tells Compose to wait for the MySQL service's health check to pass before starting the backend.

Scenario: MySQL may take several seconds to initialize its database files and users. Without a health condition, the backend might start too early and fail when it attempts its first database connection.

This condition works only when the MySQL service has a suitable health check configured, as it does in your YAML.

6. MySQL service
MySQL stores the application's book records, user information and other database content.
A. Build configuration
mysql:
  build:
    context: ./mysql
    dockerfile: Dockerfile


Compose builds the MySQL image using the Dockerfile inside ./mysql.

Scenario: The Dockerfile might customize the database image or load initialization scripts. Inspect it to determine what it actually does. A standard MySQL image can also be configured with image: mysql:<version> if you do not need a custom build.
B. Container name
container_name: mysql-db


Assigns the fixed container name mysql-db.

You can inspect its logs with:
docker logs mysql-db


The Compose service name remains mysql, which is the hostname used by the backend. The container name and service name serve different purposes.
C. Database environment variables
environment:
  MYSQL_ROOT_PASSWORD: Cloud122
  MYSQL_DATABASE: booksapp
  MYSQL_USER: admin
  MYSQL_PASSWORD: Devops123


These are commonly supported initialization variables for official MySQL images. Confirm the custom Dockerfile uses a compatible base image.
Variable	Purpose
MYSQL_ROOT_PASSWORD	Sets the MySQL root account password during initialization.
MYSQL_DATABASE	Creates the initial database named booksapp.
MYSQL_USER	Creates an additional database user named admin.
MYSQL_PASSWORD	Sets the password for the additional user.

Scenario: When MySQL initializes an empty data directory, it creates the database and user. The backend then connects using DB_HOST=mysql, DB_USER=admin, DB_PASSWORD and DB_NAME=booksapp.

The database credentials in the backend and MySQL service must match.

Note that initialization variables generally do not reset an already initialized database's users and passwords just because you edit the Compose file.
D. Publish the MySQL port
ports:
  - "3306:3306"


This maps EC2 port 3306 to container port 3306.

Scenario: A developer could connect from an authorized machine to the EC2 host's port 3306 if networking and security rules permit it.

However, your backend already connects to MySQL over the internal appnet network. Therefore, you usually do not need to publish MySQL's port at all for this architecture. You can remove the ports section for the database to avoid exposing it on the EC2 host.
E. Network
networks:
  - appnet


Attaches MySQL to the same Docker network as the backend.

Scenario: The backend uses the hostname mysql to reach the database over the Docker network, rather than going through the public IP or using localhost.
F. Persistent volume
volumes:
  - mysql_data:/var/lib/mysql


This mounts the named Docker volume mysql_data at MySQL's data directory, /var/lib/mysql.

Why is this important?

Containers can be deleted and recreated. Without persistent storage, database files stored only in a container's writable layer can be lost when that container is removed.

Scenario: You run:
docker compose down
docker compose up -d


The named volume normally remains, so MySQL can reuse its existing database data when the container starts again.

But this is not a backup. If the volume is explicitly deleted, or the underlying EC2 storage is lost, the data may be lost. For important data, use backups and a suitable recovery strategy.
7. MySQL health check
healthcheck:
  test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
  interval: 5s
  timeout: 5s
  retries: 10
  start_period: 10s


A health check lets Docker determine whether MySQL is responding to its health probe.
Setting	Explanation
test	Runs mysqladmin ping -h localhost inside the container to check MySQL availability.
interval: 5s	Checks every five seconds.
timeout: 5s	Allows up to five seconds for a check to finish.
retries: 10	Marks the container unhealthy after ten consecutive failed checks outside the start period.
start_period: 10s	Gives MySQL an initial startup grace period.

Real-world scenario: MySQL starts, but it takes 15 seconds to initialize. The backend should not attempt to start before the database is ready. The health check and depends_on: condition: service_healthy help coordinate that startup.

Note that a successful ping does not necessarily prove that every application query or database permission works correctly. The backend should also handle connection failures and retry when appropriate.
8. Docker network
networks:
  appnet:
    driver: bridge


- networks declares the networks Compose manages.
- appnet is the network's name within the Compose configuration.
- driver: bridge creates a private network for containers on the same Docker host.
Scenario: Your backend can reach the database at mysql:3306 because both containers belong to appnet. Docker's internal DNS resolves the service name mysql.

You do not need to manually discover or hardcode the MySQL container IP.
9. Docker volume
volumes:
  mysql_data:


This declares a named volume called mysql_data. It is used by the MySQL service's volume mapping:
volumes:
  - mysql_data:/var/lib/mysql


Scenario: The container is recreated during an update, but the database files remain in the named volume.

Useful commands:
docker volume ls
docker volume inspect mysql_data


Be careful with docker compose down -v: the -v flag removes the Compose-managed named volumes and can delete your database data.

10. How Docker Compose runs the complete project
When you run:
docker compose up -d --build


Compose builds the images and starts the services in dependency order. The --build flag asks Compose to build images before starting the containers.

The sequence is approximately:

1. Build images

Build frontend, backend and MySQL images from their Dockerfiles.




2. Start MySQL

Initialize the database if needed and wait for the health check.




3. Start backend

Connect to MySQL using the service hostname and database credentials.




4. Start frontend

Publish the website on EC2 port 80.



This sequence reflects your configured dependencies, but it does not guarantee that the backend or frontend is fully ready to serve requests when its container starts.
