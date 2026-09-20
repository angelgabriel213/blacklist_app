# Blacklist App

Microservicio REST para gestionar una lista negra centralizada de correos electrónicos. El proyecto separa la API en capas de rutas, servicios, repositorios, modelos y utilidades, e incluye pruebas automatizadas, Docker y CI.

## Tecnologías

- Python 3.11
- Flask
- Flask-SQLAlchemy
- Flask-Marshmallow
- PostgreSQL
- Poetry
- pytest / pytest-cov
- Docker / Docker Compose
- GitHub Actions

## Arquitectura

```text
src/
├── db/              # Configuración y modelos de persistencia
├── middleware/      # Autenticación Bearer Token
├── models/          # Modelos y schemas
├── repositories/    # Acceso a datos
├── routes/          # Endpoints REST
├── services/        # Lógica de negocio
└── utils/           # Validaciones y utilidades

tests/
├── unit/            # Pruebas unitarias
└── integration/     # Pruebas de integración
```

## Funcionalidades

- Registrar un correo en la lista negra.
- Consultar si un correo está bloqueado.
- Registrar información de la solicitud, incluida la IP de origen.
- Validar datos de entrada mediante schemas.
- Controlar autenticación mediante Bearer Token.
- Exponer un endpoint de health check.
- Ejecutar pruebas unitarias y de integración.
- Ejecutar la suite de pruebas automáticamente mediante GitHub Actions.
- Ejecutar la aplicación y PostgreSQL mediante Docker Compose.

## API

### POST /blacklists

Requiere:

```http
Authorization: Bearer <TOKEN>
Content-Type: application/json
```

Ejemplo:

```json
{
  "email": "usuario@example.com",
  "app_uuid": "123e4567-e89b-12d3-a456-426614174000",
  "blocked_reason": "Actividad de spam detectada"
}
```

### GET /blacklists/<email>

Consulta el estado de un correo. Requiere Bearer Token.

### GET /blacklists/ping

Endpoint público de health check.

## Configuración

Crea un archivo `.env` a partir de `.env.example`.

Variables principales:

```env
DATABASE_URL=postgresql://usuario:password@host:puerto/blacklist_db
BEARER_TOKEN=tu-token-secreto
```

Generación de un token aleatorio:

```bash
python3 -c "import secrets; print(secrets.token_urlsafe(32))"
```

No subas `.env` ni credenciales reales al repositorio.

## Ejecución local

```bash
cd blacklist_app
poetry install
docker compose up -d db
poetry run flask --app src.main run
```

La API local queda disponible en el puerto configurado por Flask.

## Pruebas

```bash
poetry run pytest --cov=src --cov-report=term-missing
```

La configuración del proyecto exige un mínimo de 90 % de cobertura.

## Docker

Para levantar aplicación y PostgreSQL:

```bash
docker compose up --build
```

La aplicación se ejecuta dentro del contenedor mediante Gunicorn.

## Integración continua

GitHub Actions ejecuta automáticamente la suite de pruebas en cambios dirigidos a `main` y en pull requests hacia esa rama.

## Objetivo técnico

Este proyecto demuestra experiencia práctica con desarrollo de APIs REST en Python, separación de responsabilidades, persistencia relacional, autenticación, testing automatizado, contenerización y CI.
