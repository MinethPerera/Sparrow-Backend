# ====== Build stage ======
FROM gradle:8.9-jdk21-alpine AS build
WORKDIR /app

# Copy Gradle config first (for caching)
COPY build.gradle settings.gradle ./
COPY gradle ./gradle

# Run Gradle build once to cache dependencies
RUN gradle clean build -x test --no-daemon || true

# Copy source code
COPY src ./src

# Build application JAR
RUN gradle clean build -x test --no-daemon

# ====== Runtime stage ======
FROM eclipse-temurin:21-jre-alpine AS runtime
WORKDIR /app

# Create non-root user
RUN addgroup -S spring && adduser -S spring -G spring
USER spring

# Copy the fat JAR from build
COPY --from=build /app/build/libs/*.jar /app/app.jar

# Environment variables
ENV JAVA_OPTS="-XX:+UseZGC -XX:MaxRAMPercentage=75.0"
ENV SERVER_PORT=8080

EXPOSE 8080

# Healthcheck (Actuator should be enabled)
HEALTHCHECK --interval=20s --timeout=3s --retries=5 --start-period=20s \
  CMD wget -qO- http://localhost:${SERVER_PORT}/actuator/health | grep -q '"status":"UP"' || exit 1

ENTRYPOINT ["/bin/sh","-c","java $JAVA_OPTS -jar /app/app.jar"]
