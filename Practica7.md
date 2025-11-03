# TasaCorp API. Configuración y perfiles en Quarkus

## Capítulo 7. Externalización de configuraciones y perfiles de entorno

---

## Descripción

**TasaCorp** es un sistema bancario para consulta y conversión de tasas de cambio de divisas. Este ejercicio práctico está diseñado para dominar la **configuración y gestión de perfiles** en Quarkus, cubriendo desde los conceptos básicos hasta la integración con HashiCorp Vault.

**Contexto bancario:**
- Banco peruano: TasaCorp.
- Operaciones: compra/venta de USD, EUR, MXN.
- Ambientes: desarrollo, testing, producción.
- Seguridad: secrets protegidos con Vault.

---

## Objetivos de aprendizaje

- ✅ Externalizar configuraciones con `application.properties` y `application.yaml`.
- ✅ Entender prioridades de carga (System Props > ENV > Files).
- ✅ Usar perfiles de entorno (`%dev`, `%test`, `%prod`).
- ✅ Proteger información sensible con HashiCorp Vault.
- ✅ Aplicar mejores prácticas de configuración en producción.

---

## Documentación del ejercicio

### Guías prácticas (paso a paso).

| Documento | Duración | Descripción |
|-----------|----------|-------------|
| **[README-PARTE1.md](README-PARTE1.md)** | 30 min. | **Externalización de configuraciones**<br>Properties, YAML, inyección, prioridades de carga |
| **[README-PARTE2.md](README-PARTE2.md)** | 30 min. | **Perfiles y configuración sensible**<br>%dev, %test, %prod, integración con Vault |

### Teoría profunda

| Documento | Contenido |
|-----------|-----------|
| **[TEORIA-PARTE1.md](TEORIA-PARTE1.md)** | **Fundamentos de configuración**<br>Historia, MicroProfile Config, tipos de datos, patrones, mejores prácticas |
| **[TEORIA-PARTE2.md](TEORIA-PARTE2.md)** | **Perfiles y seguridad**<br>Arquitectura de perfiles, HashiCorp Vault, gestión de secretos, casos reales |

---

## Inicio rápido

### Prerequisitos

```bash
# Java 17+
java -version

# Maven
mvn -version

# Docker Desktop (para Vault)
docker --version
```

### Crear proyecto

#### Windows
```powershell
mvn io.quarkus.platform:quarkus-maven-plugin:3.17.5:create `
    -DprojectGroupId=pe.banco `
    -DprojectArtifactId=tasacorp-api `
    -Dextensions="resteasy-reactive-jackson,config-yaml,vault"

cd tasacorp-api
```

#### macOS/Linux
```bash
mvn io.quarkus.platform:quarkus-maven-plugin:3.17.5:create \
    -DprojectGroupId=pe.banco \
    -DprojectArtifactId=tasacorp-api \
    -Dextensions="resteasy-reactive-jackson,config-yaml,vault"

cd tasacorp-api
```

### Arrancar

#### Windows
```powershell
.\mvnw.cmd quarkus:dev
```

#### macOS/Linux
```bash
./mvnw quarkus:dev
```

Abre: http://localhost:8080/api/tasas/config

---

## Estructura del proyecto

```
tasacorp-api/
├── README.md                    ← Estás aquí
├── README-PARTE1.md             ← Guía: externalización
├── README-PARTE2.md             ← Guía: perfiles + Vault
├── TEORIA-PARTE1.md             ← Teoría: configuración
├── TEORIA-PARTE2.md             ← Teoría: seguridad
├── docker-compose.yml           ← Vault para parte 2
├── pom.xml
└── src/
    └── main/
        ├── java/pe/banco/tasacorp/
        │   ├── config/              ← @ConfigMapping
        │   ├── model/               ← DTO
        │   ├── service/             ← lógica de negocio
        │   └── resource/            ← REST endpoints
        └── resources/
            ├── application.properties
            └── application.yaml
```

---

## Ruta de aprendizaje

### Parte 1. Externalización (30 min.).

1. **Leer:** [TEORIA-PARTE1.md](TEORIA-PARTE1.md) (10 min.).
2. **Practicar:** [README-PARTE1.md](README-PARTE1.md) (20 min.).
   - Crear proyecto.
   - Configurar properties y yaml.
   - Probar prioridades de carga.

**Al finalizar, dominarás:**
- `application.properties` vs `application.yaml`.
- `@ConfigProperty` vs `@ConfigMapping`.
- Prioridades: system properties > ENV > files.

### Parte 2. Perfiles y Vault (30 min.).

1. **Leer:** [TEORIA-PARTE2.md](TEORIA-PARTE2.md) (10 min.).
2. **Practicar:** [README-PARTE2.md](README-PARTE2.md) (20 min.).
   - Configurar perfiles (dev, test, prod).
   - Levantar Vault con Docker.
   - Integrar Vault con Quarkus.

**Al finalizar, dominarás:**
- Perfiles de entorno.
- Configuración específica por ambiente.
- Protección de secretos con Vault.

---

## Endpoints disponibles

| Endpoint | Descripción | Ejemplo |
|----------|-------------|---------|
| `GET /api/tasas/config` | Ver configuración actual | Ver ambiente activo |
| `GET /api/tasas/{moneda}` | Consultar tasa | `/api/tasas/USD` |
| `GET /api/tasas/convertir/{moneda}?monto=X` | Convertir monto | `/api/tasas/convertir/USD?monto=1000` |
| `GET /api/tasas/health` | Health check | Estado del servicio |

---

## Comparativa de perfiles

| Característica | DEV | TEST | PROD |
|----------------|-----|------|------|
| **Comisión** | 0.0% | 1.5% | 2.5% |
| **Límite Trans.** | Ilimitado | $1,000 | $50,000 |
| **Cache** | ❌ | ✅ | ✅ |
| **Logs** | DEBUG | INFO | ERROR |
| **API Key** | Hardcoded | Hardcoded | 🔐 Vault |

---

## Tecnologías

- **Quarkus** 3.17.5+
- **Java** 17+
- **Maven** 3.8+
- **Docker** (para Vault)
- **HashiCorp Vault** 1.15.2

---

## Recursos adicionales

- [Quarkus Configuration Guide](https://quarkus.io/guides/config)
- [Quarkus Vault Extension](https://quarkus.io/guides/vault)
- [MicroProfile Config](https://github.com/eclipse/microprofile-config)
- [HashiCorp Vault](https://www.vaultproject.io/)
- [12-Factor App](https://12factor.net/)

---

## ✅ Verificación rápida

Antes de dar por completado el ejercicio, verifica:

- [ ] El proyecto está creado y compila sin errores.
- [ ] Entiendes properties vs. yaml.
- [ ] Probaste las 3 prioridades de carga.
- [ ] Los 3 perfiles funcionan (dev, test, prod).
- [ ] Vault está corriendo y conectado.
- [ ] Los secretos se leen desde Vault en `PROD`.

---

**¿Listo para empezar? Comienza con [README-PARTE1.md](README-PARTE1.md)** 🚀

---
