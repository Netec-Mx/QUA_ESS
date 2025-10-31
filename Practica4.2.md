# Clientes seguros - Always Encrypted

**Sistema de gestión de clientes bancarios con cifrado transparente de columnas sensibles usando Always Encrypted.**

---

## Descripción

Este proyecto demuestra el uso de **Always Encrypted** en SQL Server para proteger datos sensibles de clientes:
- **Números de tarjeta:** cifrados con tipo deterministic (permite búsquedas).
- **Emails:** cifrados con tipo randomized (mayor seguridad).
- **Otros datos:** sin cifrar (nombre, teléfono).

**Característica principal:** los datos se cifran en la **aplicación** (Quarkus) y se descifran en la **aplicación**. SQL Server **nunca** ve los datos en texto plano.

---

## Arquitectura

```
┌─────────────────────────────────────────────────┐
│           APLICACIÓN QUARKUS                    │
│  • Driver JDBC Always Encrypted                 │
│  • Cifra/Descifra automáticamente              │
│  • Código NO cambia (transparente)             │
└────────────────┬────────────────────────────────┘
                 │
                 │ Datos cifrados
                 ▼
┌─────────────────────────────────────────────────┐
│              SQL SERVER                         │
│  • Almacena datos cifrados                     │
│  • Nunca ve texto plano                        │
│  • Metadata de cifrado                         │
└────────────────┬────────────────────────────────┘
                 │
                 │ Usa claves de
                 ▼
┌─────────────────────────────────────────────────┐
│          KEY STORE PROVIDER                     │
│  • Windows Certificate Store (Windows)          │
│  • Azure Key Vault (Cloud, cualquier SO)       │
│  • Column Master Key (CMK)                     │
└─────────────────────────────────────────────────┘
```

---

## ⚠️ Requisitos importantes

### Always Encrypted requiere de:

1. **Windows** con Windows Certificate Store.
    - Funciona localmente.
    - Solo Windows.

3. **Azure Key Vault** (nube).
    - Funciona en cualquier plataforma (Mac/Windows/Linux).
    - Requiere cuenta de Azure (tiene un costo mínimo).

**❌ No funciona en:**
- Mac/Linux sin Azure Key Vault.
- Docker en Mac/Linux (sin Azure).

---

## Stack técnico

- **Framework:** Quarkus 3.28.3.
- **ORM:** Hibernate ORM with Panache (patrón Repository).
- **Base de datos:** SQL Server 2022.
- **Seguridad:** Always Encrypted.
- **Driver:** Microsoft JDBC Driver 12.4.2.
- **Container:** Docker (SQL Server).

---

## Estructura del proyecto

```
clientes-seguros/
├── src/main/java/pe/banco/clientes/
│   ├── entity/
│   │   └── Cliente.java                    # Entidad JPA
│   ├── repository/
│   │   └── ClienteRepository.java          # Repository (PanacheRepositoryBase)
│   ├── dto/
│   │   └── ClienteRequest.java             # DTO para requests
│   └── resource/
│       └── ClienteResource.java            # REST endpoints
├── src/main/resources/
│   ├── application.properties              # Configuración base
│   └── import.sql                          # Datos iniciales
├── setup-always-encrypted.sql              # Script SQL para Windows
├── INSTALACION.md                          # Setup inicial del proyecto
├── TEORIA-ALWAYS-ENCRYPTED.md              # Teoría completa
├── SETUP-WINDOWS.md                        # Guía paso a paso Windows
├── SETUP-AZURE.md                          # Guía paso a paso Azure
└── README.md                               # Este archivo
```

---

## Endpoints API

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| `GET` | `/api/v1/clientes` | Listar todos los clientes |
| `GET` | `/api/v1/clientes/{id}` | Buscar cliente por ID |
| `POST` | `/api/v1/clientes` | Crear nuevo cliente |
| `GET` | `/api/v1/clientes/tarjeta/{numero}` | Buscar por número de tarjeta |

**Nota:** ❌ **No** está disponible la búsqueda por email (usa cifrado Randomized).

---

## Documentación

