# Multi-stage Dockerfile for OpenFilz API
# Stage 1: Build the application
FROM eclipse-temurin:25-jdk-noble AS build

WORKDIR /app

# Copy Maven wrapper and pom files
COPY .mvn .mvn
COPY mvnw .
COPY pom.xml .
COPY openfilz-api/pom.xml openfilz-api/

# Download dependencies (cached layer)
RUN ./mvnw dependency:go-offline -pl openfilz-api -am

# Copy source code
COPY openfilz-api/src openfilz-api/src

# Build the application
RUN ./mvnw clean package -pl openfilz-api -am -DskipTests

# Stage 2: Runtime image
FROM eclipse-temurin:25-jre-noble

WORKDIR /app

# Create non-root user
RUN groupadd -r openfilz && useradd -r -g openfilz openfilz

# Copy JAR from build stage
COPY --from=build /app/openfilz-api/target/*.jar app.jar

# Create directories for storage and set permissions
RUN mkdir -p /app/storage && \
    chown -R openfilz:openfilz /app

# Switch to non-root user
USER openfilz

# Expose port
EXPOSE 8081

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD curl -f http://localhost:8081/actuator/health || exit 1

# JVM options for container environment
ENV JAVA_OPTS="-XX:+UseContainerSupport \
    -XX:MaxRAMPercentage=75.0 \
    -XX:+UseG1GC \
    -XX:+UseStringDeduplication \
    -Djava.security.egd=file:/dev/./urandom"

# Run the application
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
