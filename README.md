

# 📦 WebPrimeDripClub – Guía de Instalación

Este repositorio orquesta el entorno completo del proyecto:

* Backend (Spring Boot)
* Frontend (React + TypeScript)
* Base de datos
* Servicios necesarios vía Docker

---

# 🖥️ Requisitos Previos

Antes de comenzar, asegúrate de tener instalado:

* Docker
* Docker Compose (v2 recomendado)
* Git

Verifica:

```bash
docker --version
docker compose version
git --version
```

---

# 🚀 Instalación para Desarrollo Local

## 1️⃣ Clonar el repositorio de infraestructura

```bash
git clone https://github.com/tu-org/webPrimeDripClub.git
cd webPrimeDripClub
```

---

## 2️⃣ Clonar backend y frontend dentro del proyecto

```bash
git clone https://github.com/tu-org/backend.git backend
git clone https://github.com/tu-org/frontend.git frontend
```

La estructura final debe verse así:

```
webPrimeDripClub/
│
├── docker-compose.yml
├── docker-compose-dev.yml
├── backend/
└── frontend/
```

---

## 3️⃣ Configurar variables de entorno

Copiar el archivo de ejemplo:

```bash
cp .env.example .env
```

Editar `.env` según tu configuración local:

```
MYSQL_DATABASE=prime_drip_club
MYSQL_USER=prime
MYSQL_PASSWORD=secret
MYSQL_ROOT_PASSWORD=root

# JWT Configuration
JWT_SECRET_KEY=your_secret_key_here
JWT_EXPIRATION_TIME=86400000
```

---

## 4️⃣ Levantar el entorno de desarrollo

```bash
docker compose -f docker-compose-dev.yml up --build
```

Esto iniciará:

* Base de datos
* Backend
* Frontend

---

## 5️⃣ Acceso a la aplicación

* Frontend → [http://localhost:3000](http://localhost:3000)
* Backend → [http://localhost:8080](http://localhost:8080)
* Base de datos → puerto definido en docker-compose

---

# 🛑 Detener el entorno

```bash
ctrl + c
```

Eliminar containers

```bash
docker compose -f docker-compose-dev.yml down
```

Para eliminar volúmenes:

```bash
docker compose -f docker-compose-dev.yml down -v
```

---

**Última actualización:** 22 de febrero de 2026  
**Versión:** 1.0  
**Autor:** Equipo de Desarrollo de NEOCODE



