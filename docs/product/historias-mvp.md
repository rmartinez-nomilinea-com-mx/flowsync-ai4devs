# Historias de usuario — FlowSync MVP

Estas historias salen del [PRD](./prd-mvp.md). Todas asumen una persona autenticada: el signup y el login ya existen. Todas trabajan sobre la **única lista compartida**.

**Convenciones:**
- "Persona" = cualquier miembro del equipo; no hay roles.
- "Libre" = tarea sin responsable.
- Las fechas absolutas se muestran en UTC con el sufijo `UTC`.
- "Última modificación" = fecha y hora del último cambio en la tarea (título, estado, responsable o fecha). Es el `updated_at` de la tarea y se muestra como `AAAA-MM-DD HH:mm UTC`. No se registra ni se muestra quién hizo el cambio.

---

## Incremento 1 — Cola compartida

### HU-01 · Crear una tarea en segundos

**Como** persona del equipo, **quiero** crear una tarea escribiendo solo su título **para** registrar trabajo sin decidir nada más.

- **Dado** que estoy en la lista, **cuando** escribo un título y pulso Enter, **entonces** la tarea aparece en la lista en `pendiente`, libre y sin fecha.
- El título es obligatorio: de 1 a 200 caracteres, sin contar los espacios de los extremos. Si está vacío, se muestra un error en castellano y no se crea nada.
- No hay más campos obligatorios ni pasos de configuración.
- Al crearla, la tarea muestra su última modificación: el momento de creación.

### HU-02 · Ver el estado del equipo de un vistazo

**Como** persona del equipo, **quiero** ver todas las tareas con su estado, responsable y frescura **para** saber quién está en qué sin preguntar.

- Cada fila muestra: título, estado, responsable ("Libre" si no tiene), fecha de vencimiento si la hay y la última modificación (p. ej. `2026-09-23 14:05 UTC`).
- Cuando la persona no tiene `full_name`, se muestra su email.
- **Orden:** primero `en curso`, luego `pendiente` y al final `hecho`. Dentro de cada estado, primero la modificada más recientemente.
- **La sección `hecho` sale plegada por defecto** y se despliega con un clic. No hay filtros en el MVP: este plegado es lo que evita que lo terminado entierre lo vivo.
- Si no hay tareas, se muestra un vacío con el campo para crear la primera.
- No se muestra ninguna información sobre personas (conexión, actividad), solo sobre tareas.

### HU-03 · Cambiar el estado en dos clics

**Como** persona que hace una tarea, **quiero** cambiar su estado desde la propia lista **para** mantenerla al día sin esfuerzo.

- **Dado** una tarea en la lista, **cuando** abro su selector de estado y elijo otro, **entonces** el cambio se guarda sin pantalla ni formulario intermedios. Son como máximo 2 clics.
- Se puede pasar de cualquier estado a cualquier otro, también hacia atrás.
- Tras el cambio, la última modificación de la fila pasa a ser la hora del cambio.
- Si el guardado falla, la fila vuelve a su estado anterior y se muestra el error.

### HU-04 · Coger una tarea libre, asignar o liberar

**Como** persona del equipo, **quiero** coger una tarea libre con un clic **para** que el resto sepa al momento que ya la estoy haciendo yo.

- **Dado** una tarea libre, **cuando** pulso "Cogerla", **entonces** paso a ser su responsable y la tarea pasa a `en curso`.
- **Dado** que otra persona cogió la tarea antes que yo, aunque mi pantalla aún la mostrara libre, **cuando** pulso "Cogerla", **entonces** la operación se rechaza. Veo quién la tiene y la fila se actualiza. Nunca se sobrescribe al responsable en silencio.
- Desde la fila puedo cambiar el responsable por cualquier persona registrada, o dejar la tarea libre, en como máximo 2 clics.
- Asignar a alguien o liberar la tarea no cambia su estado. Solo "Cogerla" lo cambia.
- Reasignar y liberar siguen la misma regla que "Cogerla". Si el responsable cambió desde que cargué la fila, la operación se rechaza y veo el responsable actual. Nunca piso un cambio de otra persona sin haberlo visto.

---

## Incremento 2 — Tiempo real

### HU-06 · Ver los cambios de otros sin refrescar

**Como** persona del equipo, **quiero** que la lista refleje los cambios de los demás mientras la tengo abierta **para** no tener que recargar ni preguntar.

