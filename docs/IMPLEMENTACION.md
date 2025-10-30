# Informe de Implementación y Despliegue: Pets API

## 1. Descripción General del Objetivo del Despliegue
El objetivo principal de este proyecto ha sido desarrollar y desplegar una API REST utilizando Spring Boot, siguiendo las mejores prácticas de la industria para la contenerización y la automatización. El despliegue se ha diseñado para ser flexible, soportando múltiples entornos a través de una configuración desacoplada y perfiles de Spring.

Los hitos clave del despliegue son:
- **Contenerización**: Empaquetar la aplicación Java en una imagen de Docker optimizada.
- **Orquestación Local**: Facilitar un entorno de desarrollo local completo y reproducible con Docker Compose, incluyendo la aplicación y una base de datos MySQL.
- **Configuración Multi-entorno**: Permitir que la misma imagen de la aplicación se conecte a diferentes bases de datos (MySQL local, PostgreSQL en la nube con Neon) sin necesidad de reconstruir la imagen.
- **Integración Continua (CI)**: Automatizar la generación y publicación de la documentación del proyecto (Javadoc) en GitHub Pages cada vez que se actualiza la rama principal.

## 2. Estructura Final del Proyecto
La estructura del proyecto se ha organizado para separar claramente el código fuente, la configuración, la documentación y los scripts de automatización.

```
pets-app/
├── .github/
│   └── workflows/
│       └── javadoc.yml       # Workflow de GitHub Actions para CI
├── .mvn/                     # Scripts del wrapper de Maven
├── docs/                     # Documentación generada (Javadoc)
│   └── IMPLEMENTACION.md     # Este informe
├── src/
│   ├── main/
│   │   ├── java/...          # Código fuente de la aplicación Spring Boot
│   │   └── resources/
│   │       ├── application.properties      # Configuración base de Spring
│   │       ├── application-local.properties # Perfil para entorno local (MySQL)
│   │       └── application-neon.properties  # Perfil para entorno Neon (PostgreSQL)
│   └── test/
├── .gitignore                # Archivos y directorios ignorados por Git
├── Dockerfile                # Define cómo construir la imagen de la aplicación
├── docker-compose.local.yml  # Orquesta el entorno de desarrollo local
├── docker-compose.neon.yml   # Orquesta la conexión a la BD de Neon
├── mvnw & mvnw.cmd           # Scripts del wrapper de Maven
├── pom.xml                   # Dependencias y configuración del proyecto Maven
└── README.md                 # Guía de inicio rápido y descripción del proyecto
```

## 3. Separación del Fichero `application.properties` en Perfiles
Para lograr flexibilidad entre entornos, se han utilizado los **perfiles de Spring**. El archivo `application.properties` base activa un perfil (`local` o `neon`) que carga una configuración específica.

1.  **`application.properties` (base):**
    Este archivo está casi vacío. Su única función podría ser definir un perfil por defecto si no se especifica ninguno externamente.

2.  **`application-local.properties`:**
    - **Activación**: Se activa con `SPRING_PROFILES_ACTIVE=local`.
    - **Propósito**: Configura la aplicación para conectarse a una base de datos **MySQL** que se ejecuta localmente (o en un contenedor Docker en la misma red).
    - **Configuración Clave**: Define la URL JDBC para MySQL, el driver, el dialecto de Hibernate y las credenciales de usuario.

3.  **`application-neon.properties`:**
    - **Activación**: Se activa con `SPRING_PROFILES_ACTIVE=neon`.
    - **Propósito**: Configura la aplicación para conectarse a una base de datos **PostgreSQL** alojada en la nube (Neon).
    - **Configuración Clave**: Las propiedades (`spring.datasource.url`, `username`, `password`) están diseñadas para ser sobreescritas por variables de entorno pasadas por Docker Compose, evitando así codificar credenciales sensibles en el código.

Este enfoque permite usar la misma imagen de Docker en diferentes entornos, simplemente cambiando las variables de entorno al ejecutar el contenedor.

## 4. Resumen del `Dockerfile` y de los Ficheros `docker-compose`

### `Dockerfile`
Se utiliza una **compilación multi-etapa** para crear una imagen final optimizada, pequeña y segura.
- **Etapa 1 (`build`)**:
  - Utiliza una imagen base con el JDK completo (`eclipse-temurin:21-jdk`).
  - Copia primero los archivos de Maven (`pom.xml`, `mvnw`, `.mvn/`) y descarga las dependencias. Esto aprovecha la caché de Docker, acelerando las compilaciones futuras si las dependencias no cambian.
  - Copia el código fuente y compila la aplicación, generando el archivo `.jar`.
- **Etapa 2 (Final)**:
  - Utiliza una imagen base mucho más ligera con solo el JRE (`eclipse-temurin:21-jre`), ya que no se necesita el JDK para ejecutar la aplicación.
  - Copia únicamente el archivo `.jar` generado en la etapa anterior.
  - Expone el puerto `8080` y define el `ENTRYPOINT` para ejecutar la aplicación.

