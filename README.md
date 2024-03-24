# Java Application with MySQL Database Connection

## Overview
This project is an enhancement of a static Java application with the integration of MySQL database connectivity. The improvements allow users to edit information within the application and save the modified data to the MySQL database.

## Step 0: Clone Git repository and create your own
- Clone the Git repository containing the Java application.
- Ensure database credentials are stored as environment variables for security and flexibility.

 
```sh
# clone repository & change into project dir
git clone git@gitlab.com:twn-devops-bootcamp/latest/07-docker/docker-exercises.git
cd docker-exercises

# remove remote repo reference and create your own local repository
rm -rf .git
git init 
git add .
git commit -m "initial commit"

# create git repository on Gitlab and push your newly created local repository to it
git remote add origin git@gitlab.com:{gitlab-user}/{gitlab-repo}.git
git push -u origin master

# you can find the environment variables defined in src/main/java/com/example/DatabaseConfig.java file

```
<img src="https://i.imgur.com/P9B13Wd.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

*****

## Step 1: Start Mysql container
- Start a MySQL container locally using the official Docker image.
- Set necessary environment variables for database connection.
- Build the Java application JAR file and start the application.
- Test the application by making changes and accessing it from a browser.


```sh
# start mysql container using docker
docker run -p 3306:3306 \
--name mysql \
-e MYSQL_ROOT_PASSWORD=rootpass \
-e MYSQL_DATABASE=team-member-projects \
-e MYSQL_USER=admin \
-e MYSQL_PASSWORD=adminpass \
-d mysql mysqld --default-authentication-plugin=mysql_native_password

<img src="https://i.imgur.com/CixuJqT.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

# create java jar file
gradle build

<img src="https://i.imgur.com/cTsc9Pu.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>


# set env vars in Terminal for the java application (these will read in DatabaseConfig.java)
export DB_USER=admin
export DB_PWD=adminpass
export DB_SERVER=localhost
export DB_NAME=team-member-projects

<img src="https://i.imgur.com/CIhA6fE.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>


# start java application
java -jar build/libs/docker-exercises-project-1.0-SNAPSHOT.jar

```
<img src="https://i.imgur.com/ai9fJaD.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

*****

## Step 2: Start Mysql GUI container
- Start a phpMyAdmin container to visualize the MySQL database.
- Access phpMyAdmin from the browser and verify connectivity to the MySQL database.

```sh
# start phpmyadmin container using the official image
docker run -p 8083:80 \
--name phpmyadmin \
--link mysql:db \
-d phpmyadmin/phpmyadmin

# access it in the browser on
localhost:8083

# login to phpmyadmin UI with either of 2 mysql user credentials:
* admin:adminpass
* root:rootpass

```
<img src="https://i.imgur.com/K0xCaLP.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<img src="https://i.imgur.com/kVhXFb0.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>


*****

## Step 3: Use docker-compose for Mysql and Phpmyadmin
- Create a docker-compose file to manage both MySQL and phpMyAdmin containers.
- Configure a volume for the MySQL database to persist data.
- Test the setup to ensure both containers function properly.

**docker-compose.yaml**
```sh
version: '3'
services:
  mysql:
    image: mysql
    ports:
      - 3306:3306
    environment:
      - MYSQL_ROOT_PASSWORD=rootpass
      - MYSQL_DATABASE=team-member-projects
      - MYSQL_USER=admin    
      - MYSQL_PASSWORD=adminpass
    volumes:
    - mysql-data:/var/lib/mysql
    container_name: mysql
    command: --default-authentication-plugin=mysql_native_password
  phpmyadmin:
    image: phpmyadmin
    environment:
      - PMA_HOST=mysql
    ports:
      - 8083:80
    container_name: phpmyadmin
volumes:
  mysql-data:
    driver: local

```

<img src="https://i.imgur.com/ojXdiDj.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>


**Start containers with docker-compose**
```sh
docker-compose -f docker-compose.yaml up    
```
<img src="https://i.imgur.com/jlCDNSp.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

*****

## Step 4: Dockerize your Java Application
- Create a Dockerfile for the Java application to package it into a Docker container.


**Dockerfile**
```sh
FROM openjdk:17.0.2-jdk
EXPOSE 8080
RUN mkdir /opt/app
COPY build/libs/docker-exercises-project-1.0-SNAPSHOT.jar /opt/app
WORKDIR /opt/app
CMD ["java", "-jar", "docker-exercises-project-1.0-SNAPSHOT.jar"]

```
<img src="https://i.imgur.com/RHFkpkF.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

