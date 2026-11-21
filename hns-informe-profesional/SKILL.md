---
name: "hns-informe-profesional"
description: "genera informes profesionales de alto impacto como documentos Word (.docx) estructurados, aplicando principios de arquitectura de la información: pirámide invertida, jerarquía visual, tipografía funcional y resumen ejecutivo TLDR.\nActivar cuando el usuario diga \"genera un informe\", \"hazme el informe de\", \"quiero un reporte de\", \"prepara el informe semanal/mensual\", \"informe de avance\", \"informe ejecutivo\", \"informe de KPIs\", \"informe de incidencias\", \"informe de proyecto\", o cualquier variante que implique producir un documento estructurado para presentar a gerencia, dirección o clientes.\nTambién activar cuando el usuario pegue datos, métricas o resultados y pida convertirlos en un documento formal. Usar siempre este skill antes de crear cualquier .docx de tipo informe, reporte o memo ejecutivo — nunca improvisar la estructura.\n"
---

# Skill: Informe Profesional de Alto Impacto

## Filosofía base

Un informe no es un volcado de datos — es una herramienta de decisión. El diseño es ingeniería
documental: su propósito es reducir la fricción cognitiva para que lo crítico destaque sin esfuerzo.

**Tres pilares que el lector busca en segundos:**
1. Comprensión rápida de la temática
2. Detección inmediata de lo importante
3. Confianza en la profesionalidad del contenido

---

## Proceso de construcción del informe

### Fase 0 — Captura de intención

Antes de generar el informe, identifica desde el contexto:

1. **Lector objetivo:** ejecutivo, técnico o mixto
2. **Sistema, área o proyecto**
3. **Período del informe**
4. **Datos disponibles:** texto pegado, Excel, CSV, capturas o archivos adjuntos
5. **Decisión esperada:** aprobación, escalamiento, seguimiento o acción

Si el usuario ya proporcionó suficiente información, **avanza sin preguntar.**

Pregunta solo si falta un dato crítico sin el cual el informe no tendría sentido:
- destinatario o nivel de lector
- tema / sistema / área
- período
- decisión esperada

**Máximo 2 preguntas. Nunca hagas un formulario completo.**

---

### Fase 1 — Arquitectura del contenido (Pirámide Invertida)

Organizar siempre en tres niveles de relevancia decreciente:

```
┌──────────────────────────────────────────┐
│  INICIO — Lo Esencial                    │
│  • Resumen ejecutivo (TLDR: 3-5 líneas)  │
│  • Quién / Qué / Cómo / Cuándo / Por qué │
│  • Conclusiones y decisiones requeridas  │
├──────────────────────────────────────────┤
│  CUERPO — El Análisis                    │
│  • KPIs y métricas principales           │
│  • Hallazgos con evidencia               │
│  • Comparativo vs. período anterior      │
├──────────────────────────────────────────┤
│  FINAL — El Detalle                      │
│  • Tablas extendidas                     │
│  • Anexos técnicos                       │
│  • Datos de soporte                      │
└──────────────────────────────────────────┘
```

**Regla de oro:** Si el lector solo lee la primera página, debe poder tomar la decisión requerida.

---

### Fase 2 — Estructura estándar del documento

**STOP: Antes de generar el .docx, construir en texto plano las 5 secciones obligatorias.**
**No avanzar a la generación si alguna de estas 5 falta.**

#### SECCIÓN 1 — PORTADA ✦ obligatoria
- Título descriptivo del informe
- Responsable | Fecha | Sistema/Área | Período

#### SECCIÓN 2 — RESUMEN EJECUTIVO (TLDR) ✦ obligatoria, siempre primera después de portada
- Párrafo de 3-5 líneas con exactamente: situación actual + hallazgo principal + acción requerida
- Semáforo de estado en línea aparte: `🟢 Normal` / `🟡 Atención` / `🔴 Crítico`
- **Si no hay TLDR, el informe no cumple. No reemplazar con "Antecedentes" ni "Introducción".**

#### SECCIÓN 3 — KPIs PRINCIPALES ✦ obligatoria
- Tabla: `Indicador | Meta | Real | Var% | Estado`
- Si no hay metas definidas, usar `[meta]` como marcador
- Si el informe es técnico (no operativo), derivar KPIs de los entregables: compilación exitosa, archivos modificados, funciones implementadas, etc.
- Máximo 6-8 filas. Los demás KPIs van a anexos.

