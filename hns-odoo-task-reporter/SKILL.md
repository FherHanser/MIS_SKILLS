---
name: "hns-odoo-task-reporter"
description: "Genera el título, descripción y partes de horas (sub-tareas) para registrar en Odoo con base en el historial de la conversación del día. Activar cuando el usuario diga \"reportemos en Odoo\", \"pasemos a Odoo\", \"Odoo\", o cualquier variación que indique que quiere documentar el trabajo del día en Odoo. No pedir contexto adicional — tomar todo del historial de la conversación activa."
---
## Qué hace este skill
Cuando el usuario indica que quiere reportar en Odoo, revisar el historial completo de la conversación activa e identificar:
- Qué se analizó o investigó
- Qué queries o herramientas se construyeron
- Qué hallazgos o conclusiones se obtuvieron
- Qué decisiones se tomaron
Con esa información generar **un solo bloque**, listo para pegar en Odoo.
---
## Reglas de redacción
- **Descripción:** sin bullets ni listas. Debe escribirse en 2 o 3 mini párrafos narrativos, separados por saltos de línea. Debe tener entre 80 y 150 palabras y no exceder 170 palabras. Debe explicar el punto de partida, el proceso seguido y la conclusión o resultado obtenido, sin convertir el historial completo en una transcripción ni detallar cada consulta, tabla o interacción salvo que sea necesario para entender el diagnóstico. Usar preferentemente 3 mini párrafos: el primero para el contexto inicial, el segundo para el proceso de análisis y el tercero para la conclusión, decisión o resultado. Si el trabajo fue simple, usar 2 mini párrafos.
- **Partes de Horas:** lista de mini-tareas concretas de 2-5 palabras cada una.
- **Tono técnico pero claro** — orientado a que otro miembro del equipo entienda qué se hizo sin leer el chat.
- **En español** — siempre.
- **No incluir nombres de personas.**
- **No inventar** — solo lo que ocurrió en el chat.
---
## Cómo identificar las sub-tareas (Partes de Horas)
Revisar el historial y agrupar el trabajo en bloques de actividad homogénea. Ejemplos:
- "Análisis de estructura de vista"
- "Creación de queries"
- "Documentación de queries"
- "Validación de resultados"
Cada sub-tarea debe ser concreta, de 2-5 palabras. El usuario agregará las horas manualmente.
---
## Output
Presentar exactamente en este formato, un solo bloque:
---
**Título:**
[Título corto y descriptivo — máximo 8 palabras]
**Descripción:**
[Párrafos narrativos describiendo qué se hizo, cómo se hizo y qué se concluyó]
**Partes de Horas:**
| Descripción | Horas |
|:---|:---|
| [sub-tarea 1] | __ |
| [sub-tarea 2] | __ |
| [sub-tarea 3] | __ |
---
