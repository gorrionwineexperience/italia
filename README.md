# Cata Interactiva Don Aviador · Vinos Italianos

Versión adaptada a 4 vinos, manteniendo la arquitectura Firebase, dashboards y minijuegos del proyecto original.

## Orden de servicio

1. Grillo — Saverio Faro
2. Passo Del Sud Negroamaro Primitivo — Tagaro
3. Pinataro Primitivo — Tagaro
4. Cinquenoci Primitivo — Tagaro

## Imágenes

La aplicación utiliza únicamente estos nombres, para que puedan sustituirse manualmente sin tocar el código:

- `img/maridaje-01.jpg` a `img/maridaje-04.jpg`
- `img/info-scene-1.jpg` a `img/info-scene-4.jpg`
- `img/logoredondo.png`

Los archivos `05` se han eliminado de esta copia y no hay referencias a ellos en el código. Las imágenes `01–04` se conservan desde el proyecto original únicamente como archivos de sustitución: debes reemplazarlas manualmente por las nuevas cuando las tengas.

## URLs principales

- `index.html` — invitado
- `index.html?host=1` — host
- `index.html?dashboard=1` — dashboard principal
- `dashboard-info.html` — pantalla de apoyo visual
- `dashboard-minijuegos.html` — ranking de juegos
- `burbujas.html` — Bubble Pop de aromas adaptado a los 4 vinos
- `linea.html` — test de equilibrio
- `tetris.html` — demo Aroma Tetris adaptada al Grillo
- `chuleta.html` — guion de cata
- `chuletaa.html` — guion ampliado y natural

## Fuentes de contenido vinícola

Información contrastada con Saverio Faro / Progetti Agricoli, Consorzio di Tutela Vini DOC Sicilia, Tagaro, ficha técnica de Passo del Sud, Regione Puglia y fichas comerciales actuales para Pinataro. La graduación puede variar ligeramente según añada.


## Correcciones acumuladas

- Bug 1: el estado `summary` devuelve correctamente los móviles desde `burbujas.html` y `linea.html` a `index.html`.
- Bug 2: los resultados de minijuegos se guardan por `gw_device` en vez de por nombre, evitando que dos participantes con el mismo nombre se sobrescriban.
- Bug 3: el progreso de Bubble Pop y del Test alcoholemia queda guardado por sesión en el navegador; recargar ya no devuelve vinos ni intentos, y un Reset del host inicia una sesión limpia.

### Bug 4 · Dashboard de minijuegos al alternar con Resumen
Al entrar en `summary`, el dashboard desconecta ahora el listener del ranking individual que estuviera activo (`bubbles` o `linea`). Esto evita que un resultado tardío del juego anterior vuelva a cambiar la vista y deje el podio final sin actualizar correctamente.

### Bugs 5–11 · Puntuaciones y equidad de minijuegos
- Línea: un intento ya no comienza hasta recibir una lectura real del sensor; si no hay datos, no consume intento ni puede producir un 100 % falso.
- Línea: `Recalibrar centro` queda bloqueado durante un intento activo.
- Línea: el cronómetro usa tiempo real y la puntuación acreditada por frame está limitada, evitando sumar segundos extra al volver de una pestaña suspendida. Cada intento queda limitado a 15 s.
- Línea: el ranking utiliza los milisegundos reales acumulados dentro de la zona durante los 3 intentos. El porcentaje sigue mostrándose como dato legible, pero el timestamp ya no desempata.
- Bubble Pop: una burbuja solo puede resolverse una vez, incluso con doble toque o multitouch.
- Bubble Pop: cada vino genera exactamente 42 burbujas: 25 aromas correctos y 17 distractores, en orden aleatorio. Los aromas correctos que queden sin resolver al terminar cuentan como escapados.
- Bubble Pop: el podio final solo incluye participantes que hayan completado los 4 vinos; los resultados parciales siguen apareciendo como provisionales en el ranking en directo.
- Bubble Pop: los timestamps ya no se usan como criterio de desempate.
- `index.html` y `dashboard-minijuegos.html` comparten las mismas reglas de Top 3 para evitar clasificaciones distintas entre móvil y pantalla.


