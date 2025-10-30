# Pets API

Este proyecto es una API REST desarrollada con Java y Spring Boot para la gestión de mascotas. La aplicación está contenerizada con Docker y configurada para ejecutarse en múltiples entornos (desarrollo local y nube) mediante el uso de perfiles de Spring.

## 🚀 Tecnologías Utilizadas
- **Backend**: Java 21, Spring Boot 3.5, Spring Data JPA
- **Build**: Maven
- **Bases de Datos**: MySQL (local), PostgreSQL (Neon)
- **Contenerización**: Docker, Docker Compose
- **Documentación**: SpringDoc (Swagger UI), Javadoc
- **CI/CD**: GitHub Actions

## ⚙️ Comandos Básicos de Ejecución

A continuación se describen los comandos para levantar la aplicación según el entorno deseado.

### 1. Entorno Local (con Docker y MySQL)

Este comando levanta la aplicación y una base de datos MySQL. Es el método recomendado para el desarrollo.

```bash
# Desde la raíz del proyecto
docker-compose -f docker-compose.local.yml up --build
```

La aplicación estará disponible en `http://localhost:8080`.

### 2. Entorno de Nube (con Docker y Neon DB)

Este comando levanta la aplicación y la conecta a una base de datos PostgreSQL en Neon.

**Prerrequisito:** Debes crear un archivo `.env` en la raíz del proyecto con tus credenciales.

**`.env` file:**
```
NEON_DATABASE_URL=jdbc:postgresql://<tu-host>.neon.tech/neondb?sslmode=require
NEON_USERNAME=<tu-usuario>
NEON_PASSWORD=<tu-contraseña>
```

**Comando de ejecución:**
```bash
# Desde la raíz del proyecto
docker-compose -f docker-compose.neon.yml up --build
```

### 3. Ejecución Nativa (sin Docker para la app)

Puedes ejecutar la aplicación directamente con Maven, conectándola a una base de datos que ya esté corriendo.

```bash
# Activa el perfil 'local' para conectar a una BD MySQL en localhost:3306
./mvnw spring-boot:run -Dspring-boot.run.profiles=local
```

## ✅ Verificación del Perfil Activo

Cuando la aplicación Spring Boot se inicia, imprime en la consola el perfil que está utilizando. Para verificarlo, busca en los logs de inicio una línea similar a esta:

```
... : The following 1 profile is active: "local"
```

O, si estás usando el perfil de Neon:

```
... : The following 1 profile is active: "neon"
```

Esto confirma que la aplicación ha cargado la configuración correcta del fichero `application-<perfil>.properties` correspondiente.
