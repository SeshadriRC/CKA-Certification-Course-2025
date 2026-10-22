Here are the two Java Dockerfile examples using Maven — one simple Dockerfile and one multistage Dockerfile, suitable for DevOps interviews.

## 1. Simple Dockerfile — Java + Maven

Project structure:

```
java-app/
├── pom.xml
├── src/
│   └── main/
│       └── java/
│           └── com/example/
│               └── Main.java
└── Dockerfile
```

Dockerfile

```
# Use Maven with JDK 21
FROM maven:3.9-eclipse-temurin-21

# Set working directory
WORKDIR /app

# Copy Maven project files
COPY pom.xml .

COPY src ./src

# Build the application and generate JAR
RUN mvn clean package -DskipTests

# Run the generated JAR
CMD ["sh", "-c", "java -jar target/*.jar"]
```

Build and run:

```
docker build -t java-maven-app .
docker run --rm java-maven-app
```

This image contains both Maven and the JDK, so it can build and run the application.

## 2. Multistage Dockerfile — Java + Maven

This is the more common production-style interview example.

```

# Stage 1: Build the application
FROM maven:3.9-eclipse-temurin-21 AS builder

# Set working directory
WORKDIR /app

# Copy Maven configuration and source code
COPY pom.xml .

# copy the src directory from your local project into a src directory inside the container's current working directory.
COPY src ./src

# Build the application and generate JAR
RUN mvn clean package -DskipTests

# Stage 2: Runtime image
FROM eclipse-temurin:21-jre

# Set working directory
WORKDIR /app

# Copy the generated JAR from builder stage
COPY --from=builder /app/target/*.jar app.jar

# Run the Java application
CMD ["java", "-jar", "app.jar"]

```


**App structure**

```bash
Local project:
java-app/
├── pom.xml
└── src/
    └── main/
        └── java/
            └── Main.java
```

Build and run:

```
docker build -t java-multistage .
docker run --rm java-multistage
```

### How multistage works

Stage 1 — Maven + JDK

Copies `pom.xml` and source code, then runs Maven to generate the JAR.

Copies only the generated JAR

Stage 2 — JRE

Runs the JAR without including Maven or the Java compiler.

### Interview difference

| Simple Dockerfile                            | Multistage Dockerfile                       |
| -------------------------------------------- | ------------------------------------------- |
| Maven and JDK are in the final image         | Maven and JDK are only in the builder stage |
| Builds and runs the application in one image | Builds in one stage and runs in another     |
| Usually larger image                         | Usually smaller final image                 |

Interview answer: “I use a multistage Docker build for Java applications. In the first stage, Maven compiles the source code and packages it into a JAR using the JDK. In the second stage, I copy only the JAR into a lightweight Java runtime image. This reduces the final image size and avoids including build tools in production.”

Note: This example assumes your Maven project generates an executable JAR. `-DskipTests` skips test execution during the build, so in a real CI/CD pipeline, tests should normally run in a separate step or be enabled during the build.
