# Práctica 1. Proyecto Quarkus: Hola, mundo
 
## Objetivos
Al finalizar la práctica, serás capaz de:
- Desarrollo de microservicios en un proyecto básico de Quarkus con endpoint REST simple.

 ## Duración aproximada
- 90 minutos.
 
##  Prerrequisitos
- **Java 17 o superior** (recomendado: Java 21 LTS).
- **Maven 3.9+** (incluido en el proyecto como Maven Wrapper).
- **IDE** (VS Code, IntelliJ IDEA, Eclipse).
- **Terminal** (Git Bash, PowerShell, terminal de macOS).

## 🛠️ Instalación del entorno

### 🍎 macOS

**Opción 1. Con Homebrew (recomendado)**

```bash
# Instalar Homebrew si no lo tienes
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Instalar Java 21
brew install openjdk@21

# Configurar Java (agregar al PATH)
echo 'export PATH="/opt/homebrew/opt/openjdk@21/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc

# Instalar Quarkus CLI
brew install quarkusio/tap/quarkus

# Verificar instalación
java -version
quarkus --version
```

**Opción 2. Con SDKMAN (para gestión avanzada de versiones)**

```bash
# Instalar SDKMAN
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"

# Instalar Java 21
sdk install java 21-tem
sdk use java 21-tem

# Instalar Quarkus CLI
sdk install quarkus

# Verificar instalación
java -version
quarkus --version
```

---

### 🪟 Windows

**Opción 1. Con Chocolatey (recomendado)**

```powershell
# 1. Instalar Chocolatey (ejecutar PowerShell como administrador)
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

# 2. Instalar Java 21
choco install openjdk21 -y

# 3. Instalar Quarkus CLI
choco install quarkus -y

# 4. Reiniciar PowerShell y verificar
java -version
quarkus --version
```

**Opción 2. Con Scoop (alternativa moderna)**

```powershell
# 1. Instalar Scoop (PowerShell normal, no requiere admin)
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
irm get.scoop.sh | iex

# 2. Agregar bucket de Java
scoop bucket add java

# 3. Instalar herramientas
scoop install openjdk21
scoop install maven
scoop install quarkus-cli

# 4. Verificar instalación
java -version
quarkus --version
```

**Opción 3. Instalación manual**