#### SECCIÓN 4 — HALLAZGOS Y ANÁLISIS ✦ obligatoria
- Subsecciones por área o tema (H2)
- Cada hallazgo sigue: qué pasó → por qué → impacto → evidencia

#### SECCIÓN 5 — PLAN DE ACCIÓN / PRÓXIMOS PASOS ✦ obligatoria, siempre al final
- Tabla: `Acción | Responsable | Fecha compromiso | Estado`
- Si no hay responsables definidos, usar `[responsable]`
- **El informe no puede terminar con una conclusión narrativa sin esta tabla.**

#### SECCIÓN 6 — ANEXOS (opcional)
- Tablas extendidas, datos técnicos, capturas de sistema

---

### Fase 3 — Estándares de diseño aplicados

Al generar el .docx, aplicar estos estándares sin excepción:

#### Tipografía
- Fuente: **Arial** (universal, sin problemas de compatibilidad)
- Cuerpo: 11pt | Interlineado 1.4 | Máx. 80 caracteres por línea
- H1: 18pt negrita | H2: 14pt negrita | H3: 12pt negrita
- Nunca más de 2 niveles de jerarquía tipográfica por sección

#### Márgenes y espacio
- Márgenes mínimos: 2 cm en todos los lados (2880 DXA aprox.)
- Espacio antes de H1: 360 | después: 240
- Espacio antes de H2: 240 | después: 180
- El espacio en blanco no es desperdicio — es respiración visual

#### Color (funcional, no decorativo) — paleta gris corporativa
- Paleta máxima: 2-3 colores — **usar grises, no azules**
- Color primario encabezados H1 (fondo): `#44546A` (gris azulado oscuro — color de la plantilla)
- Color primario encabezados H2 (texto + línea): `#44546A`
- Color H3 (texto): `#7F7F7F`
- Header de tabla: fondo `#44546A`, texto blanco
- Filas alternas tabla: blanco / `#D9D9D9` (gris claro neutro)
- Color alerta crítica: `#C00000` | alerta media: `#ED7D31`
- Fondo semáforo verde: `#E2EFDA` | amarillo: `#FFF2CC` | rojo: `#FFE7E7`
- **Prohibido:** azules `#1F4E79` o `#D6E4F0` — no corresponden a la paleta de la plantilla

#### Tablas
- Header con fondo `#44546A` y texto blanco en negrita
- Filas alternas: blanco / `#D9D9D9` (gris neutro)
- Bordes: `CCCCCC` sutiles, nunca negro grueso
- Usar sombreado de celda compatible con la librería disponible; evitar configuraciones que produzcan colores sólidos incorrectos o ilegibles
- Padding de celda: `top: 80, bottom: 80, left: 120, right: 120`

#### Gráficos (si aplica)
- Barras: comparaciones entre categorías
- Líneas: tendencias y evolución temporal
- **Prohibido:** gráficos 3D, tortas con más de 4 categorías
- Siempre etiquetar datos directamente en el gráfico

---

### Fase 4 — Generación del .docx

**OBLIGATORIO: El output siempre parte de `assets/plantilla-informe.docx`. Nunca crear un .docx desde cero.**

#### Paso 1 — Copiar y desempacar la plantilla

```bash
cp assets/plantilla-informe.docx /tmp/informe-output.docx
cd /tmp && unzip -o informe-output.docx -d informe-unpacked/
```

**CRÍTICO: Limpiar párrafos vacíos de la plantilla antes de insertar contenido.**
La plantilla tiene ~30 párrafos vacíos después de los campos de portada. Eliminarlos con Python:

```python
import re

path = '/tmp/informe-unpacked/word/document.xml'
xml = open(path).read()

# Preservar los 4 párrafos de portada + sectPr, eliminar vacíos intermedios
# Los párrafos vacíos tienen esta forma (sin <w:t> ni page break):
xml = re.sub(
    r'<w:p [^>]*>(?:(?!<w:t|w:type="page").)*?</w:p>\s*',
    lambda m: m.group(0) if '<w:t' in m.group(0) or 'w:type="page"' in m.group(0) else '',
    xml, flags=re.DOTALL
)

open(path, 'w').write(xml)
```

#### Paso 2 — Rellenar la portada en `informe-unpacked/word/document.xml`

La plantilla tiene exactamente estos 4 campos de texto que deben reemplazarse:

