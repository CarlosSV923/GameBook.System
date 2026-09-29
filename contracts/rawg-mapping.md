# Mapeo y límites de RAWG

Este documento cierra la parte RAWG de `GB-002.05`. La fuente externa es la
[documentación oficial de RAWG](https://rawg.io/apidocs) y su
[referencia OpenAPI](https://api.rawg.io/docs/). El navegador nunca llama a
RAWG directamente: el servidor de Next.js añade la API key, valida parámetros
y devuelve solo los campos necesarios.

## 1. Operaciones externas permitidas

| Necesidad de GameBook | Operación RAWG | Parámetros que puede construir Next.js | Límite del proxy |
| --- | --- | --- | --- |
| Catálogo inicial y búsqueda | `GET https://api.rawg.io/api/games` | `search`, `dates`, `platforms`, `ordering`, `page`, `page_size` | Solo valores validados; el catálogo inicial fija `ordering=-rating`. |
| Detalle de un juego | `GET https://api.rawg.io/api/games/{id}` | `id` entero RAWG proveniente de una selección o ruta validada | Nunca acepta una URL externa arbitraria ni un slug no validado desde el navegador. |
| Sugerencias públicas de juegos | `GET https://api.rawg.io/api/games` | `search`, `page=1`, `page_size=10` | Debounce, cancelación de solicitudes obsoletas y respuesta acotada. |
| Sugerencias de plataformas | `GET https://api.rawg.io/api/platforms` | `page`, `page_size` | Next.js obtiene páginas acotadas, filtra por nombre y cachea brevemente; no asume que RAWG ofrece búsqueda de plataforma si no está documentada. |

RAWG exige una API key en cada solicitud. `RAWG_API_KEY` solo existe en el
entorno servidor de Next.js, nunca en una variable `NEXT_PUBLIC_`, un bundle,
una petición del navegador o un log. La documentación oficial muestra los
parámetros `dates`, `platforms`, `page` y `page_size`, y el orden `-rating` para
consultas de mayor puntuación.

## 2. Parámetros y normalización

### 2.1 Juegos

- `search`: texto UTF-8 recortado, de 1 a 100 caracteres; el proxy no permite
  operadores, URL ni parámetros adicionales. La búsqueda de sugerencias usa
  `page_size=10`.
- `platforms`: lista de IDs enteros positivos separados por coma, producida por
  una sugerencia de plataforma. No se reenvían nombres como si fueran IDs.
- `dates`: intervalo ISO `YYYY-MM-DD,YYYY-MM-DD`. Un año se transforma en
  `YYYY-01-01,YYYY-12-31`; dos años forman un intervalo inclusivo. El proxy
  rechaza fechas imposibles y un inicio posterior al fin.
- `ordering`: solo se permite `-rating` para la vista inicial y los valores
  necesarios por una ruta explícita; no se expone un selector libre de campos
  de ordenación al navegador.
- `page`: entero entre 1 y 1000. Next.js fija `page_size=20` para catálogo y
  no reenvía un tamaño arbitrario proporcionado por el cliente.
- Si se combinan nombre, plataforma y año, Next.js envía todos los parámetros
  para que RAWG aplique la intersección; no ejecuta consultas separadas y luego
  inventa una unión local.

La respuesta paginada de RAWG se interpreta como `{ count, next, previous,
results }`. El frontend recibe una forma propia con `items`, `page`,
`hasNext` y un estado de fin; no se exponen URLs `next`/`previous` sin validar.

### 2.2 Plataformas

La referencia oficial documenta `GET /platforms` con `page`, `page_size` y una
respuesta paginada de plataformas que contiene al menos `id` y `name`. Next.js
usa páginas de hasta 50 elementos, recorta la respuesta a un máximo de 20
sugerencias y devuelve `{ id, name }`. Si el catálogo remoto no puede
consultarse, la interfaz muestra un error recuperable y no convierte una lista
parcial en una afirmación de que no existen plataformas.

## 3. Mapeo de respuestas

### 3.1 Tarjeta y favorito

| Campo GameBook | Lista RAWG | Detalle RAWG | Regla |
| --- | --- | --- | --- |
| `rawgId` | `id` | `id` | Entero positivo; es la identidad externa estable. |
| `name` | `name` | `name` | Texto recibido, sin traducir por GameBook. |
| `released` | `released` | `released` | Fecha ISO `YYYY-MM-DD` o `null`; el año visible se deriva sin inventar una fecha completa. |
| `imageUrl` | `background_image` | `background_image` | URI recibida o `null`; no se sustituye por una URL enviada por el cliente. |
| `rating` | `rating` | `rating` | Número RAWG, normalmente en la escala mostrada por RAWG; se conserva `0` si está presente y se usa `null` solo cuando falta. |
| `platforms[].id` | `platforms[].platform.id` | `platforms[].platform.id` | ID entero de RAWG para filtrar. |
| `platforms[].name` | `platforms[].platform.name` | `platforms[].platform.name` | Nombre mostrado sin traducir. |

La instantánea persistida en Game contiene solo estos campos básicos. No se
persisten descripciones, géneros, desarrolladores ni la respuesta RAWG
completa. Una actualización de instantánea ocurre únicamente después de que
el detalle haya respondido correctamente.

### 3.2 Modal de detalle

| Campo de la vista | Campo RAWG de detalle | Ausencia |
| --- | --- | --- |
| Descripción | `description` | Mostrar estado sin descripción; nunca fabricar texto. |
| Géneros | `genres[].name` | Mostrar colección vacía/estado ausente. |
| Desarrolladores | `developers[].name` | Mostrar colección vacía/estado ausente. |
| Fecha completa | `released` | Mostrar solo lo que entregue RAWG; la tarjeta puede conservar año nulo. |
| Campos de tarjeta | Los campos de §3.1 | Reutilizar la respuesta de detalle para actualizar la instantánea básica. |

El contenido externo se trata como texto no confiable: no se ejecuta HTML ni se
inyectan descripciones sin escapar. Los datos RAWG permanecen en el idioma en
que los entrega el servicio; solo los mensajes propios de GameBook se
traducen.

## 4. Sugerencias, paginación y cuota

- Las sugerencias públicas de juegos consultan `/games` con texto parcial,
  `page=1` y `page_size=10`; se cancelan respuestas obsoletas cuando cambia la
  entrada.
- Las sugerencias públicas de plataformas se obtienen del listado documentado
  de `/platforms`, se filtran por nombre en el servidor y se limitan a 20
  resultados.
- Catálogo y favoritos mantienen la paginación separada. El frontend solicita
  la siguiente página solo cuando `hasNext` es verdadero y descarta respuestas
  de filtros anteriores.
- El proxy usa cache breve solo para consultas públicas repetibles; nunca
  cachea respuestas privadas de favoritos ni mezcla datos entre cuentas.
- La cuenta gratuita de RAWG documenta hasta 20 000 solicitudes mensuales y
  exige atribución con enlace activo desde cada página que use sus datos o
  imágenes. El diseño de catálogo, detalle y cualquier vista que muestre
  instantáneas RAWG incluye un enlace visible a RAWG.
- No se implementa una sincronización masiva, un precargado indefinido ni una
  llamada RAWG por cada tarjeta de favoritos.

## 5. Ausencias, atribución y límites de seguridad

| Situación | Respuesta de Next.js |
| --- | --- |
| `released` ausente o `tba` | `released=null`; no se muestra un año inventado. |
| Imagen ausente | `imageUrl=null` y sustituto visual local; no se usa una URL arbitraria. |
| Rating ausente | `rating=null`; no se confunde con una puntuación cero. |
| Plataforma, género o desarrollador ausente | Colección vacía/estado ausente, sin valores inventados. |
| API key no configurada | Error interno controlado (`RAWG_NOT_CONFIGURED`); no se hace una llamada remota. |
| RAWG `401`/`403` o respuesta de autenticación inválida | `RAWG_AUTH_FAILED` para el frontend; nunca se revela la API key. |
| RAWG `429` o cuota agotada | `RAWG_RATE_LIMITED`; mostrar estado temporal y respetar reintentos acotados. |
| Timeout, red caída o `5xx` de RAWG | `RAWG_UNAVAILABLE`; en detalle de un favorito se conserva la tarjeta y no se sincroniza. |
| JSON no compatible o campos inesperados | `RAWG_INVALID_RESPONSE`; registrar `requestId` sin guardar el cuerpo sensible. |

Los nombres `RAWG_*` son códigos internos estables de GameBook; RAWG no se
trata como si garantizara esos códigos. El navegador recibe una forma de error
propia, sin URLs internas, API keys, stack traces ni cuerpos completos de
RAWG.

## 6. Fuentes y decisiones verificadas

- La [página oficial de API de RAWG](https://rawg.io/apidocs) exige la API key
  en cada solicitud, muestra ejemplos de `/platforms` y `/games` con `dates` y
  `platforms`, documenta `ordering=-rating` y publica las condiciones de
  atribución/cuota del plan gratuito.
- La [referencia oficial OpenAPI](https://api.rawg.io/docs/) documenta que la
  lista de juegos y plataformas es paginada con `count`, `next`, `previous` y
  `results`, y expone en juegos los campos `id`, `name`, `released`,
  `background_image`, `rating` y `platforms`; el detalle añade `description`,
  `genres` y `developers`.
- Las restricciones de proxy, códigos `RAWG_*`, debounce, cache, límites de
  página y normalización de ausencias son decisiones de GameBook para proteger
  la API key, la cuota y la experiencia; no se presentan como parámetros
  garantizados por RAWG.

No se añade una ruta pública RAWG al contrato de los microservicios ni se
genera código en esta subtarea.
