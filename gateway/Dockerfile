FROM openjdk:17-jdk-slim
WORKDIR /app

# Copy the built jar
COPY target/gateway-0.0.1-SNAPSHOT.jar app.jar
# Copy the correct application.yaml from resources
COPY src/main/resources/application.yaml /app/config/application.yaml

ENTRYPOINT ["java", "-jar", "app.jar", "--spring.config.location=file:/app/config/application.yaml"]
