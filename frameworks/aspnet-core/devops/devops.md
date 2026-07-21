# ASP.NET Core — DevOps

---

## 🐳 Docker

```dockerfile
# Dockerfile — multi-stage build
FROM mcr.microsoft.com/dotnet/sdk:10.0-alpine AS build
WORKDIR /src

# Descargar dependencias primero (mejor cache)
COPY ["MiApi/MiApi.csproj", "MiApi/"]
RUN dotnet restore "MiApi/MiApi.csproj"

# Compilar
COPY . .
WORKDIR /src/MiApi
RUN dotnet build -c Release -o /app/build

# Publicar
FROM build AS publish
RUN dotnet publish -c Release -o /app/publish \
    --no-restore /p:UseAppHost=false

# Imagen final — runtime (más pequeña que el SDK)
FROM mcr.microsoft.com/dotnet/aspnet:10.0-alpine AS final
WORKDIR /app

# Usuario sin privilegios
RUN addgroup -S dotnet && adduser -S dotnet -G dotnet
USER dotnet

COPY --from=publish /app/publish .

ENV ASPNETCORE_ENVIRONMENT=Production
ENV ASPNETCORE_URLS=http://+:8080
ENV DOTNET_EnableDiagnostics=0

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
    CMD wget -qO- http://localhost:8080/health || exit 1

ENTRYPOINT ["dotnet", "MiApi.dll"]
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
      ASPNETCORE_ENVIRONMENT: Development
      ConnectionStrings__Default: "Host=db;Port=5432;Database=mi_db;Username=postgres;Password=postgres"
      ConnectionStrings__Redis: "redis:6379"
      Jwt__Secret: "dev-secret-key-only-for-development-256bits"
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
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
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3
    networks:
      - backend

volumes:
  pgdata:

networks:
  backend:
    driver: bridge
```

