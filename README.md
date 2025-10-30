# Pets API

Este proyecto contiene una API REST desarrollada con Java y Spring Boot para gestionar un listado de mascotas. La aplicación está diseñada para ser desplegada fácilmente utilizando Docker y puede conectarse a diferentes bases de datos gracias a los perfiles de Spring.

Originalmente, este backend fue diseñado para ser consumido por un frontend simple en PHP que muestra el listado en una tabla.

## 🚀 Tecnologías Utilizadas

- **Backend**:
  - Java 21
  - Spring Boot 3.5.6
  - Spring Data JPA (Hibernate)
  - Maven
- **Bases de Datos Soportadas**:
  - MySQL 8.0 (para desarrollo local)
  - PostgreSQL (para el despliegue con Neon)
- **Documentación API**:
  - SpringDoc (Swagger UI)
- **Contenerización**:
  - Docker
  - Docker Compose
- **CI/CD**:
  - GitHub Actions (para publicar Javadoc en GitHub Pages)

## 📋 Prerrequisitos

- Java 21
- Maven
- Docker y Docker Compose
- Git

## ⚙️ Cómo Ejecutar el Proyecto (Paso a Paso)

Existen varias formas de levantar el proyecto, dependiendo de tus necesidades.

### Opción 1: Entorno Local con Docker Compose (Recomendado)

Esta es la forma más sencilla de levantar todo el entorno de desarrollo. Se levantará un contenedor para la aplicación y otro para la base de datos MySQL.

1.  **Clona el repositorio:**
    ```bash
    git clone https://github.com/tu-usuario/pets-app.git
    cd pets-app
    ```

2.  **Asegúrate de tener los scripts de Maven:**
    Si el archivo `mvnw` no existe o no tiene permisos de ejecución, puedes generarlo con:
    ```bash
    mvn -N io.takari:maven:wrapper
    # Otorga permisos de ejecución (importante en Linux/macOS)
    chmod +x mvnw
    ```

3.  **Levanta los contenedores:**
    Utiliza el archivo `docker-compose.local.yml` para iniciar los servicios.
    ```bash
    docker-compose -f docker-compose.local.yml up --build
    ```
    El flag `--build` asegura que la imagen de Docker se reconstruya si has hecho cambios en el código.

4.  **¡Listo!** La aplicación estará disponible en `http://localhost:8080`.

### Opción 2: Conexión a una Base de Datos Neon (PostgreSQL en la nube)

Este método es ideal para un entorno de *staging* o para probar la configuración de producción.

1.  **Crea un archivo `.env`** en la raíz del proyecto con tus credenciales de Neon.
    ```env
    # c:\Users\canci\Documents\GitHub\pets-app\.env
    NEON_DATABASE_URL=jdbc:postgresql://<tu-host>.neon.tech/neondb?sslmode=require
    NEON_USERNAME=<tu-usuario>
    NEON_PASSWORD=<tu-contraseña>
    ```
    **Importante:** Añade el archivo `.env` a tu `.gitignore` para no subir tus credenciales al repositorio.

2.  **Levanta el contenedor de la aplicación:**
    ```bash
    docker-compose -f docker-compose.neon.yml up --build
    ```
    Docker Compose leerá las variables del archivo `.env` y las pasará al contenedor de la aplicación.

### Opción 3: Ejecución Nativa (Sin Docker para la App)

Puedes ejecutar la aplicación directamente en tu máquina y conectarla a una base de datos (ya sea local o en un contenedor).

1.  **Inicia una base de datos MySQL.** Puedes usar Docker para ello:
    ```bash
    docker run -d -p 3306:3306 --name mysql-db -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=pets -e MYSQL_USER=user -e MYSQL_PASSWORD=password mysql:8.0
    ```

2.  **Ejecuta la aplicación Spring Boot** desde tu IDE o usando Maven. Las propiedades en `application-local.properties` están configuradas para conectarse a `localhost:3306`.
    ```bash
    ./mvnw spring-boot:run -Dspring-boot.run.profiles=local
    ```

## 📚 Documentación de la API

Una vez que la aplicación está en ejecución, puedes acceder a la interfaz de Swagger para ver y probar los endpoints disponibles:

**URL de Swagger UI:** http://localhost:8080/swagger-ui.html

## 🔄 Integración Continua (CI)

El proyecto está configurado con GitHub Actions (`.github/workflows/javadoc.yml`) para generar y desplegar automáticamente la documentación Javadoc del proyecto en GitHub Pages cada vez que se hace un `push` a la rama `main`.
