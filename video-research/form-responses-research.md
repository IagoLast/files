# Investigacion: Ver respuestas a formularios del cliente

## Decisiones del guion

- Se documentan DOS flujos para ver las respuestas a formularios de un cliente:
  1. Desde el plan semanal del cliente, donde el formulario completado aparece expandido automaticamente
  2. Desde la seccion "Respuestas a formularios" en el sidebar del cliente
- El cliente de ejemplo es [DEMO] Joe Doe (ID: 699b6249137a67a55274f46a)
- El formulario de ejemplo es "Check-in semanal" con 2 preguntas completadas
- La semana del plan es 16 feb - 22 feb 2026, con el formulario en el sabado 21

## Flujo 1: Ver respuestas desde el plan del cliente

### Paso 1: Navegar al plan del cliente

- **URL:** `https://app.trainerstudio.io/d/customers/699b6249137a67a55274f46a?date=2026-02-16`
- **Accion:** Navegacion directa (ya estamos en el plan del cliente)
- **Resultado:** Se muestra el plan semanal del cliente con los dias de la semana. En el sabado 21 se ve un bloque con el formulario "Check-in semanal" completado (icono verde de check). El formulario aparece ya expandido mostrando las preguntas y respuestas porque `isCompleted` es true.

### Paso 2: Observar el formulario completado en el plan

- **URL:** `https://app.trainerstudio.io/d/customers/699b6249137a67a55274f46a?date=2026-02-16`
- **Accion:** Observar (no requiere click -- el formulario completado se muestra expandido automaticamente)
- **Selector CSS (bloque formulario - cabecera morada):** No hay selector CSS estable; la cabecera tiene `background-color: rgb(147, 51, 234)` (purple.600). Se puede localizar por el `h2` con texto "Check-in semanal".
- **Selector Playwright (heading del formulario):** `page.locator('h2', { hasText: 'Check-in semanal' })`
- **Tipo:** heading (h2) dentro de bloque de formulario
- **Contenido visible:**
  - Cabecera morada con icono de formulario y nombre "Check-in semanal"
  - Icono verde de check (completado) en la esquina superior derecha
  - Pregunta 1: "Como valorarias tu esfuerzo del 1 al 10?" - Respuesta: "7"
  - Pregunta 2: "Como te has sentido durante los entrenamientos?" - Respuesta: "Bien pero esta semana dormi mal y estaba cansado"

### Paso 2b (opcional): Hover sobre el formulario para ver acciones

- **URL:** misma
- **Accion:** Hacer hover sobre el bloque del formulario
- **Selector Playwright:** `page.locator('h2', { hasText: 'Check-in semanal' }).locator('..')`
- **Resultado:** Aparece una barra de acciones en la parte inferior con botones: Editar formulario (icono lapiz morado), Eliminar formulario (icono papelera rojo), y Expandir/Colapsar (icono fold)

## Flujo 2: Ver respuestas desde la seccion "Respuestas a formularios"

### Paso 1: Click en "Respuestas a formularios" en el sidebar

- **URL:** `https://app.trainerstudio.io/d/customers/699b6249137a67a55274f46a?date=2026-02-16`
- **Accion:** Click en el enlace "Respuestas a formularios" del sidebar
- **Selector CSS:** `a[href*="/forms"]`
- **Selector Playwright:** `page.locator('a[href*="/forms"]')`
- **Tipo:** link
- **Obligatorio:** si
- **Resultado:** Navega a `https://app.trainerstudio.io/d/customers/699b6249137a67a55274f46a/forms`. Se muestra la pagina "Respuestas de Formularios" con una barra de filtros y una lista de tarjetas de formularios completados.

### Paso 2: Observar la lista de respuestas

- **URL:** `https://app.trainerstudio.io/d/customers/699b6249137a67a55274f46a/forms`
- **Accion:** Observar la lista de formularios completados
- **Contenido visible:**
  - Titulo: "Respuestas de Formularios"
  - Barra de filtros con: selector de formulario, selector de estado, rango de fechas, boton "Aplicar filtro"
  - Lista de tarjetas colapsadas, cada una mostrando: nombre del formulario, estado (Completado), fecha
- **Tarjetas visibles (ejemplo):**
  - Check-in semanal | Completado | 23/2/2026
  - Check-in semanal | Completado | 21/2/2026
  - Check-in semanal | Completado | 14/2/2026

### Paso 3: Click en una tarjeta para expandir respuestas

- **URL:** `https://app.trainerstudio.io/d/customers/699b6249137a67a55274f46a/forms`
- **Accion:** Click en la segunda tarjeta (21/2/2026) para expandirla y ver las respuestas
- **Selector CSS (primera tarjeta):** `.chakra-card__root:nth-child(1) details summary`
- **Selector CSS (segunda tarjeta):** `.chakra-card__root:nth-child(2) details summary`
- **Selector CSS (tercera tarjeta):** `.chakra-card__root:nth-child(3) details summary`
- **Selector Playwright (segunda tarjeta):** `page.locator('details summary').nth(1)`
- **Tipo:** details/summary (HTML nativo, expandible)
- **Obligatorio:** si
- **Resultado:** La tarjeta se expande mostrando las preguntas y respuestas del formulario:
  - Pregunta 1: "Como valorarias tu esfuerzo del 1 al 10?" - Respuesta: "7"
  - Pregunta 2: "Como te has sentido durante los entrenamientos?" - Respuesta: "Bien pero esta semana dormi mal y estaba cansado"

