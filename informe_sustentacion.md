# Informe de Cambios y Guión para Sustentación (Video)

¡Felicidades! Tu API está respondiendo perfectamente (como se ve en tu captura, el error 401 significa que la seguridad de Spring Boot está activa y rechazando peticiones sin token, lo cual es excelente). 

Usa este documento para armar el **Guión de tu video** y para entender exactamente qué le hicimos al código.

---

## 🏗️ 1. Resumen de los Cambios (Lo que debes decir en el video)

Cuando grabes, puedes empezar explicando los retos que enfrentaste y cómo los solucionaste. Aquí están los **5 grandes cambios** que hicimos en tu código:

### A. Dockerización (Multi-stage Build)
*   **Problema:** Había que garantizar que la aplicación corriera igual en la nube que en local.
*   **Solución:** Se creó un `Dockerfile` con patrón *Multi-stage*. 
    *   **Stage 1:** Usa `eclipse-temurin:17-jdk-alpine` para compilar el proyecto con Maven dentro de Docker.
    *   **Stage 2:** Usa `eclipse-temurin:17-jre-alpine` (solo el runtime) para ejecutar el `.jar`. Esto hace que el contenedor final sea súper ligero.

### B. Migración de MySQL a PostgreSQL
*   **Problema:** El plan gratuito de Render no soporta bases de datos MySQL, solo PostgreSQL.
*   **Solución:** 
    1.  **`pom.xml`**: Se eliminó la dependencia de `mysql-connector-j` y se agregó el driver de `org.postgresql`.
    2.  **`DatabaseConfig.java`**: Se cambió la URL JDBC de `jdbc:mysql://` a `jdbc:postgresql://` y se añadió `sslmode=prefer` para soportar la conexión segura que exige Render.
    3.  **`schema.sql`**: Se adaptó la sintaxis. Los tipos `ENUM` de MySQL se cambiaron por `VARCHAR` con restricciones `CHECK` en PostgreSQL, y los `DATETIME` se cambiaron a `TIMESTAMP`.

### C. Automatización de la Base de Datos (Zero-Touch)
*   **Problema:** Al desplegar en Render, no teníamos acceso fácil a la consola de la BD para crear la tabla de usuarios.
*   **Solución:** Se inyectó un `CommandLineRunner` en el archivo `DataSourceSpringConfig.java`. Gracias a esto, cada vez que la aplicación Spring Boot arranca, lee el archivo `schema.sql` y ejecuta la creación de la tabla automáticamente si no existe.

### D. Externalización de Configuración (12-Factor App)
*   **Problema:** Las credenciales de la BD y los secretos del JWT estaban quemados (hardcoded) en el código.
*   **Solución:** Se modificó el `application.properties` usando placeholders como `${DB_HOST}`, `${JWT_SECRET}`. Así, los datos sensibles se inyectan como **Variables de Entorno** directamente desde el panel de Render, manteniendo el código seguro.

### E. Solución del Bug de Lombok / Java
*   **Problema:** El proyecto fallaba al compilar porque las versiones nuevas del compilador (Java 21/25) chocaban con la versión vieja de Lombok.
*   **Solución:** Se actualizó la versión de `lombok` a la **1.18.42** en el `pom.xml`, lo que permitió un _BUILD SUCCESS_.

---

## 🎥 2. Guión Paso a Paso para Postman (Para el Video)

Durante el video, abre Postman y muestra cómo interactúas con tu API real desplegada (`https://users-management-spring-boot-hexagonal-5qny.onrender.com`).

### 🎬 Acción 1: Crear un nuevo usuario (POST)
Explica: *"Primero, vamos a registrar un nuevo usuario en el sistema a través del endpoint público."*

*   **Método:** `POST`
*   **URL:** `https://users-management-spring-boot-hexagonal-5qny.onrender.com/api/users`
*   **Headers:** `Content-Type: application/json`
*   **Body (raw -> JSON):**
```json
{
  "name": "Profesor Jhon",
  "email": "jhon.docente@universidad.edu.co",
  "password": "PasswordSeguro123!",
  "role": "ADMIN"
}
```
*   👉 **Dale a Send.** Debes recibir un `201 Created` y los datos del usuario sin la contraseña.

### 🎬 Acción 2: Hacer Login para obtener el JWT (POST)
Explica: *"Ahora, como nuestro sistema usa Arquitectura Hexagonal y seguridad JWT, vamos a autenticarnos con las credenciales que acabamos de crear."*

*   **Método:** `POST`
*   **URL:** `https://users-management-spring-boot-hexagonal-5qny.onrender.com/api/auth/login`
*   **Headers:** `Content-Type: application/json`
*   **Body (raw -> JSON):**
```json
{
  "email": "jhon.docente@universidad.edu.co",
  "password": "PasswordSeguro123!"
}
```
*   👉 **Dale a Send.** Vas a recibir un `200 OK` con un JSON que contiene un `"token"`.
*   👉 **¡COPIA ESE TOKEN AL PORTAPAPELES!**

### 🎬 Acción 3: Listar Usuarios protegidos (GET)
Explica: *"Finalmente, vamos a consultar la lista de usuarios. Este endpoint está protegido. Si intento entrar sin token, me da un 401 Unauthorized. Así que usaré el Token JWT."*

*   **Método:** `GET`
*   **URL:** `https://users-management-spring-boot-hexagonal-5qny.onrender.com/api/users`
*   **Pestaña Auth en Postman:** 
    *   Selecciona **Bearer Token**.
    *   En el campo "Token", pega el token que copiaste en el paso anterior.
*   👉 **Dale a Send.** Deberías recibir un `200 OK` con la lista de usuarios, incluyendo el que acabas de crear y el usuario admin por defecto.

---

## 📄 3. Tu Entregable Final (PDF)

Solo tienes que copiar esto, llenar tus datos en un Word, guardarlo como PDF y mandárselo al profe:

**Universidad:** [Nombre de tu U]  
**Carrera:** [Tu Carrera]  
**Asignatura:** [Nombre de tu Electiva]  
**Semestre:** [Último Semestre]  
**Código - Estudiante:** [Tu Código] - [Tu Nombre]  
**Actividad:** Despliegue en Render y Migración a PostgreSQL  
**Docente:** Jhon  

**URL del repo en GitHub:** `https://github.com/jeffcassc/users-management-spring-boot-hexagonal-ddd`  
**URL del aplicativo desplegado:** `https://users-management-spring-boot-hexagonal-5qny.onrender.com`  
**URL del video:** `[Tu link de YouTube/Drive aquí]`
