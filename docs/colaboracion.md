

---

# 👥 Colaboración con Repositorios Separados (Arquitectura Infra)

## 📌 Contexto

El proyecto **Prime Drip Club** está dividido en dos repositorios independientes:

- 📦 `prime-drip-club-backend` → Spring Boot + Java 21
- 📦 `prime-drip-club-frontend` → React + Bun

Para permitir que otros desarrolladores trabajen en el proyecto completo sin mezclar código, se implementa una tercera capa llamada:

> 🏗 **Repositorio de Infraestructura (Infra Repo)**

Este repositorio se encarga únicamente de:

- Orquestación con Docker Compose
- Configuración de entorno
- Documentación técnica

---

# 🏗 Arquitectura General

```
workspace/
│
├── prime-drip-club-backend/
├── prime-drip-club-frontend/
└──– prime-drip-club-infra/
```

Cada repositorio mantiene su independencia, pero el repo `infra` los orquesta.

---

# 📦 Repositorios

## 1️⃣ Backend Repository

Contiene:

- Código Spring Boot
- Flyway migrations
- Dockerfile
- Configuración de seguridad (JWT)

---

## 2️⃣ Frontend Repository

Contiene:

- Aplicación React + Bun
- Dockerfile
- Configuración de entorno

---

## 3️⃣ Infra Repository

Contiene:

```
prime-drip-club-infra/
│
├── docker-compose.yml
├── docker-compose-dev.yml
└── README.md
```

Este repositorio NO contiene código de aplicación.

Solo infraestructura.

---

# 🐳 docker-compose-dev.yml (Infra Repo)

Ejemplo de configuración:

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
    volumes:
      - db-data:/var/lib/mysql

  backend:
    build:
      context: ../prime-drip-club-backend
    container_name: prime-backend
    ports:
      - "8080:8080"
    depends_on:
      - db
    environment:
      SPRING_DATASOURCE_URL: jdbc:mysql://db:3306/prime_drip_club
      SPRING_DATASOURCE_USERNAME: prime
      SPRING_DATASOURCE_PASSWORD: secret

  frontend:
    build:
      context: ../prime-drip-club-frontend
    container_name: prime-frontend
    ports:
      - "3000:3000"
    depends_on:
      - backend

volumes:
  db-data:
```

---

# 🚀 Cómo levantar el proyecto completo

## 1️⃣ Clonar los tres repositorios

```bash
git clone https://github.com/tu-org/prime-drip-club-backend.git
git clone https://github.com/tu-org/prime-drip-club-frontend.git
git clone https://github.com/tu-org/prime-drip-club-infra.git
```

## 2️⃣ Organizar estructura

Todos deben quedar dentro de una misma carpeta raíz:

```
workspace/
```

## 3️⃣ Levantar servicios

Entrar al repositorio `infra`:

```bash
cd prime-drip-club-infra
docker compose -f docker-compose-dev up --build
```

---

# 🔐 Variables de Entorno (.env recomendado)

Crear un archivo `.env` dentro del repo infra:

```
MYSQL_DATABASE=prime_drip_club
MYSQL_USER=prime
MYSQL_PASSWORD=secret
MYSQL_ROOT_PASSWORD=root
```

Y modificar `docker-compose.yml`:

```yaml
environment:
  MYSQL_DATABASE: ${MYSQL_DATABASE}
  MYSQL_USER: ${MYSQL_USER}
  MYSQL_PASSWORD: ${MYSQL_PASSWORD}
  MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
```

Esto evita exponer credenciales directamente en el archivo.

---

# 🎯 Ventajas de esta Arquitectura

✅ Separación clara de responsabilidades
✅ Backend y frontend pueden desplegarse de forma independiente
✅ Escalable hacia microservicios
✅ Permite CI/CD independiente
✅ Facilita trabajo en equipo
✅ Arquitectura profesional

---

**Última actualización:** 22 de febrero de 2026  
**Versión:** 1.0  
**Autor:** Equipo de Desarrollo NECODE



