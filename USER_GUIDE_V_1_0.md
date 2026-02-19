# 📘 FlexDB API - Guía Completa para el Usuario

**Versión de API:** v1.0

Bienvenido a la documentación de referencia de FlexDB. Esta guía contiene todo lo que necesitas para integrar y aprovechar al máximo el poder de la API en tus aplicaciones, desde operaciones básicas hasta consultas avanzadas.

---

## 🚀 Primeros Pasos

### 1. Autenticación

Todas las peticiones a la API deben estar autenticadas. Para ello, necesitas incluir dos cabeceras (headers) en cada petición:

- `x-api-key`: Tu clave de API única. Esta clave te identifica como un cliente válido.
- `x-user-id`: Un identificador único para el usuario final en cuyo nombre se realiza la operación.

> **⚠️ ¡Importante!** Trata tu `x-api-key` como si fuera una contraseña. No la expongas en el código del lado del cliente (frontend).

### 2. Estructura del Endpoint

El endpoint principal de la API es:
`https://ipromos.com.mx/api/flexdb/`

Todas las operaciones se realizan a través de este endpoint base, especificando la versión y la operación en la URL.

---

## ⚙️ Conceptos Clave

- **Colección (Collection):** Similar a una tabla en una base de datos SQL. Es un contenedor para tus documentos. Por ejemplo: `users`, `products`, `orders`.
- **Documento (Document):** Similar a una fila en una tabla SQL, pero en formato JSON. Es la unidad básica de datos.
- **Identificador (`_id`):** Cada documento posee un `_id` único. Si no se proporciona al guardar, el motor genera automáticamente un **UUID v7** (un identificador universal ordenado por tiempo), lo cual es ideal para la indexación y el rendimiento.
- **Normalización de Nombres:** Los nombres de bases de datos y colecciones se normalizan internamente (a minúsculas, reemplazando espacios con guiones bajos) para máxima compatibilidad. Puedes usar nombres de hasta 255 caracteres.
- **Codificación:** El sistema soporta `utf8mb4` de forma nativa, permitiendo el uso de emojis (🎉) y cualquier carácter internacional sin configuración adicional.

---

## Endpoints Principales (CRUD)

Todas las operaciones se realizan mediante peticiones `POST` al endpoint correspondiente, enviando un cuerpo (body) en formato JSON.

### 1. 🔎 `find` - Buscar Documentos

Utiliza esta operación para buscar uno o más documentos que coincidan con tus criterios.

**Endpoint:** `POST /v1/find`

**Cuerpo (Body):**
```json
{
    "collection": "nombre_de_tu_coleccion",
    "filter": { "campo": "valor", ... },
    "options": { "limit": 10, "skip": 0, "sort": { "campo": -1 }, "fields": ["campo1", "campo2"] }
}
```

- **`collection` (requerido):** El nombre de la colección donde quieres buscar.
- **`filter` (requerido):** Un objeto JSON con los criterios de búsqueda. Un objeto vacío `{}` busca todos los documentos.
- **`options` (opcional):**
    - `limit` (number): Máximo de documentos a devolver (Default: **50**, Máximo: 10000).
    - `skip` (number): Número de documentos a saltar para paginación (Default: **0**). ⚠️ **Nota:** Esta opción tiene un bug conocido en el motor de base de datos para conjuntos de datos muy grandes. Úsalo con precaución.
    - `sort` (object): Criterio de ordenación. Ej: `{"campo": -1}` para descendente. ⚠️ **Nota:** El ordenamiento alfabético puede ser impreciso dependiendo de la configuración regional (collation) de la base de datos. El ordenamiento numérico funciona perfectamente.
    - `fields` (array): Lista de campos que quieres que la API te devuelva (proyección). Esto es clave para optimizar el rendimiento al no transferir datos innecesarios. Ej: `["nombre", "email"]`.

**Ejemplo con `curl`:**
```bash
curl --location 'https://ipromos.com.mx/api/flexdb/v1/find' \
--header 'x-api-key: tu_api_key' \
--header 'x-user-id: tu_user_id' \
--header 'Content-Type: application/json' \
--data '{
    "collection": "users",
    "filter": {
        "ciudad": "Madrid",
        "edad": { "$gte": 18 }
    },
    "options": {
        "limit": 50,
        "fields": ["nombre", "email", "perfil.verificado"]
    }
}'
```

