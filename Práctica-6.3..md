# Parte 3. Autenticación con OIDC y Keycloak

## Descripción

En esta parte, integras **OpenID Connect (OIDC)** con **Keycloak** como proveedor de identidad externo para autenticar **clientes externos**. A diferencia de las partes anteriores donde nuestra aplicación gestionaba la autenticación, aquí delegamos esta responsabilidad a Keycloak.

### Objetivos de aprendizaje

- ✅ Entender qué es OIDC y cómo difiere del JWT propio.
- ✅ Configurar Keycloak como identity provider.
- ✅ Implementar autenticación federada.
- ✅ Gestionar usuarios y roles en Keycloak.
- ✅ Validar tokens emitidos por Keycloak en Quarkus.
- ✅ Implementar autorización basada en roles externos.

---

## Comparativa: parte 2 (JWT) vs. parte 3 (OIDC)

| Aspecto | Parte 2: JWT Propio | Parte 3: OIDC + Keycloak |
|---------|---------------------|--------------------------|
| **Quién emite los tokens.** | Nuestra aplicación. | Keycloak. |
| **Quién gestiona los usuarios.** | Nuestra aplicación. | Keycloak. |
| **Quién gestiona los roles.** | Nuestra aplicación. | Keycloak. |
| **Dónde se almacenan los usuarios.** | `application.properties` (mock). | Keycloak (BD propia). |
| **Firma del token.** | La clave RSA es privada. | La clave RSA de Keycloak. |
| **Validación del token.** | La clave RSA es pública. | La clave pública de Keycloak. |
| **SSO entre apps.** | No. | Sí. |
| **Federación con otros IdP.** | No. | Sí (Google, GitHub, etc.). |
| **Complejidad inicial.** | Baja. | Media - alta. |
| **Uso típico.** | APIs internas, empleados. | Clientes externos, SaaS. |

---

## Arquitectura de la solución

```
┌────────────────────────────────────────────────────────────┐
│                    Cliente externo                         │
│              (Navegador / Aplicación)                      │
└──────────────────────┬─────────────────────────────────────┘
                       │
                       │ 1. Solicita acceso
                       ▼
           ┌───────────────────────┐
           │   Quarkus App         │
           │   (vault-api)         │
           └───────────┬───────────┘
                       │
                       │ 2. Redirige a Keycloak
                       ▼
           ┌───────────────────────┐
           │      Keycloak         │
           │  (Identity Provider)  │
           │                       │
           │  - Realm: vaultcorp   │
           │  - Usuarios           │
           │  - Roles              │
           └───────────┬───────────┘
                       │
                       │ 3. Usuario se autentica
                       │ 4. Keycloak emite Access Token
                       ▼
           ┌───────────────────────┐
           │   Cliente externo     │
           │   (guarda token)      │
           └───────────┬───────────┘
                       │
                       │ 5. Request + Bearer Token
                       ▼
           ┌───────────────────────┐
           │   Quarkus App         │
           │   - Valida token      │
           │   - Extrae roles      │
           │   - Autoriza          │
           └───────────────────────┘
```

---

## Requisitos previos

### Software necesario

| Herramienta | Versión | Para qué |
|-------------|---------|----------|
| **Docker** | 20.10+ | Ejecutar Keycloak. |
| **Java** | 17+ | Quarkus. |
| **Maven** | 3.8+ | Build del proyecto. |
| **cURL** | Cualquiera | Pruebas. |
| **Python 3** | 3.6+ | Formatear JSON (opcional). |

### Verificar Docker

```bash
docker --version
# Docker version 24.0.0 o superior
```

Si no tienes Docker instalado:
- **macOS**: `brew install docker` o descargar Docker Desktop.
- **Linux**: seguir guía oficial de Docker.
- **Windows**: Docker Desktop for Windows.

---

## Paso 1. Levantar Keycloak con Docker

### 1.1. Ejecutar el contenedor.