| Texto en XML | Reemplazar con |
|---|---|
| `<w:t>TÍTULO</w:t>` | `<w:t>Título real del informe</w:t>` |
| `<w:t>Responsable:</w:t>` | `<w:t>Responsable: Nombre Apellido</w:t>` |
| `<w:t>Fecha:</w:t>` | `<w:t>Fecha: DD/MM/AAAA</w:t>` |
| `<w:t>Sistema:</w:t>` | `<w:t>Sistema: Nombre del sistema o área</w:t>` |

#### Paso 2b — Inyectar estilos de encabezado en `informe-unpacked/word/styles.xml`

**La plantilla no define Heading1/Heading2. Sin este paso los subtítulos se ven iguales al cuerpo.**

Agregar antes del cierre `</w:styles>`:

```xml
<w:style w:type="paragraph" w:styleId="Heading1">
  <w:name w:val="heading 1"/>
  <w:basedOn w:val="Normal"/>
  <w:pPr>
    <w:spacing w:before="360" w:after="120"/>
    <w:shd w:val="clear" w:color="auto" w:fill="44546A"/>
  </w:pPr>
  <w:rPr>
    <w:rFonts w:ascii="Arial" w:hAnsi="Arial"/>
    <w:b/>
    <w:color w:val="FFFFFF"/>
    <w:sz w:val="36"/>
  </w:rPr>
</w:style>
<w:style w:type="paragraph" w:styleId="Heading2">
  <w:name w:val="heading 2"/>
  <w:basedOn w:val="Normal"/>
  <w:pPr>
    <w:spacing w:before="240" w:after="80"/>
    <w:pBdr>
      <w:bottom w:val="single" w:sz="4" w:space="1" w:color="44546A"/>
    </w:pBdr>
  </w:pPr>
  <w:rPr>
    <w:rFonts w:ascii="Arial" w:hAnsi="Arial"/>
    <w:b/>
    <w:color w:val="44546A"/>
    <w:sz w:val="28"/>
  </w:rPr>
</w:style>
<w:style w:type="paragraph" w:styleId="Heading3">
  <w:name w:val="heading 3"/>
  <w:basedOn w:val="Normal"/>
  <w:pPr><w:spacing w:before="180" w:after="60"/></w:pPr>
  <w:rPr>
    <w:rFonts w:ascii="Arial" w:hAnsi="Arial"/>
    <w:b/>
    <w:color w:val="7F7F7F"/>
    <w:sz w:val="24"/>
  </w:rPr>
</w:style>
```

**Resultado visual (paleta gris corporativa):**
- H1: fondo `#44546A`, texto blanco, 18pt negrita
- H2: texto `#44546A`, línea inferior gris, 14pt negrita
- H3: texto `#7F7F7F`, 12pt negrita

#### Paso 2c — Corregir el footer con número de página real

El footer de la plantilla tiene texto hardcodeado `22` antes del campo `PAGE`. Reemplazarlo en `word/footer1.xml`:

Buscar y eliminar: `<w:r><w:t>22</w:t></w:r>` (el texto basura antes de `<w:fldChar w:fldCharType="begin"/>`)

El footer correcto debe quedar solo con el campo de página automático:
```xml
<w:r><w:fldChar w:fldCharType="begin"/></w:r>
<w:r><w:instrText>PAGE \* MERGEFORMAT</w:instrText></w:r>
<w:r><w:fldChar w:fldCharType="separate"/></w:r>
<w:r><w:t>1</w:t></w:r>
<w:r><w:fldChar w:fldCharType="end"/></w:r>
```

#### Paso 3 — Agregar contenido del informe

#### Paso 3 — Agregar contenido del informe

Insertar el contenido **antes del `<w:sectPr>`** que cierra el `<w:body>`, en este orden exacto:

```xml
<!-- Page break después de portada -->
<w:p><w:r><w:br w:type="page"/></w:r></w:p>

<!-- SECCIÓN 2: Resumen Ejecutivo -->
<w:p><w:pPr><w:pStyle w:val="Heading1"/></w:pPr><w:r><w:t>Resumen Ejecutivo</w:t></w:r></w:p>
<w:p><w:r><w:t>[situación actual]. [hallazgo principal]. [acción requerida].</w:t></w:r></w:p>
<w:p><w:r><w:t>Estado general: 🟢 Normal / 🟡 Atención / 🔴 Crítico</w:t></w:r></w:p>

<!-- SECCIÓN 3: KPIs Principales (tabla) -->
<w:p><w:pPr><w:pStyle w:val="Heading1"/></w:pPr><w:r><w:t>KPIs Principales</w:t></w:r></w:p>
<!-- tabla: Indicador | Meta | Real | Var% | Estado -->

<!-- SECCIÓN 4: Hallazgos y Análisis -->
<w:p><w:pPr><w:pStyle w:val="Heading1"/></w:pPr><w:r><w:t>Hallazgos y Análisis</w:t></w:r></w:p>

<!-- SECCIÓN 5: Plan de Acción -->
<w:p><w:pPr><w:pStyle w:val="Heading1"/></w:pPr><w:r><w:t>Plan de Acción</w:t></w:r></w:p>
<!-- tabla: Acción | Responsable | Fecha compromiso | Estado -->

<!-- ÚLTIMA LÍNEA OBLIGATORIA — siempre al final del documento, antes del sectPr -->
<w:p>
  <w:pPr><w:jc w:val="center"/><w:rPr><w:i/><w:b/><w:color w:val="7F7F7F"/></w:rPr></w:pPr>
  <w:r><w:rPr><w:i/><w:b/><w:color w:val="7F7F7F"/></w:rPr>
    <w:t>*** última línea ***</w:t>
  </w:r>
</w:p>
```

#### Paso 4 — Reempacar

```bash
cd /tmp/informe-unpacked && zip -r ../informe-final.docx . && cp /tmp/informe-final.docx /mnt/user-data/outputs/informe-[nombre].docx
```

#### Configuración de página ya definida en la plantilla (no modificar)

- Tamaño: carta (12240 x 15840 DXA)
- Márgenes: top 709, right 1701, bottom 1417, left 1701
- Header y footer ya configurados (`rId11`, `rId12`, `rId13`, `rId14`)

Para tareas .docx complejas, seguir las instrucciones del skill de documentos disponible en el entorno.

---

### Fase 5 — Checklist de calidad antes de entregar

**Verificar cada punto. Si alguno falla, corregir antes de generar el archivo.**

- [ ] ¿La primera sección después de portada es el TLDR? (no "Antecedentes", no "Introducción")
- [ ] ¿El TLDR tiene situación actual + hallazgo principal + acción requerida?
- [ ] ¿Hay semáforo de estado visible (🟢/🟡/🔴)?
- [ ] ¿Hay tabla de KPIs con Indicador | Meta | Real | Estado?
- [ ] ¿Cada hallazgo tiene una acción asociada en el plan?
- [ ] ¿El documento cierra con tabla de Plan de Acción (no con párrafo narrativo)?
- [ ] ¿La paleta de colores usa grises `#44546A` / `#D9D9D9` (no azules `#1F4E79`)?
- [ ] ¿Las tablas tienen header gris oscuro y filas alternas gris claro?
- [ ] ¿El footer muestra número de página automático (sin texto `22` hardcodeado)?
- [ ] ¿La última línea del documento dice `*** última línea ***` centrada en gris itálica?

---

## Integridad factual y manejo de vacíos

- **No inventes** métricas, fechas, responsables, metas, estados, causas raíz ni conclusiones
- Si falta un dato, usa marcadores claros: `[fecha]`, `[responsable]`, `[meta]`, `[dato pendiente]`
- Si una conclusión es inferida, etiquétala explícitamente como **"inferencia"**
- Si los datos están incompletos, incluye una sección **"Datos pendientes o no confirmados"** al final
- No bloquees la generación del informe salvo que falte el tema central o el destinatario

---

| Tipo | Enfoque | Secciones obligatorias |
|------|---------|----------------------|
| **Ejecutivo** | Decisión / aprobación | TLDR, KPIs, Recomendación |
| **Operativo** | Estado de operaciones | TLDR, KPIs, Incidencias, Plan |
| **Técnico** | Análisis de sistemas | TLDR, Diagnóstico, Evidencia, Solución |
| **Avance de proyecto** | Progreso vs. plan | TLDR, % avance, Riesgos, Próximos pasos |
| **Incidente** | Post-mortem | TLDR, Línea de tiempo, Causa raíz, Acciones |
| **WMS/SAP** | Logística / inventario | TLDR, Fill rate, Brechas, KPIs bodega |

Para cada tipo, adaptar las secciones del Fase 2 manteniendo siempre la estructura de pirámide invertida.

---

## Referencias

- Guía filosófica completa: `references/guia-maestro.md`
- Para generación .docx, seguir las instrucciones del skill de documentos disponible en el entorno.