El resultado es una imagen final que no contiene el código fuente ni las herramientas de compilación, reduciendo su tamaño y la superficie de ataque.

### `docker-compose.local.yml`
- Define dos servicios: `app` y `db`.
- **`app`**: Construye la imagen a partir del `Dockerfile` local y la ejecuta.
  - Activa el perfil `local` de Spring.
  - Pasa las variables de entorno para la conexión a la base de datos, apuntando al servicio `db` (`jdbc:mysql://db:3306/pets`).
- **`db`**: Levanta un contenedor con la imagen oficial de `mysql:8.0`.
  - Expone el puerto `3306` para que se pueda acceder a la BD desde el exterior (por ejemplo, con un cliente de BD).
  - Configura la base de datos, usuario y contraseñas a través de variables de entorno.
  - Utiliza un volumen (`mysql_data`) para persistir los datos de la base de datos aunque el contenedor se detenga o elimine.

### `docker-compose.neon.yml`
- Define un único servicio: `app`.
- Construye y ejecuta la aplicación, activando el perfil `neon`.
- Pasa las credenciales de la base de datos Neon a través de variables de entorno (`SPRING_DATASOURCE_URL`, `SPRING_DATASOURCE_USERNAME`, etc.).
- Estas variables de entorno se cargan desde un archivo `.env` local, que está excluido de Git para proteger las credenciales.

## 5. Descripción del Flujo de Trabajo en GitHub
El flujo de trabajo se basa en un modelo de integración continua simple pero efectivo.

1.  **Desarrollo**: El desarrollo de nuevas funcionalidades o la corrección de errores se realiza en ramas separadas (ej. `feature/add-new-endpoint` o `fix/bug-in-list`).
2.  **Pull Request (PR)**: Una vez que el trabajo en la rama está completo, se abre un Pull Request hacia la rama `main`. Esto permite la revisión del código por parte de otros miembros del equipo.
3.  **Integración Continua (CI)**: Al fusionar el PR en la rama `main`, se dispara automáticamente el workflow definido en `.github/workflows/javadoc.yml`.
    - El workflow se ejecuta en un entorno de Ubuntu.
    - Clona el repositorio.
    - Configura Java 21.
    - Ejecuta `mvn clean package`. Este comando, además de compilar el proyecto, está configurado para generar la documentación Javadoc en el directorio `/target/site/apidocs`.
    - Utiliza la action `peaceiris/actions-gh-pages` para tomar el contenido de la carpeta de Javadoc y publicarlo en la rama `gh-pages` del repositorio.
4.  **Despliegue de Documentación**: GitHub Pages está configurado para servir el contenido de la rama `gh-pages`, haciendo que la documentación de la API esté siempre actualizada y accesible online.

## 6. Evidencias Gráficas de Funcionamiento

A continuación se muestran capturas de pantalla que evidencian el correcto funcionamiento del sistema.

**1. Levantando el entorno local con Docker Compose:**
*Aquí iría una captura de la terminal mostrando la salida de `docker-compose -f docker-compose.local.yml up` con los logs de la aplicación Spring y la base de datos MySQL iniciándose correctamente.*

**2. Acceso a la Documentación Swagger UI:**
*Aquí iría una captura del navegador mostrando la interfaz de Swagger en `http://localhost:8080/swagger-ui.html`, con el endpoint `/pet/list` visible.*

**3. Respuesta del Endpoint `/pet/list`:**
*Aquí iría una captura de pantalla de una herramienta como Postman o del propio Swagger UI mostrando una respuesta `200 OK` con un JSON que contiene una lista de mascotas.*

**4. Ejecución del Workflow de GitHub Actions:**
*Aquí iría una captura de la pestaña "Actions" del repositorio de GitHub, mostrando una ejecución exitosa (en verde) del workflow "Deploy to GitHub Pages".*

**5. Javadoc Desplegado en GitHub Pages:**
*Aquí iría una captura del navegador mostrando la página de Javadoc generada, accesible en la URL de GitHub Pages del repositorio.*

## 7. Conclusiones del Proyecto
El proyecto ha cumplido con éxito todos sus objetivos. Se ha creado una aplicación robusta con Spring Boot que es fácil de desarrollar, probar y desplegar gracias a la contenerización con Docker.

La separación de la configuración mediante perfiles de Spring y variables de entorno proporciona una gran flexibilidad, permitiendo que la misma aplicación funcione en diferentes entornos sin modificaciones en el código.

Finalmente, la implementación de un pipeline de CI/CD para la documentación con GitHub Actions automatiza una tarea clave, asegurando que la documentación esté siempre sincronizada con el código más reciente en la rama principal. Esto demuestra un flujo de trabajo moderno y eficiente.