```bash
docker run -p 8180:8080 \
  -e KEYCLOAK_ADMIN=admin \
  -e KEYCLOAK_ADMIN_PASSWORD=admin \
  --name keycloak-vaultcorp \
  quay.io/keycloak/keycloak:23.0.0 \
  start-dev
```

**Explicación de los parámetros:**
- `-p 8180:8080`: mapea del puerto 8180 local a el 8080 del contenedor (para no chocar con Quarkus en 8080).
- `-e KEYCLOAK_ADMIN=admin`: usuario admin de Keycloak.
- `-e KEYCLOAK_ADMIN_PASSWORD=admin`: contraseña del admin.
- `--name keycloak-vaultcorp`: nombre del contenedor.
- `start-dev`: modo desarrollo (**no** usar en producción).

### 1.2. Verificar que está corriendo.

Espera de 1 a 2 minutos y verifica en los logs:

```
Keycloak 23.0.0 on JVM started in 7.621s. Listening on: http://0.0.0.0:8080
```

Abre el navegador en: `http://localhost:8180`.

Deberías ver la página de bienvenida de Keycloak.

### 1.3. Comandos útiles de Docker.

```bash
# Ver contenedores corriendo
docker ps

# Ver logs de Keycloak
docker logs keycloak-vaultcorp

# Detener Keycloak
docker stop keycloak-vaultcorp

# Iniciar Keycloak nuevamente
docker start keycloak-vaultcorp

# Eliminar el contenedor (borra toda la configuración)
docker rm keycloak-vaultcorp
```

---

## Paso 2. Configurar Keycloak

### 2.1. Acceder a la consola de administración.

1. Navega a: `http://localhost:8180`.
2. Da click en **"Administration Console".**
3. Login:
   - **Usuario**: `admin`.
   - **Password**: `admin`.

### 2.2. Crear un Realm.

Un **Realm** es un espacio aislado para gestionar usuarios, clientes y roles.

1. En la esquina superior izquierda, da click en **"master"** (dropdown).
2. Da click en **"Create realm".**
3. **Realm name**: `vaultcorp`.
4. Da click **"Create".**

Ahora estás en el realm `vaultcorp`.

### 2.3. Crear un Client (nuestra app).

Un **Client** representa una aplicación que puede solicitar autenticación.

1. Ubícate en el menú lateral izquierdo: **Clients**.
2. Da click en **"Create client"**.

**Pantalla 1: General Settings**
- **Client type**: `OpenID Connect` ✅.
- **Client ID**: `vault-api`.
- **Name**: `VaultCorp API`.
- **Description**: (opcional) "API de gestión de secretos".
- Da click en **"Next"**.

**Pantalla 2: Capability config**
- **Client authentication**: `On` ✅.
- **Authorization**: `Off`.
- **Authentication flow**:
  - ✅ Standard flow.
  - ✅ Direct access grants.
- Da click en **"Next"**.

**Pantalla 3: Login settings**
- **Root URL**: `http://localhost:8080`.
- **Home URL**: `http://localhost:8080`.
- **Valid redirect URIs**: `http://localhost:8080/*`.
- **Valid post logout redirect URIs**: `http://localhost:8080/*`.
- **Web origins**: `http://localhost:8080`.
- Da click en **"Save"**.

### 2.4. Obtener el client cecret.

1. En la página del client `vault-api`, da click en la pestaña **"Credentials"**.
2. Copia el valor de **"Client secret"**.
3. **⚠️ Importante**: guárdalo en un lugar seguro, lo necesitarás después.

**Ejemplo**: `3dQoUzQ7Y4TQ7eknNNxeDbWiAmMjPpVn`

### 2.5. Crear roles.

Los **Realm Roles** definen permisos a nivel de realm.

1. Ubícate en el menú lateral: **"Realm roles"**.
2. Da click en **"Create role"**.

