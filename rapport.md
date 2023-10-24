# Questions of to the practice

- Why should we run the container with a flag -e to give the environment variables?

Environnements variables often contains critical informations that are not meant to be written in readable files.
For our example, if I published my code on github, my database credentials would be easily readable and malicious
people could connect to it and use my data for unauthorized purposes.
The -e command is a way to give db's credentials outside the Dockerfile.

- Why do we need a volume to be attached to our postgres container?

Docker containers are meant to be expandable. If we had to save data before killing our containers each time we update
it, we would all go insane. The use of volumes helps us making this process automatic !

**1-1 Document your database container essentials: commands and Dockerfile.**

Our app is currently composed of two containers instanciated by two images each created with a Dockerfile.
The database Dockerfile is located on "db" folder and the Adminer in "adminer" one. These two containers are
connected by the network "tp-app-network".

First of all create the network :
```bash
$ docker network create tp-app-network

```

To run the db, please use these command lines :
```bash
$ docker build -t tp-db db/.  #build the image "tp-db" using the Dockerfile located in db/.
$ docker run -d -p 55432:5432 -v /sql:/var/lib/postgresql/data --net=tp-app-network --name=tp-db tp-db 

```

This last line run in detached mode (-d) the container tp-db (--name) on the port 5432 mapped on the 55432 of your machine
(you can change 55432 for another port if it's already used). It copies the local files "db/sql/." inside the container's
filesystem. It protects data from being wiped out during container rebuilding. It connects the container to the 
"tp-app-network" network.

To run adminer it's nearly the same commands :
```bash
$ docker build -t tp-adminer adminer/.
$ docker run -d -p 88080:8080 --net=tp-app-network --name=tp-adminer tp-adminer

```


You can now connect to your app on ip : http://localhost:88080
Log with your credentials.

**1-2 Why do we need a multistage build? And explain each step of this dockerfile.**

Multi-staged build is a way to keep in cache files that we don't need to pull again when they hasn't change.
The goal is to fasten the build process when a new dependency is added.

This Dockerfile has two stages:

Build Stage:

It uses the Maven-based image to build a Java application.
Copies project files, including the pom.xml and source code.
Runs Maven to package the application (mvn package) while skipping tests (-DskipTests).
Run Stage:

Uses an Amazon Corretto-based image as the runtime.
Copies the JAR files produced in the build stage to the myapp.jar file.
Sets the entry point to run the Java application with "java -jar myapp.jar."
This Dockerfile separates the build and run environments for your Java application.

- Why do we need a reverse proxy?

It ensures that the backend servers, which handle requests, are shielded from direct 
external access. From the client's perspective, the reverse proxy server appears as 
the sole source of all content, enhancing security and reliability.

- Why is docker-compose so important?

***docker compose*** helps one building and running lots pf containers at once. It more
suitable than having to build each image manually. The same way it's way more convenient 
to write one time the ports/networks/volumes etc than each time we have to create a 
container.

**1-3 Document docker-compose most important commands.**

The most important docker compose commands are :
   - docker compose up : create the containers, volumes and networks
     - useful arguments :
       - -d : run container in detached mode
       - --build : build images
   - docker compose down : stop the containers
     - useful arguments :
       - -v : remove the data volume uses
   - docker compose ps : list running containers with their ports, states and cmds

**1-4 Document your docker-compose file.**

This Docker Compose file creates three services on a network named "app-network."

***app:***
    - Container name: "tp-app"
    - Builds from the "project" directory using its Dockerfile.
    - Exposes port 8080 on the host.
    - Depends on the "db" service.
***db:***
    - Container name: "tp-db"
    - Builds from the "db" directory using its Dockerfile.
    - Maps port 5432 from the container to the host.
    - Mounts the host directory "/sql" to "/var/lib/postgresql/data" in the container for data persistence.

***httpd:***
    - Builds from the "project/apache" directory.
    - Exposes port 80 in the container but doesn't map it to the host.
    - Depends on the "app" service.

The services are all part of the "app-network" for communication. The "app-network" is a bridge network.

