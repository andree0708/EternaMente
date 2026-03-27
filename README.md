# EternaMente - Guia de ejecucion y commit

## Donde se guardan los resultados

Los resultados del juego se guardan en la tabla `assessment_session` de PostgreSQL.

- Tabla principal: `assessment_session`
- Usuario relacionado: columna `user_id` (FK a `app_user.id`)
- Endpoint que guarda: `POST /api/assessments`

Campos relevantes en `assessment_session`:

- `age`
- `metrics` (JSONB con movimientos, errores, etc.)
- `model_version`
- `risk_score`
- `predicted_dcl`
- `created_at`
- `ml_run_at`
- `user_id`

## Requisitos

- Java 21
- Node.js 18+
- PostgreSQL (en tu caso corre en `localhost:5433`)
- Ollama instalado y activo (para analisis completo con `gemma3:4b`)
- Ollama instalado y activo (modelo recomendado local: `phi3:mini`)

## Configurar Ollama (analisis cognitivo)

En una terminal:

```bash
ollama pull phi3:mini
ollama run phi3:mini
```

El backend llama por defecto a:

- URL: `http://localhost:11434/api/generate`
- Modelo: `phi3:mini`

Puedes cambiarlo con variables/properties:

- `ollama.url`
- `ollama.model`

## Variables de entorno (Git Bash)

Desde `EternaMente/backend`:

```bash
export DATABASE_USERNAME=postgres
export DATABASE_PASSWORD=TU_PASSWORD
export DATABASE_URL=jdbc:postgresql://localhost:5433/eternamente
```

## Primera vez: arreglar esquema si hubo errores previos

Si te aparecieron errores por `user_external_id` o `user_id` nulo, ejecuta esto en pgAdmin (DB `eternamente`):

```sql
ALTER TABLE assessment_session
  ALTER COLUMN user_external_id DROP NOT NULL;

DELETE FROM assessment_session
WHERE user_id IS NULL;

ALTER TABLE assessment_session
  ALTER COLUMN user_id SET NOT NULL;
```

## Correr el proyecto

### 1) Backend

Desde `EternaMente/backend`:

```bash
./gradlew bootRun
```

Si arranco bien, queda en `http://localhost:8080`.

### 2) Frontend

En otra terminal, desde `EternaMente/frontend`:

```bash
npm install
npm run dev
```

Abrir `http://localhost:4321`.

## Flujo de prueba rapido

1. Registrarte o iniciar sesion.
2. Ir a `/game`.
3. Completar partida.
4. Click en **Guardar Resultados**.
5. Validar en consola del navegador que no haya 401/500.
6. Validar en DB:

```sql
SELECT id, user_id, age, risk_score, created_at
FROM assessment_session
ORDER BY created_at DESC
LIMIT 10;
```


## Notas utiles

- Si cambia el contenido de una migracion ya aplicada, Flyway puede fallar por checksum/descripcion.
- En desarrollo, puedes ajustar `flyway_schema_history` con cuidado o recrear la base para empezar limpio.
