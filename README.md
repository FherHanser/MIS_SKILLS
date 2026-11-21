# MIS_SKILLS

Respaldo de mis skills personales de Claude (prefijo `hns-`). La versión que uso vive en **claude.ai**; este repositorio guarda una copia para tener historial de cambios.

## Cómo restaurar o actualizar una skill

1. Comprime la carpeta de la skill en un `.zip`. La carpeta debe quedar en la raíz del zip, con su `SKILL.md` adentro.
2. En claude.ai, ve a **Settings → Capabilities → Skills** y sube el zip.
3. Claude Code descarga sola las skills de claude.ai, así que no hace falta instalarlas en la PC.

## Skills

| Skill | Para qué sirve |
|---|---|
| [hns-acta-reunion-teams](hns-acta-reunion-teams/) | Convierte la transcripción de una reunión (de Teams u otra herramienta) en un acta con resumen, acuerdos, plan con responsables, fechas y estado, riesgos y próximos pasos. |
| [hns-apple-design](hns-apple-design/) | Revisa diseños de apps móviles y de escritorio (iOS, Flutter, React Native, macOS, Tauri, Electron) según las Human Interface Guidelines de Apple. Trae 123 páginas de referencia y revisa accesibilidad, modo oscuro, navegación, formularios, íconos y UX de IA. |
| [hns-diagram-design](hns-diagram-design/) | Genera diagramas con marca propia en HTML, SVG o PNG. Cubre arquitectura, flujos, secuencia, ER, Gantt, Sankey, swimlane, organigramas, timelines y muchos tipos de gráficos. También importa desde draw.io, Mermaid, Excalidraw y PlantUML. |
| [hns-diseno-krug-atomico](hns-diseno-krug-atomico/) | Diseña y audita interfaces con dos enfoques: «No me hagas pensar» (Steve Krug, usabilidad) y Diseño Atómico (Brad Frost, sistemas de componentes). Ayuda a decidir dónde va cada control, qué texto, ícono o color usar y cómo agregar funciones sin romper lo que ya funciona. |
| [hns-email-writing-coach](hns-email-writing-coach/) | Redacta, revisa, traduce y resume correos y mensajes profesionales en español e inglés (email, WhatsApp, Slack, Teams). Incluye mensajes ejecutivos, comerciales, seguimientos, disculpas, rechazos y malas noticias. |
| [hns-informe-profesional](hns-informe-profesional/) | Genera informes ejecutivos en Word (.docx) con pirámide invertida, jerarquía visual y un resumen ejecutivo (TL;DR). Sirve para informes de avance, de KPIs, de incidencias o de proyecto dirigidos a gerencia o a clientes. |
| [hns-ingeniero-senior-pragmatico](hns-ingeniero-senior-pragmatico/) | Actúa como ingeniero de software senior: aplica Clean Architecture, Clean Code y SOLID, hace el cambio mínimo seguro, revisa la evidencia antes de tocar nada, protege las integraciones críticas (SAP, WMS, SQL Server, APIs) y usa una definición de terminado estricta. |
| [hns-mit-presentation-coach](hns-mit-presentation-coach/) | Te asesora en presentaciones con el método de Patrick Winston (MIT) y la secuencia motivadora de Monroe. Ayuda a crear y revisar charlas, decks, pitches y clases: aperturas, estructura, historias, STAR y cierres. |
| [hns-odoo-task-reporter](hns-odoo-task-reporter/) | Toma el trabajo de la conversación del día y arma el título, la descripción y los partes de horas (subtareas) para registrarlos en Odoo. |
| [hns-project-platform-kpi-auditor](hns-project-platform-kpi-auditor/) | Audita plataformas de gestión de proyectos (dashboards, tareas, estados, tiempos, bloqueos) y propone mejoras y KPIs con fundamento, siguiendo las skills de diseño. |
| [hns-sql-wms-sap](hns-sql-wms-sap/) | Genera consultas de SQL Server para el entorno WMS/SAP de Canella (EWMS_CANELLA, SBO_CANELLA, Cube_CODA). Cubre picking, fill rate, stock, interfaces, despachos, diferencias de inventario y la sincronización entre SAP y WMS. |
| [hns-ui-ux-pro-max](hns-ui-ux-pro-max/) | Ayuda a diseñar UI/UX para web, móvil y escritorio. Incluye datos que se pueden buscar: estilos, paletas, combinaciones de fuentes, guías de UX, íconos, tipos de gráfico y guías por stack tecnológico. |

## Nota

`hns-sql-wms-sap` describe bases de datos internas de la empresa. Si este repositorio se sube a GitHub, debe ser **privado**.
