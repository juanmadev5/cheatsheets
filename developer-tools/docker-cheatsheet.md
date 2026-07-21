# Docker & Docker Compose — Cheatsheet

---

## 📑 Índice

### Docker CLI
- [Imágenes — pull, build, tag, push, rmi, prune](#imágenes)
- [Contenedores — run, start, stop, kill, restart, rm](#contenedores--ciclo-de-vida)
- [Flags de docker run — puertos, volúmenes, env, red, restart, límites](#flags-principales-de-docker-run)
- [Inspección y logs — ps, logs, inspect, stats, exec, diff](#inspección-y-logs)
- [Volúmenes — create, ls, inspect, rm, prune](#volúmenes)
- [Redes — create, connect, disconnect, inspect, rm](#redes)
- [Copiar archivos — cp, export, save, load](#copiar-archivos)
- [Limpieza — prune, system df](#limpieza-del-sistema)

### Docker Compose
- [Comandos principales — up, down, start, stop, restart](#comandos-principales)
- [Inspección — ps, logs, top, config, port](#inspección)
- [Ejecución y build — exec, run, build, pull, push](#ejecución-y-build)
- [Escalar y archivos — scale, -f, --profile, env-file](#escalar-y-archivos)
- [Watch — auto-sync en desarrollo](#watch--auto-sync-en-desarrollo)
- [Estructura de docker-compose.yml — servicios, volúmenes, redes, healthcheck](#-estructura-de-docker-composeyml)

### Dockerfile
- [Instrucciones clave — FROM, WORKDIR, COPY, RUN, CMD, ENTRYPOINT](#-dockerfile--instrucciones-clave)
- [Instrucciones de referencia rápida](#instrucciones-de-referencia-rápida)
- [.dockerignore — patrones para reducir contexto de build](#-dockerignore)
- [Multi-stage builds — Go, Node.js, Java Spring Boot](#️-multi-stage-builds)

### Networking
- [Tipos de red — bridge, host, overlay, macvlan, none](#tipos-de-red)
- [Networking avanzado — inspección, DNS entre containers](#-networking-avanzado)
- [Networking en Docker Compose — redes internas, externas, aliases](#networking-en-docker-compose)

### Almacenamiento
- [Tipos de montaje — named volume, bind mount, tmpfs, anonymous](#tipos-de-montaje)
- [Volumes avanzado — backup, restore, copiar entre volumes](#-volumes-avanzado)
- [Volumes en Docker Compose — opciones, externos, NFS](#volumes-en-docker-compose)

### Seguridad y configuración
- [Docker Secrets — crear, montar, leer en aplicación](#-docker-secrets)
- [Docker Scout — análisis de vulnerabilidades (CVEs)](#-docker-scout--análisis-de-vulnerabilidades)
- [Variables de entorno — -e, --env-file, interpolación en Compose](#️-variables-de-entorno)
- [Network Security Config — bridge vs host, redes internas](#networking-en-docker-compose)

### Recursos y observabilidad
- [Límites de recursos — CPU, memoria, PID en run y Compose](#-límites-de-recursos)
- [Logging drivers — json-file, Loki, rotación, configuración global](#-logging-drivers)

### Build avanzado
- [Docker Buildx — multi-arquitectura AMD64/ARM64, caché en registry](#️-docker-buildx--multi-arquitectura)

### Infraestructura
- [Docker Context — múltiples hosts remotos por SSH](#️-docker-context--múltiples-hosts)
- [Docker Swarm — init, stacks, servicios, réplicas, rolling update, rollback](#️-docker-swarm--orquestación-básica)
- [Docker Registry — Docker Hub, ghcr.io, registry privado propio](#️-docker-registry)

### Referencia
- [Buenas prácticas](#-buenas-prácticas)

---

## 🐳 Docker CLI

### Imágenes

| Comando | Descripción |
|---|---|
| `docker pull <image>` | Descargar imagen del registry |
| `docker build -t <name> .` | Construir imagen desde Dockerfile |
| `docker build -t <name> --no-cache .` | Construir sin caché |
| `docker images` | Listar imágenes locales |
| `docker rmi <image>` | Eliminar imagen |
| `docker image prune` | Eliminar imágenes sin usar |
| `docker tag <image> <repo:tag>` | Etiquetar imagen |
| `docker push <repo:tag>` | Subir imagen al registry |
| `docker save -o img.tar <image>` | Exportar imagen a archivo |
| `docker load -i img.tar` | Importar imagen desde archivo |

---

### Contenedores — ciclo de vida

| Comando | Descripción |
|---|---|
| `docker run <image>` | Crear y arrancar contenedor |
| `docker run -d <image>` | Arrancar en segundo plano |
| `docker run --rm <image>` | Eliminar al terminar |
| `docker run -it <image> bash` | Modo interactivo |
| `docker start <id>` | Iniciar contenedor detenido |
| `docker stop <id>` | Detener (SIGTERM) |
| `docker kill <id>` | Detener (SIGKILL) |
| `docker restart <id>` | Reiniciar contenedor |
| `docker rm <id>` | Eliminar contenedor |
| `docker rm -f <id>` | Forzar eliminación |

---

### Flags principales de `docker run`

| Flag | Descripción |
|---|---|
| `-p 8080:80` | Mapear puerto `host:contenedor` |
| `-v /host:/app` | Montar volumen |
| `--name <name>` | Asignar nombre al contenedor |
| `-e KEY=VALUE` | Variable de entorno |
| `--env-file .env` | Cargar variables desde archivo |
| `--network <net>` | Conectar a una red |
| `--restart unless-stopped` | Política de reinicio |
| `--cpus 1.5` | Límite de CPU |
| `--memory 512m` | Límite de memoria |
| `--read-only` | Filesystem de solo lectura |
| `--user <uid>` | Correr como usuario específico |

---

### Inspección y logs

| Comando | Descripción |
|---|---|
| `docker ps` | Contenedores activos |
| `docker ps -a` | Todos (incluye detenidos) |
| `docker logs <id>` | Ver logs |
| `docker logs -f <id>` | Logs en tiempo real |
| `docker logs --tail 50 <id>` | Últimas 50 líneas |
| `docker inspect <id>` | Metadatos en JSON |
| `docker stats` | Uso de CPU/RAM en vivo |
| `docker top <id>` | Procesos del contenedor |
| `docker exec -it <id> sh` | Shell dentro del contenedor |
| `docker diff <id>` | Cambios en el filesystem |
| `docker port <id>` | Puertos mapeados |

---

### Volúmenes

| Comando | Descripción |
|---|---|
| `docker volume create <name>` | Crear volumen |
| `docker volume ls` | Listar volúmenes |
| `docker volume inspect <name>` | Ver detalles |
| `docker volume rm <name>` | Eliminar volumen |
| `docker volume prune` | Eliminar volúmenes no usados |

---

### Redes

| Comando | Descripción |
|---|---|
| `docker network create <name>` | Crear red |
| `docker network ls` | Listar redes |
| `docker network inspect <name>` | Ver detalles |
| `docker network connect <net> <id>` | Conectar contenedor |
| `docker network disconnect <net> <id>` | Desconectar contenedor |
| `docker network rm <name>` | Eliminar red |

---

### Copiar archivos

| Comando | Descripción |
|---|---|
| `docker cp <id>:/ruta ./local` | Copiar desde contenedor |
| `docker cp ./local <id>:/ruta` | Copiar hacia contenedor |
| `docker export <id> > cont.tar` | Exportar filesystem del contenedor |

---

### Limpieza del sistema

| Comando | Descripción |
|---|---|
| `docker system prune` | Limpiar todo lo sin uso |
| `docker system prune -a` | Incluye imágenes sin contenedor |
| `docker system df` | Ver espacio usado por Docker |
| `docker container prune` | Eliminar contenedores detenidos |
| `docker image prune -a` | Eliminar todas las imágenes sin uso |

---

## 🐙 Docker Compose

> Todos los comandos usan `docker compose` (v2). Si usás v1, reemplazá por `docker-compose`.

### Comandos principales

| Comando | Descripción |
|---|---|
| `docker compose up` | Crear y arrancar servicios |
| `docker compose up -d` | En segundo plano |
| `docker compose up --build` | Forzar rebuild antes de arrancar |
| `docker compose down` | Detener y eliminar contenedores |
| `docker compose down -v` | También elimina volúmenes |
| `docker compose down --rmi all` | También elimina imágenes |
| `docker compose start` | Iniciar servicios detenidos |
| `docker compose stop` | Detener sin eliminar |
| `docker compose restart` | Reiniciar servicios |
| `docker compose pause` | Pausar procesos |
| `docker compose unpause` | Reanudar procesos |

---

### Inspección

| Comando | Descripción |
|---|---|
| `docker compose ps` | Estado de los servicios |
| `docker compose logs` | Logs de todos los servicios |
| `docker compose logs -f <svc>` | Logs en tiempo real de un servicio |
| `docker compose logs --tail 100` | Últimas 100 líneas |
| `docker compose top` | Procesos corriendo |
| `docker compose config` | Mostrar configuración resuelta |
| `docker compose images` | Imágenes usadas |
| `docker compose port <svc> 80` | Ver puerto mapeado |

---

### Ejecución y build

| Comando | Descripción |
|---|---|
| `docker compose exec <svc> sh` | Shell en servicio activo |
| `docker compose run <svc> bash` | Contenedor temporal nuevo |
| `docker compose run --rm <svc> <cmd>` | Ejecutar comando y limpiar |
| `docker compose build` | Construir imágenes |
| `docker compose build --no-cache` | Rebuild sin caché |
| `docker compose pull` | Actualizar imágenes |
| `docker compose push` | Subir imágenes al registry |

---

### Opciones avanzadas

| Comando | Descripción |
|---|---|
| `docker compose up --scale web=3` | Escalar réplicas de un servicio |
| `docker compose -f custom.yml up` | Usar archivo alternativo |
| `docker compose -f a.yml -f b.yml up` | Combinar múltiples archivos |
| `docker compose --env-file .env.prod up` | Cargar env específico |
| `docker compose --profile dev up` | Activar perfil |
| `COMPOSE_PROJECT_NAME=myapp` | Definir nombre del proyecto |

---

### Watch — auto-sync en desarrollo

`docker compose watch` sincroniza cambios de código al container corriendo (o rebuildea) sin reiniciar `docker compose up`, reemplazando bind mounts + nodemon/similar para hot reload.

```bash
docker compose watch              # Requiere una sección `develop.watch` por servicio
docker compose up --watch         # Levantar servicios y activar watch en el mismo comando
```

```yaml
services:
  api:
    build: .
    develop:
      watch:
        - action: sync            # Copiar archivos al container sin rebuild
          path: ./src
          target: /app/src
        - action: rebuild         # Reconstruir la imagen si cambian las dependencias
          path: package.json
        - action: sync+restart    # Copiar y reiniciar el proceso del container
          path: ./config
          target: /app/config
```

---

## 📄 Estructura de `docker-compose.yml`

```yaml
services:
  web:
    build: .                        # Dockerfile en el directorio actual
    image: myapp:latest
    ports:
      - "8080:80"
    environment:
      - NODE_ENV=production
    env_file:
      - .env
    volumes:
      - ./src:/app/src              # Bind mount
      - node_modules:/app/node_modules  # Volumen nombrado
    depends_on:
      db:
        condition: service_healthy  # Esperar healthcheck
    restart: unless-stopped
    networks:
      - backend
    profiles:
      - dev                         # Solo se levanta con --profile dev

  db:
    image: postgres:18-alpine
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:8-alpine   # Desde Redis 8, licencia tri-license (SSPL/RSAL/AGPLv3) — Valkey (BSD-3) es la alternativa 100% open source
    restart: always

volumes:
  pgdata:
  node_modules:

networks:
  backend:
    driver: bridge
```

---

## 📦 Dockerfile — instrucciones clave

```dockerfile
# Multi-stage build: stage base
FROM node:24-alpine AS base
WORKDIR /app

# Copiar dependencias primero (mejor uso de caché)
COPY package*.json ./
RUN npm ci --only=production

# Stage final
FROM base AS production
COPY . .

# Variables de entorno
ENV PORT=3000 \
    NODE_ENV=production

# Puerto expuesto (documentación)
EXPOSE 3000

# No correr como root
USER node

# Healthcheck
HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD wget -qO- http://localhost:3000/health || exit 1

# ENTRYPOINT: ejecutable fijo (no sobreescribible sin --entrypoint)
ENTRYPOINT ["node"]

# CMD: argumentos por defecto (sobreescribible en docker run)
CMD ["dist/main.js"]
```

### Instrucciones de referencia rápida

| Instrucción | Descripción |
|---|---|
| `FROM <image>` | Imagen base |
| `WORKDIR <path>` | Directorio de trabajo |
| `COPY <src> <dest>` | Copiar archivos al contexto de build |
| `ADD <src> <dest>` | Como COPY, admite URLs y descomprime tarballs |
| `RUN <cmd>` | Ejecutar comando en build (crea capa) |
| `CMD ["cmd"]` | Comando por defecto al arrancar |
| `ENTRYPOINT ["cmd"]` | Punto de entrada fijo |
| `ENV KEY=VALUE` | Variable de entorno |
| `ARG NAME=default` | Argumento de build (`--build-arg`) |
| `EXPOSE <port>` | Documentar puerto (no lo publica) |
| `VOLUME <path>` | Declarar punto de montaje |
| `USER <user>` | Usuario para RUN/CMD/ENTRYPOINT |
| `HEALTHCHECK` | Verificación de salud del contenedor |
| `LABEL key=value` | Metadatos de la imagen |
| `ONBUILD <instr>` | Instrucción para imágenes derivadas |

---

## 💡 Buenas prácticas

- **Ordenar instrucciones por frecuencia de cambio**: lo más estable primero (deps) y el código fuente al final, para aprovechar el caché de capas.
- **Combinar RUNs** con `&&` para reducir capas: `RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*`
- **Usar `.dockerignore`** para excluir `node_modules`, `.git`, logs y archivos de entorno.
- **Multi-stage builds** para mantener la imagen final pequeña (sin herramientas de build).
- **Nunca correr como root** en producción; usar `USER` con un usuario sin privilegios.
- **Usar imágenes Alpine o Distroless** cuando sea posible para minimizar superficie de ataque.
- **Pinear versiones** (`node:24.4-alpine3.21`) en producción para builds reproducibles.
- **`depends_on` con `condition: service_healthy`** en lugar de scripts de espera externos.

---

## 🌐 Networking avanzado

### Tipos de red

| Driver | Descripción | Uso típico |
|---|---|---|
| `bridge` | Red privada virtual en el host (default) | Containers en el mismo host |
| `host` | El container usa la red del host directamente | Máximo rendimiento, sin aislamiento |
| `overlay` | Red multi-host (Docker Swarm) | Containers en distintos hosts |
| `macvlan` | El container tiene su propia MAC/IP en la red física | Integrar con red existente |
| `none` | Sin red | Containers completamente aislados |

```bash
# Crear redes
docker network create backend                            # bridge por defecto
docker network create --driver bridge \
  --subnet 172.20.0.0/16 \
  --gateway 172.20.0.1 \
  my-network

docker network create --driver overlay \
  --attachable \
  overlay-net                                            # Para Swarm o standalone

# Conectar/desconectar containers en runtime
docker network connect backend my-container
docker network disconnect backend my-container

# Inspeccionar — ver IPs de todos los containers conectados
docker network inspect backend

# Comunicación entre containers por nombre (DNS automático)
# Los containers en la misma red bridge personalizada se resuelven por nombre
# En docker-compose, el nombre del servicio ES el hostname
# Ejemplo: desde "api" hacer fetch a "http://db:5432" (servicio "db")

# Ver IP de un container específico
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' my-container

# Ver todos los containers con sus IPs
docker inspect -f '{{.Name}} - {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' \
  $(docker ps -q)
```

---

### Networking en Docker Compose

```yaml
services:
  api:
    build: .
    networks:
      - frontend
      - backend
    # Alias adicional para el servicio en la red
    networks:
      backend:
        aliases:
          - api-service

  db:
    image: postgres:18-alpine
    networks:
      - backend    # Solo accesible desde backend, no desde frontend

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    networks:
      - frontend   # Solo expuesto al exterior

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true   # Sin acceso a internet — solo comunicación interna

  # Usar red existente (creada fuera de compose)
  shared-net:
    external: true
    name: my-existing-network
```

---

## 💾 Volumes avanzado

### Tipos de montaje

| Tipo | Sintaxis | Descripción |
|---|---|---|
| **Named volume** | `myvolume:/app/data` | Gestionado por Docker, persiste entre containers |
| **Bind mount** | `/host/path:/container/path` | Monta directorio del host — ideal para desarrollo |
| **tmpfs** | `type=tmpfs,target=/tmp` | Solo en memoria RAM — no persiste, muy rápido |
| **Anonymous** | `/app/data` | Volume sin nombre — se elimina con el container |

```bash
# Named volumes
docker volume create pgdata
docker run -v pgdata:/var/lib/postgresql/data postgres:18-alpine

# Bind mount — solo ruta absoluta o ./relativa
docker run -v /home/user/app:/app my-image
docker run -v $(pwd):/app my-image                         # Directorio actual
docker run -v $(pwd)/src:/app/src:ro my-image              # :ro — solo lectura

# tmpfs — datos en memoria (secretos, caché temporal)
docker run --mount type=tmpfs,target=/tmp,tmpfs-size=100m my-image

# Backup de un volume
docker run --rm \
  -v pgdata:/source:ro \
  -v $(pwd):/backup \
  alpine tar czf /backup/pgdata-backup.tar.gz -C /source .

# Restaurar un volume desde backup
docker run --rm \
  -v pgdata:/target \
  -v $(pwd):/backup:ro \
  alpine tar xzf /backup/pgdata-backup.tar.gz -C /target

# Copiar datos entre volumes usando un container temporal
docker run --rm \
  -v source-vol:/from:ro \
  -v dest-vol:/to \
  alpine cp -a /from/. /to/

# Inspeccionar dónde está un volume en el host
docker volume inspect pgdata
# Mountpoint: /var/lib/docker/volumes/pgdata/_data
```

---

### Volumes en Docker Compose

```yaml
services:
  db:
    image: postgres:18-alpine
    volumes:
      # Named volume
      - pgdata:/var/lib/postgresql/data

      # Bind mount — código fuente en desarrollo
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql:ro

      # tmpfs para archivos temporales
      - type: tmpfs
        target: /tmp

  app:
    build: .
    volumes:
      # Bind mount del código (hot reload en desarrollo)
      - .:/app
      # Volume anónimo para node_modules (no sobreescribir con bind mount)
      - /app/node_modules

volumes:
  pgdata:
    driver: local

  # Volume externo (ya creado con docker volume create)
  existing-data:
    external: true
    name: my-existing-volume

  # Volume con opciones del driver
  nfs-volume:
    driver: local
    driver_opts:
      type: nfs
      o: "addr=192.168.1.100,rw"
      device: ":/exports/data"
```

---

## 🔐 Docker Secrets

Los secrets son la forma segura de pasar contraseñas, tokens y certificados a containers. A diferencia de las variables de entorno, no aparecen en `docker inspect` ni en los logs.

```bash
# Crear secret desde stdin
echo "mi_password_seguro" | docker secret create db_password -

# Crear secret desde archivo
docker secret create ssl_cert ./cert.pem
docker secret create app_config ./config.json

# Listar y eliminar secrets
docker secret ls
docker secret rm db_password
docker secret inspect db_password   # Muestra metadata, NUNCA el valor

# Los secrets se montan en /run/secrets/<nombre> dentro del container
# Solo disponibles en Docker Swarm para containers individuales
# En Compose v3+ se puede usar con deploy o localmente para desarrollo
```

```yaml
# docker-compose.yml con secrets (Compose v3.1+)
services:
  db:
    image: postgres:18-alpine
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password   # Leer desde archivo
    secrets:
      - db_password

  api:
    build: .
    secrets:
      - db_password
      - jwt_secret
    # En el container: cat /run/secrets/db_password

secrets:
  # Secret gestionado por Docker Swarm
  db_password:
    external: true

  # Secret desde archivo local (solo para desarrollo)
  jwt_secret:
    file: ./secrets/jwt_secret.txt
```

```python
# Leer secret en la aplicación
import os

def get_secret(secret_name: str) -> str:
    secret_path = f"/run/secrets/{secret_name}"
    if os.path.exists(secret_path):
        with open(secret_path) as f:
            return f.read().strip()
    # Fallback a variable de entorno en desarrollo
    return os.environ.get(secret_name.upper(), "")

db_password = get_secret("db_password")
```

---

## 🔬 Docker Scout — análisis de vulnerabilidades

Docker Scout viene integrado en la CLI (`docker scout`) y analiza imágenes en busca de CVEs conocidos, comparando contra la base de datos de vulnerabilidades y sugiriendo actualizaciones de la imagen base.

```bash
# Resumen rápido de una imagen (vulnerabilidades + recomendaciones de base image)
docker scout quickview my-image:latest

# Listado detallado de CVEs
docker scout cves my-image:latest
docker scout cves --only-severity critical,high my-image:latest

# Comparar dos imágenes (ej: antes/después de un cambio)
docker scout compare my-image:latest --to my-image:previous

# Recomendaciones de imagen base más segura/liviana
docker scout recommendations my-image:latest
```

---

## 🏗️ Multi-stage builds

Los multi-stage builds permiten separar el entorno de compilación del de producción, resultando en imágenes finales mucho más pequeñas y seguras.

```dockerfile
# Ejemplo: aplicación Go — de ~800MB a ~15MB
FROM golang:1.23-alpine AS builder
WORKDIR /build
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-w -s" -o app .

# Imagen final — solo el binario
FROM scratch                                # Imagen vacía — la más pequeña posible
COPY --from=builder /build/app /app
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
ENTRYPOINT ["/app"]


# Ejemplo: Node.js — de ~1.2GB a ~200MB
FROM node:24-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:24-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:24-alpine AS production
WORKDIR /app
ENV NODE_ENV=production
# Solo copiar dependencias de producción y el build
COPY --from=deps /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package.json .
USER node
EXPOSE 3000
CMD ["node", "dist/main.js"]


# Ejemplo: Java Spring Boot — con layer cache
FROM eclipse-temurin:25-jdk-alpine AS builder
WORKDIR /build
COPY mvnw .
COPY .mvn .mvn
COPY pom.xml .
RUN ./mvnw dependency:go-offline -q          # Cachear dependencias
COPY src ./src
RUN ./mvnw package -DskipTests -q

# Extraer layers para mejor cache en deploys sucesivos
# El jarmode "layertools" fue reemplazado por "tools" (con subcomando extract)
FROM eclipse-temurin:25-jdk-alpine AS extractor
WORKDIR /build
COPY --from=builder /build/target/*.jar app.jar
RUN java -Djarmode=tools -jar app.jar extract --layers --destination extracted

FROM eclipse-temurin:25-jre-alpine AS production  # JRE es más pequeño que JDK
WORKDIR /app
RUN addgroup -S spring && adduser -S spring -G spring
USER spring
# Copiar layers en orden de menor a mayor frecuencia de cambio
COPY --from=extractor /build/extracted/dependencies .
COPY --from=extractor /build/extracted/spring-boot-loader .
COPY --from=extractor /build/extracted/snapshot-dependencies .
COPY --from=extractor /build/extracted/application .
EXPOSE 8080
# El layer "application" extraído ya es un jar ejecutable — ya no hace falta invocar JarLauncher a mano
ENTRYPOINT ["sh", "-c", "java -jar application.jar"]


# Build selectivo de un stage específico (útil para CI)
docker build --target builder -t myapp:builder .
docker build --target production -t myapp:latest .

# Pasar ARG entre stages
ARG VERSION=dev
FROM node:24-alpine AS builder
ARG VERSION
RUN echo "Building version: $VERSION"

docker build --build-arg VERSION=1.2.3 -t myapp:1.2.3 .
```

---

## 📝 .dockerignore

El `.dockerignore` controla qué archivos se envían al daemon de Docker durante el build. Un contexto de build grande ralentiza todos los builds incluso si esos archivos no se usan.

```dockerignore
# Control de versiones
.git
.gitignore
.gitattributes

# Dependencias — se instalan dentro del container
node_modules
vendor
.venv
__pycache__
*.pyc
*.pyo

# Build outputs
dist
build
out
target
*.jar
*.war

# Variables de entorno y secretos — NUNCA en la imagen
.env
.env.*
*.pem
*.key
*.cert
secrets/

# IDE y herramientas de desarrollo
.idea
.vscode
*.swp
*.swo
.DS_Store
Thumbs.db

# Logs
*.log
logs/
npm-debug.log*

# Tests — no necesarios en producción
coverage/
.coverage
*.test.js
__tests__
spec/
test/

# Docker files (no incluir dentro de sí mismos)
Dockerfile*
docker-compose*.yml
.dockerignore

# Documentación
README.md
docs/
*.md

# CI/CD
.github
.gitlab-ci.yml
.travis.yml
Jenkinsfile

# Verificar qué se incluye en el contexto
# docker build --no-cache . 2>&1 | grep "Sending build context"
```

---

## 📊 Logging drivers

```bash
# Ver logs con opciones
docker logs my-container
docker logs --tail 100 my-container          # Últimas 100 líneas
docker logs --since 30m my-container         # Últimos 30 minutos
docker logs --since "2026-01-15T10:00:00" my-container
docker logs -f --timestamps my-container     # Stream con timestamps

# Configurar logging driver al correr un container
docker run \
  --log-driver json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  my-image

# Logging drivers disponibles
# json-file   → default, archivos JSON en el host
# syslog      → syslog del sistema
# journald    → systemd journal
# fluentd     → enviar a Fluentd/Fluentbit
# awslogs     → CloudWatch Logs
# gcplogs     → Google Cloud Logging
# splunk      → Splunk HTTP Event Collector
# none        → deshabilitar logs completamente
```

```yaml
# Logging en Docker Compose
services:
  api:
    image: my-api
    logging:
      driver: json-file
      options:
        max-size: "10m"      # Rotar al llegar a 10MB
        max-file: "3"        # Mantener máximo 3 archivos
        compress: "true"     # Comprimir archivos rotados

  # Enviar logs a Loki (stack de observabilidad moderno)
  app:
    image: my-app
    logging:
      driver: loki
      options:
        loki-url: "http://loki:3100/loki/api/v1/push"
        loki-labels: "job=my-app,env=production"

# Configuración global del daemon en /etc/docker/daemon.json
# {
#   "log-driver": "json-file",
#   "log-opts": {
#     "max-size": "10m",
#     "max-file": "3"
#   }
# }
```

---

## 🔄 Límites de recursos

```bash
# CPU
docker run --cpus 1.5 my-image              # Máximo 1.5 CPUs
docker run --cpu-shares 512 my-image        # Peso relativo (default: 1024)
docker run --cpuset-cpus "0,1" my-image     # Solo usar CPUs 0 y 1

# Memoria
docker run --memory 512m my-image           # Límite de RAM
docker run --memory 512m --memory-swap 1g my-image   # RAM + swap
docker run --memory 512m --memory-swap -1 my-image   # RAM limitada, swap ilimitado
docker run --memory-reservation 256m my-image        # Límite blando (suave)

# PID — prevenir fork bombs
docker run --pids-limit 100 my-image

# Ver uso de recursos en tiempo real
docker stats
docker stats --no-stream                    # Snapshot único (sin stream)
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"
```

```yaml
# Límites en Docker Compose (Compose v3 con deploy, o directo sin Swarm)
services:
  api:
    image: my-api
    # Límites directos (sin Swarm — Compose v2 syntax)
    mem_limit: 512m
    memswap_limit: 1g
    cpus: 1.5
    pids_limit: 100

    # Con deploy (requiere Swarm o --compatibility flag)
    deploy:
      resources:
        limits:
          cpus: "1.5"
          memory: 512M
          pids: 100
        reservations:
          cpus: "0.5"
          memory: 256M
```

---

## 🌍 Variables de entorno

```bash
# Pasar variable individual
docker run -e NODE_ENV=production my-image
docker run -e DB_HOST -e DB_PORT my-image    # Tomar valor del host

# Pasar desde archivo
docker run --env-file .env my-image
docker run --env-file .env.production my-image
```

```bash
# .env — archivo de variables (NO commitear si tiene secretos)
NODE_ENV=production
DB_HOST=postgres
DB_PORT=5432
DB_NAME=myapp
DB_USER=admin
DB_PASSWORD=supersecret
REDIS_URL=redis://redis:6379
JWT_SECRET=my-jwt-secret
API_PORT=3000
```

```yaml
# Variables de entorno en Docker Compose
services:
  api:
    image: my-api
    environment:
      # Valor fijo
      NODE_ENV: production
      PORT: "3000"
      # Interpolación desde el .env del host o shell
      DB_PASSWORD: ${DB_PASSWORD}
      API_KEY: ${API_KEY:-default-key}       # Con valor por defecto
      DEBUG: ${DEBUG:-false}

    # Cargar desde archivo
    env_file:
      - .env                                 # Base
      - .env.${ENVIRONMENT:-development}     # Override por ambiente

  db:
    image: postgres:18-alpine
    environment:
      POSTGRES_DB: ${DB_NAME:-myapp}
      POSTGRES_USER: ${DB_USER:-admin}
      POSTGRES_PASSWORD: ${DB_PASSWORD}

# El .env en el mismo directorio que docker-compose.yml
# se carga automáticamente para la interpolación ${VAR}
# pero NO se pasa a los containers a menos que uses env_file o environment
```

---

## 🏗️ Docker Buildx — multi-arquitectura

```bash
# Buildx permite crear imágenes para múltiples arquitecturas en un solo comando
# Necesario para: imágenes que corran en Apple Silicon (ARM) y servidores (AMD64)

# Ver builders disponibles
docker buildx ls

# Crear y activar un builder con soporte multi-plataforma
docker buildx create --name multiarch --driver docker-container --bootstrap --use

# Build para múltiples plataformas y push directo al registry
docker buildx build \
  --platform linux/amd64,linux/arm64,linux/arm/v7 \
  --tag usuario/myapp:latest \
  --push \
  .

# Build solo para la plataforma actual (más rápido en desarrollo)
docker buildx build --platform linux/amd64 -t myapp:latest --load .
# --push  → subir al registry
# --load  → cargar en Docker local (solo una plataforma)

# Inspeccionar qué plataformas soporta una imagen
docker buildx imagetools inspect ubuntu:26.04

# Build con caché en registry (acelera CI/CD)
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --cache-from type=registry,ref=usuario/myapp:buildcache \
  --cache-to type=registry,ref=usuario/myapp:buildcache,mode=max \
  --tag usuario/myapp:latest \
  --push \
  .

# Emulación QEMU — correr binarios ARM en x86 (para testing)
docker run --rm --platform linux/arm64 ubuntu:26.04 uname -m   # aarch64
```

---

## 🖥️ Docker Context — múltiples hosts

```bash
# Manejar múltiples entornos Docker (local, staging, producción)
# sin cambiar variables de entorno

# Ver contextos disponibles
docker context ls

# Crear contexto para servidor remoto (SSH)
docker context create production \
  --docker "host=ssh://usuario@servidor-prod.com"

docker context create staging \
  --docker "host=ssh://usuario@servidor-staging.com"

# Cambiar de contexto
docker context use production
docker ps                                   # Muestra containers del servidor remoto

# Volver al contexto local
docker context use default

# Usar contexto para un comando específico sin cambiar el default
docker --context production ps
docker --context staging logs my-container -f

# Exportar/importar contexto (para compartir con el equipo)
docker context export production > production-context.tar
docker context import production production-context.tar

# Eliminar contexto
docker context rm staging
```

---

## 🐝 Docker Swarm — orquestación básica

```bash
# Inicializar un Swarm
docker swarm init --advertise-addr 192.168.1.10

# Obtener token para unir workers
docker swarm join-token worker
docker swarm join-token manager

# Unirse al Swarm desde otro host
docker swarm join --token SWMTKN-... 192.168.1.10:2377

# Ver nodos del cluster
docker node ls
docker node inspect my-node
docker node update --availability drain my-node    # Drenar antes de mantenimiento
docker node update --availability active my-node

# Desplegar un stack (equivalente a docker-compose up en Swarm)
docker stack deploy -c docker-compose.yml my-stack
docker stack ls
docker stack ps my-stack                           # Ver tasks del stack
docker stack services my-stack
docker stack rm my-stack

# Gestionar servicios
docker service ls
docker service ps my-stack_api                     # Ver en qué nodo corre
docker service logs my-stack_api -f
docker service scale my-stack_api=5                # Escalar a 5 réplicas
docker service update --image myapp:v2 my-stack_api  # Rolling update
docker service rollback my-stack_api               # Rollback al estado anterior
docker service rm my-stack_api
```

```yaml
# docker-compose.yml optimizado para Swarm
services:
  api:
    image: myapp:latest                  # Swarm requiere imagen, no build
    deploy:
      replicas: 3
      update_config:
        parallelism: 1                   # Actualizar 1 réplica a la vez
        delay: 10s                       # Esperar 10s entre actualizaciones
        failure_action: rollback         # Rollback automático si falla
        order: start-first               # Iniciar nueva antes de detener la vieja
      rollback_config:
        parallelism: 1
        delay: 5s
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
      resources:
        limits:
          cpus: "0.5"
          memory: 256M
        reservations:
          cpus: "0.25"
          memory: 128M
      placement:
        constraints:
          - node.role == worker           # Solo en nodos worker
          - node.labels.type == api
    secrets:
      - db_password
    networks:
      - backend

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    deploy:
      replicas: 2
      placement:
        constraints:
          - node.role == manager
    networks:
      - backend
      - frontend

networks:
  backend:
    driver: overlay
    encrypted: true                      # Cifrar tráfico entre nodos
  frontend:
    driver: overlay

secrets:
  db_password:
    external: true
```

---

## 🔧 Docker Registry

```bash
# Docker Hub
docker login
docker login -u usuario -p password
docker logout

# Taggear y subir imagen
docker tag myapp:latest usuario/myapp:latest
docker tag myapp:latest usuario/myapp:1.2.3
docker push usuario/myapp:latest
docker push usuario/myapp:1.2.3

# Subir todas las tags de una imagen
docker push usuario/myapp --all-tags

# GitHub Container Registry (ghcr.io)
echo $GITHUB_TOKEN | docker login ghcr.io -u USERNAME --password-stdin
docker tag myapp:latest ghcr.io/username/myapp:latest
docker push ghcr.io/username/myapp:latest

# Registry privado propio
docker run -d \
  -p 5000:5000 \
  --name registry \
  -v registry-data:/var/lib/registry \
  -e REGISTRY_STORAGE_DELETE_ENABLED=true \
  registry:2

# Subir al registry local
docker tag myapp:latest localhost:5000/myapp:latest
docker push localhost:5000/myapp:latest
docker pull localhost:5000/myapp:latest

# Listar imágenes en registry local
curl http://localhost:5000/v2/_catalog
curl http://localhost:5000/v2/myapp/tags/list

# Eliminar imagen del registry (requiere REGISTRY_STORAGE_DELETE_ENABLED=true)
# 1. Obtener digest
DIGEST=$(curl -s -H "Accept: application/vnd.docker.distribution.manifest.v2+json" \
  http://localhost:5000/v2/myapp/manifests/latest \
  -I | grep Docker-Content-Digest | awk '{print $2}')
# 2. Eliminar por digest
curl -X DELETE "http://localhost:5000/v2/myapp/manifests/$DIGEST"
```
