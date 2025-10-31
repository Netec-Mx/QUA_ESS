# VaultCorp API: sistema de gestión de secretos con Quarkus

## Descripción del proyecto

**VaultCorp API** es un microservicio educativo desarrollado con **Quarkus** que implementa un sistema completo de gestión de secretos corporativos con **tres niveles de seguridad**:

1. **Autenticación Básica** (Basic Auth) para administradores.
2. **JWT (JSON Web Token)** para empleados internos.
3. **OIDC (OpenID Connect)** para clientes externos.

### Objetivos:

- Conocer diferentes métodos de autenticación en microservicios.
- Manejar la autorización basada en roles (`@RolesAllowed`).
- Generar y validar tokens JWT con RSA.
- Aislar datos por usuario (multi-tenancy).
- Tener buenas prácticas de seguridad en API REST.
- Contar con un testing automatizado de seguridad.

---

## Arquitectura del sistema

```
┌─────────────────────────────────────────────────────────────┐
│                  VaultCorp API (Quarkus)                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   Admin      │  │   Internal   │  │   External   │     │
│  │   Endpoints  │  │   Endpoints  │  │   Endpoints  │     │
│  │              │  │              │  │              │     │
│  │ Basic Auth   │  │     JWT      │  │     OIDC     │     │
│  │ @RolesAllowed│  │   Bearer     │  │  (Keycloak)  │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │              SecretService (In-Memory)               │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
         │                  │                  │
         ▼                  ▼                  ▼
    [Admin Users]      [Employees]       [Customers]
    (curl/Postman)     (JWT tokens)      (Keycloak)
```

---

## Requisitos previos

### Software necesario

| Herramienta | Versión Mínima | Verificación |
|-------------|----------------|--------------|
| **Java JDK** | 17+ | `java -version` |
| **Maven** | 3.8+ | `mvn -version` |
| **cURL** | Cualquiera | `curl --version` |
| **Python 3** | 3.6+ (opcional, para formatear JSON) | `python3 --version` |
| **OpenSSL** | 1.1+ (para generar llaves RSA) | `openssl version` |

### Instalación de Java y Maven (si no los tienes)

**macOS (con Homebrew):**
```bash
brew install openjdk@17
brew install maven
```

**Linux (Ubuntu/Debian):**
```bash
sudo apt update
sudo apt install openjdk-17-jdk maven
```

**Windows:**
- Descargar Java: https://adoptium.net/
- Descargar Maven: https://maven.apache.org/download.cgi

---

## Creación del proyecto desde cero

### Paso 1. Crea el proyecto Quarkus.

```bash
mvn io.quarkus:quarkus-maven-plugin:3.15.1:create \
    -DprojectGroupId=com.vaultcorp \
    -DprojectArtifactId=vault-api \
    -DprojectVersion=1.0.0-SNAPSHOT \
    -DclassName="com.vaultcorp.resource.HealthResource" \
    -Dpath="/health" \
    -Dextensions="resteasy-reactive-jackson,hibernate-validator,smallrye-jwt,oidc,security"
```

**¿Qué hace este comando?**
- Crea un proyecto Maven con groupId `com.vaultcorp`.
- Añade las extensiones necesarias:
  - `resteasy-reactive-jackson`: REST APIs con JSON.
  - `hibernate-validator`: validación de datos.
  - `smallrye-jwt`: soporte para JWT.
  - `oidc`: OpenID Connect.
  - `security`: framework de seguridad base.

### Paso 2. Navega al proyecto.

```bash
cd vault-api
```

### Paso 3. Agrega una extensión adicional.

```bash
./mvnw quarkus:add-extension -Dextensions="elytron-security-properties-file"
```

Esta extensión permite configurar usuarios embebidos para Basic Auth.

---

## Estructura del proyecto

