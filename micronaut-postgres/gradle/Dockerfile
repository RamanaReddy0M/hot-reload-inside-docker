FROM openjdk:11

WORKDIR /app

COPY gradle gradle
COPY gradlew build.gradle gradle.properties ./
COPY src ./src

RUN ./gradlew clean build -x test

CMD ["./gradlew", "run", "-t"]
