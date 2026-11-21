---
name: "hns-diseno-krug-atomico"
description: Diseña, audita y mejora interfaces con dos lentes combinados — «No me hagas pensar» (Steve Krug, usabilidad) y Diseño Atómico (Brad Frost, sistemas de componentes) — en español. Úsalo siempre que el usuario pida diseñar o rediseñar una pantalla, un control, un flujo, un modal o un componente; revisar o auditar una UI, una captura o un mockup; preguntar «¿esto cumple los principios de diseño?», «¿es intuitivo?», «¿está bien ubicado?», «¿qué color/icono/texto va aquí?»; o agregar una función nueva a una app con interfaz (web, escritorio, móvil), aunque no mencione a Krug ni a Atomic Design. También al redactar textos de interfaz (botones, avisos, mensajes de error) y al decidir dónde poner una opción nueva sin romper lo que ya funciona.
---

# Diseño: No me hagas pensar + Diseño Atómico

Dos lentes que se complementan:

- **Krug** responde «¿la persona entiende esto sin pensar?» — cada pantalla, cada control, cada palabra.
- **Diseño Atómico** responde «¿esto encaja en el sistema?» — que lo nuevo se arme con piezas que ya existen y que todo se vea y se comporte igual.

Una interfaz puede ser consistente y aun así confusa (falla Krug), o clara pero hecha de piezas sueltas que se contradicen entre pantallas (falla Atómico). Revisa siempre con los dos.

## Las 3 leyes de Krug (el resumen que guía todo)

1. **No me hagas pensar.** Todo debe ser obvio: qué es, si se puede tocar y qué pasa al tocarlo.
2. **Clics sin dudas.** Tres clics obvios valen más que uno que obliga a pensar; importa la certeza de cada paso, no cuántos son.
3. **Omite palabras.** Quita la mitad del texto y luego la mitad de lo que quede: la gente ojea, no lee.

Para la capa visual y de plataforma (contraste, tamaños táctiles, modo oscuro, movimiento, accesibilidad del sistema) apóyate en el skill **apple-design** si está disponible: es el tercer principio que suele acompañar a estos dos.

## Cuándo leer las referencias

- `references/krug.md` — los principios de Krug explicados y su lista de verificación. Léelo al auditar o cuando dudes si algo «hace pensar».
- `references/atomico.md` — niveles, tokens, inventario y cómo agregar algo nuevo sin romper el sistema. Léelo al crear componentes o al tocar estilos.
- `references/criterios-practicos.md` — decisiones concretas que ya se probaron en un producto real (vista previa al pasar el mouse vs. acción al tocar, confirmar gastos, avisos de primera vez, movimiento, controles que se ocultan…). Léelo antes de proponer un patrón de interacción.

## Flujo de trabajo

### 1. Entiende qué intenta hacer la persona usuaria
Antes de opinar sobre colores o posición, nombra la tarea en una frase: «quiere volver a la nota que escribió», «quiere saber si el libro ya terminó de traducirse». El diseño se juzga contra esa tarea, no contra el gusto.

### 2. Mira con ojos de Krug (primer vistazo)
Pregúntate, en este orden:
1. **¿Se entiende en 3 segundos qué es y qué puedo hacer?** Si hace falta leer un párrafo, falla.
2. **¿Lo que se puede tocar parece tocable, y lo que no, no?** (cursor, forma, contraste, estado al pasar el mouse).
3. **¿Cada elección es obvia?** Si hay que dudar entre dos botones, sobra uno o falta un nombre mejor.
4. **¿Sobra algo?** Texto de relleno, instrucciones, adornos, dos caminos para lo mismo.
5. **¿Sorprende?** Nada debe pasar «solo» sin que la persona lo pida (difuminar la pantalla al rozar algo, mover lo que estaba leyendo, gastar dinero).

### 3. Ubícalo en el sistema (Atómico)
1. ¿Qué nivel es? átomo (botón, color, icono) → molécula (campo + etiqueta + botón) → organismo (barra, tarjeta, panel) → plantilla → página.
2. ¿Ya existe una pieza que lo resuelva? **Reutiliza antes de crear.** Si existe algo parecido, extiéndelo (una variante) en vez de hacer uno nuevo.
3. ¿Usa los tokens del sistema (colores, espacios, radios, tipografía, sombras) o valores sueltos? Valores sueltos = deuda.
4. ¿Tiene todos sus estados? normal, hover, foco visible, presionado, deshabilitado, cargando, vacío, error.
5. ¿Funciona en todos los temas/modos que ya existen sin tocarlos?

