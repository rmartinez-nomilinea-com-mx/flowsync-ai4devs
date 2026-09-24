# PRD — FlowSync MVP

> Caso de estudio del curso, no un cliente real. Alcance recortado al mínimo (2026-09-23). Historias en [historias-mvp.md](./historias-mvp.md).

## 1. Problema

En un equipo remoto pequeño nadie ve en qué está cada uno sin interrumpir a alguien. El "¿en qué estás?" llega de dos formas: la ronda de la daily, que se come la mitad de sus 15 minutos, y los mensajes sueltos por chat.

**Coste concreto:** dos personas tocaron el mismo módulo la misma semana porque una empezó sin que la otra lo supiera. Se perdieron dos días.

## 2. Usuarios

- **Quiénes:** los miembros de un equipo remoto de 3–10 personas, con roles planos. Todos ven y hacen lo mismo.
- **Quién se beneficia:** los pares, no un lead.
  - Quien iba a empezar algo que otra persona ya tenía.
  - Quien interrumpe para preguntar.
  - Quien recibe la interrupción.
- **Caso de estudio:** un equipo de 6 personas de un SaaS, en 3 husos horarios, con un gestor de tareas pesado y una daily de 15 minutos.
- **No es nuestro usuario:** un equipo que necesite planificar, estimar o reportar hacia arriba.

## 3. Propuesta de valor

Una lista compartida donde se ve **quién está en qué y qué está libre** sin preguntar, y en la que actualizar tu tarea cuesta dos clics.

Sirve para dos decisiones:
1. **No empezar algo que otra persona ya tiene.**
2. **Elegir lo siguiente sabiendo qué está libre** (libre = sin responsable).

**Por qué se mantiene al día:** quien la escribe también la usa. Es su cola de trabajo y, al actualizarla, deja de recibir preguntas.

**Éxito a una semana de uso real:** el equipo cancela la ronda de "¿en qué estás?" de la daily y nadie pide que vuelva.

## 4. Alcance

Esto es todo lo que se construye. Lo que no aparece aquí está fuera.

1. **Crear una tarea escribiendo solo el título.** Nace en `pendiente` y libre.
2. **Ver la lista del equipo.** Cada tarea muestra título, estado, responsable (o "Libre") y la **fecha y hora de su última modificación en UTC**.
   - Orden fijo: `en curso` → `pendiente` → `hecho`. Dentro de cada estado, primero la modificada más recientemente.
   - La sección `hecho` sale plegada.
3. **Cambiar el estado** entre `pendiente`, `en curso` y `hecho`, en cualquier dirección y con un máximo de 2 clics. Cualquiera puede cambiar cualquier tarea.
4. **Coger una tarea libre:** un clic te hace responsable y la pasa a `en curso`. Si otra persona la cogió antes, se rechaza y ves quién la tiene.
5. **Soltar una tarea:** cualquiera puede dejar libre una tarea asignada, sin cambiar su estado. Sirve para que una tarea abandonada no quede bloqueada.
6. **La lista se refresca sola:** se consulta periódicamente (≤ 30 s) y al volver a la pestaña. No hace falta recargar ni preguntar.

Base existente que se reutiliza sin cambios: el registro y el login.

## 5. NO-alcance

Cada punto es una decisión explícita. Si alguien lo pide durante el MVP, la respuesta es no.

### Campos y operaciones sobre tareas
- **Fecha de vencimiento** y marca de vencida. Estaba en el contexto inicial y se recorta porque no sirve a ninguna de las dos decisiones.
- **Editar el título.** Si está mal, se crea otra tarea y la errónea se pasa a `hecho`.
- **Borrar tareas.** Lo terminado o erróneo va a `hecho`.
- **Asignar una tarea a otra persona.** Cada uno se asigna la suya con "Coger"; repartir trabajo es cosa de un lead, y aquí no hay lead.
- Descripción, comentarios, adjuntos, subtareas, checklists, dependencias o enlaces entre tareas.
- Etiquetas, áreas, módulos, prioridad, tipo o cualquier campo extra. **Detectar trabajo duplicado es cosa de las personas**, mirando lo que está `en curso`.
- Más de un responsable por tarea.
- Estados configurables, un estado "bloqueada" o reglas de transición.
- Historial de cambios o registro de quién hizo la última modificación.

### Vista y consumo
- **Filtros** (por responsable, estado o texto) y búsqueda. El contexto inicial pedía filtrar por estado; lo sustituyen el orden fijo y `hecho` plegado.
- Ordenación manual, arrastrar y soltar, tablero kanban u otras vistas.
- Vista "qué ha cambiado desde tu última visita".
- Paginación y archivado. Se asume que la lista de un equipo de ≤ 10 personas cabe en una página.
- Hora local de cada persona: todas las fechas y horas se muestran en UTC.
- Móvil nativo, modo offline y accesibilidad más allá de la que dan los componentes base.

### Tiempo real y señales
- **Push del servidor** (SSE, WebSockets). Basta con la consulta periódica.
- Notificaciones de cualquier tipo: push, email, sonido, badges o resúmenes diarios.
- **Presencia:** quién está conectado, última conexión, "escribiendo…" o indicadores de actividad de personas. Se rechaza a propósito porque es vigilancia.
- Chat, videollamada y edición colaborativa simultánea.

### Equipo, cuentas y acceso
- **Varios equipos o espacios**, y personas en más de uno. Un despliegue es un equipo.
- **Control de acceso a nivel de aplicación:** invitaciones, dominios permitidos o aprobación de altas. El aislamiento se hace **por red** (VPN, red privada o allowlist de IPs). Exponer el despliegue a Internet con el registro abierto está fuera de uso soportado.
- Roles, permisos y administradores.
- Perfil editable, avatares, recuperación de contraseña, SSO/OAuth y 2FA.
- Borrar o desactivar cuentas.

### Planificación y gestión
- Sprints, estimaciones, épicas, backlog priorizado, roadmaps, capacidad o carga de trabajo.
- Informes, métricas, dashboards, exportaciones o analítica de uso.
- Bloqueos e impedimentos. La parte de bloqueos de la daily sigue existiendo y este MVP no la resuelve.

### Integraciones
- Estado derivado de Git, PRs, CI o calendario.
- Importar tareas de otro gestor o sincronizarse con él. FlowSync sustituye al gestor, no convive con él.
- API pública, webhooks, integración con Slack o chat, y cualquier OAuth de terceros.
