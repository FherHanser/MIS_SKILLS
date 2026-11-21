---
name: "hns-acta-reunion-teams"
description: "Convierte una transcripción de reunión (Teams u otra) en un acta: resumen, acuerdos, plan con responsables, fechas y estado, riesgos y próximos pasos."
---

# Acta de reunión desde transcripción

Usar cuando el usuario pegue o adjunte la transcripción de una reunión y pida acta, minuta, resumen, tareas, "qué quedó" o "hazlo como Read AI".

## Contexto del usuario
- El usuario es Fernand Hanser (TI, Canella), responsable del seguimiento de proyectos.
- Participantes frecuentes: César Abal (negocio/inventarios), Raúl Sao (desarrollo BI: cubo de Analysis Services, SQL, Power BI), Willy (jefe de TI, asigna requerimientos), Carlos Rosales (desarrollo SAP B1).
- Vocabulario: cubo, Analysis Services, base de pruebas o productiva, semáforo, maestro de bodegas, UDT/UDF de SAP, store (stored procedure), DAX, cierre, base, diario.
- Las transcripciones de Teams traen mucho ruido (palabras en inglés sin sentido como "Call", "Cortana" o "Bing", y frases cortadas). Hay que ignorar ese ruido e interpretar según el contexto técnico.

## Pasos
1. Leer la transcripción completa.
2. Deducir la fecha de la reunión a partir de las referencias del texto ("ayer al 12", "lunes 16") y marcarla como deducida.
3. Separar lo que es un acuerdo de lo que solo fue una propuesta. No convertir sugerencias en decisiones.
4. Detectar tareas implícitas, como "lo hago mañana" o "apúntalo para el lunes". Convertir fechas relativas en fechas absolutas.
5. Crear el acta como documento (Docs) con estas secciones:
   - Datos generales: fecha, participantes con su rol, mencionados, objetivo y reunión anterior.
   - Resumen ejecutivo: 1 o 2 párrafos, empezando por el estado del proyecto.
   - Acuerdos y decisiones: lista numerada.
   - Plan de trabajo: tabla con #, Frente, Tarea, Responsable, Fecha y Estado (desplegable con Pendiente / En curso / Hecho), agrupada por frente.
   - Tareas por responsable: casillas por persona con los compromisos que no están en el plan.
   - Riesgos y temas sin dueño: tabla con Riesgo, Detalle y Responsable, marcando "Por definir" donde no haya dueño.
   - Próximos pasos: fecha de la próxima revisión y pendientes, más una nota de que el acta sale de una transcripción automática.
6. Escribir en español de Guatemala, con frases cortas y sin adornos.
7. Al final, ofrecer redactar los entregables derivados: un requerimiento formal para Willy, un correo de seguimiento o el mensaje para el chat del equipo.

## Reglas
- No inventar fechas ni responsables. Si no se definieron, poner "Por definir".
- Señalar los temas que quedaron sin dueño, porque suelen ser los riesgos reales.
- Si se pide a otra área (por ejemplo SAP/Carlos), indicar que el trámite es un requerimiento formal a Willy.