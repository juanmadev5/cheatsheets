# Spring Boot — DevOps

---

## 🐳 Docker

```dockerfile
# Dockerfile — multi-stage build
FROM eclipse-temurin:25-jdk-alpine AS builder
WORKDIR /build

# Descargar dependencias primero (mejor cache)
COPY mvnw .
COPY .mvn .mvn
COPY pom.xml .
RUN ./mvnw dependency:go-offline -q

# Compilar
COPY src ./src
RUN ./mvnw package -DskipTests -q

# Extraer layers para cache eficiente entre deploys
# El jarmode "layertools" fue reemplazado por "tools" (con subcomando extract)
FROM eclipse-temurin:25-jdk-alpine AS extractor
WORKDIR /build
COPY --from=builder /build/target/*.jar application.jar
RUN java -Djarmode=tools -jar application.jar extract --layers --destination extracted

# Imagen final — JRE (más pequeño que JDK)
FROM eclipse-temurin:25-jre-alpine
WORKDIR /app

# Usuario sin privilegios
RUN addgroup -S spring && adduser -S spring -G spring

# Copiar layers en orden de menor a mayor frecuencia de cambio
COPY --from=extractor /build/extracted/dependencies .
COPY --from=extractor /build/extracted/spring-boot-loader .
COPY --from=extractor /build/extracted/snapshot-dependencies .
COPY --from=extractor /build/extracted/application .

USER spring

# Variables de entorno con valores por defecto
ENV JAVA_OPTS="-Xms256m -Xmx512m -XX:+UseG1GC"
ENV SPRING_PROFILES_ACTIVE=prod

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
    CMD wget -qO- http://localhost:8080/api/actuator/health || exit 1

# El layer "application" extraído ya es un jar ejecutable — ya no hace falta invocar JarLauncher a mano
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar application.jar"]
```

```yaml
# docker-compose.yml — entorno de desarrollo
services:
  api:
    build: .
    container_name: mi-api
    ports:
      - "8080:8080"
    environment:
      SPRING_PROFILES_ACTIVE: dev
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/mi_db
      SPRING_DATASOURCE_USERNAME: postgres
      SPRING_DATASOURCE_PASSWORD: postgres
      SPRING_DATA_REDIS_HOST: redis
      APP_JWT_SECRET: dev-secret-key-only-for-development-256bits
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    volumes:
      - ./logs:/app/logs
    restart: unless-stopped
    networks:
      - backend

  db:
    image: postgres:16-alpine
    container_name: mi-db
    environment:
      POSTGRES_DB: mi_db
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres -d mi_db"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - backend

  redis:
    image: redis:7-alpine
    container_name: mi-redis
    command: redis-server --save 60 1 --loglevel warning
    ports:
      - "6379:6379"
    volumes:
      - redisdata:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3
    networks:
      - backend

  mailhog:
    image: mailhog/mailhog
    container_name: mi-mailhog
    ports:
      - "1025:1025"   # SMTP
      - "8025:8025"   # Web UI
    networks:
      - backend

volumes:
  pgdata:
  redisdata:

networks:
  backend:
    driver: bridge
```