**Rol 1: customer**
- **Role name**: `customer`.
- **Description**: "cliente básico".
- Da click en **"Save"**.

**Rol 2: premium-customer**
- Da click en **"Create role"** nuevamente.
- **Role name**: `premium-customer`.
- **Description**: "cliente premium".
- Da click en **"Save"**.

### 2.6. Crear usuarios.

1. Ubícate el menú lateral: **Users**.
2. Da click en **"Create new user"**.

**Usuario 1: cliente básico**
- **Username**: `client001`.
- **Email**: `cliente1@empresa.com`.
- **First name**: `Carlos`.
- **Last name**: `Gómez`.
- **Email verified**: `On` ✅.
- **Required user actions**: (dejar vacío).
- Da click en **"Create"**.

Ahora asigna una contraseña:
- Da click en la pestaña **"Credentials"**.
- Da click en **"Set password"**.
- **Password**: `pass001`.
- **Password confirmation**: `pass001`.
- **Temporary**: `Off` ✅.
- Da click en **"Save"** y confirma.

Ahora asigna el rol:
- Da click en la pestaña **"Role mapping"**.
- Da click en **"Assign role"**.
- Selecciona **`customer`**.
- Da click en **"Assign"**.

**Usuario 2: cliente premium**
- Repetir los pasos anteriores con:
  - **Username**: `client002`.
  - **Email**: `cliente2@empresa.com`.
  - **First name**: `María`.
  - **Last name**: `López`.
  - **Password**: `pass002`.
  - **Rol**: `premium-customer`.

✅ **Resumen de usuarios creados:**

| Username | Password | Email | Rol |
|----------|----------|-------|-----|
| `client001` | `pass001` | cliente1@empresa.com | `customer` |
| `client002` | `pass002` | cliente2@empresa.com | `premium-customer` |

---

## Paso 3. Configurar Quarkus

### 3.1. Habilitar OIDC en `application.properties`.

Edita `src/main/resources/application.properties` y agrega al final:

```properties
# Configuración OIDC con Keycloak (Parte 3)
quarkus.oidc.enabled=true
quarkus.oidc.auth-server-url=http://localhost:8180/realms/vaultcorp
quarkus.oidc.client-id=vault-api
quarkus.oidc.credentials.secret=TU-CLIENT-SECRET-AQUI
quarkus.oidc.tls.verification=none
quarkus.oidc.application-type=service
```

**⚠️ Importante**: reemplaza `TU-CLIENT-SECRET-AQUI` con el client secret que copiaste en el paso 2.4.

**Explicación de las propiedades:**

| Propiedad | Descripción |
|-----------|-------------|
| `quarkus.oidc.enabled` | Habilita OIDC (true). |
| `quarkus.oidc.auth-server-url` | URL del realm de Keycloak. |
| `quarkus.oidc.client-id` | ID del client creado en Keycloak. |
| `quarkus.oidc.credentials.secret` | Client secret de Keycloak. |
| `quarkus.oidc.tls.verification` | Desactivar verificación SSL en dev. |
| `quarkus.oidc.application-type` | Tipo: service (API backend). |

### 3.2. Verificar la configuración completa.

Tu `application.properties` debería tener las 3 partes:

```properties
# Parte 1: Basic Auth
quarkus.http.auth.basic=true
quarkus.security.users.embedded.enabled=true
...

# Parte 2: JWT
mp.jwt.verify.publickey.location=publicKey.pem
mp.jwt.verify.issuer=https://vaultcorp.com
...

# Parte 3: OIDC
quarkus.oidc.enabled=true
quarkus.oidc.auth-server-url=http://localhost:8180/realms/vaultcorp
...
```

---

## Paso 4. Crear endpoints externos

### 4.1. Actualizar `Roles.java`.

Agrega los nuevos roles de clientes:

