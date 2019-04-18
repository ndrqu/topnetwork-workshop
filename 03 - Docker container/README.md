# Docker container

## Table of Contents

- [Docker container](#docker-container)
  - [Table of Contents](#table-of-contents)
    - [Run Hello World! Docker](#run-hello-world-docker)
    - [Running a DB in a Container](#running-a-db-in-a-container)
    - [WordPress with Docker](#wordpress-with-docker)

### Run Hello World! Docker

Open a bash terminal on your laptop.

You can check that the Docker installation was successful with the following test program:

```bash
docker run hello-world
```

You should see an output similar to the example below:

```bash
latest: Pulling from library/hello-world
1b930d010525: Pull complete 
Digest: sha256:92695bc579f31df7a63da6922075d0666e565ceccad16b59c3374d2cf4e8e50e
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the hello-world image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
 ```

 Docker should now be installed and working correctly. Continue on below with the rest of the WordPress setup.

### Running a DB in a Container

Before installing WordPress with Docker you will need to have somewhere to store the data. MariaDB is a community-developed relational database management system and a drop-in replacement for MySQL. It is [officially available on Docker Hub](https://hub.docker.com/_/mariadb/) and provides easy instructions with up to date images.

Start off by making a new directory where you wish to store the files for WordPress and MariaDB for example in your home directory.

```bash
mkdir ~/wordpress && cd ~/wordpress
```

Downloading and installing a new MariaDB container can all be performed with a single command. Before jumping in check the required parameters.

MariaDB Environment variables, these are marked in the Docker command with -e:

- -e MYSQL_ROOT_PASSWORD= Set your own password here. We suggest as a good practice to generate a password using [random.org](https://www.random.org) password generator;
- -e MYSQL_DATABASE= Creates and names a new database e.g. wordpress;

Docker parameters:

- –name wordpressdb – Names the container;
- -v “$PWD/database”:/var/lib/mysql – Creates a data directory linked to the container storage to ensure data persistence;
- -d – Tells Docker to run the container in daemon;
- mariadb:latest – Finally defines what to install and which version;

Then run the command below while replacing the <password> with your own.

```bash
docker run -e MYSQL_ROOT_PASSWORD=<password> -e MYSQL_DATABASE=wordpress --name wordpressdb -v "$PWD/database":/var/lib/mysql -d mariadb:latest
```

```bash
...
Status: Downloaded newer image for mariadb:latest
23df0ec2e48beb1fb8704ba612e9eb083f4193ecceb11102bc91232955cccc54
```

If Docker was successful at creating the container, you should see a code at the end of the output similar to the example above. You can confirm that the MariaDB container is running by using the following command:

```bash
docker ps
```

Check the status for your MariaDB install, it should show “Up” and the time it has been running like in the example output below.

```bash
CONTAINER ID IMAGE          COMMAND                CREATED        STATUS        PORTS      NAMES
14649c5b7e9a mariadb:latest "/docker-entrypoint.s" 12 seconds ago Up 12 seconds 3306/tcp   wordpressdb
```

Other useful commands for working with containers are ‘start’, ‘stop’ and ‘remove’.

```bash
docker start <container name>
docker stop <container name>
docker rm <container name>
```

You can find out more about available commands and options to specific commands.

```bash
docker --help
docker <command> --help
```

Full command-line documentation is also available over at [Docker support page](https://docs.docker.com/engine/reference/commandline/cli/).

### WordPress with Docker

Applications in containers run isolated from one another in the userspace of the host operating system sharing the kernel with other containers. This reduces the overhead required to run packaged software while also enabling the containers to run on any kind of infrastructure. To allow applications within different containers to work with one another Docker supports container linking.

WordPress is also made [officially available on Docker Hub](https://hub.docker.com/_/wordpress/), pull the image using with the command below. When the version to download is not specified Docker will fetch the latest available.

```bash
docker pull wordpress
```

WordPress container also takes environment variables and Docker parameters:

* -e WORDPRESS_DB_PASSWORD= Set the same database password here.
* –name wordpress – Gives the container a name.
* –link wordpressdb:mysql – Links the WordPress container with the MariaDB container so that the applications can interact.
* -p 80:80 – Tells Docker to pass connections from your server’s HTTP port to the containers internal port 80.
* -v “$PWD/html”:/var/www/html – Sets the WordPress files accessible from outside the container. The volume files will remain even if the container was removed.
* -d – Makes the container run on background
* wordpress – Tells Docker what to install. Uses the package downloaded earlier with the docker pull wordpress -command.

Run the command below while replacing the \<password\> as you did for the MariaDB container.

```bash
docker run -e WORDPRESS_DB_PASSWORD=<password> --name wordpress --link wordpressdb:mysql -p 80:80 -v "$PWD/html":/var/www/html -d wordpress
```

Then open your laptop’s IP address in a web browser to test the installation. You should be redirected to the initial WordPress setup page at http://\<your IP\>/wp-admin/install.php. 
Go through the setup wizard and you are done.
