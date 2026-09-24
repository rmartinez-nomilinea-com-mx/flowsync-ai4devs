# Historias de usuario — FlowSync MVP

Estas historias salen del [PRD](./prd-mvp.md) y cubren solo su §4 (Alcance). Todo lo demás está en §5 (NO-alcance).

**Convenciones:**
- Todas asumen una persona autenticada; el registro y el login ya existen.
- Todas trabajan sobre la única lista compartida.
- "Libre" = tarea sin responsable.
- "Última modificación" = el `updated_at` de la tarea, que se muestra como `AAAA-MM-DD HH:mm UTC`.

---

### HU-01 · Crear una tarea con solo el título

**Como** persona del equipo, **quiero** crear una tarea escribiendo su título **para** registrar trabajo sin decidir nada más.

- **Dado** que estoy en la lista, **cuando** escribo un título y pulso Enter, **entonces** la tarea aparece en `pendiente`, libre, con su última modificación igual a la hora de creación.
- El título debe tener de 1 a 200 caracteres, sin contar los espacios de los extremos. Si está vacío, se muestra un error en castellano y no se crea nada.

### HU-02 · Ver quién está en qué

**Como** persona del equipo, **quiero** ver todas las tareas con estado, responsable y última modificación **para** saber quién está en qué sin preguntar.

- Cada fila muestra el título, el estado, el responsable ("Libre" si no tiene) y la última modificación en UTC.
- Si el responsable no tiene `full_name`, se muestra su email.
- El orden es `en curso` → `pendiente` → `hecho`. Dentro de cada estado, primero la modificada más recientemente.
- La sección `hecho` sale plegada y se despliega con un clic.
- Si no hay tareas, se muestra un estado vacío con el campo para crear la primera.

### HU-03 · Cambiar el estado en dos clics

**Como** persona del equipo, **quiero** cambiar el estado de una tarea desde la lista **para** mantenerla al día sin esfuerzo.

- Desde la fila elijo `pendiente`, `en curso` o `hecho`, con un máximo de 2 clics y sin formularios.
- Se puede pasar de cualquier estado a cualquier otro.
- Tras el cambio, la última modificación de la tarea se actualiza.
- Si el guardado falla, la fila vuelve a su valor anterior y se muestra el error.

### HU-04 · Coger y soltar una tarea

**Como** persona del equipo, **quiero** coger una tarea libre con un clic **para** que el resto vea al momento que la estoy haciendo yo.

- **Dado** una tarea libre, **cuando** pulso "Coger", **entonces** paso a ser su responsable y la tarea pasa a `en curso`.
- **Dado** que otra persona la cogió antes, aunque mi pantalla aún la mostrara libre, **cuando** pulso "Coger", **entonces** la operación se rechaza, veo quién la tiene y la fila se actualiza. Nunca se sobrescribe al responsable.
- **Dado** una tarea con responsable, **cuando** cualquiera pulsa "Soltar", **entonces** la tarea queda libre y conserva su estado.
- No hay forma de asignar una tarea a otra persona.

### HU-05 · La lista se refresca sola

**Como** persona del equipo, **quiero** que la lista abierta refleje los cambios de los demás **para** no tener que recargar ni preguntar.

- Con la lista abierta, los cambios de otras personas aparecen en ≤ 30 s sin recargar.
- Al volver a la pestaña, la lista se refresca de inmediato.
- El refresco no genera aviso, sonido ni notificación, y no pierde lo que estoy escribiendo en el campo de nueva tarea.

---

## Notas técnicas

Esto no son requisitos; es un punto de partida que respeta la arquitectura descrita en `CLAUDE.md`.

- **Tabla `tasks`:** `id`, `title` (string 200, not null), `status` (`pending` | `in_progress` | `done`, default `pending`), `assignee_id` (FK `users`, nullable), `created_at`, `updated_at`. Se crea con migración; `database/schema.ts` se regenera, no se edita.
- **API bajo `/api/v1/tasks`,** protegida con `middleware.auth()`:
  - `GET /tasks`
  - `POST /tasks` con `{ title }`
  - `PATCH /tasks/:id` con `{ status }`
  - `POST /tasks/:id/claim`: `UPDATE … WHERE assignee_id IS NULL`. Si no afecta a ninguna fila, devuelve `409` con el responsable actual.
  - `POST /tasks/:id/release`
- Las respuestas van siempre por `serialize()` con un `TaskTransformer`, que incluye `assignee` con `{ id, fullName, email }` y `updatedAt` en ISO 8601 UTC.
- **Frontend:** las llamadas nuevas van en `lib/api.ts` y la lista en `pages/`. El refresco usa `setInterval` (≤ 30 s) más el evento `visibilitychange`.
