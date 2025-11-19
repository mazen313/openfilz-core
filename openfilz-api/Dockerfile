# Multi-stage Dockerfile for OpenFilz API
# Stage 1: Build the application
FROM maven:3.9-eclipse-temurin-25 AS build

WORKDIR /app

# Copy pom files for dependency resolution
COPY pom.xml .
COPY openfilz-api/pom.xml openfilz-api/
COPY openfilz-gateway/pom.xml openfilz-gateway/

# Download dependencies (cached layer)
RUN mvn dependency:go-offline -pl openfilz-api -am

# Copy source code
COPY openfilz-api/src openfilz-api/src

# Build the application and create executable JAR
# Use verify phase to trigger spring-boot:repackage in post-integration-test phase
RUN mvn clean verify -pl openfilz-api -am -DskipTests

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

# Environment variables with defaults
# Database Configuration
ENV DB_HOST=localhost \
    DB_PORT=5432 \
    DB_NAME=dms_db \
    DB_USER=dms_user \
    DB_PASSWORD=dms_password

# Storage Configuration
ENV STORAGE_TYPE=local \
    STORAGE_LOCAL_BASE_PATH=/app/storage \
    MINIO_ENDPOINT=http://localhost:9000 \
    MINIO_ACCESS_KEY=minioadmin \
    MINIO_SECRET_KEY=minioadmin \
    MINIO_BUCKET_NAME=dms-bucket

# Security Configuration
ENV KEYCLOAK_ISSUER_URI=http://localhost:8080/realms/your-realm \
    OPENFILZ_SECURITY_NO_AUTH=true \
    OPENFILZ_SECURITY_WORM_MODE=false

# OpenSearch Configuration (Optional)
ENV OPENSEARCH_URL=http://localhost:9200

# Application Configuration
ENV SERVER_PORT=8081 \
    SPRING_PROFILES_ACTIVE=default

# Logging Configuration
ENV LOGGING_LEVEL_ORG_OPENFILZ=INFO

# JVM options for container environment
ENV JAVA_OPTS="-XX:+UseContainerSupport \
    -XX:MaxRAMPercentage=75.0 \
    -XX:+UseG1GC \
    -XX:+UseStringDeduplication \
    -Djava.security.egd=file:/dev/./urandom"

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD curl -f http://localhost:8081/actuator/health || exit 1

# Run the application
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
