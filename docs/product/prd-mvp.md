# PRD — FlowSync MVP: estado del equipo sin preguntar

> Caso de estudio del curso, no un cliente real. Estado: **borrador aprobado para historias** (2026-09-23).

## 1. Problema

En un equipo remoto pequeño nadie ve el estado del equipo sin interrumpir a alguien. El "¿en qué estás?" llega de dos formas: la ronda de la daily, que se come la mitad de sus 15 minutos, y los mensajes sueltos por chat.

Episodio que lo concreta: dos personas tocaron el mismo módulo la misma semana porque una empezó sin que la otra lo supiera. Se perdieron dos días.

**Quién se beneficia:** los pares, no un lead. No hay reporte hacia arriba.
- Los dos devs que descubren tarde que iban a lo mismo.
- Quien interrumpe a otro para preguntar.
- Quien recibe la interrupción.

## 2. Usuario objetivo

- Equipos remotos de 3–10 personas con roles planos: todos ven y editan lo mismo.
- **Primer usuario (caso de estudio):** un equipo de 6 personas de un SaaS, repartido en 3 husos horarios. Hoy usan un gestor de tareas pesado y una daily de 15 minutos por videollamada.
- **No es nuestro usuario:** un equipo que necesite sprints, estimaciones, épicas, backlog priorizado o informes.

## 3. Qué decisión cambia

1. **No empezar algo que otra persona ya está tocando.**
2. **Elegir lo siguiente sabiendo qué está libre.** Libre = tarea sin responsable.

Si la única respuesta fuera "sentirse informado", el tiempo real no compensaría lo que cuesta.

## 4. Propuesta

FlowSync es **donde se hace el trabajo, no donde se cuenta**. Sustituye al gestor de tareas: crea las tareas, no las importa. Convivir con otro gestor exigiría actualizar dos veces, y así muere esta categoría de producto.

Hay una única lista compartida de tareas, que cumple dos funciones:

- **Para quien escribe**, es su cola de trabajo. La mira para decidir qué coge, y actualizarla le quita interrupciones. Por eso lo sostiene: cobra en el momento.
- **Para los demás**, es el estado del equipo de un vistazo, fresco, sin refrescar ni preguntar.

**Forma de la señal:** un resumen que espera, no un aviso que interrumpe. "Llego por la mañana o vuelvo de una reunión y veo qué se ha movido." No hay notificaciones push.

**Frescura, no presencia:** el estado es de la **tarea**, no de la persona. No hay "quién está conectado" ni indicadores de actividad. Lo rechazamos a propósito, porque eso es vigilancia.

## 5. Alcance del MVP

### Dentro

| Capacidad | Decisión |
|---|---|
| Tarea | Campos: **título** (el único obligatorio), **responsable** (opcional; vacío = libre), **estado**, **fecha de vencimiento** (opcional). |
| Estados | Tres, fijos y no configurables: `pendiente` → `en curso` → `hecho`. Se puede mover a cualquiera, también hacia atrás. Estado inicial: `pendiente`. |
| Coger una tarea | Un clic sobre una tarea libre te la asigna y la pasa a `en curso`. Si otra persona la cogió antes, se rechaza. |
| Última actualización | **En cada tarea**: "actualizado hace X por Nombre". Es el antídoto visible contra la información vieja. |
| Lista | Una sola lista compartida por todo el equipo. |
| Filtros | Por **responsable** (mías / sin asignar / persona concreta / todas) y por **estado**. Se pueden combinar. |
| Vencimiento | La fecha es un día sin hora. Está **vencida** si hoy, en **UTC**, es posterior a esa fecha y la tarea no está `hecho`. |
| Fechas y horas | Las fechas absolutas se muestran **en UTC** para todo el equipo, para que nadie traduzca husos al hablar de fechas. Los tiempos relativos ("hace 2 h") no dependen de la zona horaria. |
| Tiempo real | Los cambios de otros aparecen sin refrescar: altas, ediciones, cambios de estado o responsable, y borrados. |
| Edición / borrado | Cualquiera edita o borra cualquier tarea (roles planos). |
| Cuentas | El registro y el login ya existen. Cualquier cuenta registrada pertenece al único espacio. |

