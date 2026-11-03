# infra

Orquestación local del Salón de Estudio Colaborativo (Mini-proyecto 2 de Proyecto Integrador I, Universidad del Valle).

Este repositorio no tiene código de la aplicación. Tiene el `docker-compose.yml` que levanta el sistema completo en tu equipo y la documentación que aplica a todos los servicios.

## Arquitectura

| Servicio | Repositorio | Producción | En local con Docker |
|---|---|---|---|
| Frontend (React) | [`frontend`](https://github.com/pi1-salon-estudio/frontend) | https://pi1-salon-estudio.vercel.app | http://localhost:8080 |
| API REST | [`backend-api`](https://github.com/pi1-salon-estudio/backend-api) | https://pi1-salon-estudio-api.onrender.com ([Swagger](https://pi1-salon-estudio-api.onrender.com/docs)) | http://localhost:3000 |
| Tiempo real (Socket.IO) | [`backend-realtime`](https://github.com/pi1-salon-estudio/backend-realtime) | https://pi1-salon-estudio-realtime.onrender.com | http://localhost:4000 |
| Base de datos | Firestore (Firebase) | Proyecto `pi1-salon-estudio`, región `us-east4` | La misma de producción |

El frontend nunca consulta Firestore. Los datos pasan por `backend-api` y los eventos en tiempo real por `backend-realtime`.

En producción no se usa este compose. Vercel y Render despliegan cada servicio desde su propio repositorio cuando se fusiona un PR a `main`.

## Requisitos

- Docker Desktop.
- Los cuatro repositorios clonados en la misma carpeta:

```
pi1-salon-estudio/
  frontend/
  backend-api/
  backend-realtime/
  infra/
```

```powershell
git clone https://github.com/pi1-salon-estudio/frontend.git
git clone https://github.com/pi1-salon-estudio/backend-api.git
git clone https://github.com/pi1-salon-estudio/backend-realtime.git
git clone https://github.com/pi1-salon-estudio/infra.git
```

- La llave de la cuenta de servicio de Firebase, guardada fuera de cualquier repositorio. Pídesela al responsable de la infraestructura.

## Levantar el sistema

```powershell
cd infra
Copy-Item .env.example .env
```

Abre `.env` y pon la ruta de la llave en `FIREBASE_ADMIN_JSON`, con `/` en lugar de `\`. Después:

```powershell
docker compose up --build
```

La primera vez tarda unos minutos porque construye las tres imágenes. Cuando termine:

- http://localhost:8080 abre el frontend.
- http://localhost:3000/health y http://localhost:4000/health responden `{"estado":"ok"}`.
- http://localhost:3000/docs abre Swagger.

El frontend arranca después de que los dos backends pasan su health check. Así, cuando abres el navegador, la API ya responde.

| Comando | Qué hace |
|---|---|
| `docker compose up --build` | Construye las imágenes con el código actual y levanta todo |
| `docker compose up -d` | Levanta todo en segundo plano |
| `docker compose ps` | Muestra el estado y el health check de cada servicio |
| `docker compose logs -f backend-api` | Muestra los logs de un servicio |
| `docker compose down` | Detiene y elimina los contenedores |

## Cómo maneja la llave de Firebase

La llave entra como un secret de Docker Compose. Compose la monta como archivo de solo lectura en `/run/secrets/firebase_admin` dentro del contenedor de `backend-api`, y `GOOGLE_APPLICATION_CREDENTIALS` apunta a esa ruta. No se copia en ninguna imagen. `docker compose config` muestra la ruta del archivo, pero nunca su contenido.

Si `FIREBASE_ADMIN_JSON` no está definida, compose se detiene con un mensaje que explica qué falta.

## Problemas comunes

- **Puerto ocupado (`port is already allocated`).** Otro contenedor o programa usa el 8080, el 3000 o el 4000. Detén ese contenedor en Docker Desktop o cambia el puerto de la izquierda en `docker-compose.yml`, por ejemplo `"8081:80"`. Si cambias el puerto del frontend, agrégalo también en `CORS_ORIGINS` de los dos backends.
- **La API responde 503 `BD_NO_DISPONIBLE`.** La ruta de `FIREBASE_ADMIN_JSON` no apunta a la llave o la llave no es válida.
- **Cambié código y no se ve.** Vuelve a correr con `--build`. Sin esa opción, compose reutiliza las imágenes que ya existen.

## Convenciones del proyecto

- `main` está protegida en los cuatro repositorios: los cambios entran por PR.
- Una rama por historia o tarea, por ejemplo `feature/TS-00-base-tecnica-api`.
- Commits con [Conventional Commits](https://www.conventionalcommits.org/es/v1.0.0/).
- Cada PR explica qué cambia, cómo se probó y la historia de Jira relacionada.
- Nunca se suben llaves ni archivos `.env`. Todos los repositorios los bloquean en `.gitignore`.
