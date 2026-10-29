# apptdea – Sistema de reservas

Aplicación full stack para gestionar reservas: permite listar, crear y cancelar reservas. El backend expone una API REST con Spring Boot y PostgreSQL, y el frontend en React consume esa API.

Proyecto del **Laboratorio Spring Boot con Cursor** – Tecnológico de Antioquia, 2026.

Autor: [Daniel Salgado Mazo // Juan David Castro // Duvan Pinilla]

## Stack tecnológico

### Backend
- Java 25
- Spring Boot 4.0.9 (Spring Web)
- Spring Data JPA (Hibernate)
- PostgreSQL 17
- springdoc-openapi (documentación con Swagger UI)
- Spring Boot DevTools
- Maven

### Frontend
- React
- TypeScript
- Vite
- Axios
- Node.js 24

## Arquitectura

### Backend (arquitectura por capas)

```
Cliente HTTP -> Controller -> Service -> Repository -> PostgreSQL
                    |            |
                   DTO        Entity
```

- **controller**: expone los endpoints REST y transforma las entidades en DTO.
- **service**: contiene las reglas de negocio y las transacciones.
- **repository**: acceso a datos con Spring Data JPA.
- **entity**: entidades JPA (`Reservation`, `ReservationStatus`).
- **dto**: objetos de transferencia implementados con `records` de Java.
- **exception**: excepciones de negocio y manejador global que responde con errores HTTP.
- **config**: filtro CORS para permitir peticiones desde `http://localhost:3000`.

### Frontend
- **services**: acceso a la API con Axios (`ReservationService.ts`); la URL del backend se lee de una variable de entorno.
- **hooks**: lógica de estado y de formularios (`useReservations`, `useReservationForm`).
- **components**: vistas (`ReservationForm`, `ReservationTable`, `Toast`) con su CSS.

## Estructura del proyecto

```
apptdea/
├── .cursor/rules/springboot.mdc
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── services/
│   │   ├── App.tsx
│   │   └── main.tsx
│   ├── .env
│   └── package.json
├── src/main/java/com/udea/apptdea/
│   ├── config/
│   ├── controller/
│   ├── dto/
│   ├── entity/
│   ├── exception/
│   ├── repository/
│   ├── service/
│   └── ApptdeaApplication.java
├── src/main/resources/application.properties
└── pom.xml
```

## Reglas de negocio

- No se puede crear una reserva si ya existe otra en la misma fecha y hora.
- Cancelar una reserva cambia su estado de `ACTIVE` a `CANCELLED`.
- No se puede cancelar una reserva que no existe ni una que ya está cancelada.

## API REST

| Método | Ruta | Descripción | Respuestas |
|--------|------|-------------|------------|
| GET | `/reservas` | Lista todas las reservas | 200 |
| POST | `/reservas` | Crea una reserva | 201, 409 si el horario está ocupado |
| DELETE | `/reservas/{id}` | Cancela una reserva | 204, 404 si no existe, 409 si ya estaba cancelada |

Ejemplo de cuerpo para `POST /reservas`:

```json
{
  "customerName": "Diego",
  "reservationDate": "2026-11-01",
  "reservationTime": "13:00",
  "serviceName": "Capacitación"
}
```

Con la aplicación en marcha, la documentación interactiva está en
`http://localhost:8080/swagger-ui/index.html`.

## Requisitos previos

- JDK 25
- Node.js 24
- PostgreSQL 17 (con pgAdmin, opcional)

## Configuración y ejecución

### 1. Base de datos
Crear una base de datos llamada `reservationtdea` en PostgreSQL. Hibernate crea las tablas al arrancar (`ddl-auto=update`).

### 2. Credenciales
Ajustar usuario y contraseña en `src/main/resources/application.properties`:

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/reservationtdea
spring.datasource.username=postgres
spring.datasource.password=root
```

### 3. Backend
```bash
./mvnw spring-boot:run        # Linux / macOS
.\mvnw.cmd spring-boot:run    # Windows
```
El servidor arranca en `http://localhost:8080`.

### 4. Frontend
Dentro de la carpeta `frontend`, crear un archivo `.env` con:

```
VITE_API_BASE_URL=http://localhost:8080
```

Luego:

```bash
cd frontend
npm install
npm run dev -- --port 3000
```
La aplicación queda en `http://localhost:3000`. Se usa el puerto 3000 porque es el único origen que acepta el filtro CORS del backend.

## Notas

- La contraseña en `application.properties` está en texto plano solo por tratarse de un entorno de desarrollo local.
- El proyecto se desarrolló con apoyo de herramientas de IA siguiendo la guía del laboratorio; los prompts utilizados están documentados en el informe.