### Setup inicial
- **[INSTALACION.md](./INSTALACION.md)** - Crear proyecto desde cero con Docker + SQL Server.

### Teoría
- **[TEORIA-ALWAYS-ENCRYPTED.md](./TEORIA-ALWAYS-ENCRYPTED.md)** - Conceptos, arquitectura, tipos de cifrado.

### Configuración Always Encrypted

**Opción A. Windows (recomendado para desarrollo).**
- **[SETUP-WINDOWS.md](./SETUP-WINDOWS.md)** - Paso a paso con Windows Certificate Store.

**Opción B. Azure (recomendado para producción).**
- **[SETUP-AZURE.md](./SETUP-AZURE.md)** - Paso a paso con Azure Key Vault.

---

## Estado del Proyecto
**Completado:**

- ✅ Proyecto Quarkus configurado.
- ✅ Entidad `Cliente` (con campos para cifrar).
- ✅ Repository con `PanacheRepositoryBase`.
- ✅ REST API completa (CRUD).
- ✅ SQL Server en Docker.
- ✅ Base de datos BancoDB creada.
- ✅ Datos de prueba.
- ✅ Script SQL para Always Encrypted.
- ✅ Documentación completa (teoría + setup).

**Pendiente (requiere Windows o Azure):**

- Crear certificado/key (CMK).
- Configurar Always Encrypted en SQL Server.
- Probar cifrado end-to-end.
- Validar búsquedas con cifrado determinístico.

---

## Inicio rápido

**Paso 1. Levantar SQL Server.**

```bash
docker start sqlserver
```

Si no tienes el contenedor:
```bash
docker run -e 'ACCEPT_EULA=Y' -e 'SA_PASSWORD=Pass123!Admin' \
  -p 1433:1433 --name sqlserver \
  -d mcr.microsoft.com/mssql/server:2022-latest
```

**Paso 2. Verificar la base de datos.**

```bash
docker exec sqlserver /opt/mssql-tools18/bin/sqlcmd \
  -S localhost -U sa -P 'Pass123!Admin' -C \
  -Q "SELECT name FROM sys.databases WHERE name = 'BancoDB'"
```

Si no existe:
```bash
docker exec sqlserver /opt/mssql-tools18/bin/sqlcmd \
  -S localhost -U sa -P 'Pass123!Admin' -C \
  -Q "CREATE DATABASE BancoDB"
```

**Paso 3. Ejecutar la aplicación (SIN Always Encrypted).**

```bash
./mvnw quarkus:dev
```

#### Accesos:
- API: http://localhost:8080/api/v1/clientes
- Dev UI: http://localhost:8080/q/dev

**Paso 4. Probar API.**

```bash
# Listar clientes
curl http://localhost:8080/api/v1/clientes

# Crear cliente
curl -X POST http://localhost:8080/api/v1/clientes \
  -H "Content-Type: application/json" \
  -d '{
    "nombre": "Juan Pérez",
    "numeroTarjeta": "4532-1234-5678-9012",
    "email": "juan.perez@banco.com",
    "telefono": "+56912345678"
  }'
```

---

## Configurar Always Encrypted

**⚠️ Importante:** Este paso requiere Windows O Azure.

### Opción A. Windows

1. Abre **[SETUP-WINDOWS.md](./SETUP-WINDOWS.md)**.
2. Sigue el paso a paso.
3. Ejecuta `setup-always-encrypted.sql` (después de crear el certificado).
4. Actualiza `application.properties` con `columnEncryptionSetting=Enabled`.

### Opción B. Azure

1. Abre **[SETUP-AZURE.md](./SETUP-AZURE.md)**.
2. Crea una cuenta Azure (gratis con un crédito inicial de $200 USD).
3. Configura la Key Vault y el Service Principal.
4. Actualiza `application.properties` con credenciales Azure.

---

## Verificar cifrado

### Con Always Encrypted habilitado:

```bash
# Insertar cliente (datos se cifran automáticamente)
curl -X POST http://localhost:8080/api/v1/clientes \
  -H "Content-Type: application/json" \
  -d '{
    "nombre": "María González",
    "numeroTarjeta": "5412-9876-5432-1098",
    "email": "maria.gonzalez@banco.com",
    "telefono": "+56987654321"
  }'
```