```java
package com.vaultcorp.security;

public class Roles {
    // Parte 1: Roles administrativos
    public static final String VAULT_ADMIN = "vault-admin";
    public static final String VAULT_AUDITOR = "vault-auditor";
    
    // Parte 2: Roles de empleados
    public static final String EMPLOYEE = "employee";
    
    // Parte 3: Roles de clientes (OIDC)
    public static final String CUSTOMER = "customer";
    public static final String PREMIUM_CUSTOMER = "premium-customer";

    private Roles() {}
}
```

### 4.2. Crear `ExternalSecretResource.java`.

Crea `src/main/java/com/vaultcorp/resource/ExternalSecretResource.java`:

```java
package com.vaultcorp.resource;

import com.vaultcorp.model.Secret;
import com.vaultcorp.model.SecretLevel;
import com.vaultcorp.security.Roles;
import com.vaultcorp.service.SecretService;
import io.quarkus.security.identity.SecurityIdentity;
import jakarta.annotation.security.RolesAllowed;
import jakarta.inject.Inject;
import jakarta.ws.rs.*;
import jakarta.ws.rs.core.MediaType;
import jakarta.ws.rs.core.Response;

import java.util.List;
import java.util.Map;

@Path("/api/external/secrets")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
public class ExternalSecretResource {

    @Inject
    SecurityIdentity securityIdentity;

    @Inject
    SecretService secretService;

    // Endpoint para clientes básicos y premium: secretos públicos
    @GET
    @Path("/public")
    @RolesAllowed({Roles.CUSTOMER, Roles.PREMIUM_CUSTOMER})
    public Response getPublicSecrets() {
        List<Secret> publicSecrets = secretService.getSecretsByLevel(SecretLevel.PUBLIC);
        
        return Response.ok(Map.of(
            "user", securityIdentity.getPrincipal().getName(),
            "roles", securityIdentity.getRoles(),
            "totalSecrets", publicSecrets.size(),
            "secrets", publicSecrets
        )).build();
    }

    // Endpoint solo para clientes premium: secretos confidenciales
    @GET
    @Path("/confidential")
    @RolesAllowed(Roles.PREMIUM_CUSTOMER)
    public Response getConfidentialSecrets() {
        List<Secret> confidentialSecrets = secretService.getSecretsByLevel(SecretLevel.CONFIDENTIAL);
        
        return Response.ok(Map.of(
            "user", securityIdentity.getPrincipal().getName(),
            "roles", securityIdentity.getRoles(),
            "level", "CONFIDENTIAL",
            "totalSecrets", confidentialSecrets.size(),
            "secrets", confidentialSecrets
        )).build();
    }

    // Endpoint de perfil para ambos tipos de clientes
    @GET
    @Path("/profile")
    @RolesAllowed({Roles.CUSTOMER, Roles.PREMIUM_CUSTOMER})
    public Response getProfile() {
        return Response.ok(Map.of(
            "username", securityIdentity.getPrincipal().getName(),
            "roles", securityIdentity.getRoles(),
            "authMethod", "OIDC (Keycloak)"
        )).build();
    }
}
```

### 4.3. Actualizar `SecretService.java`.

Agrega datos de prueba con niveles `PUBLIC` y `CONFIDENTIAL`:

```java
private void initializeMockData() {
    // TOP_SECRET (solo admin)
    Secret s1 = new Secret();
    s1.setName("API Key Producción");
    s1.setContent("sk_prod_ABC123XYZ");
    s1.setLevel(SecretLevel.TOP_SECRET);
    s1.setOwnerId("admin");
    secrets.add(s1);

    // INTERNAL (empleados)
    Secret s2 = new Secret();
    s2.setName("Contraseña BD Dev");
    s2.setContent("devPass2024");
    s2.setLevel(SecretLevel.INTERNAL);
    s2.setOwnerId("emp001");
    secrets.add(s2);

    // PUBLIC (clientes básicos)
    Secret s3 = new Secret();
    s3.setName("Manual de Usuario");
    s3.setContent("https://docs.vaultcorp.com/manual");
    s3.setLevel(SecretLevel.PUBLIC);
    s3.setOwnerId("marketing");
    secrets.add(s3);

    Secret s4 = new Secret();
    s4.setName("API Pública de Consultas");
    s4.setContent("https://api.vaultcorp.com/public/v1");
    s4.setLevel(SecretLevel.PUBLIC);
    s4.setOwnerId("marketing");
    secrets.add(s4);

    // CONFIDENTIAL (solo premium)
    Secret s5 = new Secret();
    s5.setName("Credenciales AWS S3");
    s5.setContent("AKIAIOSFODNN7EXAMPLE");
    s5.setLevel(SecretLevel.CONFIDENTIAL);
    s5.setOwnerId("ops");
    secrets.add(s5);

    Secret s6 = new Secret();
    s6.setName("Token Analytics Premium");
    s6.setContent("analytics_premium_token_xyz789");
    s6.setLevel(SecretLevel.CONFIDENTIAL);
    s6.setOwnerId("analytics");
    secrets.add(s6);
}
```

---

## Paso 5. Probar la integración

### 5.1. Levantar la aplicación.

```bash
./mvnw quarkus:dev
```

Verifica que no haya errores de configuración.

### 5.2. Obtener "Access Token" de Keycloak.

**Para cliente básico (customer):**

```bash
curl -X POST http://localhost:8180/realms/vaultcorp/protocol/openid-connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=password" \
  -d "client_id=vault-api" \
  -d "client_secret=TU-CLIENT-SECRET" \
  -d "username=client001" \
  -d "password=pass001"
```

Respuesta:
```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI...",
  "expires_in": 300,
  "refresh_expires_in": 1800,
  "token_type": "Bearer",
  ...
}
```

**Guardar el token en variable:**

```bash
TOKEN=$(curl -s -X POST http://localhost:8180/realms/vaultcorp/protocol/openid-connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=password" \
  -d "client_id=vault-api" \
  -d "client_secret=TU-CLIENT-SECRET" \
  -d "username=client001" \
  -d "password=pass001" \
  | grep -o '"access_token":"[^"]*"' | cut -d'"' -f4)
```

### 5.3. Probar endpoints.

**Ver perfil:**
```bash
curl http://localhost:8080/api/external/secrets/profile \
  -H "Authorization: Bearer $TOKEN"
```

**Ver secretos públicos:**
```bash
curl http://localhost:8080/api/external/secrets/public \
  -H "Authorization: Bearer $TOKEN"
```

**Intentar ver secretos confidenciales (debe fallar con 403):**
```bash
curl -i http://localhost:8080/api/external/secrets/confidential \
  -H "Authorization: Bearer $TOKEN"
# HTTP/1.1 403 Forbidden
```

### 5.4. Ejecutar script de pruebas automatizado.

**Importante**: primero edita el script y agrega tu client secret.

```bash
nano test-part3-oidc.sh
```

Busca y modifica:
```bash
CLIENT_SECRET="TU-CLIENT-SECRET-AQUI"
```

Pon tu secret real.

Luego ejecuta:

```bash
chmod +x test-part3-oidc.sh
./test-part3-oidc.sh
```

---

## API Reference. Endpoints Parte 3.

| Método | Ruta | Rol Requerido | Descripción |
|--------|------|---------------|-------------|
| `GET` | `/api/external/secrets/profile` | `customer`, `premium-customer` | Ver perfil del usuario. |
| `GET` | `/api/external/secrets/public` | `customer`, `premium-customer` | Listar secretos `PUBLIC`. |
| `GET` | `/api/external/secrets/confidential` | `premium-customer` | Listar secretos `CONFIDENTIAL`. |

**Headers necesarios:**
```
Authorization: Bearer <access-token-de-keycloak>
```

---

## Persistir configuración de Keycloak