- **Dado** que tengo la lista abierta, **cuando** otra persona crea, edita, cambia el estado o el responsable, o borra una tarea, **entonces** mi lista lo refleja en ≤ 5 s sin recargar, respetando el orden.
- El cambio no genera aviso, sonido ni notificación. Como mucho, un resaltado breve en la fila afectada.
- La última modificación de la fila afectada se actualiza con el evento.
- Si se pierde la conexión en tiempo real, la lista se resincroniza al recuperarla y no queda desactualizada en silencio. Mientras tanto se ve una indicación discreta de "sin conexión".
- Solo se emiten eventos de tareas. Nunca se emiten ni se exponen eventos sobre personas (conexión, actividad).

---

## Incremento 3 — Plazos y mantenimiento

### HU-07 · Fecha de vencimiento y tareas vencidas

**Como** persona del equipo, **quiero** poner una fecha de vencimiento opcional **para** ver de un vistazo qué se ha pasado de plazo.

- La fecha es un día sin hora. Se pone, cambia o quita desde la fila y se muestra como `AAAA-MM-DD` (UTC).
- Una tarea está **vencida** si la fecha actual en UTC es posterior a su fecha y no está en `hecho`. Las vencidas llevan una marca visual clara.
- La marca de vencida es la misma para todo el equipo, esté en el huso horario que esté.
- Una tarea en `hecho` nunca se marca como vencida.

### HU-08 · Editar el título y borrar una tarea

**Como** persona del equipo, **quiero** corregir el título o borrar una tarea **para** que la lista siga reflejando la realidad.

- El título se edita en línea. Se aplican las mismas reglas que al crearla (HU-01).
- Cualquiera puede borrar cualquier tarea, previa confirmación. El borrado es definitivo, sin papelera.
- En ambos casos los demás lo ven en tiempo real (HU-06).

---

## Siguiente iteración (fuera del MVP)

### HU-05 · Filtrar por responsable y estado (aplazada)

Queda fuera del MVP por decisión de alcance: con 3–10 personas la lista ordenada por estado y con `hecho` plegado basta para verla de un vistazo. Se retomará si la lista deja de caber en ese vistazo. Criterios originales:

**Como** persona del equipo, **quiero** filtrar la lista **para** centrarme en mi cola o en lo que está libre.

- Filtro de responsable: **Mías** · **Sin asignar** · **una persona** · **Todas** (por defecto).
- Filtro de estado: uno o varios de `pendiente` / `en curso` / `hecho`. Por defecto, todos.
- Los dos filtros se combinan. Por ejemplo, "Sin asignar + pendiente" responde a "¿qué puedo coger?".
- Los filtros viven en la URL: un enlace o una recarga mantiene la vista.
- Si una tarea deja de cumplir el filtro al editarla, sale de la vista.

### HU-09 · Qué se ha movido desde mi última visita

**Como** persona que vuelve de una reunión o empieza el día, **quiero** ver qué tareas han cambiado desde mi última visita **para** ponerme al día sin recorrer toda la lista.

Pendiente de definir: qué cuenta como "visita" y si hace falta guardar un historial de cambios de estado.

---

## Notas técnicas para el Incremento 1

Esto no son requisitos; es un punto de partida que respeta la arquitectura del repo (ver `CLAUDE.md`).

- **Tabla `tasks`:**
  - `id`
  - `title` (string 200, not null)
  - `status` (`pending` | `in_progress` | `done`, default `pending`)
  - `assignee_id` (FK `users`, nullable)
  - `due_date` (date, nullable)
  - `created_at`, `updated_at`

  Se crea con migración; `database/schema.ts` se regenera, no se edita.
- **API bajo `/api/v1/tasks`, protegida con `middleware.auth()`:**
  - `GET /tasks`, sin parámetros de filtro en el MVP
  - `POST /tasks`
  - `PATCH /tasks/:id`
  - `POST /tasks/:id/claim` (HU-04). Hace una actualización condicional `WHERE assignee_id IS NULL`; si no hay filas afectadas, devuelve `409` con el responsable actual.
  - Un `PATCH` que cambia `assigneeId` debe incluir `expectedAssigneeId`: actualización condicional y `409` si no coincide, igual que `claim`.
  - `DELETE /tasks/:id`
  - `GET /users` para el selector de responsable.
- Las respuestas van siempre por `serialize()` con un `TaskTransformer`. Este incluye `assignee` con `{ id, fullName, email }` y `updatedAt` en ISO 8601 UTC.
- Las fechas se guardan y se devuelven en UTC (ISO 8601). "Vencida" se calcula en el cliente con la fecha UTC actual.
- **Frontend:** las llamadas nuevas van en `lib/api.ts`. La lista va en `pages/`.
- **Tiempo real (Incremento 2):** SSE con `@adonisjs/transmit`. El backend emite `task.created|updated|deleted` en un único canal de tareas y el cliente aplica los eventos sobre su estado local.
