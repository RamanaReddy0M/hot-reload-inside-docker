## Micronaut + Postgres + Maven/Gradle + Docker

## Run Application Inside Docker

### Step-1:

Ensure working of hot reload in local machine. For a micronaut application it's pretty straight
forward i.e, we don't need to add any dependency or plugin, just need to run the __maven/gradle__
command inside working directory:

__Maven__:

````
./mvnw mn:run
````

__Gradle:__

```
./gradlew run -t
```

Here `-t` enable continuous build that means gradle should re-build the project if any changes
triggered in __src__ directory.

__Maven / Gradle:__
While application is running, change code base by adding some `System.out.println` statement and
save the changes. Then the changes should be auto-compiled and updated without having to
restart the application(hot reload).

### Step-2:

Create `Dockerfile`, `docker-compose.yml` and `.env`(used to pass values to variables used inside
docker-compose.yml) inside the working directory.

Then project structure is:

```
<working-dir>
├── ...
├── src
|     └── ...
├── .env
├── Dockerfile
├── docker-compose.yml
└── README.md
```

__Maven:__

- [Dockerfile](./maven/Dockerfile)
- [docker-compose.yml](./maven/docker-compose.yml)
- [.env](./maven/.env)

__Gradle:__

- [Dockerfile](./gradle/Dockerfile)
- [docker-compose.yml](./gradle/docker-compose.yml)
- [.env](./gradle/.env)

Now, copy the content from `Dockerfile`, `docker-compose.yml`
and `.env` files (based on build tool) to newly
created __Dockerfile__ and __docker-compose.yml__ files in your project.

### Step-3:

Make appropriate changes in [docker-compose.yml](./docker-compose.yml) and [.env](./.env) like:

[__docker-compose.yml__](docker-compose.yml)

```yaml
version: '3.8'
services:
  spring-boot-postgres-maven:
    image: <your-image-name>
    container_name: <your-container-name>
  ....
```

[__.env__](.env)

```
...
DB_NAME=<your-db-name>
...
```

### Step-4:

__Maven/Gradle:__

Now, run the application inside docker:

Simply do `docker-compose up` inside working directory
> $ docker-compose up

Then you may get error like `permission denied exception for ./mvnw file`
or `permission denied exception for ./gradlew file`

To solve this issue, execute the command:

> chmod u+x ./mvnw

Similarly, for gradle do `chmod u+x ./graldew`

Again do a `docker-compose up`, this time you don't get any errors but
for the first build it may take up to 7-10 minutes to pull
images(openjdk:11 and postgres:14.1-alpine) and download dependencies. If everything runs
successfully, by doing `docker ps` you
would see a similar outcome(image and container names may differ) for __maven / gradle__:

```
➜ micronaut-postgres ✗ docker ps
CONTAINER ID   IMAGE                             COMMAND                  CREATED             STATUS             PORTS                                            NAMES
c30ca301b5dd   micronaut-postgres-image         "./gradlew run -t"       About an hour ago   Up About an hour   0.0.0.0:8000->8000/tcp, 0.0.0.0:8080->8080/tcp   micronaut-postgres-container
263e83e4296f   postgres:14.1-alpine              "docker-entrypoint.s…"   About an hour ago   Up About an hour   0.0.0.0:5432->5432/tcp                           postgres-container

```

Now, while the application is up and running inside docker, make the changes to the code base, then
you would see that application running inside docker should restart automatically.

## Debugging

### Maven:

Logs inside docker after doing `docker-compose up`:

