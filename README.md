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

# create java jar file
gradle build

# set env vars in Terminal for the java application (these will read in DatabaseConfig.java)
export DB_USER=admin
export DB_PWD=adminpass
export DB_SERVER=localhost
export DB_NAME=team-member-projects

# start java application
java -jar build/libs/docker-exercises-project-1.0-SNAPSHOT.jar

```

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

**Start containers with docker-compose**
```sh
docker-compose -f docker-compose.yaml up    
```

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
****

