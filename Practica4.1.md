# Proyecto Quarkus - Persistencia reactiva con Panache

Ejercicio práctico para demostrar la persistencia reactiva usando **Quarkus 3.28.3**, **Hibernate Reactive Panache** y **PostgreSQL**.

---

## Requisitos previos

- Java 21.
- Maven 3.8+.
- PostgreSQL 12+ (corriendo en `localhost:5432`).
- cURL (incluido en Windows 10+, macOS y Linux).

---

## Creación del proyecto

### Paso 1. Crea el proyecto Quarkus.

```bash
mvn io.quarkus.platform:quarkus-maven-plugin:3.28.3:create \
    -DprojectGroupId=pe.banco \
    -DprojectArtifactId=productos-reactive \
    -DprojectVersion=1.0.0-SNAPSHOT \
    -Dextensions="resteasy-reactive-jackson,hibernate-reactive-panache,reactive-pg-client,smallrye-openapi"
```

### Paso 2. Entra al proyecto.

```bash
cd productos-reactive
```

### Paso 3. Crea la base de datos en PostgreSQL.

```bash
psql -U postgres -c "CREATE DATABASE productos_db;"
```

### Paso 4. Configura `application.properties`.

```properties
# PostgreSQL reactivo
quarkus.datasource.db-kind=postgresql
quarkus.datasource.username=postgres
quarkus.datasource.password=postgres
quarkus.datasource.reactive.url=postgresql://localhost:5432/productos_db
quarkus.datasource.jdbc=false

# Hibernate
quarkus.hibernate-orm.database.generation=drop-and-create
quarkus.hibernate-orm.log.sql=true
quarkus.hibernate-orm.sql-load-script=import.sql

# HTTP
quarkus.http.port=8080
```

### Paso 5. Crea datos iniciales (`import.sql`).

```sql
INSERT INTO Producto (id, nombre, descripcion, precio, stock) VALUES (1, 'Laptop Dell XPS', 'Laptop de alto rendimiento', 1500.00, 10);
INSERT INTO Producto (id, nombre, descripcion, precio, stock) VALUES (2, 'Mouse Logitech', 'Mouse inalámbrico', 25.50, 50);
INSERT INTO Producto (id, nombre, descripcion, precio, stock) VALUES (3, 'Teclado Mecánico', 'Teclado RGB', 89.99, 30);
ALTER SEQUENCE Producto_SEQ RESTART WITH 4;
```

---

## Estructura del proyecto

```
pe.banco.productos
├── entity/
│   └── Producto.java              # Entidad JPA que extiende PanacheEntity
├── repository/
│   └── ProductoRepository.java    # Repository reactivo (PanacheRepositoryBase)
├── dto/
│   └── ProductoRequest.java       # DTO para requests
└── resource/
    └── ProductoReactivoResource.java  # REST endpoints reactivos
```

---

## Ejecuta el proyecto

```bash
./mvnw quarkus:dev
```

**Accesos:**
- API: http://localhost:8080/api/v1/productos/reactivo
- Swagger UI: http://localhost:8080/q/swagger-ui
- Dev UI: http://localhost:8080/q/dev

---

## Pruebas con cURL

### 1. Lista todos los productos.

```bash
curl http://localhost:8080/api/v1/productos/reactivo
```

**Respuesta esperada:**
```json
[
  {
    "id": 1,
    "nombre": "Laptop Dell XPS",
    "descripcion": "Laptop de alto rendimiento",
    "precio": 1500.0,
    "stock": 10
  },
  ...
]
```

---

### 2. Busca un producto por ID.

```bash
curl http://localhost:8080/api/v1/productos/reactivo/1
```

---

### 3. Crea un nuevo producto.

```bash
curl -X POST http://localhost:8080/api/v1/productos/reactivo \
  -H "Content-Type: application/json" \
  -d '{
    "nombre": "Monitor LG 27",
    "descripcion": "Monitor gaming 144Hz",
    "precio": 450.00,
    "stock": 8
  }'
```

**Respuesta:**
```json
{
  "id": 4,
  "nombre": "Monitor LG 27",
  "descripcion": "Monitor gaming 144Hz",
  "precio": 450.0,
  "stock": 8
}
```

---

### 4. Actualiza un producto.

```bash
curl -X PUT http://localhost:8080/api/v1/productos/reactivo/1 \
  -H "Content-Type: application/json" \
  -d '{
    "nombre": "Laptop Dell XPS 15",
    "descripcion": "Laptop actualizada",
    "precio": 1600.00,
    "stock": 15
  }'
```

---

### 5. Elimina un producto.

```bash
curl -X DELETE http://localhost:8080/api/v1/productos/reactivo/2
```

---

### 6. Busca productos con stock bajo.

```bash
curl http://localhost:8080/api/v1/productos/reactivo/stock-bajo/20
```

**Retorna todos los productos con stock menor a 20 unidades.**

---

### 7. Haz una carga masiva (demuestra concurrencia reactiva).

```bash
curl -X POST http://localhost:8080/api/v1/productos/reactivo/carga-masiva/100
```

**Crea 100 productos de forma reactiva.** Este endpoint demuestra las ventajas de la programación reactiva en operaciones masivas.

---