1. **Descargar Java 21**
   - Ve a [Adoptium](https://adoptium.net/).
   - Descarga "Eclipse Temurin 21 (LTS)".
   - Instala siguiendo el wizard.

2. **Configurar variables de entorno**
   - Abre "Variables de entorno del sistema".
   - Crea `JAVA_HOME` apuntando a: `C:\Program Files\Eclipse Adoptium\jdk-21.x.x`.
   - Agrega a `PATH`: `%JAVA_HOME%\bin`.

3. **Descargar Quarkus CLI**
   - Ve a [Quarkus CLI Releases](https://github.com/quarkusio/quarkus/releases)
   - Descarga y agrega al `PATH`.

4. **Verificar**
   ```cmd
   java -version
   quarkus --version
   ```

**Opción 4. WSL2 + SDKMAN (para desarrolladores avanzados)**

```bash
# 1. Instalar WSL2 (PowerShell como admin)
wsl --install

# 2. Reiniciar y abrir Ubuntu/WSL.
# 3. Seguir los pasos de SDKMAN de macOS
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"
sdk install java 21-tem
sdk install quarkus
```

---

## Instrucciones
### Tarea 1
#### Opción 1. Con Quarkus CLI (recomendado)

```bash
# Crear proyecto con extensión REST
quarkus create app cl.alchemicaldata:banco \
    --extension=rest \
    --no-wrapper

# Entrar al directorio
cd banco
```

#### Opción 2. Con Quarkus CLI sin código (educativo)

```bash
# Crear proyecto limpio
quarkus create app cl.alchemicaldata:banco --no-code

# Entrar al directorio
cd banco

# Agregar extensión REST después
./mvnw quarkus:add-extension -Dextensions="rest"
```

#### Opción 3. Desde Maven Archetype

```bash
mvn io.quarkus.platform:quarkus-maven-plugin:3.15.1:create \
    -DprojectGroupId=cl.alchemicaldata \
    -DprojectArtifactId=banco \
    -DprojectVersion=1.0.0-SNAPSHOT \
    -Dextensions=rest
    
cd banco
```

#### Opción 4. Desde la web (más visual)

1. Ir a [code.quarkus.io](https://code.quarkus.io)
2. Configurar:
   - **Group:** `cl.alchemicaldata`
   - **Artifact:** `banco`
   - **Build Tool:** Maven
   - **Java Version:** 21
3. Agregar extensión: **RESTEasy Reactive**
4. Generar y descargar ZIP
5. Descomprimir y abrir el proyecto

---

### Tarea 2. Estructurar el proyecto

```
banco/
├── mvnw                          # Maven Wrapper (macOS/Linux)
├── mvnw.cmd                      # Maven Wrapper (Windows)
├── pom.xml                       # Configuración Maven
├── README.md                     # Documentación
├── src/
│   ├── main/
│   │   ├── docker/               # Dockerfiles (JVM y Native)
│   │   ├── java/
│   │   │   └── cl/alchemicaldata/banco/
│   │   │       └── HelloResource.java
│   │   └── resources/
│   │       └── application.properties
│   └── test/
│       └── java/
│           └── cl/alchemicaldata/banco/
│               └── HelloResourceTest.java
├── target/                       # Archivos compilados (generado)
└── .mvn/                         # Configuración Maven Wrapper
```

---

### Tarea 3. Configuración inicial

**Paso 1.** Posicionarse en el directorio del proyecto.

```bash
# macOS/Linux/Git Bash
cd banco

# Windows CMD
cd banco
```

**Paso 2.** Dar permisos al Maven Wrapper (solo macOS/Linux/Git Bash).

```bash
chmod +x mvnw
```

**Paso 3.** Verificar que la extensión REST está instalada.

Revisar que el archivo `pom.xml` contenga:

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-rest</artifactId>
</dependency>
```

Si no está, agregar:

**macOS/Linux/Git Bash:**
```bash
./mvnw quarkus:add-extension -Dextensions="rest"
```

**Windows (CMD/PowerShell):**
```cmd
mvnw.cmd quarkus:add-extension -Dextensions="rest"
```

---

 
### Tarea 4. Crear el endpoint `HelloResource`

**Paso 1.** Crear el archivo: `src/main/java/cl/alchemicaldata/banco/HelloResource.java`

```java
package cl.alchemicaldata.banco;

import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;
import jakarta.ws.rs.Produces;
import jakarta.ws.rs.core.MediaType;

@Path("/hello")
public class HelloResource {

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    public String hello() {
        return "Hola mundo desde Quarkus 🧉";
    }
}
```

---

### Tarea 5. Ejecución del proyecto

#### Modo: desarrollo (hot reload automático)

**macOS/Linux/Git Bash:**
```bash
./mvnw quarkus:dev
```

**Windows (CMD/PowerShell):**
```cmd
mvnw.cmd quarkus:dev
```

## Resultado esperado

```
__  ____  __  _____   ___  __ ____  ______ 
 --/ __ \/ / / / _ | / _ \/ //_/ / / / __/ 
 -/ /_/ / /_/ / __ |/ , _/ ,< / /_/ /\ \   
--\___\_\____/_/ |_/_/|_/_/|_|\____/___/   

INFO  [io.quarkus] (Quarkus Main Thread) banco 1.0.0-SNAPSHOT on JVM started in 1.234s
INFO  [io.quarkus] (Quarkus Main Thread) Listening on: http://localhost:8080

Tests paused
Press [e] to edit command line args, [r] to resume testing, [h] for more options>
```

**Accesos**
- **Endpoint:** http://localhost:8080/hello
- **Dev UI:** http://localhost:8080/q/dev
- **Health Check:** http://localhost:8080/q/health
- **Metrics:** http://localhost:8080/q/metrics

**Paso 1.** Compilar sin ejecutar

**macOS/Linux/Git Bash:**
```bash
./mvnw clean compile
```

**Windows:**
```cmd
mvnw.cmd clean compile
```

**Paso 2.** Empaquetar aplicación (JAR)

**macOS/Linux/Git Bash:**
```bash
./mvnw package
```

**Windows:**
```cmd
mvnw.cmd package
```

**Paso 3.** Ejecutar JAR empaquetado

**macOS/Linux/Git Bash:**
```bash
java -jar target/quarkus-app/quarkus-run.jar
```

**Windows:**
```cmd
java -jar target\quarkus-app\quarkus-run.jar
```

---

### Tarea 6. Pruebas

**Paso 1.** Probar el endpoint manualmente

**Opción 1. Navegador**
```
http://localhost:8080/hello
```

**Opción 2. curl (macOS/Linux/Git Bash)**
```bash
curl http://localhost:8080/hello
```

**Opción 3. PowerShell (Windows)**
```powershell
Invoke-WebRequest -Uri http://localhost:8080/hello | Select-Object -Expand Content
```

**Opción 4. Postman/Insomnia**
- Method: GET
- URL: http://localhost:8080/hello

**Paso 2.** Ejecutar tests automatizados

**macOS/Linux/Git Bash:**
```bash
./mvnw test
```

**Windows:**
```cmd
mvnw.cmd test
```

---

### Tarea 6. Docker (opcional)

**Paso 1.** Construir imagen Docker (JVM Mode)

```bash
docker build -f src/main/docker/Dockerfile.jvm -t banco:1.0.0-jvm .
```

**Paso 2.** Ejecutar contenedor

```bash
docker run -i --rm -p 8080:8080 banco:1.0.0-jvm
```

**Paso 3.** Construir imagen nativa (requiere GraalVM)

```bash
./mvnw package -Pnative -Dquarkus.native.container-build=true
docker build -f src/main/docker/Dockerfile.native -t banco:1.0.0-native .
```

---

## 🔍 Comandos útiles en modo dev

Cuando la aplicación está corriendo con `quarkus:dev`, puedes usar estas teclas:

| Tecla | Acción |
|-------|--------|
| **`w`** | Abrir Dev UI en navegador |
| **`d`** | Abrir documentación |
| **`r`** | Ejecutar tests |
| **`s`** | Ver métricas |
| **`h`** | Ver todas las opciones |
| **`q`** | Salir de la aplicación |
| **`Ctrl+C`** | Forzar salida |

---

## ⚙️ Configuración (`application.properties`)

Archivo: `src/main/resources/application.properties`

```properties
# Puerto del servidor (default: 8080)
quarkus.http.port=8080

# Habilitar CORS en desarrollo
quarkus.http.cors=true

# Nivel de log
quarkus.log.level=INFO
quarkus.log.console.level=INFO

# Hot reload (activado por defecto en dev mode)
quarkus.live-reload.instrumentation=true
```

---

## 🚨 Solución de problemas comunes

### ❌ Error: `jakarta.ws.rs not found` o` imports` subrayados en rojo.

**Causa:** falta la extensión `REST`.

**Solución**
```bash
# macOS/Linux/Git Bash
./mvnw quarkus:add-extension -Dextensions="rest"

# Windows
mvnw.cmd quarkus:add-extension -Dextensions="rest"
```

### ❌ Error: `Permission denied: ./mvnw` (macOS/Linux).

**Causa:** el wrapper no tiene permisos de ejecución.

**Solución**
```bash
chmod +x mvnw
```

### ❌ Error: `Port 8080 already in use`.

**Causa:** otro proceso está usando el puerto 8080.

**Solución 1. Cambiar puerto.**

En `application.properties`:
```properties
quarkus.http.port=8081
```

**Solución 2. Liberar puerto.**

 Para macOS/Linux:
```bash
lsof -ti:8080 | xargs kill -9
```

Para Windows PowerShell como admin:
```powershell
Get-Process -Id (Get-NetTCPConnection -LocalPort 8080).OwningProcess | Stop-Process
```

### ❌ Error: `JAVA_HOME is not set` (Windows).

**Solución**
```cmd
# Verificar si Java está instalado
java -version

# Configurar JAVA_HOME (PowerShell como admin)
[System.Environment]::SetEnvironmentVariable("JAVA_HOME", "C:\Program Files\Eclipse Adoptium\jdk-21.x.x", "Machine")

# Reiniciar PowerShell
```

### ❌ Error: Maven Wrapper no funciona en Windows.

**Causa:** estás usando `./mvnw` en `CMD/PowerShell`.

**Solución:** usar `mvnw.cmd`.
```cmd
mvnw.cmd quarkus:dev
```

### ❌ Error: `Failed to execute goal... dependencies could not be resolved`.

**Causa:** Maven no puede descargar dependencias.

**Solución**
```bash
# Limpiar caché de Maven y reintentar
./mvnw dependency:purge-local-repository
./mvnw clean install
```

---

## 📚 Recursos adicionales

### Documentación oficial
- [Guías de Quarkus](https://quarkus.io/guides/)
- [Quarkus Cheat Sheet](https://lordofthejars.github.io/quarkus-cheat-sheet/)
- [REST con Quarkus](https://quarkus.io/guides/rest)
- [Quarkus Dev Services](https://quarkus.io/guides/dev-services)

### Comunidad
- [Quarkus GitHub](https://github.com/quarkusio/quarkus)
- [Stack Overflow - Tag: quarkus](https://stackoverflow.com/questions/tagged/quarkus)
- [Quarkus Zulip Chat](https://quarkusio.zulipchat.com/)

### Extensiones útiles para VS Code
- **Extension Pack for Java** (Microsoft)
- **Quarkus Tools** (Red Hat)
- **REST Client** (Huachao Mao)
- **Thunder Client** (RangaV Vadhineni)

---

## 📝 Notas importantes 

### Para usuarios de Windows
- Se recomienda usar **PowerShell** o **Git Bash** en lugar de CMD.
- Si usas Git Bash, los comandos son iguales a macOS/Linux (`./mvnw`).
- Si usan CMD/PowerShell, deben usar `mvnw.cmd` en lugar de `./mvnw`.
- **Chocolatey** facilita la instalación, considera usarlo.

### Para usuarios de macOS
- **Homebrew** es la forma más simple de instalar todo.
- **SDKMAN** es útil si necesitas cambiar versiones de Java frecuentemente.
- El terminal por defecto (zsh) funciona perfecto.

### Buenas prácticas
1. **Siempre** estar en el directorio raíz del proyecto (donde está `pom.xml`).
2. **Verificar** que Java y Maven estén instalados antes de empezar.
3. **Usar modo dev** (`quarkus:dev`) durante desarrollo para hot reload.
4. **Revisar logs** cuando algo falle, Quarkus da mensajes claros.
5. **Explorar Dev UI** (`http://localhost:8080/q/dev`) tiene muchas herramientas útiles.
