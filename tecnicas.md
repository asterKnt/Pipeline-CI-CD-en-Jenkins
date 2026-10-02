# Matriz de Técnicas de Pruebas ISTQB - HDT 4

## Resumen de la Suite de Regresión

La suite de regresión (`bruno/regresion/`) fue diseñada aplicando técnicas de diseño de pruebas de caja negra basadas en el estándar ISTQB:
- **BVA (Boundary Value Analysis):** Análisis de Valores Frontera.
- **EP (Equivalence Partitioning):** Partición de Equivalencia.
- **DT (Decision Table / Business Rules):** Tablas de Decisión / Reglas de Negocio.
- **ST (State Transition):** Transición de Estados.

---

## Tabla de Casos de Prueba y Técnicas

| # | Request (`.bru`) | Técnica ISTQB | Valor Probado | Resultado Esperado | Resultado Obtenido | Estado |
|---|---|---|---|---|---|---|
| 01 | `01-reset.bru` | Setup / Baseline | Endpoint `POST /reset` | `200 OK`, restablece el estado en memoria a 3 libros iniciales. | `200 OK`, `{"status": "reset", "books": 3}` | PASS |
| 02 | `02-bva-year-min-valid.bru` | BVA | `year = 1450` (Límite inferior válido) | `201 Created`, libro persistido con año 1450. | `201 Created`, `year = 1450` | PASS |
| 03 | `03-bva-year-below-min-invalid.bru` | BVA | `year = 1449` (Valor inmediatamente inferior al mínimo) | `422 Unprocessable Entity` por validación de esquema. | `422 Unprocessable Entity` | PASS |
| 04 | `04-bva-year-max-valid.bru` | BVA | `year = 2100` (Límite superior válido) | `201 Created`, libro persistido con año 2100. | `201 Created`, `year = 2100` | PASS |
| 05 | `05-bva-year-above-max-invalid.bru` | BVA | `year = 2101` (Valor inmediatamente superior al máximo) | `422 Unprocessable Entity` por validación de esquema. | `422 Unprocessable Entity` | PASS |
| 06 | `06-bva-copies-zero-valid.bru` | BVA | `copies = 0` (Límite inferior válido de existencias) | `201 Created`, libro persistido con copies = 0. | `201 Created`, `copies = 0` | PASS |
| 07 | `07-bva-copies-negative-invalid.bru` | BVA | `copies = -1` (Valor inferior al mínimo permitido) | `422 Unprocessable Entity` por validación de esquema. | `422 Unprocessable Entity` | PASS |
| 08 | `08-ep-empty-title-invalid.bru` | EP | `title = ""` (Cadena vacía, partición de longitud < 1) | `422 Unprocessable Entity` por regla de longitud obligatoria [1, 120]. | `422 Unprocessable Entity` | PASS |
| 09 | `09-ep-filter-author-case-insensitive.bru` | EP | Query param `?author=feathers` (Partición válida en minúsculas) | `200 OK`, retorno de elementos cuyo autor contiene la subcadena sin distinguir mayúsculas. | `200 OK`, lista contiene "Michael Feathers" | PASS |
| 10 | `10-ep-get-nonexistent-id-404.bru` | EP | `book_id = 99999` (Partición inválida de ID no existente) | `404 Not Found` con mensaje descriptivo. | `404 Not Found` | PASS |
| 11 | `11-decision-put-missing-fields-invalid.bru` | DT | `PUT /books/1` enviando solo `{"title": "..."}` sin `author` ni `year` | `422 Unprocessable Entity`, pues PUT exige el reemplazo total del recurso. | `422 Unprocessable Entity` | PASS |
| 12 | `12-decision-patch-partial-valid.bru` | DT | `PATCH /books/1` enviando únicamente `{"copies": 10}` | `200 OK`, actualiza únicamente el campo enviado y preserva los existentes. | `200 OK`, `copies = 10` y demás atributos intactos | PASS |
| 13 | `13-state-transition-delete-lifecycle.bru` | ST | `DELETE /books/2` sobre recurso existente | `204 No Content`, transición de estado: Activo -> Eliminado. | `204 No Content` | PASS |
| 14 | `14-state-transition-verify-deleted-404.bru` | ST | `GET /books/2` inmediatamente después de la eliminación | `404 Not Found`, verificación de que el recurso permanece en estado Eliminado. | `404 Not Found` | PASS |

---

## Hallazgos y Discrepancias Frente a la Especificación

Durante la ejecución de la suite de regresión contra la API (`api.py`):
1. **Conformidad con la Especificación:** Todas las validaciones de límites de dominio (`year` en [1450, 2100], `copies` en [0, 999], `title` y `author` con longitud mínima de 1) se cumplen de acuerdo con la especificación.
2. **Diferenciación de Métodos HTTP:** La distinción entre reemplazo total (`PUT` con validación estricta de campos obligatorios requeridos por `BookIn`) y actualización parcial (`PATCH` con campos opcionales vía `BookPatch`) funciona según lo esperado.
3. **Consistencia en Ciclo de Vida:** La eliminación de recursos (`DELETE`) opera de forma destructiva y produce `204 No Content`, cambiando el estado a no disponible y retornando `404 Not Found` en subsecuentes lecturas o reintentos de borrado.
