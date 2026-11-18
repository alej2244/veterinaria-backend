# Backend - Veterinaria San Marcos (DSY1104)

Repositorio correspondiente al desarrollo del backend para el proyecto semestral **Sistema de Gestión y Atención Web – Veterinaria San Marcos**[cite: 72, 81].

* **Integrantes:** Martin Cortes, Christian Montecinos, Alejandro Naranjo
* **Repositorio Frontend (Unidad 2):** https://github.com/MartinCC77/veterinaria-Evaluacion1[cite: 81]

---

## 1. Arquitectura y Decisiones Técnicas

Para darle persistencia real al sistema y reemplazar el uso de `localStorage` que teníamos en React, definimos la siguiente arquitectura[cite: 62, 81]:

* **Framework y Lenguaje:** Java 21 con **Spring Boot** (proyecto Maven bajo el paquete `cl.duoc.veterinariabackend`). Elegimos Spring Boot porque ya lo probamos en la guía 17 y nos da una estructura ordenada para separar controladores, servicios y seguridad.
* **Despliegue:** Instancia **EC2 (Ubuntu)** en AWS Academy[cite: 83]. La aplicación correrá escuchando en el puerto `8080`, configurando las reglas de entrada necesarias en el Security Group[cite: 83].
* **Base de Datos:** **MySQL**[cite: 83]. Como manejamos usuarios, categorías, servicios y citas relacionadas entre sí[cite: 81], necesitamos un motor relacional que asegure la integridad de los datos mediante llaves foráneas.
* **Dependencias principales:**
  * `Spring Web`: para armar los endpoints REST y manejar JSON[cite: 83].
  * `Spring Data JPA` + `MySQL Driver`: para conectar con la base de datos y hacer consultas sin escribir SQL repetitivo[cite: 83].
  * `Spring Security` + `jjwt`: para proteger rutas, validar roles y generar los tokens JWT[cite: 62, 65].
  * `BCrypt`: para encriptar las claves antes de guardarlas.
  * `Springdoc OpenAPI (Swagger UI)`: para probar y documentar la API desde el navegador.

---

## 2. Conexión con el Frontend

En el repositorio de React, las funciones dentro de `src/services/` (`usuarioService.js`, `servicioService.js`, `categoriaService.js`, `citaService.js`) dejarán de leer y escribir en el navegador para hacer peticiones HTTP hacia este backend[cite: 63, 82].

La interfaz mostrará estados de carga mientras espera la respuesta, mensajes de error si falla una validación (`400`) y redirección al login si el token vence (`401`) o el rol no tiene acceso (`403`)[cite: 63, 66].

### Ejemplo de flujo: Solicitud de Cita (`POST /api/citas`)
1. El cliente llena el formulario en React y valida los campos básicos (que no vayan vacíos y que la fecha sea válida)[cite: 81].
2. React envía el JSON a `POST /api/citas`. Si el usuario está logueado, manda su token en el header `Authorization: Bearer <token>`[cite: 67, 81].
3. El controlador recibe la petición y valida la estructura con `@Valid`.
4. El servicio (`CitaService`) busca el servicio en MySQL para obtener su precio real (aplicando precio de oferta si corresponde) y, si el cliente marcó urgencia, suma **$10.000** al total[cite: 81]. Además, fuerza que toda cita nazca con estado `pendiente`[cite: 81].
5. Se guarda el registro en MySQL y el backend responde `201 Created` con los datos confirmados para que React muestre el comprobante[cite: 66, 81].

---

## 3. Modelo de Datos y Seguridad de Claves

### Diagrama Entidad-Relación (MER)

```mermaid
erDiagram
    USUARIO ||--o{ CITA : "solicita"
    CATEGORIA ||--o{ SERVICIO : "agrupa"
    SERVICIO ||--o{ CITA : "tiene"

    USUARIO {
        bigint id PK
        varchar nombre
        varchar correo UK
        varchar password_hash
        varchar rol
        boolean activo
    }

    CATEGORIA {
        bigint id PK
        varchar nombre UK
    }

    SERVICIO {
        bigint id PK
        varchar codigo UK
        varchar nombre
        varchar especie
        int duracion_min
        int precio
        boolean oferta
        int precio_oferta
        text observaciones
        varchar imagen
        bigint categoria_id FK
    }

    CITA {
        bigint id PK
        varchar dueno
        varchar correo
        varchar mascota
        varchar especie
        date fecha
        varchar hora
        text motivo
        boolean urgencia
        int precio_final
        varchar estado
        datetime creada_en
        bigint usuario_id FK
        bigint servicio_id FK
    }
