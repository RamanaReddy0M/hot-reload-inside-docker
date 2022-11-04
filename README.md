[![Documentation](https://img.shields.io/static/v1?label=Documentation&message=v1.0&color=<COLOR>)](https://ramanareddy0m.github.io/hot-reload-inside-docker/docs/intro)

# hot-reload-inside-docker

If you're looking to run a SpringBoot or Micronaut application inside docker with hot reload then
you can use this repo.

https://user-images.githubusercontent.com/90540245/199573628-164be40b-4eda-41d5-b743-1e6ca8ef3221.mp4

### Outcomes

- Application built inside docker.
- Code changes are auto-compiled and updated without having to restart the app or container.
- Remote debugging using IntelliJIDEA.

## Prerequisites

- Make sure that you have Docker and Docker-Compose installed
    - Windows or macOS: [Install Docker Desktop](https://www.docker.com/get-started/)
    - Linux: [Install Docker](https://www.docker.com/get-started/) and
      then [Docker Compose](https://github.com/docker/compose)

## Description

Run the application first time by hitting run button in IDE. While application is running, make some
changes to the code in `src` directory and save the changes. This time code changes
should be auto-compiled and updated without having to restart the application manually, and that is
what we call a __Hot Reload__.

Most of the frameworks support this feature locally. To have same behaviour inside
docker we have created `Dockerfile` and `docker-compose.yml` templates
for the java related technologies like SpringBoot and Micronaut. Additionally, you can debug the
application running inside docker through IntelliJIDEA. To know more about it refer [Remote Debugging Using IntellijIDEA. ](https://github.com/RamanaReddy0M/hot-reload-inside-docker/blob/master/README.md#remote-debugging-using-intellijidea)

## Quick Start

__Navigation Table__

| Framework      | Database | <img src="./resources/maven-icon.svg" alt="maven" width="100" height="60">                                  | <img src="./resources/gradle-icon.svg" alt="maven" width="100" height="45"> |
|----------------|----------|-------------------------------------------------------------------------------------------------------------|-------|
| __SpringBoot__ | postgres | [<img src="./resources/folder_type_docker.svg" alt="folder" width="80" height="40"/>](./spring-postgres)    |[<img src="./resources/folder_type_docker.svg" alt="folder" width="80" height="40"/>](./spring-postgres)|
| __Micronaut__  | postgres | [<img src="./resources/folder_type_docker.svg" alt="folder" width="80" height="40"/>](./micronaut-postgres) | [<img src="./resources/folder_type_docker.svg" alt="folder" width="80" height="40"/>](./micronaut-postgres)|

__Step-1:__ Click
on <img src="./resources/folder_type_docker.svg" alt="folder" width="40" height="30"/> icon
based on framework and build tool used in project to navigate to specific docker files. Check
the `README.md` per framework for the detailed steps.

__Step-2:__ Now copy the `Dockerfile`, `docker-compose.yml` and `.env` files from repository where
you have been navigated, to the working directory and make appropriate changes.

__Step-3:__ Simply run:

```
cd <working-dir>
docker-compose up
```

## Explanation

### Key Concepts

- Volume Mapping
- Gradle / Maven Dependency Caching

### Volume Mapping

The essential component is __mounting__ current directory from local machine to `app`(WORKDIR)
directory
inside Docker container. SpringBoot / Micronaut application by default comes with Maven / Gradle
Wrappers, this would allow us to run the application within Docker container.

### Gradle / Maven Dependency Caching

Mounting current directory into Docker container helps to have source code and the
build tool within Docker container, but the source code within Docker is dependent on many external
libraries(dependencies) which are not present in current directory.

There are two ways to solve this issue:

- Mounting `.m2` / `.gradle` from Docker Host to Docker container.
- Caching all the dependencies while building the Docker image

Mounting root level directories is not an option to choose. But there are some other issue with
Gradle
caching, if you are mounting `.gradle` and running the application with Gradle build tool within
Docker container, then docker acquires lock for gradle cache, that means we
can't run any application with Gradle build in local machine until Docker container is stopped.

Caching all the dependencies while building Docker image is a good option during development phase.
Since we are downloading all the dependencies image size would be larger(depends on dependencies).

### Comparison between Framework, Feature and Build Tools for Hot-Reload and Remote-Debugging

Table depicts the way to enable hot-reload / remote-debugging feature for the Java related
technologies(SpringBoot / Micronaut) between the Maven and Gradle build tools.

| Framework | Feature          | <img src="./resources/maven-icon.svg" alt="maven" width="100" height="60">                                                                                                    | <img src="./resources/gradle-icon.svg" alt="maven" width="100" height="45">                                                                                                                                             |
|----------|------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| SpringBoot | hot-reload       | __Step-1:__ Add `spring-boot-devtools` dependency to `pom.xml`. </br> __Step-2:__ Run `./mvnw spring-boot:run` inside working directory.                                      | __Step-1:__ Add `org.springframework.boot:spring-boot-devtools` dependency to `build.gradle`. </br> __step-2__: Run `./gradlew bootRun` inside working directory.                                                       |
|          | remote-debugging | Just run the command `./mvnw spring-boot:run -Dspring-boot.run.jvmArguments="-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:8000"` inside working directory. | __Step-1:__ Add task </br>`bootRun { jvmArgs=["-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:8000"] }`</br> to `build.gradle`</br></br> __Step-2__: Run `./gradlew bootRun` inside working directory. |
|Micronaut| hot-reload       | Run `./mvnw mn:run -Dmn.watch=true` inside working directory                                                                                                                  | Run `./gradlew run -t` inside working directory, Here `-t` enables continious build.                                                                                                                                    |
|          | remote-debugging | Just run the command `./mvnw mn:run -Dmn.debug -Dmn.debug.host=* -Dmn.debug.port=8000` inside working directory.                                                              | __Step-1:__ Add task </br>`run { jvmArgs=["-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:8000"] }`</br> to `build.gradle`</br></br> __Step-2__: Run `./gradlew run -t` inside working directory.      

## Remote Debugging Using IntelliJIDEA

https://user-images.githubusercontent.com/90540245/199643735-1462e99f-61ba-4e6b-84f9-6ae60fb3b686.mp4

> __Note:__ Check
> out [Comparison between Framework, Feature and Build Tools for Hot-Reload and Remote-Debugging]()
> section
> if you are using different build tool or framework.