*****

## Step 5: Build and push Java Application Docker Image
- Set up a Docker hosted repository on Nexus.
- Build the Java application image locally and push it to the repository.

```sh
# create jar file - docker-exercises-project-1.0-SNAPSHOT.jar
gradle build

# create docker image - {repo-name}/{image-name}:{image-tag}
docker build -t {repo-name}/java-app:1.0-SNAPSHOT .

# push docker to remote docker repo {repo-name}
docker push {repo-name}/java-app:1.0-SNAPSHOT

```
<img src="https://i.imgur.com/HhxAGU4.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<img src="https://i.imgur.com/qun3Osp.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<img src="https://i.imgur.com/efSKTk8.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>


****

## Step 6: Add application to docker-compose

Update the docker-compose file to include your Java application's Docker image. Configure all necessary environment variables. Ensure that the application and MySQL containers use environment variables for configuration.

**docker-compose-with-app.yaml**

```sh
version: '3'
services:
  my-java-app:
    image: java-mysql-app:1.0 # specify the full image name with repository name
    environment:
      - DB_USER=${DB_USER}
      - DB_PWD=${DB_PWD}
      - DB_SERVER=${DB_SERVER}
      - DB_NAME=${DB_NAME}
    ports:
    - 8080:8080
    container_name: my-java-app
    depends_on:
      - mysql
  mysql:
    image: mysql
    ports:
      - 3306:3306
    environment:
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - MYSQL_DATABASE=${DB_NAME}
      - MYSQL_USER=${DB_USER}
      - MYSQL_PASSWORD=${DB_PWD}
    volumes:
    - mysql-data:/var/lib/mysql
    container_name: mysql
    command: --default-authentication-plugin=mysql_native_password
  phpmyadmin:
    image: phpmyadmin
    ports:
      - 8083:80
    environment:
      - PMA_HOST=${PMA_HOST}
      - PMA_PORT=${PMA_PORT}
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
    container_name: phpmyadmin
    depends_on:
      - mysql
volumes:
  mysql-data:
    driver: local

```

<img src="https://i.imgur.com/ezBnFLj.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

**steps:**
```sh
# set all needed environment variables
export DB_USER=admin
export DB_PWD=adminpass
export DB_SERVER=mysql
export DB_NAME=team-member-projects

export MYSQL_ROOT_PASSWORD=rootpass

export PMA_HOST=mysql
export PMA_PORT=3306

# start all 3 containers 
docker-compose -f docker-compose-with-app.yaml up    
```
<img src="https://i.imgur.com/tex9DwD.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<img src="https://i.imgur.com/vqGEDpe.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>


*****

## Step 7: Run application on server with docker-compose

Finally your docker-compose file is completed and you want to run your application on the server with docker-compose. For that you need to do the following:

- Set insecure docker repository on server, because Nexus uses http
- Run docker login on the server to be allowed to pull the image
- Your application index.html has a hardcoded localhost as a HOST to send requests to the backend. You need to fix that and set the server IP address instead, because the server is going to be the host when you deploy the application on a remote server. (Don't forget to rebuild and push the image and if needed adjust the docker-compose file)
- Copy docker-compose.yaml to the server
- Set the needed environment variables for all containers in docker-compose
- Run docker-compose to start all 3 containers



**Dockerfile**

```sh
# on Linux server - to add an insecure docker registry, add the file /etc/docker/daemon.json with the following content
{
  "insecure-registries" : [ "{repo-address}:{repo-port}" ]
}

# restart docker for the configuration to take affect
sudo service docker restart

# check the insecure repository was added - last section "Insecure Registries:"
docker info

# do docker login to repo
docker login {repo-address}:{repo-port}

# change hardcoded HOST env var in src/main/resources/static/index.html file, line 48
const HOST = "{server-ip-address}";

# rebuild the application and image and push to repo
gradle build
docker build -t {repo-name}/java-app:1.0-SNAPSHOT .
docker push {repo-name}/java-app:1.0-SNAPSHOT 

# copy docker-compose file to remote server
scp -i ~/.ssh/id_rsa docker-compose.yaml {server-user}:{server-ip}:/home/{server-user}

# ssh into the remote server
# set all env vars as you did in exercise 6
# run docker compose file
# open port 8080 on server to access java application
```

<img src="https://i.imgur.com/2VrcmOv.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>


## Step 8: Open ports

Open the necessary ports on the server's firewall to allow access to the application from the browser. Test access from the browser to ensure everything is working as expected.
