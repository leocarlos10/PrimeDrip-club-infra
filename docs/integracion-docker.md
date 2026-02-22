---

# 🐳 Infraestructura Docker – Prime Drip Club

Arquitectura dockerizada para:

* ⚛️ Frontend con **Bun**
* ☕ Backend con **Spring Boot 3 + Java 21**
* 🐬 MySQL 8
* 🛫 Flyway para migraciones
* 🔐 Spring Security + JWT

---

# 🏗 Estructura del Proyecto

```
prime-drip-club/
│
├── backend/
│   ├── Dockerfile
│   └── (Spring Boot project)
│
├── frontend/
│   ├── Dockerfile
│   └── (Bun + React)
│
└── docker-compose.yml
```

---

# 🧱 1️⃣ Base de Datos – MySQL

## Servicio en `docker-compose.yml`

```yaml
version: "3.9"

services:
  prime-db:
    image: mysql:8.0.45
    restart: unless-stopped
    environment:
       MYSQL_DATABASE: primedrip_club_db
       MYSQL_ROOT_PASSWORD: root
       MYSQL_USER: NeoCode
       MYSQL_PASSWORD: NeoCode1005;
    ports:
      - "3306:3306"
    volumes:
      - prime-db-data:/var/lib/mysql
```

## Volumen Persistente

```yaml
volumes:
  prime-db-data:
```

📌 MySQL guardará los datos aunque el contenedor se elimine.

---

# 🧱 2️⃣ Backend – Spring Boot + Java 21

Tu proyecto usa:

```xml
<java.version>21</java.version>
```

Usamos imágenes compatibles con Java 21.

---

## 📄 Dockerfile (`/backend/Dockerfile`)

