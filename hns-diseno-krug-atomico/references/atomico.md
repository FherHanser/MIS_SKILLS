# Diseño Atómico — cómo construir sin romper el sistema

Resumen en palabras propias de la metodología de Brad Frost, orientado a código real (HTML/CSS/JS, componentes).

## Contenido
1. Los cinco niveles
2. Tokens: la capa debajo de los átomos
3. Reutilizar antes de crear
4. Estados y variantes
5. Inventario de la interfaz
6. Agregar algo nuevo paso a paso
7. Señales de deuda de sistema

---

## 1. Los cinco niveles
| Nivel | Qué es | Ejemplos |
|---|---|---|
| **Átomos** | Piezas mínimas que no se dividen sin perder sentido | color, tipografía, icono, botón, campo, etiqueta, interruptor |
| **Moléculas** | Átomos que funcionan juntos con un propósito | campo + etiqueta + ayuda; buscador (campo + botón); selector segmentado |
| **Organismos** | Secciones con identidad propia | barra superior, tarjeta de libro, panel de notas, barra de selección |
| **Plantillas** | La estructura de una pantalla con organismos ubicados, sin contenido real | lector: barra arriba + hoja + pie |
| **Páginas** | La plantilla con contenido real; donde se prueba con casos extremos | el lector con un libro largo, uno vacío, uno con títulos enormes |

No es un proceso lineal: se va y vuelve entre niveles. Lo valioso es poder decir «esto es una variante de tal molécula» en lugar de «esto es algo nuevo».

## 2. Tokens: la capa debajo de los átomos
Los tokens son las decisiones de diseño con nombre: colores, espacios, radios, sombras, tipografías, duraciones de animación.
- Define colores por **función**, no por valor: `--text`, `--muted`, `--accent`, `--line`, `--surface`, `--danger`, no `--cafe-claro`.
- Los temas (claro, oscuro, sepia, contraste alto) **redefinen los tokens**, nunca los componentes. Si un componente necesita reglas por tema, le falta un token.
- Un valor escrito a mano (`#a65c25`, `13px`, `border-radius: 7px`) dentro de un componente es una señal de alerta: o existe un token para eso, o hace falta crearlo.
- Respeta las preferencias del sistema como tokens: `prefers-reduced-motion`, `prefers-contrast`, `prefers-color-scheme`.

## 3. Reutilizar antes de crear
Antes de crear un componente:
1. Busca en el código uno que haga lo mismo o casi (por clase, por nombre, por aspecto).
2. Si existe igual: úsalo.
3. Si existe parecido: crea una **variante** (modificador) de ese, no una copia.
4. Solo si no existe nada: crea uno nuevo, con tokens, estados y nombre claro.

Ejemplo: si ya hay una tarjeta flotante para «Traducir», la vista de «Original» o «Definir» debe ser la misma tarjeta con otro contenido y encabezado, no una tarjeta nueva. La persona aprende un solo patrón.

## 4. Estados y variantes
Todo componente interactivo necesita, como mínimo: normal · hover · foco visible (teclado) · presionado/activo · deshabilitado. Según el caso: cargando · vacío · error · seleccionado.
- El foco visible no es opcional y debe usar el color del sistema (no el anillo negro por defecto si rompe la armonía, pero sí uno visible).
- Las variantes se declaran (`.btn.primary`, `.btn.quiet`), no se improvisan con estilos en línea.

## 5. Inventario de la interfaz
Para auditar un sistema existente: captura y agrupa todo lo que hay por tipo (botones, campos, tarjetas, avisos, iconos, colores en uso). Lo que aparece en tres versiones distintas debería ser una. El inventario hace visibles las inconsistencias que pantalla por pantalla parecen inofensivas.

## 6. Agregar algo nuevo paso a paso
1. **Nombra la tarea** que resuelve (Krug primero).
2. **Elige el nivel**: ¿es un átomo nuevo, una molécula con átomos existentes, o un organismo?
3. **Busca la pieza más cercana** que ya exista y parte de ella.
4. **Usa tokens** para todo valor visual; si falta uno, créalo en la raíz y en cada tema.
5. **Define estados** y comportamiento con teclado.
6. **Ubícalo donde la persona lo buscaría**, junto a lo relacionado (en el mismo grupo de la barra, en el mismo panel de ajustes), sin desplazar lo que ya funciona.
7. **Pruébalo en todos los temas y modos** existentes, y con contenido extremo (texto muy largo, vacío, imágenes grandes).

## 7. Señales de deuda de sistema
- El mismo concepto con tres nombres en la interfaz.
- Componentes «casi iguales» que difieren en 2 px o un tono.
- Estilos que dependen de la página en la que están en vez de su variante.
- Reglas `!important` para pelear con otro componente.
- Un tema que se ve bien y otro roto tras un cambio.
- Un archivo de estilos que solo crece: cada cambio agrega reglas y ninguno quita.
