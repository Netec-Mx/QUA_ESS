# Sistema de evaluación crediticia - Quarkus

Sistema bancario para evaluar solicitudes de crédito de consumo mediante un algoritmo de scoring crediticio automatizado.

## Documentación del proyecto

<div align="center">

### 📖 [IR A TEORÍA](./TEORIA.md) | 🧪 [IR A TESTS](./TESTS.md)

</div>

## Características

- ✅ **Evaluación crediticia automática** con algoritmo de scoring (0-1000 puntos).
- ✅ **Validación exhaustiva** de datos (RUT chileno, email, rangos, etc.).
- ✅ **Persistencia con Panache** (PostgreSQL).
- ✅ **Dev Services** (base de datos automática en desarrollo).
- ✅ **Tests completos** (unitarios, integración, REST).
- ✅ **Compilación nativa** con GraalVM.
- ✅ **Manejo de errores** con Exception Mappers.
- ✅ **Validadores custom** (`@ValidRut`).

---

## Requisitos previos

### Ambos sistemas (Mac y Windows)

- **Java 21** ([Amazon Corretto](https://docs.aws.amazon.com/corretto/latest/corretto-21-ug/downloads-list.html) u [Oracle JDK](https://www.oracle.com/java/technologies/downloads/#java21)).
- **Maven 3.9+** ([Descargar](https://maven.apache.org/download.cgi)).
- **Docker Desktop** (opcional, para producción con PostgreSQL real).

### Verificar instalación:

**Mac/Linux:**
```bash
java -version
mvn -version
```

**Windows (PowerShell):**
```powershell
java -version
mvn -version
```

---

## Instalación y ejecución

### Clonar/descargar el proyecto.

```bash
cd evaluacion-crediticia
```

### Ejecutar en modo desarrollo.

Quarkus levantará automáticamente PostgreSQL con **Dev Services** (Testcontainers).

**Mac/Linux:**
```bash
./mvnw quarkus:dev
```

**Windows (CMD):**
```cmd
mvnw.cmd quarkus:dev
```

**Windows (PowerShell):**
```powershell
.\mvnw.cmd quarkus:dev
```

La aplicación estará disponible en: **http://localhost:8080**

**Hot Reload activado:** Cualquier cambio en el código se refleja automáticamente.

---

## Ejecutar tests

### Tests unitarios y de integración.

**Mac/Linux:**
```bash
./mvnw clean test
```

**Windows:**
```cmd
mvnw.cmd clean test
```

### Tests nativos (GraalVM).

**Mac/Linux:**
```bash
./mvnw verify -Dnative
```

**Windows:**
```cmd
mvnw.cmd verify -Dnative
```

### Ver cobertura de código.

**Mac/Linux:**
```bash
./mvnw clean verify
open target/site/jacoco/index.html
```

**Windows:**
```cmd
mvnw.cmd clean verify
start target\site\jacoco\index.html
```

---

## Compilación

### Compilar JAR ejecutable (JVM).

**Mac/Linux:**
```bash
./mvnw clean package
java -jar target/quarkus-app/quarkus-run.jar
```

**Windows:**
```cmd
mvnw.cmd clean package
java -jar target\quarkus-app\quarkus-run.jar
```

**Tiempo de arranque JVM:** ~ 1.5 segundos.
**Memoria RSS:** ~ 150 MB .

### Compilar binario nativo (GraalVM).

Requiere **GraalVM 21** con Native Image instalado ([Descargar](https://www.graalvm.org/downloads/))

**Mac/Linux:**
```bash
./mvnw package -Dnative
./target/evaluacion-crediticia-1.0.0-SNAPSHOT-runner
```

**Windows:**
```cmd
mvnw.cmd package -Dnative
target\evaluacion-crediticia-1.0.0-SNAPSHOT-runner.exe
```

**Tiempo de arranque nativo:** ~ 0.015 segundos.
**Memoria RSS:** ~ 30 MB .

---

## Probar la API

### Solicitud aprobada (perfil excelente).

**Mac/Linux (curl):**
```bash
curl -X POST http://localhost:8080/api/v1/creditos/evaluar \
  -H "Content-Type: application/json" \
  -d '{
    "rut": "12345678-5",
    "nombreCompleto": "Juan Pérez Test",
    "email": "juan.test@email.cl",
    "edad": 35,
    "ingresosMensuales": 2500000,
    "deudasActuales": 300000,
    "montoSolicitado": 5000000,
    "mesesEnEmpleoActual": 36
  }'
```

**Windows (PowerShell):**
```powershell
$body = @{
    rut = "12345678-5"
    nombreCompleto = "Juan Pérez Test"
    email = "juan.test@email.cl"
    edad = 35
    ingresosMensuales = 2500000
    deudasActuales = 300000
    montoSolicitado = 5000000
    mesesEnEmpleoActual = 36
} | ConvertTo-Json

Invoke-RestMethod -Uri http://localhost:8080/api/v1/creditos/evaluar `
  -Method Post `
  -ContentType "application/json" `
  -Body $body
```

**Respuesta esperada:**
```json
{
  "solicitudId": 6,
  "scoreCrediticio": 850,
  "aprobada": true,
  "razonEvaluacion": "Aprobado: Excelente perfil crediticio. Felicitaciones.",
  "estado": "APROBADA"
}
```

---

### Solicitud rechazada (DTI alto).

**Mac/Linux:**
```bash
curl -X POST http://localhost:8080/api/v1/creditos/evaluar \
  -H "Content-Type: application/json" \
  -d '{
    "rut": "23456789-6",
    "nombreCompleto": "María Silva Test",
    "email": "maria.test@email.cl",
    "edad": 28,
    "ingresosMensuales": 1000000,
    "deudasActuales": 700000,
    "montoSolicitado": 3000000,
    "mesesEnEmpleoActual": 6
  }'
```

**Windows (PowerShell):**
```powershell
$body = @{
    rut = "23456789-6"
    nombreCompleto = "María Silva Test"
    email = "maria.test@email.cl"
    edad = 28
    ingresosMensuales = 1000000
    deudasActuales = 700000
    montoSolicitado = 3000000
    mesesEnEmpleoActual = 6
} | ConvertTo-Json

Invoke-RestMethod -Uri http://localhost:8080/api/v1/creditos/evaluar `
  -Method Post `
  -ContentType "application/json" `
  -Body $body
```

**Respuesta esperada:**
```json
{
  "solicitudId": 7,
  "scoreCrediticio": 420,
  "aprobada": false,
  "razonEvaluacion": "Rechazado: Ratio deuda/ingreso (70.00%) supera el límite permitido (50%).",
  "estado": "RECHAZADA"
}
```

---

### Validación de RUT inválido.

**Mac/Linux:**
```bash
curl -X POST http://localhost:8080/api/v1/creditos/evaluar \
  -H "Content-Type: application/json" \
  -d '{
    "rut": "12345678-9",
    "nombreCompleto": "Test Inválido",
    "email": "test@email.cl",
    "edad": 30,
    "ingresosMensuales": 2000000,
    "deudasActuales": 300000,
    "montoSolicitado": 4000000,
    "mesesEnEmpleoActual": 12
  }'
```

**Respuesta esperada (400 Bad Request):**
```json
{
  "error": "RUT chileno inválido"
}
```

---

### Listar todas las solicitudes.

**Mac/Linux:**
```bash
curl http://localhost:8080/api/v1/creditos
```

**Windows:**
```powershell
Invoke-RestMethod -Uri http://localhost:8080/api/v1/creditos
```

---

### Obtener solicitud por ID.

**Mac/Linux:**
```bash
curl http://localhost:8080/api/v1/creditos/1
```

**Windows:**
```powershell
Invoke-RestMethod -Uri http://localhost:8080/api/v1/creditos/1
```

---

## Algoritmo de scoring

El sistema evalúa múltiples factores para calcular el score crediticio.

### Factores evaluados:

| Factor | Peso | Descripción |
|--------|------|-------------|
| **DTI (Debt-to-Income)** | Alto | Ratio deuda/ingreso. Límite: 50 % |
| **Estabilidad laboral** | Medio | Meses en empleo actual |
| **Capacidad de pago** | Alto | Cuota estimada vs. 30 % del ingreso |
| **Edad** | Bajo | Rango óptimo: 25 - 55 años |
| **Monto solicitado** | Medio | Ratio monto/ingreso mensual |

### Escala de score:

- **800 - 1000:** excelente (aprobación inmediata, mejores tasas).
- **650 - 799:** bueno (aprobación estándar).
- **500 - 649:** regular (requiere análisis manual).
- **0 - 499:** malo (rechazo automático).

### Umbral de aprobación: **650 puntos.**

---

## Estructura del proyecto

```
evaluacion-crediticia/
├── src/
│   ├── main/
│   │   ├── java/pe/banco/evaluacion/
│   │   │   ├── entidades/          # Entidades JPA
│   │   │   │   └── SolicitudCredito.java
│   │   │   ├── repositorios/       # Repositorios Panache
│   │   │   │   └── SolicitudCreditoRepository.java
│   │   │   ├── servicios/          # Lógica de negocio
│   │   │   │   ├── ScoringService.java
│   │   │   │   └── ValidacionService.java
│   │   │   ├── recursos/           # Endpoints REST
│   │   │   │   └── CreditoRecurso.java
│   │   │   ├── dtos/               # DTOs
│   │   │   │   ├── SolicitudCreditoDTO.java
│   │   │   │   └── RespuestaEvaluacionDTO.java
│   │   │   ├── validadores/        # Validadores custom
│   │   │   │   ├── ValidRut.java
│   │   │   │   └── ValidadorRut.java
│   │   │   └── excepciones/        # Exception mappers
│   │   │       ├── ValidationExceptionMapper.java
│   │   │       └── GenericExceptionMapper.java
│   │   └── resources/
│   │       ├── application.properties
│   │       └── import.sql
│   └── test/
│       └── java/pe/banco/evaluacion/
│           ├── servicios/          # Tests de servicios
│           ├── recursos/           # Tests de endpoints
│           ├── repositorios/       # Tests de repositorios
│           ├── validadores/        # Tests de validadores
│           └── NativeImageIT.java  # Test nativo
├── pom.xml
└── README.md
```

---

## Cobertura de tests

El proyecto incluye:

- ✅ **Tests unitarios** (servicios, validadores).
- ✅ **Tests de integración** (repositorios con BD).
- ✅ **Tests REST** (endpoints con REST Assured).
- ✅ **Tests parametrizados** (múltiples escenarios).
- ✅ **Tests nativos** (compilación GraalVM).

**Cobertura objetivo:** > 85 % .

---

## Docker

### Crear imagen Docker (JVM).

**Mac/Linux:**
```bash
./mvnw package
docker build -f src/main/docker/Dockerfile.jvm -t evaluacion-crediticia:jvm .
docker run -i --rm -p 8080:8080 evaluacion-crediticia:jvm
```

**Windows:**
```cmd
mvnw.cmd package
docker build -f src/main/docker/Dockerfile.jvm -t evaluacion-crediticia:jvm .
docker run -i --rm -p 8080:8080 evaluacion-crediticia:jvm
```

### Crear imagen Docker (nativa).

**Mac/Linux:**
```bash
./mvnw package -Dnative -Dquarkus.native.container-build=true
docker build -f src/main/docker/Dockerfile.native -t evaluacion-crediticia:native .
docker run -i --rm -p 8080:8080 evaluacion-crediticia:native
```

**Windows:**
```cmd
mvnw.cmd package -Dnative -Dquarkus.native.container-build=true
docker build -f src/main/docker/Dockerfile.native -t evaluacion-crediticia:native .
docker run -i --rm -p 8080:8080 evaluacion-crediticia:native
```
 
---

## Troubleshooting

### ❌ Problema: los tests fallan con "Connection refused".

**Solución:** asegúrate de tener Docker Desktop ejecutándose (Dev Services lo necesita).

**Mac/Linux:**
```bash
docker ps
```

**Windows:**
```powershell
docker ps
```

Si Docker no está corriendo, inícialo desde Docker Desktop.

---

### ❌ Problema: Puerto 8080 ya en uso.

**Solución:** cambia el puerto en `application.properties`:

```properties
quarkus.http.port=8081
```

O mata el proceso que usa el puerto:

**Mac/Linux:**
```bash
lsof -ti:8080 | xargs kill -9
```

**Windows:**
```powershell
Get-Process -Id (Get-NetTCPConnection -LocalPort 8080).OwningProcess | Stop-Process
```

---

### ❌ Problema: "JAVA_HOME not set".

**Mac/Linux:**
```bash
export JAVA_HOME=$(/usr/libexec/java_home -v 21)
```

**Windows (PowerShell):**
```powershell
$env:JAVA_HOME = "C:\Program Files\Java\jdk-21"
```

---

## Recursos adicionales

- [Documentación Quarkus](https://quarkus.io/guides/)
- [Panache Guide](https://quarkus.io/guides/hibernate-orm-panache)
- [Testing Guide](https://quarkus.io/guides/getting-started-testing)
- [Native Build Guide](https://quarkus.io/guides/building-native-image)
- [Container Guide](https://quarkus.io/guides/container-image)

---
