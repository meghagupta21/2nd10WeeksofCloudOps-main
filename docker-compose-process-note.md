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

EC2 + Docker + MySQL/RDS

Complete Deployment Setup — Amazon Linux 2023

Goal: run the frontend and backend in Docker containers on EC2. Choose either Amazon RDS MySQL or a MySQL container for the database.

1. Create EC2 and connect

Launch an Amazon Linux 2023 EC2 instance and connect using EC2 Instance Connect or SSH.

Security group: allow SSH (22) from your IP and HTTP (80) from intended website users.

If using RDS, allow MySQL (3306) from the EC2 security group only—not from the public internet.

2. Install Git and Docker

sudo dnf install git -y
sudo dnf install docker -y

# Start Docker now and enable it after reboot
sudo systemctl enable --now docker

# Allow ec2-user to run Docker commands
sudo usermod -aG docker ec2-user
newgrp docker

# Verify
git --version
docker version
docker ps

Explanation: dnf installs packages; systemctl starts/enables Docker; usermod adds ec2-user to the Docker group; newgrp activates the group in the current shell; docker ps lists running containers.

Security: Do NOT run sudo chmod 777 /var/run/docker.sock. It grants broad access to Docker, which can effectively provide root-level control of the host. Docker group membership is also highly privileged; add only trusted users.

3. Install Docker Compose

Preferred: install the Compose plugin if available in the configured repositories.

sudo dnf install docker-compose-plugin -y
docker compose version

If the package is unavailable, install the standalone binary instead:

sudo curl -fL "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose version

Use docker compose for the plugin, or docker-compose for the standalone executable. Do not need to install both.

4. Install Docker Buildx (optional for many basic deployments)

mkdir -p ~/.docker/cli-plugins
curl -fL https://github.com/docker/buildx/releases/download/v0.33.0/buildx-v0.33.0.linux-amd64 -o ~/.docker/cli-plugins/docker-buildx
chmod +x ~/.docker/cli-plugins/docker-buildx
docker buildx version

Buildx provides advanced image-building features. The linux-amd64 binary is for x86-64; choose the matching ARM64 binary for an ARM EC2 instance.

5. Clone the GitHub repository

git clone https://github.com/CloudTechDevOps/2nd10WeeksofCloudOps-main.git
cd 2nd10WeeksofCloudOps-main
ls -la

git clone downloads the source; cd enters the repository; ls -la lists files. Check the actual repository structure and locate docker-compose.yaml.

6. Configure the database

Option A — Amazon RDS MySQL

Create an RDS MySQL database and record its endpoint, database name, username and password.

Update the environment variables expected by the backend in docker-compose.yaml or the project environment file. Variable names depend on the project.

Configure the RDS security group to accept port 3306 from the EC2 security group. Confirm routing and credentials.

Keep passwords out of source control; use an environment file with restricted permissions or a secrets manager.

Option B — MySQL container

Inspect docker-compose.yaml to see whether a database service is already defined.

Configure the backend to connect to the database service name (for example, db), not localhost, when they are separate containers on the Compose network.

Use persistent storage/volumes if database data must survive container replacement.

7. Start the application containers

Run from the directory containing docker-compose.yaml.

# Compose plugin
docker compose up -d

# OR, if using standalone Compose
docker-compose up -d

# Check containers and logs
docker ps
docker compose logs -f

up creates/starts the services; -d runs them in the background; docker ps lists running containers; logs -f streams logs. Replace docker compose with docker-compose if using the standalone binary. Check service names and logs if a container exits.

8. Install a MySQL-compatible client (RDS import only)

sudo dnf search mariadb
sudo dnf install mariadb105 -y
mysql --version

Package availability can vary by repository. If mariadb105 is unavailable, install a compatible client package. A client is enough to import SQL into RDS; a database server on EC2 is not required.

9. Import test.sql into RDS (RDS import only)

Go to the directory containing test.sql. Verify the file path before running the import.

cd backend
ls -la

mysql -h YOUR_RDS_ENDPOINT \
  -P 3306 \
  -u admin \
  -p YOUR_DATABASE < test.sql

Replace YOUR_RDS_ENDPOINT and YOUR_DATABASE with real values. -h is the endpoint, -P is the port, -u is the username, -p prompts for the password, and < test.sql sends the SQL statements to MySQL. Check whether the script expects an existing database before importing; inspect it first because it may create or modify tables/data.

10. Open and test the website

http://YOUR_EC2_PUBLIC_IP

# Troubleshooting
docker ps
docker compose logs --tail=100
curl -I http://localhost

The EC2 security group must allow port 80, and the Compose file must publish the frontend/reverse-proxy port to host port 80.

Confirm containers are healthy and the backend can reach the database.

Add a book in the UI and verify it persists after refreshing the page.
