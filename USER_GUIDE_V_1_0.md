# 📘 FlexDB API - Guía de Inicio Rápido

**Versión de API:** v1.1

Bienvenido a la documentación de FlexDB. Aquí encontrarás todo lo que necesitas para empezar a integrar y utilizar nuestra API en tus aplicaciones.

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
- **`options` (opcional):** Opciones para paginar, ordenar o limitar los resultados.

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
        "limit": 50
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

- **`documents`:** Un array que contiene uno o más objetos JSON a guardar.

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
    "options": { "multi": true }
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

Aquí tienes los operadores más comunes para usar dentro del objeto `filter`.

| Operador | Descripción | Ejemplo |
|---|---|---|
| **(implícito)** | Igual a | `{"ciudad": "Madrid"}` |
| `$gt` | Mayor que | `{"edad": { "$gt": 18 }}` |
| `$gte` | Mayor o igual que | `{"precio": { "$gte": 99.99 }}` |
| `$lt` | Menor que | `{"stock": { "$lt": 10 }}` |
| `$lte` | Menor o igual que | `{"calificacion": { "$lte": 5 }}` |
| `$ne` | No es igual a | `{"status": { "$ne": "archivado" }}` |
| `$in` | El valor está en un array | `{"categoria": { "$in": ["tecnologia", "hogar"] }}` |
| `$nin` | El valor no está en un array | `{"rol": { "$nin": ["admin", "superadmin"] }}` |
| `$like` | Búsqueda de texto (comodín `%`) | `{"nombre": { "$like": "Laptop%" }}` |
| `$between` | El valor está entre dos números | `{"amount": { "$between": [100, 500] }}` |
| `$or` | Cumple una de varias condiciones | `{"$or": [{"stock": 0}, {"activo": false}]}` |
| `$exists` | El campo existe (o no) | `{"email": { "$exists": true }}` |

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
