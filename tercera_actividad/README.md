## DOCKER COMPOSER TERCERA ACTIVIDAD

Este repositorio contiene un archivo `docker-compose.yml` que define una aplicación postgres y que se puede implementar desde docker .

## Servicios

Correciones que se mejoraron y se implementaron

### 1. Servicio de Aplicación (`app`)
- **Imagen**: `nginx:latest`
- **Puertos**: Mapea el puerto `8080` en el host al puerto `80` en el contenedor.
- **Variables de Entorno**:
  - `NODE_ENV`: Establecido en `production`.
- **Dependencias**: Depende del servicio `database`.
- **Red**: Conectado a la red personalizada `app_network`.

### 2. Servicio de Base de Datos (`database`)
- **Imagen**: `postgres:alpine`
- **Volúmenes**:
  - `pg_data`: Mapeado a `/var/lib/postgresql/data` en el contenedor para almacenamiento persistente.
- **Variables de Entorno**:
  - `POSTGRES_USER`: `admin`
  - `POSTGRES_PASSWORD`: `password123`
  - `POSTGRES_DB`: `my_database`
- **Puertos**: Mapea el puerto `5432` en el host al puerto `5432` en el contenedor.
- **Red**: Conectado a la red personalizada `app_network`.

### 3. Servicio de Redis (`redis`)
- **Imagen**: `redis`
- **Comando**: Configura Redis para requerir una contraseña usando `--requirepass "redispass123"`.
- **Puertos**: Mapea el puerto `6379` en el host al puerto `6379` en el contenedor.

## Volúmenes
- **`pg_data`**: Proporciona almacenamiento persistente para la base de datos Postgres.

## Redes
- **`app_network`**: Una red de puente personalizada que permite la comunicación entre los servicios.

## Sugerencias para Mejorar

### 1. Gestión de Variables de Entorno
- Usa un archivo `.env` para almacenar credenciales sensibles (por ejemplo, `POSTGRES_PASSWORD`, contraseña de Redis) en lugar de escribirlas directamente en el archivo `docker-compose.yml`.

### 2. Comprobaciones de Salud
- Agrega secciones de `healthcheck` para monitorear la disponibilidad de los servicios. Por ejemplo, el servicio de base de datos puede incluir:
  ```yaml
  healthcheck:
    test: ["CMD", "pg_isready", "-U", "admin"]
    interval: 30s
    timeout: 10s
    retries: 5
```