## Nuevo mini-juego · Flappy Wine

- Nuevo archivo `flappy.html`.
- El host lo activa con `miniGames/currentScreen = "flappy"`.
- Cada invitado dispone de 3 intentos por sesión; recargar no recupera intentos.
- Ranking: mejor número de puertas superadas; la distancia exacta y después el total de puertas de los 3 intentos sirven como desempate, sin usar timestamps.
- Resultados: `miniGames/flappy/results/{deviceId}`.
- Integrado en el dashboard y en el podio final junto a Bubble Pop y Línea.


## Clasificación general de mini-juegos

Se ha añadido un campeón general calculado únicamente entre participantes que hayan completado los tres mini-juegos. Cada juego pesa exactamente un 33,3 %.

Para hacer comparables las escalas, cada resultado se normaliza de 0 a 100 respecto al mejor resultado de la sesión entre los participantes elegibles:

- Bubble Pop: puntos Bubble del jugador / mejor puntuación Bubble.
- Test alcoholemia: milisegundos totales dentro de la zona / mejor tiempo de la sesión.
- Flappy Wine: mejor distancia recorrida / mejor distancia de la sesión.
- Puntuación global = (Bubble normalizado + Línea normalizado + Flappy normalizado) / 3.

No se usan timestamps para desempatar. Si dos participantes obtienen exactamente la misma puntuación global, quedan empatados a efectos de cálculo.

## Refuerzo de sesión, anti-recarga y sincronización

- Bubble Pop consume el vino en el momento de iniciar la partida. Si la página se recarga o se abandona durante el juego, ese vino se recupera como `interrupted` y no puede repetirse.
- En Bubble Pop, de las 25 burbujas correctas por vino, toda la que no se acierta cuenta como escapada. La fórmula se mantiene explícitamente como `(+2 × aciertos) - (3 × fallos) - (2 × escapadas)`.
- Línea consume el intento justo cuando, tras validar el sensor, comienzan los 15 segundos. Si se recarga durante el intento, queda consumido como `interrupted`.
- `index.html` ya no redirige a un invitado tardío hacia un minijuego hasta que tenga nombre y `sessionVersion` válida. Además, `burbujas.html`, `linea.html` y `flappy.html` validan por sí mismos la identidad/sesión contra Firebase y devuelven a `index.html` si no es válida.
- Bubble Pop, Línea y Flappy Wine reintentan la publicación de resultados guardados localmente al volver a abrir el juego. Así, un fallo temporal de Firebase/Wi-Fi no obliga a repetir una partida ya consumida.

## Demo visual · Flappy Wine

- Antes del primer intento de la sesión, `flappy.html` reproduce automáticamente una demostración visual de unos segundos.
- La demo muestra el impulso de cada toque, la caída por gravedad y cómo atravesar el hueco entre barricas.
- Incluye indicaciones visuales, pulsación animada y botón `Saltar`.
- La demo no consume intentos, no escribe puntuación y no modifica Firebase.
- Tras verla, queda disponible el botón `👀 Ver demo` para repetirla antes de cualquier intento pendiente.

- Precisión Bubble: se calcula globalmente como `aciertos totales / (aciertos totales + fallos totales) × 100`; no se usa una media de porcentajes por vino.

## Fórmula definitiva Bubble Pop

- Acierto: +2 puntos.
- Fallo (tocar una burbuja incorrecta): -2 puntos.
- Aroma correcto que se escapa: -1 punto.
- Burbuja incorrecta que se escapa: 0 puntos.
- Fórmula: `(aciertos × 2) - (fallos × 2) - escapadas buenas`.
- Precisión: `aciertos / (aciertos + fallos) × 100`.


## Podios y clasificación general
- Flappy Wine: el resultado final es la media de puertas de los 3 intentos; la distancia media desempata.
- Clasificación general absoluta 0–100: Bubble = puntos/200, Línea = tiempo dentro/45 s, Flappy = media de puertas/28.
- Cada mini-juego pesa 33,3 % y un 100/100 representa un rendimiento máximo real, no ser el mejor de la sesión.