```
vault-api/
├── src/
│   ├── main/
│   │   ├── java/com/vaultcorp/
│   │   │   ├── dto/                    # Data Transfer Objects
│   │   │   │   ├── LoginRequest.java
│   │   │   │   ├── SecretRequest.java
│   │   │   │   └── TokenResponse.java
│   │   │   ├── model/                  # Entidades del dominio
│   │   │   │   ├── Secret.java
│   │   │   │   └── SecretLevel.java
│   │   │   ├── resource/               # REST Controllers
│   │   │   │   ├── AdminSecretResource.java
│   │   │   │   ├── AuthResource.java
│   │   │   │   └── InternalSecretResource.java
│   │   │   ├── security/               # Configuración de seguridad
│   │   │   │   └── Roles.java
│   │   │   └── service/                # Lógica de negocio
│   │   │       ├── JwtService.java
│   │   │       └── SecretService.java
│   │   └── resources/
│   │       ├── application.properties  # Configuración
│   │       ├── privateKey.pem          # Llave privada RSA (JWT)
│   │       └── publicKey.pem           # Llave pública RSA (JWT)
│   └── test/                           # Tests
├── pom.xml                             # Dependencias Maven
├── test-part1-security.sh              # Script de pruebas Parte 1
├── test-part2-jwt.sh                   # Script de pruebas Parte 2
├── README.md                           # Este archivo
├── README-PARTE2.md                    # Guía detallada Parte 2
└── TEORIA-PARTE2.md                    # Teoría JWT
```

---

## Configuración inicial

### 1. Configura `application.properties`.

Edita `src/main/resources/application.properties`:

```properties
# Configuración básica
quarkus.http.port=8080

# Habilitar seguridad en modo desarrollo
quarkus.security.auth.enabled-in-dev-mode=true

# Usuarios embebidos para autenticación básica (Parte 1)
quarkus.security.users.embedded.enabled=true
quarkus.security.users.embedded.plain-text=true
quarkus.security.users.embedded.users.admin=admin123
quarkus.security.users.embedded.roles.admin=vault-admin
quarkus.security.users.embedded.users.auditor=auditor123
quarkus.security.users.embedded.roles.auditor=vault-auditor
quarkus.security.users.embedded.users.employee=employee123
quarkus.security.users.embedded.roles.employee=employee

# Habilitar autenticación básica
quarkus.http.auth.basic=true

# Configuración JWT (Parte 2)
mp.jwt.verify.publickey.location=publicKey.pem
mp.jwt.verify.issuer=https://vaultcorp.com
smallrye.jwt.sign.key.location=privateKey.pem
mp.jwt.verify.clock.skew=60

# Deshabilitar OIDC temporalmente (hasta Parte 3)
quarkus.oidc.enabled=false
```

### 2. Genera llaves RSA para JWT.

```bash
# Generar llave privada (2048 bits)
openssl genrsa -out src/main/resources/privateKey.pem 2048

# Generar llave pública desde la privada
openssl rsa -pubout -in src/main/resources/privateKey.pem -out src/main/resources/publicKey.pem
```

**¿Por qué RSA?** Porque permite que diferentes microservicios validen tokens sin compartir la clave privada.

---

## Parte 1. Autenticación y autorización básica

### Conceptos clave.

- **Basic Auth**: usuario y contraseña en cada request.
- **`@PermitAll`**: endpoint público sin autenticación.
- **`@RolesAllowed`**: endpoint protegido por roles.
- **`HTTP 401`**: no autenticado.
- **`HTTP 403`**: autenticado pero sin permiso.

### Usuarios y roles.

| Username | Password | Rol | Permisos |
|----------|----------|-----|----------|
| `admin` | `admin123` | `vault-admin` | ✅ Leer, crear, eliminar secretos |
| `auditor` | `auditor123` | `vault-auditor` | ✅ Solo lectura (stats) |
| `employee` | `employee123` | `employee` | ✅ Login JWT |

### Endpoints administrativos.

```bash
# ✅ Público: Health Check.
curl http://localhost:8080/api/admin/secrets/health

# 🔒 Admin: listar todos los secretos.
curl -u admin:admin123 http://localhost:8080/api/admin/secrets/all

# 🔒 Admin: eliminar un secreto.
curl -X DELETE -u admin:admin123 http://localhost:8080/api/admin/secrets/{id}

# 🔒 Admin/Auditor: ver estadísticas.
curl -u auditor:auditor123 http://localhost:8080/api/admin/secrets/stats
```

### Ejecutar pruebas automatizadas.

```bash
# Asegúrate de que Quarkus esté corriendo
./mvnw quarkus:dev

# En otra terminal
./test-part1-security.sh
```

**Documentación completa:** revisa el código fuente de `AdminSecretResource.java`.

---

## Parte 2. Autenticación con JWT

### Conceptos clave.

- **JWT**: token autocontenido con información del usuario.
- **Stateless**: el servidor no guarda sesiones.
- **Bearer Token**: se envía en header `Authorization: Bearer <token>`.
- **Claims**: información dentro del JWT (sub, email, groups, exp).
- **RSA Signature**: firma criptográfica que garantiza integridad.

