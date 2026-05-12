 Backend API - Infraestructura y Despliegue

Este repositorio contiene la lógica de negocio y la persistencia de datos de la aplicación, diseñada bajo una arquitectura de contenedores y desplegada de forma automatizada en AWS.

## 1. Explicación de la Arquitectura (IE6, IE7, IE8)
El sistema sigue un flujo de datos de N-capas:
- **Usuario** → Accede vía Navegador al Frontend.
- **Frontend (React)** → Realiza peticiones REST al Backend.
- **Backend (Node/Java)** → Procesa la lógica y consulta la base de datos.
- **Base de Datos (MySQL)** → Almacena la información de forma persistente.

**Infraestructura:**
- **EC2 Privada:** El backend reside en una instancia dentro de una subred privada, protegida de accesos externos directos.
- **Comunicación:** Se utiliza una red interna de Docker para conectar el Backend con MySQL.



## 2. Dockerización (IE1)
El proyecto utiliza un `Dockerfile` optimizado:
- **Imagen Base:** `node:20-alpine` (o la que uses) para reducir la superficie de ataque y el tamaño de la imagen.
- **Multi-stage Build:** Separamos la etapa de construcción (build) de la de ejecución para eliminar dependencias de desarrollo en producción.
- **Seguridad:** Se recomienda el uso de usuarios non-root para la ejecución del proceso principal.

## 3. Orquestación con Docker Compose (IE2, IE3)
El archivo `docker-compose.yml` gestiona dos servicios clave:
- **db:** Servidor MySQL 8.
- **backend:** La API que conecta con la DB.

**Persistencia (IE3):**
Se utiliza un **Named Volume** (`mysql_data`) para la base de datos. 
> *Justificación:* Esto garantiza que los datos no se pierdan si el contenedor se reinicia, se detiene o se actualiza la imagen.

## 4. Pipeline CI/CD (IE4)
El flujo de automatización con **GitHub Actions** es el siguiente:
1. **Trigger:** Push a la rama `main`.
2. **Build:** Creación de la imagen Docker.
3. **Push:** Envío de la imagen a **Docker Hub**.
4. **Deploy:** Conexión vía SSH a la **EC2** de AWS, descarga de la nueva imagen y reinicio de servicios con `docker-compose`.

## 5. Variables de Entorno
Es necesario configurar los siguientes Secrets en GitHub:
- `DOCKER_USERNAME`: Usuario de Docker Hub.
- `DOCKER_PASSWORD`: Token de acceso a Docker Hub.
- `EC2_HOST`: IP Pública de la instancia.
- `EC2_SSH_KEY`: Clave privada `.pem`.

## 6. Cómo ejecutar localmente
```bash
# Clonar el repo
git clone <URL_BACKEND>

# Levantar servicios
docker-compose up -d --build
