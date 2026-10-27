-- crate a amazon-linux ec2 instance and connect to the instance

-- create a rds  mysql database for this project or you can use container directly

-- install the git,docker and docker compose by follwing commands.run the following commands individually
```sh
sudo yum install git -y
sudo yum install docker -y #linux 2023
sudo usermod -aG docker ec2-user
newgrp docker
sudo service docker start
sudo chmod 777 /var/run/docker.sock
docker version
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose version
```
- bildx install process
```
mkdir -p ~/.docker/cli-plugins

curl -L https://github.com/docker/buildx/releases/download/v0.33.0/buildx-v0.33.0.linux-amd64 \
  -o ~/.docker/cli-plugins/docker-buildx

chmod +x ~/.docker/cli-plugins/docker-buildx
docker buildx version

sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
docker-compose version

```
-- after above process clone the repo by follwing command
```sh   
git clone https://github.com/CloudTechDevOps/2nd10WeeksofCloudOps-main.git
```  
-- switch to repo 
```sh    
cd 2nd10WeeksofCloudOps-main
```  

-- update the rds detailes in docker-compose.yaml if you are using rds only other wise just go through current yaml

-- after completeing the above process just run docker compose file by following command
```sh    
docker-compose up -d
```   
-- for above command will deploy the frontend,backend containers

-- next install the mariadb or mysql on your servr for database inilize only for rds process
```sh    
sudo yum install mariadb105-server -y
```    
--after that chaange into you project backend dirictroy then run the follwing command  only of rds 
```sh    
mysql -h <rds-end-point> -u admin -p<passowrd> < test.sql
mysql -h book-rds.c1u4kewc6r37.ap-south-1.rds.amazonaws.com -u admin -pveeranarni < test.sql   ## example command chamge the values
```

-- then search the public-ip:80 port in broswer you will get the appliaction deployment then add the book
    --------------------------THANKYOU------------------------------------------------------------









    ///Alternative Megha////////

    # Deploy a Dockerized Application on Amazon Linux EC2

## Step 1: Create an EC2 Instance

- Create an **Amazon Linux 2023 EC2 instance** in AWS.
- Configure the security group to allow:
  - **SSH (22):** Only your IP address.
  - **HTTP (80):** For accessing the frontend.
  - **Backend port:** Only if required by your application.
- Connect to the EC2 instance using SSH or the AWS EC2 Instance Connect feature.

## Step 2: Create the MySQL Database

You can use either of the following approaches:

**Option 1: Amazon RDS MySQL**

- Create an RDS MySQL database.
- Note the database endpoint, port, database name, username, and password.
- Configure security groups so the EC2 instance can connect to RDS on port `3306`.
- Keep the RDS database private whenever possible.

**Option 2: MySQL Container**

- Use the MySQL service already defined in your `docker-compose.yaml`.
- Docker Compose will create and run the MySQL container along with the frontend and backend.
- Ensure that a persistent Docker volume is configured for database storage.

> Choose only one database approach. If you use RDS, update the Compose configuration so the backend connects to RDS instead of the MySQL container.

## Step 3: Install Git, Docker, and Docker Compose

Run the following commands **individually** on your EC2 instance.

### 3.1 Install Git

```bash
sudo dnf install git -y
git --version
```

### 3.2 Install Docker

```bash
sudo dnf install docker -y
```

Start Docker and enable it to start automatically after reboot:

```bash
sudo systemctl enable --now docker
```

Add the EC2 user to the Docker group:

```bash
sudo usermod -aG docker ec2-user
```

Apply the group membership in your current session:

```bash
newgrp docker
```

Verify Docker:

```bash
docker version
docker ps
```

If Docker permission errors persist, reconnect to the EC2 instance and try again. Avoid using `chmod 777 /var/run/docker.sock`, because it grants unnecessarily broad access to the Docker daemon.

### 3.3 Install Docker Buildx

Create the Docker CLI plugins directory:

```bash
mkdir -p ~/.docker/cli-plugins
```

Download Buildx for the current Linux architecture:

```bash
curl -SL https://github.com/docker/buildx/releases/latest/download/buildx-$(uname -s | tr '[:upper:]' '[:lower:]')-$(uname -m) \
  -o ~/.docker/cli-plugins/docker-buildx
```

Make the file executable:

```bash
chmod +x ~/.docker/cli-plugins/docker-buildx
```

Verify the installation:

```bash
docker buildx version
```

or

ARCH=$(uname -m)

if [ "$ARCH" = "x86_64" ]; then
  BUILDX_ARCH="amd64"
elif [ "$ARCH" = "aarch64" ]; then
  BUILDX_ARCH="arm64"
else
  echo "Unsupported architecture: $ARCH"
  exit 1
fi

curl -fSL "https://github.com/docker/buildx/releases/latest/download/buildx-v0.33.0.linux-${BUILDX_ARCH}" \
  -o /root/.docker/cli-plugins/docker-buildx

### 3.4 Install Docker Compose

Install the standalone Docker Compose binary:

```bash
sudo curl -SL "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" \
  -o /usr/local/bin/docker-compose
```

Make it executable:

```bash
sudo chmod +x /usr/local/bin/docker-compose
```

Verify the installation:

```bash
docker-compose version
```

**Note:** The standalone `docker-compose` command is supported, but the modern Docker Compose plugin uses `docker compose` with a space. Use the command supported by your installation.

## Step 4: Clone the Git Repository

Clone the project repository:

```bash
git clone https://github.com/CloudTechDevOps/2nd10WeeksofCloudOps-main.git
```

Move into the project directory:

```bash
cd 2nd10WeeksofCloudOps-main
```

Verify the project files:

```bash
ls -la
```

Locate the `docker-compose.yaml` or `docker-compose.yml` file before continuing.

## Step 5: Configure the Database

### Option A: If You Are Using RDS MySQL

Open the Compose file:

```bash
vi docker-compose.yaml
```

Update the backend's database environment variables to match the application's expected variable names.

Example:

```yaml
environment:
  DB_HOST: your-rds-endpoint.amazonaws.com
  DB_PORT: "3306"
  DB_NAME: booksapp
  DB_USER: admin
  DB_PASSWORD: ${DB_PASSWORD}
```

Replace the example endpoint and database details with your actual values.

**Important:**
- Confirm the variable names expected by the backend source code. The application may use `DB_HOST`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`, or different names.
- Store database passwords in an environment file or secret manager instead of committing them to Git.
- Ensure the RDS security group permits MySQL connections from the EC2 instance's security group.
- Do not expose the RDS database publicly unless there is a specific, justified requirement.

### Option B: If You Are Using the MySQL Container

Review the existing Compose file and check that:

- The MySQL service has the required database name and credentials.
- The backend uses the MySQL **service name** as `DB_HOST`, for example, `mysql`.
- The backend and MySQL services share a Docker network.
- A persistent volume is mounted at the MySQL data directory.
- Database initialization scripts are configured correctly.

If the existing Compose file already contains the required settings, no changes may be necessary.

## Step 6: Start the Application Using Docker Compose

Run the command from the directory containing the Compose file.

For the standalone Compose installation:

```bash
docker-compose up -d --build
```

Or, if the modern Compose plugin is installed:

```bash
docker compose up -d --build
```

This command builds the configured images and starts the services defined in the Compose file, such as the frontend, backend, and MySQL container if configured.

Check the running containers:

```bash
docker ps
```

Check the service status:

```bash
docker-compose ps
```

View application logs:

```bash
docker-compose logs -f
```

To inspect a particular service:

```bash
docker-compose logs -f backend
```

Replace `backend` with the actual service name in your Compose file.

## Step 7: Initialize the Database — RDS Only

**Skip this step if you are using the MySQL container and its database initialization is already configured.**

Install a MySQL-compatible client on Amazon Linux 2023:

```bash
sudo dnf install mariadb105 -y
```

Check the client:

```bash
mysql --version
```

Navigate to the directory containing `test.sql`, or use its full path:

```bash
cd /path/to/project/backend
```

Connect to RDS and import the SQL file:

```bash
mysql -h <RDS_ENDPOINT> -P 3306 -u <DB_USERNAME> -p <DB_NAME> < test.sql
```

Replace the placeholders with your actual values. The `-p` option prompts for the password securely.

Example format:

```bash
mysql -h your-rds-endpoint.amazonaws.com -P 3306 -u admin -p booksapp < test.sql
```

Before importing, confirm that:
- `test.sql` exists at the specified path.
- The database named `booksapp` already exists, or create it first.
- The database user has permission to create or modify the required tables.
- The EC2 instance can reach the RDS endpoint on port `3306`.

## Step 8: Verify the Deployment

Check that the containers are running:

```bash
docker ps
```

If a service is not running, inspect its logs:

```bash
docker-compose logs --tail=100
```

Open a browser and navigate to:

```text
http://<EC2_PUBLIC_IP>
```

Replace `<EC2_PUBLIC_IP>` with your EC2 instance's public IPv4 address.

If the application loads successfully, test the main features, including adding a book and confirming that the database operation works.

If the frontend loads but API requests fail, verify the backend URL used by the frontend. A browser cannot resolve Docker-internal service names such as `backend`; the frontend must use an API address reachable from the browser, or an appropriate reverse-proxy configuration.

## Step 9: Useful Docker Commands

Check running containers:

```bash
docker ps
```

Check all containers, including stopped containers:

```bash
docker ps -a
```

View logs:

```bash
docker-compose logs -f
```

Restart the application:

```bash
docker-compose restart
```

Stop and remove the Compose containers and network:

```bash
docker-compose down
```

Rebuild and restart the application after code changes:

```bash
docker-compose up -d --build
```

> **Warning:** Avoid `docker-compose down -v` unless you intentionally want to delete the Compose-managed volumes. Removing the database volume can permanently delete the stored data if no backup exists.

---

# Deployment Completed — Thank You!



#