### Flujo de autenticación.

```
1. Login           2. Token          3. Request         4. Validación
┌────────┐        ┌────────┐        ┌────────┐        ┌────────┐
│ POST   │───────>│ Server │───────>│ GET    │───────>│ Server │
│ /login │ user/  │ genera │  JWT   │ /api   │ Bearer │ valida │
│        │ pass   │  JWT   │ token  │        │ token  │  firma │
└────────┘        └────────┘        └────────┘        └────────┘
```

### Usuarios para JWT.

| Username | Password | Email | Rol |
|----------|----------|-------|-----|
| `emp001` | `pass001` | juan.perez@vaultcorp.com | `employee` |
| `emp002` | `pass002` | maria.gonzalez@vaultcorp.com | `employee` |
| `emp003` | `pass003` | carlos.rodriguez@vaultcorp.com | `employee` |

### Flujo completo paso a paso.

#### 1. Hacer login y obtener token.

```bash
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "emp001",
    "password": "pass001"
  }'
```

**Respuesta:**
```json
{
  "token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "type": "Bearer",
  "expiresIn": 3600
}
```

#### 2. Guardar el token.

```bash
# Método 1: Manual
TOKEN="eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9..."

# Método 2: Automático
TOKEN=$(curl -s -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"emp001","password":"pass001"}' \
  | grep -o '"token":"[^"]*"' | cut -d'"' -f4)
```

#### 3. Ver el perfil.

```bash
curl http://localhost:8080/api/internal/secrets/profile \
  -H "Authorization: Bearer $TOKEN"
```

#### 4. Crear un secreto.

```bash
curl -X POST http://localhost:8080/api/internal/secrets \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Mi Secreto",
    "content": "información confidencial",
    "level": "INTERNAL"
  }'
```

#### 5. Listar "mis secretos".

```bash
curl http://localhost:8080/api/internal/secrets/my-secrets \
  -H "Authorization: Bearer $TOKEN"
```

#### 6. Ejecutar pruebas automatizadas.

```bash
# Asegúrate de que Quarkus esté corriendo
./mvnw quarkus:dev

# En otra terminal
./test-part2-jwt.sh
```

**Documentación detallada:** ver `README-PARTE2.md` y `TEORIA-PARTE2.md`.

---

## Cómo ejecutar el proyecto

### Opción 1. Modo desarrollo (recomendado).

```bash
# Iniciar en modo dev con hot reload
./mvnw quarkus:dev
```

**Ventajas:**
- Hot reload automático al cambiar código.
- Dev UI disponible en http://localhost:8080/q/dev .
- Logs en tiempo real.

### Opción 2. Compilar y ejecutar JAR.

```bash
# Compilar
./mvnw clean package

# Ejecutar
java -jar target/quarkus-app/quarkus-run.jar
```

### Opción 3. Compilación nativa con GraalVM (avanzado).

```bash
# Compilar binario nativo
./mvnw package -Dnative

# Ejecutar (arranque < 0.05s)
./target/vault-api-1.0.0-SNAPSHOT-runner
```

---

## Testing completo

### Pruebas automatizadas.

```bash
# Parte 1: Autenticación Básica
./test-part1-security.sh

# Parte 2: JWT
./test-part2-jwt.sh
```

### Pruebas manuales: checklist.

#### Parte 1. Basic auth.

- [ ] Health check funciona sin autenticación.
- [ ] Admin puede listar todos los secretos.
- [ ] Admin puede eliminar secretos.
- [ ] Auditor puede ver estadísticas.
- [ ] Auditor no puede eliminar secretos.
- [ ] Requests sin credenciales son rechazadas (401).

#### Parte 2. JWT.

- [ ] Login genera token válido.
- [ ] Token contiene claims correctos (sub, email, groups).
- [ ] Perfil muestra información del usuario.
- [ ] Empleado puede crear secretos propios.
- [ ] Empleado solo ve sus propios secretos.
- [ ] Segundo empleado no ve secretos del primero.
- [ ] Token expirado es rechazado (401).

---

## API Reference: resumen de endpoints

### Endpoints públicos

| Método | Ruta | Descripción |
|--------|------|-------------|
| `GET` | `/api/admin/secrets/health` | Health check |
| `POST` | `/api/auth/login` | Generar JWT |

### Endpoints con basic auth

