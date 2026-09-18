# Blacklist App

Microservicio REST para la gestión centralizada de la lista negra global de emails de la compañía. Permite a los sistemas internos consultar si un email está bloqueado, así como agregar nuevos emails a la lista negra.

## Contexto del proyecto

Una compañía multinacional gestiona cientos de aplicaciones internas, cada una con su propia lógica de bloqueo de emails, lo que ha generado inconsistencias y problemas legales. Este microservicio centraliza esa lógica en un único punto de verdad.

## Stack tecnológico

| Tecnología | Uso |
|---|---|
| Python 3.11 | Lenguaje base |
| Flask 3.0 | Framework web |
| Flask-SQLAlchemy | ORM para PostgreSQL |
| Flask-RESTful | Estructura de API REST |
| Flask-Marshmallow | Serialización/validación de datos |
| Flask-JWT-Extended | Soporte de autenticación por token |
| PostgreSQL 14 | Base de datos relacional |
| Poetry | Gestión de dependencias y entornos virtuales |
| pytest / pytest-cov | Testing y cobertura de código |
| Docker / Docker Compose | Contenerización y orquestación local |

## Estructura del proyecto

blacklist_app/
├── src/
│ ├── db/ # Configuración e inicialización de la base de datos
│ ├── middleware/ # Autenticación por Bearer Token
│ ├── models/ # Modelos SQLAlchemy y schemas de Marshmallow
│ ├── repositories/ # Capa de acceso a datos
│ ├── routes/ # Definición de endpoints REST
│ ├── services/ # Lógica de negocio
│ └── utils/ # Utilidades (validaciones, IP del cliente, etc.)
├── tests/
│ ├── unit/ # Tests unitarios con mocks
│ └── integration/ # Tests de integración contra los endpoints reales
├── Dockerfile
├── docker-compose.yml
└── pyproject.toml


## Endpoints disponibles

### `POST /blacklists`
Agrega un email a la lista negra global. Requiere autenticación Bearer Token.

**Request body:**
```json
{
  "email": "usuario@example.com",
  "app_uuid": "123e4567-e89b-12d3-a456-426614174000",
  "blocked_reason": "Actividad de spam detectada"
}
```

**Respuesta (201):**
```json
{
  "id": "a1b2c3d4-...",
  "email": "usuario@example.com",
  "message": "Email added to blacklist successfully",
  "created_at": "2026-08-28T10:00:00"
}
```

Internamente, el servicio también registra la IP de origen de la solicitud y la fecha/hora del registro.

### `GET /blacklists/<email>`
Consulta si un email está en la lista negra global. Requiere autenticación Bearer Token.

**Respuesta (200):**
```json
{
  "is_blacklisted": true,
  "email": "usuario@example.com",
  "blocked_reason": "Actividad de spam detectada"
}
```

### `GET /blacklists/ping`
Health check del servicio, sin autenticación.

## Variables de entorno

Copia `.env.example` a `.env` y configura:

DATABASE_URL=postgresql://usuario:password@host:puerto/blacklist_db
BEARER_TOKEN=tu-token-secreto


Para generar un token seguro:
```bash
python3 -c "import secrets; print(secrets.token_urlsafe(32))"
```

## Cómo correr el proyecto localmente

```bash
cd blacklist_app
poetry install
docker compose up -d db      # Levanta solo la base de datos PostgreSQL
poetry run flask --app src.main run
```

## Cómo correr los tests

```bash
poetry run pytest --cov=src --cov-report=term-missing
```

El proyecto cuenta con **50 tests** (unitarios y de integración) que alcanzan una **cobertura de 97.21%**, superando el mínimo del 90% exigido.

- **Tests unitarios**: validan servicios, repositorio, middleware de autenticación, schemas y utilidades de forma aislada, usando mocks para no depender de la base de datos.
- **Tests de integración**: prueban los endpoints reales end-to-end contra una base de datos SQLite temporal, incluyendo casos de éxito, validaciones fallidas, conflictos (email duplicado) y autenticación.

## Integración continua

Cada `push` a la rama `master` ejecuta automáticamente el workflow definido en `.github/workflows/pruebas.yml`, el cual:

1. Instala Python 3.11 y Poetry
2. Instala las dependencias del proyecto
3. Corre la suite completa de tests con reporte de cobertura
4. Falla el pipeline si la cobertura baja del 90% (configurado en `pyproject.toml`)

Esto garantiza que ningún cambio se integre a la rama principal sin pasar las pruebas.

## Docker

El proyecto incluye `Dockerfile` y `docker-compose.yml` para levantar la aplicación junto a una instancia de PostgreSQL de forma reproducible:

```bash
docker compose up --build
```

La app queda expuesta en `http://localhost:5001`.