```
.......
micronaut-postgres-container  | Listening for transport dt_socket at address: 8000
micronaut-postgres-container  |  __  __ _                                  _   
micronaut-postgres-container  | |  \/  (_) ___ _ __ ___  _ __   __ _ _   _| |_ 
micronaut-postgres-container  | | |\/| | |/ __| '__/ _ \| '_ \ / _` | | | | __|
micronaut-postgres-container  | | |  | | | (__| | | (_) | | | | (_| | |_| | |_ 
micronaut-postgres-container  | |_|  |_|_|\___|_|  \___/|_| |_|\__,_|\__,_|\__|
micronaut-postgres-container  |   Micronaut (v3.7.2)
.......
```

If you observe the log `Listening for transport dt_socket at address: 8000`, that means application
is running in debug mode at port 8000 inside docker.

Application is running in debug mode because of command inside docker-compose:

__docker-compose.yml__

```yaml
...
command: ./mvnw spring-boot:run -Dspring-boot.run.jvmArguments="-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:${DEBUG_PORT_ON_CONTAINER}"
...
```

### Gradle:

In case of gradle to run application in debug mode add `run` task to `build.gradle`:

__run task__:

```
run {
	jvmArgs=["-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:8000"]
}
```

After that do `docker-compose up` inside working directory, then the logs inside docker is:

```
...
micronaut-postgres-container  | Starting a Gradle Daemon, 1 incompatible and 1 stopped Daemons could not be reused, use --status for details
micronaut-postgres-container  | > Task :compileJava UP-TO-DATE
micronaut-postgres-container  | > Task :processResources UP-TO-DATE
micronaut-postgres-container  | > Task :classes UP-TO-DATE
micronaut-postgres-container  | 
micronaut-postgres-container  | > Task :run
micronaut-postgres-container  | Listening for transport dt_socket at address: 8000
micronaut-postgres-container  |  __  __ _                                  _   
micronaut-postgres-container  | |  \/  (_) ___ _ __ ___  _ __   __ _ _   _| |_ 
micronaut-postgres-container  | | |\/| | |/ __| '__/ _ \| '_ \ / _` | | | | __|
micronaut-postgres-container  | | |  | | | (__| | | (_) | | | | (_| | |_| | |_ 
micronaut-postgres-container  | |_|  |_|_|\___|_|  \___/|_| |_|\__,_|\__,_|\__|
micronaut-postgres-container  |   Micronaut (v3.7.2)

.....
```

If you observe the log `Listening for transport dt_socket at address: 8000`, that means application
is running in debug mode at port 8000 inside docker.

__Maven / Gradle__:

Inside __jvmArgs__ we have used the property `address=*:8000` which tells from where we should
attach debugger. Here `*` is
placed in place of __host__ that means we can attach to debugger from any host within the same
network.

Then follow steps in [Remote Debugging Using IntelliJIDEA](../README.md) to attach a debugger.

## Details About Docker-Compose File

```yaml
version: '3.8'
services:
  micronaut-postgres:
    image: micronaut-postgres-maven-image
    container_name: micronaut-postgres-maven-container
    networks:
      - student-grading-network
    build:
      context: .
    env_file: .env
    depends_on:
      - db
    ports:
      - ${APPLICATION_PORT_ON_DOCKER_HOST}:${APPLICATION_PORT_ON_CONTAINER}
      - ${DEBUG_PORT_ON_DOCKER_HOST}:${DEBUG_PORT_ON_CONTAINER}
    volumes:
      - ./:/app
    command: ./mvnw mn:run -Dmn.debug -Dmn.debug.host=* -Dmn.debug.port=${DEBUG_PORT_ON_CONTAINER}

  db:
    container_name: postgres-container
    image: postgres:14.1-alpine
    env_file: .env
    ports:
      - ${DB_PORT_ON_DOCKER_HOST}:${DB_PORT_ON_CONTAINER}
    volumes:
      - db:/var/lib/postgresql/data
    networks:
      - student-grading-network

volumes:
  db:

networks:
  student-grading-network:
```

Here each service acts as a new container. Since our application is dependent on `db` service, we
need
to take care of a few things:

- `micronaut-postgres` service shouldn't start before `db` service. And that is why we
  used `depends_on` property under `micronaut-postgres`.
- `micronaut-postgres` and `db` services both have to be on the same network, so that they
  can communicate with each other. If we don't provide any network to services, they might run in
  isolated networks which leads to communication link failure between the application and the
  database.
- Finally, for hot reloading of the app inside docker, our current directory(where the source code
  exists) should be mounted to the working directory inside the container.

```yaml
    volumes:
      - ./:/app
```

### Reference

Refer the [__micronaut-postgres__](https://github.com/RamanaReddy0M/hot-reload-inside-docker-examples/tree/master/micronaut-postgres)
sample project for complete source code.