| Método | Ruta | Rol Requerido | Descripción |
|--------|------|---------------|-------------|
| `GET` | `/api/admin/secrets/all` | `vault-admin` | Listar todos los secretos |
| `DELETE` | `/api/admin/secrets/{id}` | `vault-admin` | Eliminar secreto |
| `GET` | `/api/admin/secrets/stats` | `vault-admin`, `vault-auditor` | Ver estadísticas |

### Endpoints con JWT

| Método | Ruta | Rol Requerido | Descripción |
|--------|------|---------------|-------------|
| `GET` | `/api/internal/secrets/profile` | `employee` | Ver perfil |
| `GET` | `/api/internal/secrets/my-secrets` | `employee` | Listar secretos propios |
| `POST` | `/api/internal/secrets` | `employee` | Crear secreto |

---

## Troubleshooting

### ❌ Problema: "Port 8080 already in use".

**Solución:**
```bash
# Encontrar proceso usando el puerto
lsof -i :8080

# Matar el proceso
kill -9 <PID>

# O cambiar el puerto
./mvnw quarkus:dev -Dquarkus.http.port=8081
```

### ❌ Problema: "Token issued to client quarkus-app is not active".

**Causa:** conflicto entre extensiones OIDC y JWT.

**Solución:** verificar que `application.properties` contenga
```properties
quarkus.oidc.enabled=false
```

### ❌ Problema: "401 Unauthorized" con basic auth.

**Verificar:**
1. ¿Estás usando el formato correcto? `curl -u username:password`.
2. ¿El usuario existe en `application.properties`?
3. ¿La contraseña es correcta?

### ❌ Problema: "401 Unauthorized" con JWT.

**Verificar:**
1. ¿El header es correcto? `Authorization: Bearer <token>`.
2. ¿El token expiró? Los tokens duran 1 hora.
3. ¿Las llaves RSA existen en `src/main/resources/`?

### ❌ Problema: no puedo ver secretos de otro usuario.

**¡Eso es correcto!** Es una característica de seguridad (aislamiento multi-tenancy).

---

## Recursos de aprendizaje

### Documentación oficial:

- [Quarkus Security](https://quarkus.io/guides/security).
- [Quarkus JWT](https://quarkus.io/guides/security-jwt).
- [RFC 7519 - JWT Standard](https://tools.ietf.org/html/rfc7519).

### Herramientas útiles:

- [jwt.io](https://jwt.io), debugger de JWT.
- [Quarkus Dev UI](http://localhost:8080/q/dev), interfaz de desarrollo.
- [Postman](https://www.postman.com/), testing de API.

### Archivos del proyecto:

- `README-PARTE2.md`, guía detallada de JWT.
- `TEORIA-PARTE2.md`, teoría completa de JWT.
- `test-part1-security.sh`, script de pruebas, parte 1.
- `test-part2-jwt.sh`, script de pruebas, parte 2.

---

## Próximos pasos

### Parte 3. OIDC con Keycloak.

- Integración con proveedores de identidad externos.
- OpenID Connect (OIDC).
- Single Sign-On (SSO).
- Keycloak como Identity Provider.
- Federación de identidades.

### Mejoras adicionales:

- [ ] Persistencia con PostgreSQL.
- [ ] Cifrado de secretos en base de datos.
- [ ] Auditoría completa de accesos.
- [ ] Rate limiting.
- [ ] API versioning.
- [ ] OpenAPI/Swagger documentation.
- [ ] Docker Compose setup.
- [ ] CI/CD con GitHub Actions.

### Puntos clave:

1. **JWT no es encriptación.** El payload es visible.
2. **Stateless = escalabilidad.** Pero dificulta la revocación.
3. **La expiración es crítica.** Limita la ventana de ataque.
4. **RSA en microservicios.** Separación de responsabilidades.
5. **Aislamiento por diseño.** Filtrar siempre por usuario autenticado.

---

## ✅ Checklist de verificación final.

Antes de dar por completado el ejercicio:

- [ ] El proyecto compila sin errores.
- [ ] Quarkus arranca en modo dev.
- [ ] Health check responde.
- [ ] Basic auth funciona con admin y auditor.
- [ ] Login genera un JWT válido.
- [ ] JWT permite el acceso a endpoints protegidos.
- [ ] El aislamiento entre usuarios funciona.
- [ ] Los scripts de prueba se ejecutan sin errores.
- [ ] Puedes decodificar un JWT en jwt.io.
- [ ] Entiendes la diferencia entre 401 y 403.

---
