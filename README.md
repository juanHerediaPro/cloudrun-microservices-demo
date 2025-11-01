# Microservicios Backend - Java Spring Boot

Este proyecto contiene dos microservicios simples construidos con Java y Spring Boot, listos para desplegar en la nube con MySQL/Cloud SQL.

## Microservicios

### 1. User Service (Puerto 8081)
Gestiona usuarios con operaciones CRUD completas.

**Endpoints:**
- `GET /api/users` - Obtener todos los usuarios
- `GET /api/users/{id}` - Obtener usuario por ID
- `POST /api/users` - Crear nuevo usuario
- `PUT /api/users/{id}` - Actualizar usuario
- `DELETE /api/users/{id}` - Eliminar usuario

### 2. Product Service (Puerto 8082)
Gestiona productos con operaciones CRUD y búsqueda.

**Endpoints:**
- `GET /api/products` - Obtener todos los productos
- `GET /api/products/{id}` - Obtener producto por ID
- `GET /api/products/search?name={name}` - Buscar productos por nombre
- `POST /api/products` - Crear nuevo producto
- `PUT /api/products/{id}` - Actualizar producto
- `DELETE /api/products/{id}` - Eliminar producto

## Requisitos

- Java 17 o superior
- Maven 3.6+
- MySQL 8.0+ o Cloud SQL (MySQL)
- Docker (para construir imágenes)

## Base de Datos

Ambos microservicios se conectan a la misma base de datos MySQL. Necesitas crear la base de datos antes de ejecutar los servicios:

\`\`\`sql
CREATE DATABASE microservices_db;
\`\`\`

## Variables de Entorno

Configura estas variables de entorno para conectar a tu base de datos:

- `DB_HOST` - Host de la base de datos (default: localhost)
- `DB_PORT` - Puerto de MySQL (default: 3306)
- `DB_NAME` - Nombre de la base de datos (default: microservices_db)
- `DB_USER` - Usuario de MySQL (default: root)
- `DB_PASSWORD` - Contraseña de MySQL (default: password)

## Construcción de Imágenes Docker

### User Service
\`\`\`bash
cd user-service
docker build -t user-service:latest .
\`\`\`

### Product Service
\`\`\`bash
cd product-service
docker build -t product-service:latest .
\`\`\`

## Despliegue en la Nube

### Google Cloud Platform (Cloud Run + Cloud SQL)

1. **Crear instancia de Cloud SQL (MySQL)**
\`\`\`bash
gcloud sql instances create microservices-db \
  --database-version=MYSQL_8_0 \
  --tier=db-f1-micro \
  --region=us-central1
\`\`\`

2. **Crear base de datos**
\`\`\`bash
gcloud sql databases create microservices_db --instance=microservices-db
\`\`\`

3. **Construir y subir imágenes a Container Registry**
\`\`\`bash
# User Service
cd user-service
docker build -t gcr.io/[PROJECT-ID]/user-service:latest .
docker push gcr.io/[PROJECT-ID]/user-service:latest

# Product Service
cd product-service
docker build -t gcr.io/[PROJECT-ID]/product-service:latest .
docker push gcr.io/[PROJECT-ID]/product-service:latest
\`\`\`

4. **Desplegar en Cloud Run**
\`\`\`bash
# User Service
gcloud run deploy user-service \
  --image gcr.io/[PROJECT-ID]/user-service:latest \
  --platform managed \
  --region us-central1 \
  --add-cloudsql-instances [PROJECT-ID]:us-central1:microservices-db \
  --set-env-vars DB_HOST=/cloudsql/[PROJECT-ID]:us-central1:microservices-db \
  --set-env-vars DB_NAME=microservices_db \
  --set-env-vars DB_USER=root \
  --set-env-vars DB_PASSWORD=[YOUR-PASSWORD] \
  --allow-unauthenticated

# Product Service
gcloud run deploy product-service \
  --image gcr.io/[PROJECT-ID]/product-service:latest \
  --platform managed \
  --region us-central1 \
  --add-cloudsql-instances [PROJECT-ID]:us-central1:microservices-db \
  --set-env-vars DB_HOST=/cloudsql/[PROJECT-ID]:us-central1:microservices-db \
  --set-env-vars DB_NAME=microservices_db \
  --set-env-vars DB_USER=root \
  --set-env-vars DB_PASSWORD=[YOUR-PASSWORD] \
  --allow-unauthenticated
\`\`\`

### AWS (ECS + RDS)

1. **Crear instancia RDS MySQL**
2. **Subir imágenes a ECR**
3. **Crear Task Definitions con variables de entorno**
4. **Desplegar servicios en ECS**

### Azure (Container Apps + Azure Database for MySQL)

1. **Crear Azure Database for MySQL**
2. **Subir imágenes a Azure Container Registry**
3. **Crear Container Apps con variables de entorno**

## Ejecución Local

### Con Maven
\`\`\`bash
# User Service
cd user-service
mvn spring-boot:run

# Product Service
cd product-service
mvn spring-boot:run
\`\`\`

### Con Docker
\`\`\`bash
# User Service
docker run -p 8081:8081 \
  -e DB_HOST=host.docker.internal \
  -e DB_NAME=microservices_db \
  -e DB_USER=root \
  -e DB_PASSWORD=password \
  user-service:latest

# Product Service
docker run -p 8082:8082 \
  -e DB_HOST=host.docker.internal \
  -e DB_NAME=microservices_db \
  -e DB_USER=root \
  -e DB_PASSWORD=password \
  product-service:latest
\`\`\`

## Ejemplos de uso

### Crear un usuario
\`\`\`bash
curl -X POST http://localhost:8081/api/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Juan Pérez","email":"juan@example.com","phone":"123456789"}'
\`\`\`

### Crear un producto
\`\`\`bash
curl -X POST http://localhost:8082/api/products \
  -H "Content-Type: application/json" \
  -d '{"name":"Laptop","description":"Laptop gaming","price":1500.00,"stock":10}'
\`\`\`

## Notas de Seguridad

- En producción, usa secretos seguros para las contraseñas de base de datos
- Configura SSL/TLS para las conexiones a la base de datos
- Implementa autenticación y autorización según tus necesidades
- Usa Cloud SQL Proxy para conexiones seguras en GCP
