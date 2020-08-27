# Fresh Drupal 8 with Docker Installation 

This repo contains Dockerfiles and a docker-compose.yml which can be used to 
spin up a Drupal 8 development environment in Docker containers. The Drupal 8
development environments consists of three interconnected containers: A MySQL
database container, a Drupal 8 container and a NGINX web server container.

# Table of Contents

* [Pre-Requisites](#pre-requisites)
* [Docker Installation](#docker-installation)
  * [Verifying Your Docker Installation](#verifying-your-docker-installation)
* [Configuration](#configuration)
  * [NGINX](#nginx)
  * [Drupal](#drupal)
  * [MySQL](#mysql)
* [Running the Fresh Drupal 8 with Docker Environment](#Running-the-fresh-drupal-8-with-docker-environment)
  * [Starting the Development Environment](#starting-the-development-environment)
  * [Initial Setup](#initial-setup)
  * [Interacting with the MySQL Container](#interacting-with-the-mysql-container)
  * [Stopping the Development Environment](#stopping-the-development-environment)
  * [Restarting the Stopped Environments Again](#Restarting-the-Stopped-Environments-Again)
  * [Re-execute a Fresh Environment](#re-execute-a-fresh-environment)


# Pre-Requisites

To utilize the Fresh Drupal 8 Docker environment, the following software
must be availble on the host where the the environment is to be deployed.

* Docker 

# Docker Installation   

Install Docker Desktop for Windows. See [this page](https://docs.docker.com/docker-for-windows/install/)

## Verifying Your Docker Installation

The Docker and docker-compose installation may be verified by running the 
following commands in a Windows PowerShell/command line terminal.

```bash
# Verify that Docker is working correctly.
$ docker run --rm hello-world:latest
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
1b930d010525: Pull complete
Digest: sha256:6540fc08ee6e6b7b63468dc3317e3303aae178cb8a45ed3123180328bcc1d20f
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

# Verify that docker-compose is installed and available.
$ docker-compose --version
docker-compose version 1.23.2, build 1110ad01
```

If these commands do not succeed, please review the Docker installation
instructions for your platform and try re-installing.

# Configuration

This section outlines the configuration for the three containers that
constitute the Drupal 8 Docker development environment.

## NGINX

The Drupal 8 Docker development environment utilizes an NGINX 1.17 container as 
its frontend web server.

The environment variables set in the NGINX container may be viewed in the 
[nginx.env](nginx/resources/nginx.env) file.

The NGINX server block for the Drupal application may be viewed in the
[default.conf](nginx/resources/default.conf) file.

## Drupal

The environment variables set in the Drupal container may be viewed in the 
[drupal.env](drupal/resources/drupal.env) file.

## MySQL

The Drupal 8 Docker development environment utilizes a MySQL 8 container as its
backend database.

The environment variables set in the MySQL container may be viewed in the 
[mysql.env](mysql/resources/mysql.env) file.

# Running the Fresh Drupal 8 with Docker Environment

`docker-compose.yml` will build out a Fresh Drupal 8 environment with no existing site.
Use this docker-compose.yml when you want to create a Drupal 8 development environment with no existing data.

## Starting the Development Environment

```bash
# Execute in the top level directory where docker-compose.yml is.
$ docker-compose up -d --build

Or it can split into two commends

$ docker-compose build
$ docker-compose up
```

This command will spin up a MySQL 8 database container, a Drupal 8 container and  
an NGINX web server frontend container.

Once completed, Drupal may be accessed at `http://localhost:8080`.

## Initial Setup

Once after the steps are executed, there are a few initial setup steps the must be completed.

1. Once the Drupal container is up, navigate to `http://localhost:8080/index.php`
   Select the "Standard" installation profile on the "Choose Profile" tab.
2. On the "Set up database" tab, select "MySQL, MariaDB, Percona Server, or equivalent"
   as the database type. For the fields, use the values in [mysql.env](mysql/resources/mysql.env).
   The database name should be the value of  `MYSQL_DATABASE`, the database username
   should be the value of `MYSQL_USER` and  the database password should be the
   value of `MYSQL_PASSWORD`. Under advanced options, set the "Host" option to `mysql`
   to match the name of the MySQL service.
3. Enter whatever values you wish on the "Configure site" tab.
4. Once the Drupal installation is complete, you are now ready to use the Drupal environment.

## Interacting with the MySQL Container

MySQL container is exposed for local connections. 

### Connect to the MySQL container via the `mysql` client.

You may connect to the MySQL container via a local client on the 
same host as your development environment using the host `127.0.0.1` and the 
port `3306`.

```bash 
$ mysql -h 127.0.0.1 -P 3306 -u drupaluser --password=drupalpassword
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 90
Server version: 8.0.16 MySQL Community Server - GPL

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| drupaldb           |
| information_schema |
+--------------------+
2 rows in set (0.00 sec)
```

### Connect to MySQL via the `mysql` client within the MySQL container.

It is also possible to connect to the database using the MySQL client within
the MySQL container if no local client is available.
```bash
# Get the ID of the MySQL container.
$ docker ps -qf "ancestor=fresh-drupal-8-docker_mysql"
ab50798d19f6

# Or get the container(which list all the containers)
$ docker ps -a

# Start the MySQL client within the container.
$ docker exec -it ab50798d19f6 mysql --user=drupaluser --password=drupalpassword
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 91
Server version: 8.0.16 MySQL Community Server - GPL

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| drupaldb           |
| information_schema |
+--------------------+
2 rows in set (0.01 sec)
```

### Dump a database within the MySQL container.

The MySQL container also contains the `mysqldump` binary, which may be used to 
easily dump databases within the MySQL container.

```bash
# Get the ID of the MySQL container.
$ docker ps -qf "ancestor=fresh-drupal-8-docker_mysql"
ab50798d19f6

# Or get the container(which list all the containers)
$ docker ps -a

# Dump the database to the local host file drupaldb_dump.sql.
$ docker exec -i ab50798d19f6 /usr/bin/mysqldump --user=drupaluser --password=drupalpassword drupaldb > drupaldb_dump.sql

# Dump the database to the local host file drupaldb_dump.sql and print to stdout.
$ docker exec -i ab50798d19f6 /usr/bin/mysqldump --user=drupaluser --password=drupalpassword drupaldb | tee drupaldb_dump.sql
```

## Stopping the Development Environments

The containers may be stopped via `docker-compose stop`.

```bash
# Stopping the containers.
$ docker-compose stop

```

## Restarting the Stopped Environments Again

Edits made to the Drupal site and changes made to the MySQL database will be 
persisted across restarts of the Drupal 8 development containers.
```bash
## Restart containers
$ docker-compose up -d --build

# Or it can split into two commends

$ docker-compose build
$ docker-compose up
```

## Re-execute a Fresh Environment

If you wish to restart a fresh Drupal 8 Docker development environment, execute
the following commands to remove the persistent volumes for the containers and 
restart them with new volumes. Be sure to copy data you wish to save before
removing the persistent volumes.

```bash 
# Standard containers
# Stop all containers and delete persistent volumes.
$ docker-compose down -v

# Restart containers with fresh volumes
$ docker-compose up -d --build

# Or it can split into two commends

$ docker-compose build
$ docker-compose up

```