### Verificar en SQL Server (sin Always Encrypted):

```bash
docker exec -it sqlserver /opt/mssql-tools18/bin/sqlcmd \
  -S localhost -U sa -P 'Pass123!Admin' -C

USE BancoDB;
SELECT * FROM Cliente;
GO
```

**Deberías ver:**
- `nombre`: texto plano.
- `numero_tarjeta`: `0x01A7B3C9...` (binario cifrado).
- `email`: `0x02X5Y9Z3...` (binario cifrado).
- `telefono`: texto plano.

---

## Comparación: Sin vs. con Always Encrypted

| Aspecto | Sin Cifrado | Con Always Encrypted |
|---------|-------------|---------------------|
| **En la BD** | Texto plano | Binario cifrado |
| **En la app** | Texto plano | Texto plano (transparente) |
| **DBA ve** | ✅ Todo | ❌ Solo datos cifrados |
| **Búsquedas** | ✅ Todas | ⚠️ Solo determinísticas |
| **Seguridad** | Baja | Alta |

---

## Conceptos clave

### Column Master Key (CMK)
- Llave maestra que protege las CEK.
- Almacenada en Windows Certificate Store o Azure Key Vault.
- **Nunca** sale del Key Store.

### Column Encryption Key (CEK)
- Llave que realmente cifra los datos.
- Cifrada por el CMK.
- Almacenada (cifrada) en SQL Server.

### Cifrado determinístico
- Mismo valor → mismo cifrado.
- Permite búsquedas por igualdad (`WHERE =`).
- Menos seguro (patrones detectables).

### Cifrado aleatorio (randomized)
- Mismo valor → cifrado diferente cada vez.
- **No** permite búsquedas.
- Más seguro.

---

## Solución de problemas

### ❌ Error: "Cannot find the certificate"
- **Causa:** el certificado no existe o el thumbprint es incorrecto.
- **Solución:** verificar el certificado en Windows Certificate Store.

### ❌ Error: "Column encryption setting is not enabled"
- **Causa:** falta la configuración en el connection string.
- **Solución:** agregar `columnEncryptionSetting=Enabled`.

### ❌ No puedo buscar por email
- **Es esperado.** Email usa cifrado Randomized que no permite búsquedas.

### ❌ Docker en Mac no funciona con Always Encrypted
- **Es correcto.** Requiere Windows Certificate Store o Azure Key Vault.

---

## 📖 Recursos adicionales

- [Microsoft Docs - Always Encrypted](https://docs.microsoft.com/en-us/sql/relational-databases/security/encryption/always-encrypted-database-engine)
- [Quarkus SQL Server Guide](https://quarkus.io/guides/datasource)
- [Azure Key Vault Docs](https://docs.microsoft.com/en-us/azure/key-vault/)

---

## Para los estudiantes

### Requisitos del sistema:
- **Windows 10/11** (para ejercicio completo).
- O **cuenta Azure** (alternativa cloud).
- Docker Desktop.
- Java 21.
- Maven 3.8+.

### Recomendación:
1. Leer **TEORIA-ALWAYS-ENCRYPTED.md** primero.
2. Seguir **INSTALACION.md** para setup básico.
3. Probar API sin cifrado.
4. Configurar Always Encrypted con **SETUP-WINDOWS.md** o **SETUP-AZURE.md**.

---

## Próximos pasos

### Antes del curso:
1. [ ] Instalar Windows en Parallels.
2. [ ] Seguir SETUP-WINDOWS.md completo.
3. [ ] Validar que todo funciona.
4. [ ] Generar screenshots del proceso.
5. [ ] Probar ejercicio end-to-end.

### Durante el curso:
1. [ ] Demo en vivo con Windows.
2. [ ] Explicación de teoría (TEORIA-ALWAYS-ENCRYPTED.md).
3. [ ] Ejercicio práctico guiado.
4. [ ] Validación del cifrado en SQL Server

---