### 2. 💾 `save` - Guardar Documentos

Utiliza esta operación para crear uno o varios documentos nuevos.

**Endpoint:** `POST /v1/save`

**Cuerpo (Body):**
```json
{
    "collection": "nombre_de_tu_coleccion",
    "documents": [
        { "campo1": "valor1", ... },
        { "campo1": "valor2", ... }
    ]
}
```

- **`documents`:** Puede ser un **objeto único** para insertar un solo documento, o un **array de objetos** para realizar una inserción masiva (batch insert) de forma optimizada.

**Ejemplo con `curl`:**
```bash
curl --location 'https://ipromos.com.mx/api/flexdb/v1/save' \
--header 'x-api-key: tu_api_key' \
--header 'x-user-id: tu_user_id' \
--header 'Content-Type: application/json' \
--data '{
    "collection": "products",
    "documents": [
        { "name": "Laptop Pro", "price": 1200, "stock": 15 }
    ]
}'
```

### 3. ✏️ `update` - Actualizar Documentos

Utiliza esta operación para modificar documentos existentes.

**Endpoint:** `POST /v1/update`

**Cuerpo (Body):**
```json
{
    "collection": "nombre_de_tu_coleccion",
    "filter": { "campo_a_buscar": "valor" },
    "update": { "$set": { "campo_a_cambiar": "nuevo_valor" } },
    "options": { "multi": true, "upsert": false }
}
```

- **`update`:** Un objeto que especifica los cambios. Usa operadores como `$set` (modificar campo) o `$inc` (incrementar un número).
- **`options.multi`:** Si es `true`, actualiza todos los documentos que coincidan. Por defecto, solo actualiza el primero.
- **`options.upsert`:** Si es `true`, crea un nuevo documento si no encuentra coincidencias con el filtro.

#### Operadores de Actualización Comunes
| Operador | Descripción | Ejemplo |
|---|---|---|
| `$set` | Modifica el valor de un campo | `{"$set": {"status": "active"}}` |
| `$inc` | Incrementa un valor numérico | `{"$inc": {"views": 1}}` |
| `$unset` | Elimina un campo del documento | `{"$unset": {"temporary_field": ""}}` |

**Ejemplo con `curl`:**
```bash
curl --location 'https://ipromos.com.mx/api/flexdb/v1/update' \
--header 'x-api-key: tu_api_key' \
--header 'x-user-id: tu_user_id' \
--header 'Content-Type: application/json' \
--data '{
    "collection": "products",
    "filter": { "name": "Laptop Pro" },
    "update": { "$set": { "price": 1150 } }
}'
```

### 4. 🗑️ `delete` - Eliminar Documentos

Utiliza esta operación para eliminar documentos.

**Endpoint:** `POST /v1/delete`

**Cuerpo (Body):**
```json
{
    "collection": "nombre_de_tu_coleccion",
    "filter": { "campo": "valor" },
    "options": { "multi": true }
}
```

- **`options.multi`:** Si es `true`, elimina todos los documentos que coincidan. Por defecto, solo elimina el primero. **¡Úsalo con cuidado!**

---

## 📖 Cheatsheet de Filtros (Operators)

FlexDB soporta una sintaxis de consulta rica y compatible con MongoDB. Los operadores se clasifican en cuatro categorías según su naturaleza.

#### 🔵 Operadores Primarios (Comparación y Lógica)
Son el núcleo del motor y se implementan directamente.