### Paso 4 (opcional): Observar el contenido expandido

- **URL:** misma
- **Accion:** Observar las preguntas y respuestas
- **Selector CSS (area de respuestas expandida):** `details[open] .chakra-card__body:last-child`
- **Selector Playwright:** `page.locator('details[open] .chakra-card__body').last()`
- **Contenido:** Las preguntas aparecen en negrita (font-weight: bold, font-size: xs) y las respuestas debajo en texto normal (font-size: xs)

## Selectores de referencia

### Sidebar - Navegacion del cliente

| Elemento | Selector CSS (para effects) | Selector Playwright |
|---|---|---|
| Link "Plan" | `a[href$="/699b6249137a67a55274f46a"]` | `page.locator('a', { hasText: 'Plan' }).first()` |
| Link "Respuestas a formularios" | `a[href*="/forms"]` | `page.locator('a[href*="/forms"]')` |

### Plan - Formulario completado

| Elemento | Selector CSS (para effects) | Selector Playwright |
|---|---|---|
| Heading del formulario | `h2` (unico en la vista del plan) | `page.locator('h2', { hasText: 'Check-in semanal' })` |
| Bloque completo del formulario | Usar boundingBox del h2 parent (no hay selector CSS estable por clases hash de Chakra) | `page.locator('h2', { hasText: 'Check-in semanal' }).locator('xpath=ancestor::div[contains(@style,"border")]').first()` |
| Pregunta 1 texto | No hay selector CSS estable | `page.getByText('Como valorarias tu esfuerzo del 1 al 10?')` |
| Respuesta 1 texto | No hay selector CSS estable | `page.getByText('7').first()` |
| Pregunta 2 texto | No hay selector CSS estable | `page.getByText('Como te has sentido durante los entrenamientos?')` |
| Respuesta 2 texto | No hay selector CSS estable | `page.getByText('Bien pero esta semana dormi mal')` |

### Seccion Formularios - Filtros

| Elemento | Selector CSS (para effects) | Selector Playwright |
|---|---|---|
| Select "Seleccionar formulario" | `[role="combobox"]:nth-of-type(1)` (primer combobox) | `page.locator('[role="combobox"]').first()` |
| Select "Todos los estados" | `[role="combobox"]:nth-of-type(2)` (segundo combobox, puede no funcionar por wrapper) | `page.locator('[role="combobox"]').nth(1)` |
| Input fecha "desde" | `input[type="date"]:first-of-type` | `page.locator('input[type="date"]').first()` |
| Input fecha "hasta" | `input[type="date"]:last-of-type` | `page.locator('input[type="date"]').last()` |
| Boton "Aplicar filtro" | No hay selector CSS facil (usar boundingBox) | `page.locator('button', { hasText: 'Aplicar filtro' })` |

### Seccion Formularios - Tarjetas de respuestas

| Elemento | Selector CSS (para effects) | Selector Playwright |
|---|---|---|
| Primera tarjeta (summary) | `.chakra-card__root:nth-child(1) summary` | `page.locator('details summary').nth(0)` |
| Segunda tarjeta (summary) | `.chakra-card__root:nth-child(2) summary` | `page.locator('details summary').nth(1)` |
| Tercera tarjeta (summary) | `.chakra-card__root:nth-child(3) summary` | `page.locator('details summary').nth(2)` |
| Contenido expandido (cuando open) | `details[open] .chakra-card__body:last-child` | `page.locator('details[open] .chakra-card__body').last()` |
| Tarjeta expandida completa | `.chakra-card__root:has(details[open])` | `page.locator('.chakra-card__root').filter({ has: page.locator('details[open]') })` |

## Notas

- **Formulario en el plan se auto-expande:** Cuando `isCompleted` es `true`, el componente `FormBlock` inicializa `showInstructions` como `true`, por lo que las respuestas se muestran automaticamente sin necesidad de hacer click. El estado inicial viene del codigo: `useState(props.value.isCompleted)`.
- **Cabecera morada:** El bloque del formulario en el plan tiene una cabecera morada (`bg="purple.600"`, `rgb(147, 51, 234)`) que lo distingue visualmente de otros tipos de items (ejercicios, tareas, etc.).
- **Icono de completado:** Un icono verde de check (`TbCircleCheckFilled`) aparece en la esquina superior derecha cuando el formulario esta completado.
- **Clases CSS hash:** Chakra UI genera clases CSS con hash (ej: `css-11r7b9m`) que no son estables entre builds. Los selectores deben basarse en: atributos de rol (`[role="combobox"]`), elementos HTML nativos (`details`, `summary`, `h2`), o clases estables de Chakra (`.chakra-card__root`, `.chakra-card__body`).
- **HTML `<details>` nativo:** Las tarjetas de respuestas en la seccion de formularios usan `<details>/<summary>` HTML nativo, lo que permite expandir/colapsar con un simple click en el summary. El atributo `[open]` se agrega automaticamente al expandir.
- **Filtros disponibles:** La barra de filtros permite: filtrar por nombre de formulario (multi-select), filtrar por estado (Completado/Incompleto), y filtrar por rango de fechas (desde/hasta). El boton "Aplicar filtro" ejecuta el filtrado. Hay un boton "Limpiar filtros" que aparece solo cuando hay filtros activos.
- **Estructura de cada respuesta:** Cada tarjeta muestra en el summary: nombre del formulario, estado con icono, y fecha. Al expandir muestra las preguntas (bold, xs) y respuestas (normal, xs) en una lista vertical.
