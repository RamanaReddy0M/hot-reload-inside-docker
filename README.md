# hot-reload-inside-docker

If you're looking to run a SpringBoot or Micronaut application inside docker with hot reload then
you can use this repo.


https://user-images.githubusercontent.com/90540245/199573628-164be40b-4eda-41d5-b743-1e6ca8ef3221.mp4


### Outcomes

- Application built inside docker.
- Code changes are auto-compiled and updated without having to restart the app or container.
- Remote debugging using IntelliJIDEA.

## Explanation

Run the application first time by hitting run button in IDE. While application is running, change
code base by adding some `System.out.println` statement and save the changes. This time code changes
should be auto-compiled and updated without having to restart the application(hot reload).

Most of the frameworks itself does support this feature. Use some file watcher tools
like [modd](https://github.com/cortesi/modd) if framework doesn't support hot reload by default.

To have same behaviour inside docker we have created `Dockerfile` and `docker-compose.yml` templates
for the java related technologies like SpringBoot and Micronaut:

| Framework      | Database | <img src="./resources/maven-icon.svg" alt="maven" width="100" height="60">                                  | <img src="./resources/gradle-icon.svg" alt="maven" width="100" height="45"> |
|----------------|----------|-------------------------------------------------------------------------------------------------------------|-------|
| __SpringBoot__ | postgres | [<img src="./resources/folder_type_docker.svg" alt="folder" width="80" height="40"/>](./spring-postgres)    |[<img src="./resources/folder_type_docker.svg" alt="folder" width="80" height="40"/>](./spring-postgres)|
| __Micronaut__  | postgres | [<img src="./resources/folder_type_docker.svg" alt="folder" width="80" height="40"/>](./micronaut-postgres) | [<img src="./resources/folder_type_docker.svg" alt="folder" width="80" height="40"/>](./micronaut-postgres)|

Click on <img src="./resources/folder_type_docker.svg" alt="folder" width="40" height="30"/> icon
based on framework and build tool used in project to navigate to specific docker files.

## Quick Comparison

| Framework |Feature | <img src="./resources/maven-icon.svg" alt="maven" width="100" height="60">                                                   |<img src="./resources/gradle-icon.svg" alt="maven" width="100" height="45">|
|----------|-----|------------------------------------------------------------------------------------------------------------------------------|-----|
| SpringBoot |hot-reload| __step-1:__ Add `spring-boot-devtools` dependency to `pom.xml`. </br> __step-2:__ Run `./mvnw spring-boot:run` inside working directory. | __step-1:__ Add `org.springframework.boot:spring-boot-devtools` dependency to `build.gradle`. </br> __step-2__: Run `./gradlew bootRun` inside working directory.|
|          |remote-debugging| Just run the command `./mvnw spring-boot:run -Dspring-boot.run.jvmArguments="-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:8000"` inside working directory. | __step-1:__ Add task </br>`bootRun { jvmArgs=["-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:8000"] }`</br> to `build.gradle`</br></br> __step-2__: Run `./gradlew bootRun` inside working directory.|
|Micronaut|hot-reload| Run `./mvnw mn:run -Dmn.watch=true` inside working directory                                                                 | Run `./gradlew run -t` inside working directory, Here `-t` enables continious build.|
|          |remote-debugging| Just run the command `./mvnw mn:run -Dmn.debug -Dmn.debug.host=* -Dmn.debug.port=8000` inside working directory.| __step-1:__ Add task </br>`run { jvmArgs=["-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:8000"] }`</br> to `build.gradle`</br></br> __step-2__: Run `./gradlew run -t` inside working directory.

## Remote Debugging Using IntelliJIDEA

https://user-images.githubusercontent.com/90540245/199643735-1462e99f-61ba-4e6b-84f9-6ae60fb3b686.mp4
