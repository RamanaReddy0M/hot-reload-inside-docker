# hot-reload-inside-docker

If you're looking to run a SpringBoot or Micronaut application inside docker with hot reload then you can use this repo.


### Outcomes

- Application built inside docker.
- Code changes auto-compiled and updated without having to restart the app or container.
- Remote debugging using IntelliJIDEA.

## Explanation


Run the application first time by hitting run button in IDE. While application is running, change
code base by adding some `System.out.println` statement and save the changes. This time code changes
should be auto-compiled and updated without having to restart the application(hot reload).

Most of the frameworks itself does support this feature. Use some file watcher tools like [modd](https://github.com/cortesi/modd) if framework doesn't support hot reload by default. 

To have same behaviour inside docker we have created `Dockerfile` and `docker-compose.yml` templates for the java related technologies like SpringBoot and Micronaut:



|                      | <img src="./resources/maven-icon.svg" alt="maven" width="100" height="60">                                       | <img src="./resources/gradle-icon.svg" alt="maven" width="100" height="45"> |
|----------------------|------------------------------------------------------------------------------------------------------------------|-------|
| __spring-postgres__  |[<img src="./resources/folder_type_docker.svg" alt="folder" width="80" height="40"/>](./spring-postgres/maven)    |[<img src="./resources/folder_type_docker.svg" alt="folder" width="80" height="40"/>](./spring-postgres/gradle)|
| __micronaut-postgres__ | [<img src="./resources/folder_type_docker.svg" alt="folder" width="80" height="40"/>](./micronaut-postgres/maven) | [<img src="./resources/folder_type_docker.svg" alt="folder" width="80" height="40"/>](./micronaut-postgres/gradle)|

Click on <img src="./resources/folder_type_docker.svg" alt="folder" width="40" height="30"/> icon
based on framework and build tool used in project to navigate to specific docker files.