```dockerfile
# Build stage
FROM maven:3.9-eclipse-temurin-21 AS build

WORKDIR /app

COPY pom.xml .
COPY src ./src

RUN mvn clean package -DskipTests

# Runtime stage
FROM eclipse-temurin:21-jdk

WORKDIR /app

COPY --from=build /app/target/*.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

### 🔥 ¿Por qué multistage build?

- Reduce tamaño de la imagen final
- No incluye Maven en producción
- Más profesional

---

# 📦 Explicación paso a paso del Dockerfile (Spring Boot + Maven)

Este Dockerfile utiliza **multi-stage build** para compilar y ejecutar una aplicación Java optimizando el tamaño final de la imagen.

---

## 🏗️ Etapa 1: Build (Compilación)

```dockerfile
FROM maven:3.9-eclipse-temurin-21 AS build
```

* Usa la imagen oficial de **Apache Maven** con Java 21.
* Incluye todas las herramientas necesarias para compilar el proyecto.
* Se le asigna el nombre `build` a esta etapa para poder referenciarla después.

---

```dockerfile
WORKDIR /app
```

* Define `/app` como directorio de trabajo dentro del contenedor.
* Si la carpeta no existe, Docker la crea automáticamente.

---

```dockerfile
COPY pom.xml .
```

* Copia el archivo `pom.xml` al contenedor.
* Este archivo contiene las dependencias y configuración del proyecto.

---

```dockerfile
COPY src ./src
```

* Copia el código fuente de la aplicación al contenedor.

---

```dockerfile
RUN mvn clean package -DskipTests
```

* Ejecuta el comando de Maven para:

  * Limpiar compilaciones anteriores
  * Descargar dependencias
  * Compilar el proyecto
  * Generar el archivo `.jar`
* La opción `-DskipTests` evita ejecutar pruebas para acelerar el build.

Resultado generado:

```
/app/target/miapp.jar
```

Aquí termina la etapa de compilación.

---

# 🚀 Etapa 2: Runtime (Ejecución)

```dockerfile
FROM eclipse-temurin:21-jdk
```

* Usa una imagen más ligera con solo Java.
* No incluye Maven ni herramientas de compilación.
* Reduce el tamaño final y mejora la seguridad.

---

```dockerfile
WORKDIR /app
```

* Define nuevamente el directorio de trabajo.

---

```dockerfile
COPY --from=build /app/target/*.jar app.jar
```

* Copia el archivo `.jar` generado en la etapa `build`.
* `--from=build` indica que el archivo proviene de la primera etapa.
* Se renombra como `app.jar`.

---

```dockerfile
EXPOSE 8080
```

* Indica que la aplicación escuchará en el puerto 8080.
* Es solo informativo (no publica el puerto automáticamente).

---

```dockerfile
ENTRYPOINT ["java", "-jar", "app.jar"]
```

* Define el comando que se ejecutará al iniciar el contenedor.
* Lanza la aplicación Java.

---

# 🧠 ¿Por qué usar multi-stage build?

Ventajas:

* ✅ Imagen final más pequeña
* ✅ No incluye Maven en producción
* ✅ Más segura
* ✅ Separación clara entre compilación y ejecución
* ✅ Mejor práctica para producción

---

# 📌 Flujo completo

1. Docker usa Maven para compilar el proyecto.
2. Genera el archivo `.jar`.
3. Crea una nueva imagen limpia con solo Java.
4. Copia el `.jar`.
5. Ejecuta la aplicación.

---

# 🎯 Resumen conceptual

| Etapa     | Función                                 |
| --------- | --------------------------------------- |
| build     | Compilar la aplicación                  |
| runtime   | Ejecutar la aplicación                  |
| resultado | Imagen optimizada lista para producción |

---

## Servicio Backend en docker-compose

```yaml
 backend:
    build: ./backend
    restart: unless-stopped
    ports:
      - "8080:8080"
    depends_on:
      - prime-db
    environment:
      - SPRING_DATASOURCE_URL: jdbc:mysql://db:3306/prime_drip_club_db
      - SPRING_DATASOURCE_USERNAME: NeoCode
      - SPRING_DATASOURCE_PASSWORD: NeoCode1005;
      - SPRING_DATASOURCE_DRIVER_CLASS_NAME: com.mysql.cj.jdbc.Driver
```

⚠️ Importante:

Dentro de Docker, el host de la base de datos NO es `localhost`.

Es el nombre del servicio:

```
jdbc:mysql://db:3306/prime_drip_club
```

---

# 🛫 Flyway

Ya tienes en tu `pom.xml`:

```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>

<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-mysql</artifactId>
</dependency>
```

Si tus migraciones están en:

```
src/main/resources/db/migration
```

Spring Boot ejecutará automáticamente las migraciones cuando inicie el contenedor.

No necesitas un contenedor adicional.

---

# 🧱 3️⃣ Frontend – React + Bun

Como el frontend está hecho con **Bun**, usamos la imagen oficial.

---

## 📄 Dockerfile (`/frontend/Dockerfile`)

```dockerfile
# 🏗 Etapa 1 - Build
FROM oven/bun:1 AS build

WORKDIR /app

COPY package.json bun.lockb ./
RUN bun install

COPY . .
RUN bun run build

# 🚀 Etapa 2 - Servidor Web
FROM nginx:alpine

COPY --from=build /app/dist /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]

```

---

## Servicio Frontend

```yaml
frontend:
  build: ./frontend
  restart: unless-stopped
  ports:
    - "80:80"
  depends_on:
    - backend
```

---

# 📦 docker-compose.yml Completo

```yaml
version: "3.9"

services:
  db:
    image: mysql:8.3
    container_name: prime-db
    restart: always
    environment:
      MYSQL_DATABASE: prime_drip_club
      MYSQL_ROOT_PASSWORD: root
      MYSQL_USER: prime
      MYSQL_PASSWORD: secret
    ports:
      - "3306:3306"
    volumes:
      - db-data:/var/lib/mysql

  backend:
    build: ./backend
    container_name: prime-backend
    restart: always
    ports:
      - "8080:8080"
    depends_on:
      - db
    environment:
      SPRING_DATASOURCE_URL: jdbc:mysql://db:3306/prime_drip_club
      SPRING_DATASOURCE_USERNAME: prime
      SPRING_DATASOURCE_PASSWORD: secret

  frontend:
    build: ./frontend
    container_name: prime-frontend
    restart: always
    ports:
      - "3000:3000"
    depends_on:
      - backend

volumes:
  db-data:
```

---

# 🚀 Cómo Levantar el Proyecto

Desde la raíz:

```bash
docker compose up --build
```

Accesos:

- Frontend → [http://localhost:3000](http://localhost:3000)
- Backend → [http://localhost:8080](http://localhost:8080)
- MySQL → interno dentro de Docker

---

# 🧠 Consideraciones Importantes

## 🔹 1. Cambiar URLs del frontend

Si el frontend consume:

```
http://localhost:8080
```

Dentro de Docker debe usar:

```
http://backend:8080
```

Porque `backend` es el nombre del servicio.

---

