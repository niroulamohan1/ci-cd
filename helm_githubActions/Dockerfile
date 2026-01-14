# Multi-stage build for Spring Boot
FROM maven:3.8.5-openjdk-17 as builder
WORKDIR /build
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn clean package -DskipTests

# Runtime
FROM openjdk:17-slim
WORKDIR /app
COPY --from=builder /build/target/*.jar app.jar
RUN useradd -m -u 1000 appuser && chown -R appuser:appuser /app
USER appuser
EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
    CMD curl -f http://localhost:8080/actuator/health || exit 1
ENTRYPOINT ["java", "-Xmx512m", "-Xms256m", "-jar", "app.jar"]
