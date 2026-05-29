# Stage 1: build
FROM eclipse-temurin:17-jdk-alpine AS builder
WORKDIR /app
COPY mvnw pom.xml ./
COPY .mvn .mvn
RUN chmod +x mvnw
COPY src ./src
RUN ./mvnw -B -DskipTests package

# Stage 2: runtime (non-root)
FROM eclipse-temurin:17-jre-alpine AS runtime
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar
USER appuser
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