| Operador | Descripción | Ejemplo |
|---|---|---|
| `$eq` | Igual a (flexible, ignora tipo de dato). | `{"edad": 18}` |
| `$eqExact` | Igualdad estricta (valida valor y tipo). | `{"activo": {"$eqExact": true}}` |
| `$gt` / `$gte` | Mayor que / Mayor o igual que. | `{"edad": { "$gt": 18 }}` |
| `$lt` / `$lte` | Menor que / Menor o igual que. | `{"precio": { "$lt": 100 }}` |
| `$in` | El valor se encuentra en una lista. | `{"rol": { "$in": ["admin", "editor"] }}` |
| `$and` | Y lógico (normalmente implícito). | `{"$and": [{"a":1}, {"b":2}]}` |
| `$or` | O lógico. | `{"$or": [{"a":1}, {"b":2}]}` |
| `$not` | Negación de una condición. | `{"edad": { "$not": { "$lt": 18 } }}` |
| `$regex` | Expresión regular (PCRE). | `{"email": { "$regex": "^admin" }}` |
| `$isnull` | El valor del campo es `null`. | `{"deleted_at": { "$isnull": true }}` |

#### 🟢 Alias Matemáticos y Lógicos
Se expanden a combinaciones de operadores primarios para facilitar la escritura.

| Alias | Se expande a | Descripción | Ejemplo |
|---|---|---|---|
| `$ne` | `NOT $eq` | No es igual a | `{"status": {"$ne": "deleted"}}` |
| `$nin` | `NOT $in` | No está en lista | `{"id": {"$nin": [1, 2]}}` |
| `$between`| `$gte AND $lte` | Rango inclusivo | `{"age": {"$between": [18, 30]}}` |
| `$nor` | `NOT $or` | Ni uno ni otro | `{"$nor": [{"a":1}, {"b":2}]}` |
| `$xor` | O exclusivo | Solo una condición verdadera | `{"$xor": [{"a":1}, {"b":2}]}` |

#### 🟡 Alias de Utilidad (Convenience)
Atajos para operaciones comunes de texto y estructura.

| Alias | Descripción | Ejemplo |
|---|---|---|
| `$exists` | El campo existe | `{"email": {"$exists": true}}` |
| `$isnotnull`| No es nulo | `{"name": {"$isnotnull": true}}` |
| `$hasValue` | No nulo y no vacío | `{"desc": {"$hasValue": true}}` |
| `$like` | SQL LIKE (case-sensitive) | `{"name": {"$like": "J%"}}` |
| `$ilike` | SQL LIKE (case-insensitive) | `{"name": {"$ilike": "j%"}}` |
| `$contains` | Búsqueda universal (string/array/obj) | `{"tags": {"$contains": "promo"}}` |
| `$any` | Intersección de arrays (OR) | `{"tags": {"$any": ["a", "b"]}}` |
| `$hasAll` | Contiene todos (AND) | `{"tags": {"$hasAll": ["a", "b"]}}` |

#### 🔴 Operadores Complejos
Requieren lógica avanzada del motor.

| Operador | Descripción | Ejemplo |
|---|---|---|
| `$size` | Tamaño de array | `{"tags": {"$size": 3}}` |
| `$type` | Tipo de dato SQL | `{"age": {"$type": "INTEGER"}}` |
| `$elemMatch`| Coincidencia en objetos de array | `{"users": {"$elemMatch": {"active": true}}}` |

### Campos Anidados

Puedes buscar en campos dentro de objetos anidados usando la notación de puntos.

```json
"filter": {
    "direccion.ciudad": "Barcelona",
    "perfil.preferencias.notificaciones": true
}
```

---

## 💡 Endpoints de Utilidad

### `GET /health`

Devuelve el estado de salud de la API y su conexión a la base de datos. Es útil para monitoreo.

**Ejemplo con `curl`:**
```bash
curl --location 'https://ipromos.com.mx/api/flexdb/health'
```

---

## 🚨 Manejo de Errores

La API utiliza códigos de estado HTTP estándar para indicar el éxito o fracaso de una petición.

- **`200 OK`**: La petición fue exitosa.
- **`400 Bad Request`**: La petición está mal formada (ej. JSON inválido).
- **`401 Unauthorized`**: Falta la `x-api-key` o es inválida.
- **`404 Not Found`**: El endpoint o la ruta no existe.
- **`429 Too Many Requests`**: Has excedido el límite de peticiones permitidas.
- **`500 Internal Server Error`**: Ocurrió un error inesperado en el servidor.

El cuerpo de la respuesta de un error contendrá un objeto JSON con más detalles:
```json
{
    "ok": 0,
    "errmsg": "Mensaje descriptivo del error."
}
```
