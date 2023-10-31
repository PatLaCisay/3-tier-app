# Lucas BARREAU - Questions of to the practice

## Docker practice
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
$ docker run -d -p 8090:8080 --net=tp-app-network --name=tp-adminer tp-adminer

```


You can now connect to your app on ip : http://localhost:8090
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

The services are all part of the "app-network" for communication. The "app-network" is a bridge 
network.

**1-5 Document your publication commands and published images in dockerhub.**

The "tag" argument gives a tag to the local image. After using :
```bash
$ docker login
```
To log to your DockerHub account. After that the push command publish your image into DockerHub.
You can afterward change the version tag to update your image if you want.

As you can see in the docker-compose.yaml we commented the previous "context/build" and use our 
freshly published image.

- Why do we put our images into an online repo?

If another dev wants to use our images, he just have to pull them from DockerHub and won't have to 
build them again. This process saves time, a crucial factor of the following practice : pipelines.

## GitHub action practice

**2-1 What are testcontainers?**

Test containers are java libs that allows you to run a bunch of containers to test your app.

**2-2 Document your Github Actions configurations.**

***Events Triggering CI Jobs:***

- When code is pushed to the repository, specifically on the "main" and "develop" branches.
- When pull requests are created or updated.

***Jobs:***

- One job named "test-backend" is defined.
- Job Details:

The "test-backend" job will run on an Ubuntu 22.04 operating system.

***Job Steps:***

 - 1 Checkout Code:
    - Uses the "actions/checkout" action to fetch the code from the GitHub repository.
 - 2 Set up JDK 17:
    - Uses the "actions/setup-java" action to set up Java Development Kit (JDK) version 17 from the 
      "adopt" distribution.
 - 3 Build and Test with Maven:
    - Changes the working directory to a directory called "project."
    - Executes the "mvn clean verify" command, typically used for building and testing Java projects
      with Apache Maven.
  
 
- Secured Variables, why?

The Dockerhub credentials are critical data. If they are in clear in the code and the repo is public,
a malicious person might use them to connect to your Dockerhub account. Using Github secrets is a good
practice to hide them.

- Why did we put needs: test-backend on this job?

We can't build an app that tests have not succeeded. That's why we put this "need" parameter.

- For what purpose do we need to push docker images?

If a dev want's to test our app, it's more convenient if images are already build. It's saving his/her
time.

**2-3 Document your quality gate configuration.**

Overview of the Quality Gate data :

   - Quality Gate: Passed
   - Code Smells: 0
   - Bugs: 0
   - Vulnerabilities: 2
   - Security Hotspots: 3
   - Coverage: 0.0%
   - Duplications: 0
   - Latest Analysis (Main Branch): Passed
   - Last Analysis: 30th October 2023

The Quality Gate passed, indicating good code quality with no code smells or bugs. However, there 
are 2 vulnerabilities and 3 security hotspots. Code coverage is at 0%, and there are no code 
duplications. The latest analysis on the main branch passed on October 30, 2023.

## Ansible

**3-1 Document your inventory and base commands**

This first command uses the basic setup config to test the connection to the server.
```bash
$ ansible all -i ansible/inventories/setup.yml -m ping
```
Ping results :
```
lbarreau.takima.cloud | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}

```
This second command shows the OS of the server's machine.
```bash
$ ansible all -i ansible/inventories/setup.yml -m setup -a "filter=ansible_distribution*"
lbarreau.takima.cloud | SUCCESS => {
    "ansible_facts": {
        "ansible_distribution": "CentOS",
        "ansible_distribution_file_parsed": true,
        "ansible_distribution_file_path": "/etc/redhat-release",
        "ansible_distribution_file_variety": "RedHat",
        "ansible_distribution_major_version": "7",
        "ansible_distribution_release": "Core",
        "ansible_distribution_version": "7.9",
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false
}

```
This third command resets the Apache http server installed in the server.
```bash
$ ansible all -i ansible/inventories/setup.yml -m yum -a "name=httpd state=absent" --become
```
Apache :
```JSON
lbarreau.takima.cloud | SUCCESS => {
    "ansible_facts": {
      "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "msg": "",
    "rc": 0,
    "results": [
    "httpd is not installed"
    ]
}


```

**3-2 Document your playbook**

### Installing Docker on CentOS

This Ansible playbook is used to install Docker on CentOS. It consists of the following tasks:

- ***Role Inclusion***:
    - The playbook starts by including the "docker" role. This role is expected to contain Docker-related tasks, making the playbook more modular and organized.

- ***Install device-mapper-persistent-data***:
    - In this task, the `yum` module is used to install the "device-mapper-persistent-data" package. It ensures that this essential package is up to date.

- ***Install lvm2***:
    - The "Install lvm2" task utilizes the `yum` module to install the "lvm2" package, ensuring it is in the latest state.

- ***Add Docker Repository***:
    - The "add repo docker" task employs the `command` module to add the Docker repository for CentOS. It configures the system to fetch Docker packages from the specified repository URL.

- ***Install Docker***:
    - In the "Install Docker" task, the `yum` module is used to install the "docker-ce" package. This step brings Docker to the system, ensuring it is present.

- ***Make sure Docker is running***:
    - The last task, "Make sure Docker is running," uses the `service` module to ensure that the Docker service is started and set to run automatically during system boot. It uses the "docker" tag for easy categorization.

This playbook simplifies the process of installing Docker on CentOS distrib, making use of roles for better organization and ensuring that the required dependencies and services are set up correctly.

**3-4 Document your docker_container tasks configuration.**

Each docker_container task configuration works the same way has a docker-compose service.