### ❌ Problema

Al usar `start-dev`, Keycloak usa una base de datos H2 **en memoria**. Si detienes el contenedor, **se pierde toda la configuración**.

### Solución 1. Exportar/importar configuración.

**Exportar:**

```bash
docker exec -it keycloak-vaultcorp /opt/keycloak/bin/kc.sh export \
  --dir /tmp/keycloak-export \
  --realm vaultcorp

docker cp keycloak-vaultcorp:/tmp/keycloak-export ./keycloak-backup
```

**Importar:**

```bash
docker run -p 8180:8080 \
  -e KEYCLOAK_ADMIN=admin \
  -e KEYCLOAK_ADMIN_PASSWORD=admin \
  -v $(pwd)/keycloak-backup:/opt/keycloak/data/import \
  --name keycloak-vaultcorp \
  quay.io/keycloak/keycloak:23.0.0 \
  start-dev --import-realm
```

### Solución 2. Usar PostgreSQL (producción).

**docker-compose.yml:**

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: keycloak
      POSTGRES_USER: keycloak
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  keycloak:
    image: quay.io/keycloak/keycloak:23.0.0
    environment:
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: admin
      KC_DB: postgres
      KC_DB_URL: jdbc:postgresql://postgres:5432/keycloak
      KC_DB_USERNAME: keycloak
      KC_DB_PASSWORD: password
    ports:
      - "8180:8080"
    depends_on:
      - postgres
    command: start-dev

volumes:
  postgres_data:
```

Ejecutar:
```bash
docker-compose up -d
```

Detener:
```bash
docker-compose down
```

**Ventaja:** La configuración persiste en el volumen de PostgreSQL.

---

## Troubleshooting

### ❌ Problema: "Connection refused" al conectar con Keycloak.

**Causa:** Keycloak no está corriendo.

**Solución:**
```bash
docker ps
# Si no aparece keycloak-vaultcorp:
docker start keycloak-vaultcorp
```

### ❌ Problema: "401 Unauthorized" al usar token.

**Verificar:**
1. ¿El token expiró? Los tokens OIDC expiran en 5 minutos (300s).
2. ¿Copiaste bien el `client_secret` en `application.properties`?
3. ¿La URL de Keycloak es correcta?

**Generar nuevo token:**
```bash
# Repetir el curl de obtención de token
```

### ❌ Problema: "403 Forbidden" con cliente básico.

**¡Eso es correcto!** El cliente básico (`customer`) **no** puede acceder a secretos `CONFIDENTIAL`.

### ❌ Problema: Keycloak reinicia y pierdo la configuración.

**Solución:** usa Docker Compose con PostgreSQL (ver sección "Persistir Configuración").

### ❌ Problema: Puerto 8180 ya en uso.

**Solución:**
```bash
# Cambiar el puerto al levantar Keycloak
docker run -p 8280:8080 ...
```

Y actualizar en `application.properties`:
```properties
quarkus.oidc.auth-server-url=http://localhost:8280/realms/vaultcorp
```

---

## Conceptos clave

### ¿Qué es OIDC?

**OpenID Connect (OIDC)** es un protocolo de autenticación construido sobre OAuth 2.0. Permite que las aplicaciones deleguen la autenticación a un **Identity Provider** (IdP) como Keycloak.

### Flujo OIDC simplificado.

```
1. Usuario → App: "quiero acceder".
2. App → Keycloak: "autentica a este usuario".
3. Usuario → Keycloak: ingresa credenciales.
4. Keycloak → App: "aquí está el token, está autenticado".
5. App: valida token y autoriza el acceso.
```

### Componentes OIDC

| Componente | Descripción | En este ejercicio |
|------------|-------------|-------------------|
| **Identity Provider (IdP)** | Servicio que autentica usuarios. | Keycloak. |
| **Relying Party (RP)** | Aplicación que confía en el IdP. | Tu app Quarkus. |
| **Access Token** | Token JWT para acceder a recursos. | Emitido por Keycloak. |
| **ID Token** | Token con info del usuario. | Emitido por Keycloak. |
| **Realm** | Espacio aislado de usuarios/roles. | `vaultcorp`. |
| **Client** | Aplicación registrada en el IdP. | `vault-api`. |

### Ventajas de OIDC

- ✅ **Single Sign-On (SSO)**: un login para múltiples apps.
- ✅ **Federación**: integración con Google, GitHub, etc.
- ✅ **Centralización**: gestión de usuarios en un solo lugar.
- ✅ **Seguridad**: identity provider especializado.
- ✅ **Escalabilidad**: separación de responsabilidades.

### Desventajas de OIDC

- ❌ **Dependencia externa**: requiere IdP corriendo.
- ❌ **Complejidad**: más componentes que gestionar.
- ❌ **Latencia**: validación de tokens contra Keycloak.
- ❌ **Setup inicial**: configuración más compleja.

---

## Comparativa final. Parte 1 vs. 2 vs. 3

| Aspecto | Parte 1 (Basic Auth) | Parte 2 (JWT) | Parte 3 (OIDC) |
|---------|----------------------|---------------|----------------|
| **Usuarios** | Admins/Auditores | Empleados | Clientes externos |
| **Endpoints** | `/api/admin/*` | `/api/internal/*` | `/api/external/*` |
| **Método** | Credenciales en cada request | Token JWT propio | Token OIDC de Keycloak |
| **Gestión de usuarios** | application.properties | Endpoint /login | Keycloak |
| **Gestión de roles** | application.properties | Código Java | Keycloak |
| **Expiración** | No | Sí (1 hora) | Sí (5 minutos) |
| **SSO** | No | No | Sí ✅ |
| **Complejidad** | Baja | Media | Alta |
| **Uso típico** | APIs internas | Apps móviles/SPAs | SaaS, multi-tenant |

---

## ✅ Checklist de verificación

- [ ] Keycloak está corriendo en Docker.
- [ ] Realm `vaultcorp` creado.
- [ ] Client `vault-api` configurado con client secret.
- [ ] Roles `customer` y `premium-customer` creados.
- [ ] Usuarios `client001` y `client002` creados con contraseñas.
- [ ] Roles asignados a usuarios correctamente.
- [ ] `application.properties` tiene configuración OIDC completa.
- [ ] `ExternalSecretResource.java` creado.
- [ ] Cliente básico puede ver secretos `PUBLIC`.
- [ ] Cliente básico **no** puede ver secretos `CONFIDENTIAL` (403).
- [ ] Cliente premium **sí** puede ver secretos `CONFIDENTIAL`.
- [ ] Script `test-part3-oidc.sh` ejecuta sin errores.

---

## Próximos pasos (opcional)

### Mejoras avanzadas.

1. **Integración con Google/GitHub**.
   - Configurar Identity Providers en Keycloak.
   - Permitir login social.

2. **Multi-tenancy con Organizations**.
   - Usar claim `organization` en tokens.
   - Filtrar secretos por organización.

3. **Refresh Tokens**.
   - Implementar renovación automática de tokens.
   - Mejorar la experiencia del usuario.

4. **Logout funcional**.
   - Endpoint de logout que invalida tokens en Keycloak.
   - Blacklist de tokens revocados.

5. **Admin UI con React**.
   - Interfaz gráfica para gestionar secretos.
   - Login con Keycloak desde navegador.

---

## Recursos adicionales

- [Keycloak Documentation](https://www.keycloak.org/documentation)
- [Quarkus OIDC Guide](https://quarkus.io/guides/security-oidc-bearer-token-authentication)
- [OpenID Connect Spec](https://openid.net/connect/)
- [OAuth 2.0 RFC 6749](https://tools.ietf.org/html/rfc6749)

---
