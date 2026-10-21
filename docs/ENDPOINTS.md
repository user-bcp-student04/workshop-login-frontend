# Contrato de endpoints observado por ReqFlow Frontend

Este documento describe las solicitudes y respuestas que el cliente Angular espera. Se reconstruyó desde el frontend; no verifica el comportamiento de los servicios backend.

## Autenticación

- **Método y ruta:** `POST http://localhost:8080/auth/login`
- **Solicitud:** JSON con `username`, `password` y `client` (`angular-workshop`). El cliente usa timeout de 5000 ms.
- **Respuesta esperada por el cliente:** `data.token` truthy y `data.user`; no se valida un esquema adicional antes de almacenar los valores.
- **Éxito:** guarda `token` en `localStorage` como `workshop_token`, serializa `user` en `workshop_user`, conserva preferencias en `workshop_preferences` y navega a `/dashboard`.
- **Fallo:** muestra el mensaje de `response.data.message`, el mensaje del error o un texto alternativo.

## Consulta de tareas

- **Método y ruta:** `GET http://localhost:8000/api/tasks?owner=<username guardado>`.
- **Autorización enviada:** `Authorization: Bearer <workshop_token>`.
- **Respuesta esperada por el cliente:** una lista en `data`; para mostrar cada tarea utiliza `title`, `description`, `status`, `priority` y `owner`.
- **Presentación:** excluye de la lista visible las tareas con estado `ARCHIVED`, agrega una etiqueta derivada de prioridad y propietario, y ordena por prioridad.
- **Sesión ausente:** si no encuentra `workshop_token`, navega a `/login` sin consultar tareas.
- **Fallo de consulta:** muestra `response.data.detail`, el mensaje del error o un texto alternativo.

## Cierre de sesión

El botón de logout elimina `workshop_token` y `workshop_user` de `localStorage` y navega a `/login`. Las preferencias `workshop_preferences` se conservan.

## Rutas locales

- `/` y rutas desconocidas redirigen a `/login`.
- `/login` carga el componente de login.
- `/dashboard` carga el dashboard; la comprobación de token está dentro de su inicialización.