### Fuera (decisión explícita)

- Sprints, estimaciones, épicas, backlog priorizado, informes y flujos configurables.
- Etiquetas, áreas, módulos o cualquier campo extra en la tarea. **La detección de trabajo duplicado la hacen las personas** mirando qué está `en curso`.
- Bloqueos. La parte de bloqueos de la daily sigue existiendo y este MVP no la resuelve.
- Chat, comentarios, videollamada y edición colaborativa simultánea.
- Presencia, "conectado ahora" e indicadores de actividad de personas.
- Notificaciones push y emails.
- Estado derivado de fuentes externas (Git/PRs, CI, calendario) y cualquier integración u OAuth de terceros.
- Importar tareas de otro gestor.
- **Varios equipos o espacios**, y personas en más de uno.
- Permisos o jerarquía de roles.
- Vista "cambios desde tu última visita". Es candidata a la siguiente iteración (ver §9).

## 6. Supuestos

- **A1 — Un despliegue = un equipo.** No existe la entidad "equipo" ni "espacio". Todo usuario registrado ve todas las tareas.
- **A2 — El aislamiento es de red, no de aplicación.** El despliegue solo es accesible desde la red del equipo (VPN, red privada o allowlist de IPs). El registro abierto es aceptable **solo** bajo esa condición. Si el despliegue se expone a Internet, cualquiera podría registrarse y ver o editar todo. Es responsabilidad de operación, no del producto.
- **A3 — Los duplicados se detectan a ojo.** Basta con que el trabajo `en curso` sea visible para que el equipo evite el episodio de §1. No es seguro: dos tareas con títulos distintos pueden tocar el mismo módulo. Se valida con el uso.
- **A4 — Los husos horarios se unifican en UTC.** El equipo acepta leer las fechas absolutas en UTC.
- **A5 — Nombre visible.** `full_name` es opcional en el alta actual. Cuando falta, se muestra el email.

## 7. Riesgos

| # | Riesgo | Mitigación en el MVP |
|---|---|---|
| **R1** | **La información se queda vieja** y la lista deja de reflejar la realidad. Es el riesgo #1: si ocurre, el producto pierde el sentido. | Actualizar cuesta ≤ 2 clics sobre una lista ya abierta. No hay campos obligatorios salvo el título. La marca "actualizado hace X" hace visible lo viejo. No se obliga a nadie. |
| R2 | Dos personas cogen la misma tarea a la vez. | "Coger" solo funciona si la tarea sigue libre. Si no, se rechaza con un aviso. |
| R3 | Solapamiento en un mismo módulo con tareas distintas (A3). | Se asume. Se observa en la semana de prueba. |
| R4 | El despliegue queda expuesto fuera de la red del equipo (A2). | Se documenta en el despliegue. No hay control en la aplicación. |

## 8. Métrica de éxito

- **Para el usuario:** deja de hacer la ronda de "¿en qué estás?" en la daily porque el estado del equipo se ve de un vistazo.
- **Criterio a una semana de uso real:** el equipo **cancela esa ronda y nadie pide que vuelva**. Si la siguen haciendo igual, no funcionó.
- **Señal de R1 (observación, sin analítica):** al empezar la daily, ¿las tareas `en curso` tienen "actualizado hace" de menos de un día laborable?

## 9. Plan de construcción

Una vertical fina y usable de punta a punta. Mejor una capacidad terminada que tres a medias. El detalle está en [historias-mvp.md](./historias-mvp.md).

1. **Incremento 1 — Cola compartida:** crear, listar, cambiar el estado, coger o asignar, y filtrar. Ya se puede usar con recarga manual.
2. **Incremento 2 — Tiempo real:** los cambios aparecen sin refrescar. Es lo que da sentido al producto; sin esto el MVP no está terminado.
3. **Incremento 3 — Plazos y mantenimiento:** fecha de vencimiento con marca de vencida, editar el título y borrar.
4. **Siguiente iteración (fuera del MVP):** "qué se ha movido desde tu última visita".