### 4. Propón (una recomendación, no un catálogo)
- Da **una** propuesta concreta y el porqué en una o dos líneas, ligado al principio que la justifica.
- Si de verdad hay una elección abierta que es de la persona (cuál de dos quitar, qué valor por defecto), **pregunta** en una línea; no decidas en silencio.
- **No quites** elementos que la persona ya aprobó o que forman la identidad visual del producto argumentando «duplicidad». «Un control, un camino» aplica a controles, no a elementos de identidad.
- Agrega sin invadir: lo nuevo debe caber en el lenguaje visual existente; no rediseñes lo que funciona para encajar lo nuevo.

### 5. Verifica
- **Prueba del tronco** (ver krug.md): desde cualquier pantalla, ¿se sabe dónde estoy, qué es esto y cómo vuelvo?
- Revisa los estados y los temas (claro, oscuro, contraste alto, movimiento reducido).
- Si puedes ver la interfaz (captura, navegador), mírala de verdad antes de afirmar que quedó bien.

## Cómo responder (Krug también aplica a tus respuestas)

La persona que lee tu respuesta también es usuaria. Aplica lo mismo:
- **Primero la respuesta** en 1–3 líneas: «Casi. Hay una sola falla: …».
- **Una idea por mensaje y por imagen.** Si muestras una captura, que demuestre una sola cosa entendible sin leer el pie.
- Detalle, tablas y alternativas solo si ayudan a decidir o si las piden.
- Nada de jerga para el usuario final (ratios de contraste, nombres de tokens) en material que verá la persona usuaria; esa jerga es para el equipo.
- Si te equivocaste en un diagnóstico, dilo en una línea y sigue.

### Formato para una auditoría o «¿cumple los principios?»

```
**Veredicto:** <Sí / Casi / No> — <la razón principal en una frase>

**Lo que falla** (de mayor a menor impacto)
1. <qué> — <por qué hace pensar o rompe el sistema> → <qué haría>

**Lo que está bien** (solo si ayuda a no tocarlo)

¿<una sola pregunta si hay una decisión abierta>?
```

Mantén la lista corta: los 1–3 problemas que más cuestan. Un problema menor que no hace pensar a nadie no merece entrar.

### Cuando reescribas textos de interfaz: muestra la poda

Haz visible el recorte (tercera ley), así la persona ve qué se ganó sin tener que comparar de memoria:

```
Antes:   «Haga clic aquí para guardar los cambios que ha realizado en su nota»
Después: «Guardar nota»
```

### Cuando pidan crear algo nuevo desde cero (no para retoques ni auditorías)

Solo entonces entrega el desglose completo, porque ahí sí ayuda a construir:

1. **Desglose atómico** — átomos, moléculas y organismos; marca cuáles ya existen en el proyecto (se reutilizan) y cuáles son nuevos.
2. **Por qué se entiende sin pensar** — jerarquía visual (qué destaca y por qué), certeza en cada clic (qué ve la persona antes y después de tocar) y la poda de texto.
3. **Código** — en el stack del proyecto (sus variables CSS, sus componentes, su framework); si no hay proyecto, HTML + CSS con variables. Incluye estados (hover, foco visible, activo, deshabilitado), atributos de accesibilidad (`aria-label`, `role`) y comportamiento con teclado.

Para una pregunta puntual («¿está bien ubicado?», «¿qué color va aquí?») no uses este formato: responde con el veredicto y la recomendación.

## Señales rápidas de que algo falla

| Señal | Principio |
|---|---|
| La persona pregunta «¿y esto qué hace?» o «¿dónde está…?» | Krug: no es autoevidente |
| Dos botones que hacen casi lo mismo, o el mismo en dos lugares con nombres distintos | Krug: elección que hace pensar / Atómico: pieza duplicada |
| Un color, tamaño o radio que no existe en ningún otro lado | Atómico: valor fuera de tokens |
| Un componente nuevo que «se parece» a uno existente pero no es igual | Atómico: variante no declarada |
| Texto que explica cómo usar la pantalla | Krug: la pantalla no se explica sola |
| Algo cambia o se mueve sin que la persona lo pida | Krug: sorpresa |
| Un control importante solo se descubre por accidente | Krug: falta afordancia o señal |
| Un modal/tooltip que corta contenido sin avisar que sigue | Krug: hace creer que eso es todo |
| Funciona en el tema claro pero no en oscuro / contraste alto | Atómico: el componente no usa tokens |