## Endpoints disponibles

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| `GET` | `/api/v1/productos/reactivo` | Listar todos |
| `GET` | `/api/v1/productos/reactivo/{id}` | Buscar por ID |
| `POST` | `/api/v1/productos/reactivo` | Crear producto |
| `PUT` | `/api/v1/productos/reactivo/{id}` | Actualizar producto |
| `DELETE` | `/api/v1/productos/reactivo/{id}` | Eliminar producto |
| `GET` | `/api/v1/productos/reactivo/stock-bajo/{umbral}` | Stock bajo |
| `POST` | `/api/v1/productos/reactivo/carga-masiva/{cantidad}` | Carga masiva |

---

## Alternativas a cURL

### Postman (Windows/Mac/Linux)
1. Descargar: https://www.postman.com/downloads/
2. Importar endpoints desde Swagger: http://localhost:8080/q/swagger-ui

### Thunder Client (VS Code)
1. Instalar extensión "Thunder Client" en VS Code.
2. Crear requests con interfaz gráfica.

### Swagger UI (incluido) - Recomendado para Windows

**URL:** http://localhost:8080/q/swagger-ui

#### Cómo usar Swagger UI.

1. **Abrir en el navegador:** http://localhost:8080/q/swagger-ui
2. **Expandir** el endpoint que quieres probar (click en la fila).
3. Dar click en **"Try it out".**
4. **Completar los parámetros** según la tabla.
5. Dar click en **"Execute".**
6. **Ver la respuesta** en la sección "Response body".

---

#### Datos para cada endpoint en Swagger:

##### GET. Listar todos los productos
- **Endpoint:** `/api/v1/productos/reactivo`.
- **Parámetros:** ninguno.
- **Action:** da click en "Execute".

---

##### GET. Buscar por ID
- **Endpoint:** `/api/v1/productos/reactivo/{id}`.
- **Parámetro `id`:** `1`.
- **Action:** da click en "Execute".

---

##### POST. Crear producto
- **Endpoint:** `/api/v1/productos/reactivo`.
- **Request body:**
```json
{
  "nombre": "Monitor LG 27",
  "descripcion": "Monitor gaming 144Hz",
  "precio": 450.00,
  "stock": 8
}
```
- **Action:** pega el JSON y da click en "Execute".

---

##### PUT. Actualizar producto
- **Endpoint:** `/api/v1/productos/reactivo/{id}`.
- **Parámetro `id`:** `1`.
- **Request body:**
```json
{
  "nombre": "Laptop Dell XPS 15 Actualizada",
  "descripcion": "Laptop de última generación",
  "precio": 1600.00,
  "stock": 15
}
```
- **Action:** pega el JSON y da click en "Execute".

---

##### DELETE. Eliminar producto
- **Endpoint:** `/api/v1/productos/reactivo/{id}`.
- **Parámetro `id`:** `2`.
- **Action:** da click en "Execute".

---

##### GET. Stock bajo
- **Endpoint:** `/api/v1/productos/reactivo/stock-bajo/{umbral}`.
- **Parámetro `umbral`:** `20`.
- **Action:** da click en "Execute".
- **Resultado:** muestra productos con stock menor a 20.

---

##### POST. Carga masiva (demuestra concurrencia).
- **Endpoint:** `/api/v1/productos/reactivo/carga-masiva/{cantidad}`.
- **Parámetro `cantidad`:** `100`.
- **Action:** da click en "Execute".
- **Resultado:** crea 100 productos reactivamente.

---

**💡 Tip:** Swagger UI es la forma más fácil de probar la API en Windows sin instalar algo adicional.

---

## Conceptos reactivos demostrados

### `Uni<T>` Operación asíncrona que retorna un solo valor.
```java
public Uni<List<Producto>> listarTodos() {
    return repository.listAll();
}
```

### Composición reactiva.
```java
return repository.findById(id)
    .onItem().ifNotNull().transform(producto -> Response.ok(producto).build())
    .onItem().ifNull().continueWith(Response.status(404).build());
```

### Transacciones reactivas.
```java
return Panache.withTransaction(() -> repository.persist(producto));
```

### Operaciones en lote.
```java
return repository.persistirLote(productos);
```

---

## Tecnologías utilizadas

- **Quarkus 3.28.3:** framework Java supersónico.
- **Hibernate Reactive Panache:** ORM reactivo simplificado.
- **PostgreSQL:** base de datos relacional.
- **SmallRye Mutiny:** librería reactiva (Uni/Multi).
- **RESTEasy Reactive:** REST endpoints reactivos.
- **SmallRye OpenAPI:** documentación automática (Swagger).

---

## Recursos adicionales

- [Quarkus - Hibernate Reactive Panache](https://quarkus.io/guides/hibernate-reactive-panache)
- [SmallRye Mutiny](https://smallrye.io/smallrye-mutiny/)
- [Reactive Programming](https://www.reactivemanifesto.org/)

---

## Ejercicio propuesto

1. Ejecutar carga masiva de 500 productos.
2. Observar los logs SQL.
3. Comparar tiempos con diferentes cantidades.
4. Analizar: ¿por qué el enfoque reactivo es más eficiente en alta concurrencia?

---

## Solución de problemas

### ❌ Error: "Unable to find JDBC driver"
**Solución:** verificar que `quarkus.datasource.jdbc=false` esté en `application.properties`.

### ❌ Error: "Connection refused"
**Solución:** verificar que PostgreSQL esté corriendo:
```bash
psql -U postgres -c "SELECT version();"
```

### ❌ Tabla vacía
**Solución:** verificar que `import.sql` esté en `src/main/resources/` y que use IDs explícitos.

