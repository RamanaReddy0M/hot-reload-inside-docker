## SpringBoot + Postgres + Maven/Gradle + Docker

## Run Application Inside Docker
### Step-1:

Ensure working of hot reload in local machine.

__Maven__:

To ensure that hot reload works locally, we need to
add `spring-boot-devtools` dependency to `pom.xml` which helps to re-run the application when the
changes are detected.

````xml

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
</dependency>
````

__Gradle:__

For gradle add `spring-boot-devtools` dependency to `build.gradle` file

```yaml
developmentOnly 'org.springframework.boot:spring-boot-devtools'
```

__Maven / Gradle:__
> __Note__: If you're using __IntelliJIDEA__ as development tool, after adding `devtools` dependency
> enable these properties:
>
> 1.Navigate to - __IntelliJIDEA -> Preferences -> Build, Execution, Deployment -> Compiler__ and
> enable `Build project automatically`.
>
> 2.Navigate to - __IntelliJIDEA -> Preferences -> Advanced Settings__ and
> enable `Allow auto-make to start even if developed application is currently running`.


After applying these changes run the application. While application is running, change
code base by adding some `System.out.println` statement and save the changes. This time code changes
should be auto-compiled and updated without having to restart the application(hot reload).

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
➜  spring-postgres ✗ docker ps
CONTAINER ID   IMAGE                             COMMAND                  CREATED          STATUS          PORTS                                            NAMES
8247f3b42566   spring-boot-postgres-image       "./mvnw spring-boot:…"   29 seconds ago   Up 25 seconds   0.0.0.0:8000->8000/tcp, 0.0.0.0:8080->8080/tcp    spring-boot-postgres-container
04a7dbf0c0e3   postgres:14.1-alpine              "docker-entrypoint.s…"   4 minutes ago    Up 4 minutes    5432/tcp                                          postgres-container
```

Now, while the application is up and running inside docker, make the changes to the code base, then
you would see that application running inside docker should restart automatically.

## Debugging

### Maven:

Logs inside docker after doing `docker-compose up`:

```
.......
spring-boot-postgres-maven-container  | Listening for transport dt_socket at address: 8000
spring-boot-postgres-maven-container  | 06:32:14.319 [Thread-0] DEBUG org.springframework.boot.devtools.restart.classloader.RestartClassLoader - Created RestartClassLoader org.springframework.boot.devtools.restart.classloader.RestartClassLoader@4ab3d40a
spring-boot-postgres-maven-container  | 
spring-boot-postgres-maven-container  |   .   ____          _            __ _ _
spring-boot-postgres-maven-container  |  /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
spring-boot-postgres-maven-container  | ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
spring-boot-postgres-maven-container  |  \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
spring-boot-postgres-maven-container  |   '  |____| .__|_| |_|_| |_\__, | / / / /
spring-boot-postgres-maven-container  |  =========|_|==============|___/=/_/_/_/
spring-boot-postgres-maven-container  |  :: Spring Boot ::                (v2.7.5)
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

In case of gradle to run application in debug mode add `bootRun` task to `build.gradle`:

__bootRun task__:
```
bootRun {
	jvmArgs=["-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:8000"]
}
```

After that do `docker-compose up` inside working directory, then the logs inside docker is:
```
...
spring-postgres-gradle-container  | > Task :run
spring-postgres-gradle-container  | Listening for transport dt_socket at address: 8000
spring-postgres-gradle-container  | 06:32:14.319 [Thread-0] DEBUG org.springframework.boot.devtools.restart.classloader.RestartClassLoader - Created RestartClassLoader org.springframework.boot.devtools.restart.classloader.RestartClassLoader@4ab3d40a
spring-postgres-gradle-container  | 
spring-postgres-gradle-container  |   .   ____          _            __ _ _
spring-postgres-gradle-container  |  /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
spring-postgres-gradle-container  | ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
spring-postgres-gradle-container  |  \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
spring-postgres-gradle-container  |   '  |____| .__|_| |_|_| |_\__, | / / / /
spring-postgres-gradle-container  |  =========|_|==============|___/=/_/_/_/
spring-postgres-gradle-container  |  :: Spring Boot ::                (v2.7.5)
.....
```
If you observe the log `Listening for transport dt_socket at address: 8000`, that means application is running in debug mode at port 8000 inside docker.

__Maven / Gradle__:

Inside __jvmArgs__ we have used the property `address=*:8000` which tells from where we should attach debugger. Here `*` is
placed in place of __host__ that means we can attach to debugger from any host within the same network.

Then follow steps in [Remote Debugging Using IntelliJIDEA](../README.md) to attach a debugger.

## Details About Docker-Compose File
```yaml
version: '3.8'
services:
  spring-boot-postgres:
    image: spring-boot-postgres-maven-image
    container_name: spring-boot-postgres-maven-container
    networks:
      - spring-boot-postgres-network
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
    command: ./mvnw spring-boot:run -Dspring-boot.run.jvmArguments="-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:${DEBUG_PORT_ON_CONTAINER}"

  db:
    container_name: postgres-container
    image: postgres:14.1-alpine
    env_file: .env
    ports:
      - ${DB_PORT_ON_DOCKER_HOST}:${DB_PORT_ON_CONTAINER}
    volumes:
      - db:/var/lib/postgresql/data
    networks:
      - spring-boot-postgres-network

volumes:
  db:

networks:
  spring-boot-postgres-network:
```
Here each service acts as a new container. Since our application is dependent on `db` service, we
need
to take care of a few things:

- `spring-boot-postgres` service shouldn't start before `db` service. And that is why we
  used `depends_on` property under `spring-boot-postgres`.
- `spring-boot-postgres` and `db` services both have to be on the same network, so that they
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

Refer the [__spring-postgres__](https://github.com/RamanaReddy0M/hot-reload-inside-docker-examples/tree/master/spring-boot-postgres)
sample project for complete source code.