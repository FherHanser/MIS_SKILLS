# Criterios prácticos (probados en un producto real)

Decisiones que salieron de iterar con una persona usuaria exigente en una app de lectura. Están generalizadas: aplícalas a cualquier interfaz cuando el caso se parezca.

## Contenido
1. Vista previa vs. acción
2. Lo que tapa contenido
3. Controles que se ocultan
4. Movimiento
5. Acciones con costo o irreversibles
6. Invitaciones y descubrimiento
7. Selección y barras contextuales
8. Color y contraste
9. Identidad visual y metáforas
10. Texto de interfaz
11. Agregar funciones sin romper
12. Cómo trabajar con la persona

---

## 1. Vista previa vs. acción
- **Pasar el mouse = vista previa ligera** (un globito, un resaltado). No debe cambiar el estado de la pantalla.
- **Tocar = acción o vista completa.** Por ejemplo: pasar sobre un subrayado muestra su nota resumida; tocar el globito abre la nota completa.
- Un efecto fuerte (difuminar la pantalla, abrir un panel) nunca se dispara solo con pasar el mouse: sorprende y molesta al rozar.
- Si una vista previa se puede tocar, dale señal (cursor de mano, borde al pasar) y un pequeño respiro (~300 ms) para que el mouse llegue a ella sin que desaparezca.

## 2. Lo que tapa contenido
- Un tooltip es para algo breve. Si el contenido puede ser largo (una nota, un párrafo, una imagen), usa una tarjeta que se adapte: ancho con límites (`clamp`), alto máximo relativo a la ventana (~70 %) y desplazamiento interno.
- **Nunca recortes en silencio.** Si algo no cabe, desvanece el final y di que sigue («Leer completa ›»). Un corte a media palabra hace creer que eso es todo.
- La tarjeta se ancla a lo que explica (arriba o abajo según el espacio) y no tapa ese elemento.
- Para enfocar sin perder contexto: atenúa/difumina el fondo **excepto** lo que se está comentando, que queda nítido.
- Cerrar siempre por tres vías: Esc, × y tocar fuera. Si había edición, tocar fuera guarda (no se pierde lo escrito).

## 3. Controles que se ocultan
- En pantallas de inmersión (lectura, video), barras, flechas y hasta el puntero se ocultan tras ~2 s sin mover el mouse, y vuelven al primer movimiento real.
- No se ocultan mientras haya algo abierto (panel, menú, tarjeta).
- Ignora los «movimientos» falsos que genera el navegador cuando cambia el contenido bajo el puntero (compara la posición real).
- Un control persistente pequeño (p. ej., música sonando) se atenúa y se reduce a un icono; al tocarlo lleva al control principal, no abre un segundo control duplicado.

## 4. Movimiento
- El movimiento debe **acompañar, no decorar**: decir «cambió» y «hacia dónde». Un fundido breve (~200 ms) con un desplazamiento mínimo (~10 px) basta.
- Efectos realistas espectaculares (hoja que se curva en 3D) distraen en tareas de concentración; ofrecerlos solo como opción, nunca por defecto.
- Respeta `prefers-reduced-motion` (sin animación) y `prefers-reduced-transparency`/`prefers-contrast` (sin difuminados ni texturas).

## 5. Acciones con costo o irreversibles
- Antes de gastar dinero o tiempo largo, confirma diciendo **qué, cuánto y una sola vez**: «El libro entero (46 371 palabras) por ≈ US$1.01, una sola vez. [Cancelar] [Traducir]».
- Muestra el estado mientras ocurre («se está traduciendo; sigue leyendo») y avisa rápido si falla, con la causa en palabras humanas y cómo reintentar.
- Borrar = papelera recuperable y opción de deshacer, no borrado definitivo.
- Cambios masivos (reorganizar, recalcular) se muestran primero como vista previa («esto moverá 12 elementos») y se guardan con «Deshacer».

## 6. Invitaciones y descubrimiento
- Una función que la persona no sabe que existe se ofrece **una vez**, en contexto y descartable: «Este libro está en inglés · Leerlo en español ×». Si la descarta, no vuelve.
- La misma opción vive también en su lugar permanente (ajustes), para cuando la busque.
- Etiquetas que cambian según el contexto deben decir lo que hacen ahí: si el libro ya está en español, el botón «Traducir» pasa a «Original».

## 7. Selección y barras contextuales
- La barra que aparece al seleccionar se acerca a la selección pero no la tapa.
- Selección de **una palabra** → globito compacto con la acción más probable (definir) y «más»; selección larga → barra completa. Lo que no aplica, no aparece.
- Las zonas invisibles de acción (pasar página tocando el margen) se limitan a los márgenes reales; nunca invaden el texto, o un doble clic para seleccionar dispara otra cosa. Señálalas con una flecha discreta al pasar.

## 8. Color y contraste
- Usa tonos de acento derivados del sistema en vez de blancos puros o negros duros en elementos flotantes; el blanco brillante sobre un fondo cálido rompe la armonía.
- El anillo de foco por defecto del navegador puede romper la armonía, pero el foco **debe verse**: usa el color de acento.
- Mensajes de confirmación fuera del botón (debajo), no dentro, para que el botón no cambie de tamaño.
- Botones que alternan texto («Copiar» → «Copiada ✓») con ancho fijo para que nada salte.
- Calibra contraste de texto largo a ≥ 7:1 para lectura cómoda; el tono puede ser cálido sin perder contraste.

## 9. Identidad visual y metáforas
- Los elementos que dan identidad (en una app de libros: lomo, cinta marcapáginas, repisa, papel) no se quitan por «redundantes»: no son controles, son el lenguaje del producto.
- Un modo o tema nuevo se **agrega** como opción; no se modifican los existentes que la persona ya eligió.
- Las texturas o efectos de realismo (grano de papel, el reverso de la hoja traslúcido) deben **sentirse sin leerse**: muy baja opacidad y desenfoque suficiente para que no compitan con el contenido.

## 10. Texto de interfaz
- Botones con verbo y resultado: «Guardar como nota», «Ver en Notas», «Traducir con Claude».
- Notas de estado en una línea y en lenguaje de la persona: «Traducido en tu equipo, gratis.», no «Motor local CTranslate2».
- Nada de cifras técnicas en material para usuarios finales.
- Evita títulos ambiguos («Nota» para el apunte de una nota): nombra según lo que la persona ve.

## 11. Agregar funciones sin romper
- Antes de proponer, ubica dónde vive lo relacionado y agrégalo ahí (mismo grupo de la barra, mismo panel), siguiendo el orden y espaciado existentes.
- Verifica que lo que ya funcionaba sigue igual (en código: compara la salida antes/después en todos los casos, no solo en el que se arregló).
- Si una mejora implica quitar algo visible que la persona aprobó, pregunta antes.

## 12. Cómo trabajar con la persona
- Cuando pregunta «¿esto cumple los principios?», responde con un veredicto claro primero.
- Una recomendación, con su porqué; pregunta solo si la decisión es genuinamente suya.
- Una idea por mensaje y por imagen; nada que la obligue a preguntar «¿qué demuestras?».
- Si un diagnóstico estaba mal, dilo directo y corrige.
