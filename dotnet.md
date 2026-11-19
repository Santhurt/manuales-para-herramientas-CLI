# Manual del CLI de .NET (`dotnet`)

> Manual de referencia rápida para crear, compilar, probar y publicar aplicaciones .NET desde la terminal.
> Basado en el SDK de **.NET 10** (la mayoría de comandos funcionan igual en .NET 8 y 9; las diferencias se indican).

---

## Índice

1. [Chuleta rápida](#1-chuleta-rápida)
2. [Creación y administración de proyectos](#2-creación-y-administración-de-proyectos)
3. [Soluciones y dependencias multiproyecto](#3-soluciones-y-dependencias-multiproyecto)
4. [Gestión de paquetes (NuGet)](#4-gestión-de-paquetes-nuget)
5. [Compilación y ejecución](#5-compilación-y-ejecución)
6. [Testeo](#6-testeo)
7. [Publicación y distribución](#7-publicación-y-distribución)
8. [Gestión de herramientas y workloads](#8-gestión-de-herramientas-y-workloads)
9. [Información y diagnóstico del entorno](#9-información-y-diagnóstico-del-entorno)
10. [Solución de problemas](#10-solución-de-problemas)

---

## 1. Chuleta rápida

Los comandos que se usan el 90% del tiempo.

| Acción | Comando |
|---|---|
| Crear app de consola | `dotnet new console -n MiApp` |
| Crear solución | `dotnet new sln -n MiSolucion` |
| Añadir proyecto a la solución | `dotnet sln add src/MiApp` |
| Referenciar otro proyecto | `dotnet reference add ../Core` |
| Añadir paquete NuGet | `dotnet package add Serilog` |
| Restaurar dependencias | `dotnet restore` |
| Compilar | `dotnet build` |
| Ejecutar | `dotnet run` |
| Ejecutar con recarga en caliente | `dotnet watch` |
| Ejecutar tests | `dotnet test` |
| Publicar en Release | `dotnet publish -c Release` |
| Instalar herramienta global | `dotnet tool install -g dotnet-ef` |
| Ver info del entorno | `dotnet --info` |

**Proyecto completo en una línea:**

```bash
dotnet new console -n Hola && cd Hola && dotnet run
```

> **Nota de sintaxis (.NET 10):** los comandos pasan a formato *sustantivo-verbo*
> (`dotnet package add`, `dotnet reference add`). La sintaxis antigua
> (`dotnet add package`, `dotnet add reference`) sigue funcionando.

---

## 2. Creación y administración de proyectos

### Plantillas (`dotnet new`)

```bash
dotnet new list                     # Listar plantillas instaladas
dotnet new list web                 # Filtrar por nombre
dotnet new list --tag Test          # Filtrar por etiqueta
dotnet new search blazor            # Buscar plantillas en NuGet.org
dotnet new console --help           # Ver opciones de una plantilla
```

### Plantillas de proyecto más comunes

| Plantilla | Descripción |
|---|---|
| `console` | Aplicación de consola |
| `classlib` | Biblioteca de clases |
| `web` | ASP.NET Core vacío |
| `webapi` | API REST (Minimal API o controladores) |
| `mvc` | ASP.NET Core MVC |
| `razor` | Razor Pages |
| `blazor` | Blazor Web App |
| `grpc` | Servicio gRPC |
| `worker` | Servicio en segundo plano (Worker Service) |
| `xunit` / `nunit` / `mstest` | Proyectos de tests |
| `avalonia.app`* | App de escritorio multiplataforma |

\* Requiere instalar la plantilla: `dotnet new install Avalonia.Templates`

### Plantillas de archivo (*item templates*)

| Plantilla | Genera |
|---|---|
| `gitignore` | `.gitignore` para .NET |
| `editorconfig` | `.editorconfig` con reglas de estilo |
| `globaljson` | `global.json` (fija versión del SDK) |
| `nugetconfig` | `nuget.config` |
| `buildprops` | `Directory.Build.props` |
| `packagesprops` | `Directory.Packages.props` (gestión central de paquetes) |
| `tool-manifest` | `.config/dotnet-tools.json` |
| `apicontroller` | Controlador de API |
| `mstest-class` | Clase de tests MSTest |

### Opciones comunes de `dotnet new`

| Flag | Descripción |
|---|---|
| `-n`, `--name` | Nombre del proyecto (y de la carpeta) |
| `-o`, `--output` | Directorio de salida |
| `-f`, `--framework` | Framework destino (`net10.0`, `net9.0`, `net8.0`) |
| `--language` | `C#` (por defecto), `F#`, `VB` |
| `--force` | Sobrescribir archivos existentes |
| `--dry-run` | Mostrar qué se crearía, sin crear nada |
| `--use-program-main` | Usar `Main` explícito en lugar de *top-level statements* |
| `--no-restore` | No ejecutar `restore` tras crear |

### Ejemplos

```bash
# Consola con Main clásico y framework concreto
dotnet new console -n Calculadora -f net9.0 --use-program-main

# API con controladores y sin OpenAPI
dotnet new webapi -n Api -o src/Api --use-controllers --no-openapi

# Archivos de configuración del repositorio
dotnet new gitignore
dotnet new editorconfig
dotnet new globaljson --sdk-version 10.0.100

# Controlador de API dentro del proyecto actual
dotnet new apicontroller -n UsuariosController -o Controllers
```

### Gestión de plantillas

```bash
dotnet new install Avalonia.Templates          # Instalar paquete de plantillas
dotnet new install ./mi-plantilla/             # Instalar desde carpeta local
dotnet new uninstall                           # Listar plantillas desinstalables
dotnet new uninstall Avalonia.Templates        # Desinstalar
dotnet new update                              # Actualizar paquetes de plantillas
```

### Apps de un solo archivo (.NET 10+)

Ejecutar un `.cs` suelto sin `.csproj`:

```bash
dotnet run app.cs                      # Compila y ejecuta
dotnet project convert app.cs          # Convertir a proyecto completo
```

Directivas dentro del archivo:

```csharp
#:package Humanizer@2.14.1
#:sdk Microsoft.NET.Sdk.Web
#:property TargetFramework=net10.0
```

Con shebang (`#!/usr/bin/env dotnet`) y `chmod +x app.cs` se puede ejecutar como script: `./app.cs`.

### Secretos de usuario (desarrollo)

```bash
dotnet user-secrets init
dotnet user-secrets set "ConnectionStrings:Db" "Host=localhost;Database=app"
dotnet user-secrets list
dotnet user-secrets remove "ConnectionStrings:Db"
dotnet user-secrets clear
```

> Se guardan fuera del repo en `~/.microsoft/usersecrets/<UserSecretsId>/secrets.json`.

### Formato de código

```bash
dotnet format                          # Aplicar estilo + analizadores
dotnet format --verify-no-changes      # Solo verificar (útil en CI)
dotnet format whitespace               # Solo espacios/indentación
dotnet format style                    # Solo reglas de estilo de código
dotnet format analyzers                # Solo diagnósticos de analizadores
```

---

## 3. Soluciones y dependencias multiproyecto

### Archivo de solución

| Formato | Descripción |
|---|---|
| `.sln` | Formato clásico de Visual Studio |
| `.slnx` | Formato XML simplificado (por defecto desde .NET 10) |

```bash
dotnet new sln -n MiSolucion                  # Crear solución (.slnx en .NET 10)
dotnet new sln -n MiSolucion --format sln     # Forzar formato clásico
dotnet sln migrate MiSolucion.sln             # Convertir .sln → .slnx
```

### Gestionar proyectos de la solución

```bash
dotnet sln add src/Api/Api.csproj                       # Añadir un proyecto
dotnet sln add src/**/*.csproj                          # Añadir varios (glob de bash)
dotnet sln add tests/Api.Tests --solution-folder tests  # En una carpeta de solución
dotnet sln add libs/Core --in-root                      # En la raíz, sin carpeta
dotnet sln list                                         # Listar proyectos
dotnet sln remove src/Viejo/Viejo.csproj                # Quitar proyecto
dotnet sln MiSolucion.slnx list                         # Si hay varias soluciones
```

> `dotnet sln` y `dotnet solution` son equivalentes.

### Referencias entre proyectos

```bash
# Desde la carpeta del proyecto que depende
dotnet reference add ../Core/Core.csproj
dotnet reference list
dotnet reference remove ../Core/Core.csproj

# Indicando el proyecto origen explícitamente
dotnet reference add src/Core --project src/Api/Api.csproj

# Sintaxis antigua (.NET 8/9)
dotnet add src/Api reference src/Core
```

### Ejemplo: estructura por capas desde cero

```bash
mkdir Tienda && cd Tienda
dotnet new sln -n Tienda
dotnet new gitignore

dotnet new classlib -n Tienda.Domain         -o src/Tienda.Domain
dotnet new classlib -n Tienda.Application    -o src/Tienda.Application
dotnet new classlib -n Tienda.Infrastructure -o src/Tienda.Infrastructure
dotnet new webapi   -n Tienda.Api            -o src/Tienda.Api
dotnet new xunit    -n Tienda.Tests          -o tests/Tienda.Tests

dotnet sln add src/*/*.csproj
dotnet sln add tests/*/*.csproj --solution-folder tests

dotnet reference add src/Tienda.Domain         --project src/Tienda.Application
dotnet reference add src/Tienda.Application    --project src/Tienda.Infrastructure
dotnet reference add src/Tienda.Application src/Tienda.Infrastructure --project src/Tienda.Api
dotnet reference add src/Tienda.Application    --project tests/Tienda.Tests

dotnet build
```

Resultado:

```
Tienda/
├── Tienda.slnx
├── src/
│   ├── Tienda.Domain/
│   ├── Tienda.Application/       → Domain
│   ├── Tienda.Infrastructure/    → Application
│   └── Tienda.Api/               → Application, Infrastructure
└── tests/
    └── Tienda.Tests/             → Application
```

### Configuración compartida: `Directory.Build.props`

Se aplica automáticamente a todos los proyectos en esa carpeta y subcarpetas.

```bash
dotnet new buildprops
```

```xml
<Project>
  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
  </PropertyGroup>
</Project>
```

### Gestión central de versiones: `Directory.Packages.props`

Las versiones de paquetes se declaran una vez y los `.csproj` solo indican el nombre.

```bash
dotnet new packagesprops
```

```xml
<Project>
  <PropertyGroup>
    <ManagePackageVersionsCentrally>true</ManagePackageVersionsCentrally>
  </PropertyGroup>
  <ItemGroup>
    <PackageVersion Include="Serilog" Version="4.2.0" />
    <PackageVersion Include="xunit" Version="2.9.3" />
  </ItemGroup>
</Project>
```

```xml
<!-- En cada .csproj, sin versión -->
<PackageReference Include="Serilog" />
```

> Con CPM activo, `dotnet package add` escribe la versión en `Directory.Packages.props` automáticamente.

---

## 4. Gestión de paquetes (NuGet)

### Comandos principales

```bash
dotnet package search Serilog                       # Buscar en NuGet.org
dotnet package search Serilog --exact-match         # Ver todas las versiones
dotnet package add Serilog                          # Añadir (última estable)
dotnet package add Serilog --version 4.2.0          # Versión concreta
dotnet package add Serilog@4.2.0                    # Forma abreviada (.NET 10)
dotnet package add Npgsql --prerelease              # Permitir prerelease
dotnet package add Serilog --project src/Api        # A otro proyecto
dotnet package remove Serilog                       # Quitar
dotnet package list                                 # Listar paquetes
dotnet package update                               # Actualizar todos (.NET 10)
dotnet package update Serilog                       # Actualizar uno
dotnet package update --vulnerable                  # Solo los vulnerables
```

Sintaxis antigua (.NET 8/9):

```bash
dotnet add package Serilog
dotnet remove package Serilog
dotnet list package
```

### Auditoría de dependencias

| Comando | Muestra |
|---|---|
| `dotnet package list --outdated` | Paquetes con versiones nuevas |
| `dotnet package list --outdated --highest-minor` | Solo actualizaciones menores |
| `dotnet package list --vulnerable` | Paquetes con vulnerabilidades conocidas |
| `dotnet package list --deprecated` | Paquetes obsoletos |
| `dotnet package list --include-transitive` | Incluye dependencias indirectas |
| `dotnet package list --format json` | Salida JSON (para scripts/CI) |
| `dotnet nuget why Newtonsoft.Json` | Por qué se incluye un paquete transitivo |

> Todos funcionan sobre una solución completa si se ejecutan en su carpeta.

### Restauración

```bash
dotnet restore                          # Restaurar paquetes del proyecto/solución
dotnet restore --force                  # Re-evaluar todas las dependencias
dotnet restore --no-cache               # Ignorar caché HTTP
dotnet restore --locked-mode            # Fallar si cambia packages.lock.json (CI)
```

> `build`, `run`, `test`, `publish` y `pack` hacen `restore` implícito.
> Usa `--no-restore` para saltarlo.

### Orígenes (feeds) de paquetes

```bash
dotnet nuget list source
dotnet nuget add source https://pkgs.dev.azure.com/org/_packaging/feed/nuget/v3/index.json -n MiFeed
dotnet nuget add source ~/paquetes-locales -n Local          # Carpeta local como feed
dotnet nuget add source https://nuget.pkg.github.com/USUARIO/index.json -n github \
    -u USUARIO -p TOKEN --store-password-in-clear-text
dotnet nuget disable source MiFeed
dotnet nuget enable source MiFeed
dotnet nuget remove source MiFeed
dotnet nuget config paths                                    # Ver qué nuget.config se aplican
```

### Caché local

```bash
dotnet nuget locals all --list           # Ver rutas de cachés
dotnet nuget locals all --clear          # Limpiar todas las cachés
dotnet nuget locals http-cache --clear   # Solo la caché HTTP
```

| Caché | Ruta en Linux |
|---|---|
| `global-packages` | `~/.nuget/packages/` |
| `http-cache` | `~/.local/share/NuGet/http-cache/` |
| `temp` | `/tmp/NuGetScratch<usuario>/` |
| `plugins-cache` | `~/.local/share/NuGet/plugin-cache/` |

---

## 5. Compilación y ejecución

### Flujo básico

```bash
dotnet restore     # 1. Descarga dependencias
dotnet build       # 2. Compila (incluye restore)
dotnet run         # 3. Compila y ejecuta (incluye build)
dotnet clean       # Borra salidas de compilación (bin/obj)
```

### Opciones comunes (`build`, `run`, `test`, `publish`)

| Flag | Descripción | Por defecto |
|---|---|---|
| `-c`, `--configuration` | `Debug` o `Release` | `Debug` (`Release` en `publish` desde .NET 8) |
| `-f`, `--framework` | Framework destino (multi-target) | Único definido |
| `-r`, `--runtime` | RID destino (`linux-x64`, `win-x64`…) | Plataforma actual |
| `-o`, `--output` | Carpeta de salida | `bin/<config>/<tfm>/` |
| `-v`, `--verbosity` | `q`, `m`, `n`, `d`, `diag` | `m` |
| `--no-restore` | No restaurar | — |
| `--no-build` | No compilar (`run`, `test`, `publish`) | — |
| `--no-dependencies` | No compilar referencias de proyecto | — |
| `-p:Prop=Valor` | Definir una propiedad MSBuild | — |
| `-bl` | Generar log binario `msbuild.binlog` | — |
| `--tl:off` | Desactivar el *terminal logger* | `auto` |

### Compilación

```bash
dotnet build                                 # Debug
dotnet build -c Release                      # Release
dotnet build src/Api/Api.csproj              # Un proyecto concreto
dotnet build -p:Version=1.2.3                # Pasar propiedad MSBuild
dotnet build -warnaserror                    # Tratar warnings como errores
dotnet build -bl                             # Log binario (ver con MSBuild Structured Log Viewer)
```

### Ejecución

```bash
dotnet run                                   # Proyecto de la carpeta actual
dotnet run --project src/Api                 # Otro proyecto
dotnet run -c Release
dotnet run -- --puerto 8080 -v               # Argumentos para la app tras "--"
dotnet run --launch-profile https            # Perfil de Properties/launchSettings.json
dotnet run --no-launch-profile               # Ignorar launchSettings.json
dotnet run -e ASPNETCORE_ENVIRONMENT=Staging # Variable de entorno (.NET 9+)
dotnet bin/Debug/net10.0/MiApp.dll           # Ejecutar un ensamblado ya compilado
```

### Recarga en caliente (`dotnet watch`)

```bash
dotnet watch                                 # = dotnet watch run
dotnet watch --project src/Api
dotnet watch test                            # Re-ejecutar tests al guardar
dotnet watch build                           # Solo recompilar al guardar
dotnet watch --no-hot-reload                 # Reiniciar en vez de aplicar cambios
dotnet watch -- --puerto 8080                # Argumentos para la app
```

> En la terminal de `watch`: **Ctrl+R** fuerza reinicio, **Ctrl+C** sale.

### MSBuild directo

```bash
dotnet msbuild -t:Rebuild                    # Ejecutar un target
dotnet msbuild -getProperty:TargetFramework  # Consultar una propiedad
dotnet msbuild -preprocess:salida.xml        # Ver el proyecto expandido completo
dotnet build-server shutdown                 # Parar servidores de compilación (Roslyn, MSBuild)
```

---

## 6. Testeo

### Crear un proyecto de tests

```bash
dotnet new xunit -n MiApp.Tests -o tests/MiApp.Tests   # o nunit / mstest
dotnet sln add tests/MiApp.Tests
dotnet reference add src/MiApp --project tests/MiApp.Tests
```

### Ejecutar tests

```bash
dotnet test                                  # Toda la solución/proyecto
dotnet test tests/MiApp.Tests                # Un proyecto concreto
dotnet test -c Release --no-build            # Sin recompilar (CI)
dotnet test --list-tests                     # Listar tests sin ejecutarlos
dotnet test -v n                             # Ver nombre de cada test
dotnet test --logger "console;verbosity=detailed"
dotnet test --blame-hang-timeout 2m          # Detectar tests colgados
```

### Filtrar tests (`--filter`)

| Expresión | Ejecuta |
|---|---|
| `--filter "FullyQualifiedName~Usuarios"` | Tests cuyo nombre completo contiene `Usuarios` |
| `--filter "Name=Suma_DevuelveResultado"` | Un test concreto |
| `--filter "ClassName=MiApp.Tests.CalcTests"` | Una clase |
| `--filter "Category=Integracion"` | Por categoría (`[Trait]` en xUnit) |
| `--filter "Priority!=1"` | Excluir prioridad 1 |
| `--filter "Category=Unit\|Category=Fast"` | OR lógico (`\|`) |
| `--filter "Category=Unit&Name~Login"` | AND lógico (`&`) |

Operadores: `=` exacto, `!=` distinto, `~` contiene, `!~` no contiene.

### Resultados y cobertura

```bash
# Informe TRX / JUnit
dotnet test --logger trx --results-directory ./TestResults
dotnet test --logger "junit;LogFilePath=resultados.xml"   # requiere JunitXml.TestLogger

# Cobertura de código (coverlet.collector, incluido en las plantillas)
dotnet test --collect:"XPlat Code Coverage"

# Generar informe HTML de cobertura
dotnet tool install -g dotnet-reportgenerator-globaltool
reportgenerator -reports:"**/coverage.cobertura.xml" -targetdir:coverage -reporttypes:Html
xdg-open coverage/index.html
```

### Microsoft.Testing.Platform (MTP)

Plataforma de tests moderna que sustituye a VSTest. En .NET 10 se activa vía `global.json`:

```json
{
  "test": {
    "runner": "Microsoft.Testing.Platform"
  }
}
```

```bash
dotnet test --project tests/MiApp.Tests      # Con MTP se indica --project/--solution
dotnet test --coverage                       # Cobertura con Microsoft.Testing.Extensions.CodeCoverage
```

> Los proyectos con MTP también son ejecutables: `dotnet run --project tests/MiApp.Tests`.

---

## 7. Publicación y distribución

### Modos de publicación

| Modo | Comando | Requiere runtime en destino | Tamaño |
|---|---|---|---|
| Dependiente del framework | `dotnet publish -c Release` | Sí | Muy pequeño |
| Autocontenido | `dotnet publish -r linux-x64 --self-contained` | No | Grande (~70 MB) |
| Archivo único | `... -p:PublishSingleFile=true` | Según modo | Un solo ejecutable |
| Recortado (*trimmed*) | `... -p:PublishTrimmed=true` | No | Reducido |
| ReadyToRun | `... -p:PublishReadyToRun=true` | Según modo | Mayor, arranque más rápido |
| Native AOT | `... -p:PublishAot=true` | No | Pequeño, nativo, arranque instantáneo |

### Ejemplos

```bash
# Publicación estándar (bin/Release/net10.0/publish/)
dotnet publish -c Release

# Carpeta de salida personalizada
dotnet publish -c Release -o ./dist

# Ejecutable único autocontenido para Linux
dotnet publish -c Release -r linux-x64 --self-contained \
    -p:PublishSingleFile=true -p:IncludeNativeLibrariesForSelfExtract=true

# Ejecutable para Windows desde Linux
dotnet publish -c Release -r win-x64 --self-contained -p:PublishSingleFile=true

# Native AOT (necesita clang y zlib-devel en Fedora)
dotnet publish -c Release -r linux-x64 -p:PublishAot=true

# Fijar versión del ensamblado
dotnet publish -c Release -p:Version=2.1.0
```

> Las propiedades `-p:` se pueden fijar permanentemente en el `.csproj`
> (`<PublishSingleFile>true</PublishSingleFile>`, etc.).

### Identificadores de runtime (RID) comunes

| RID | Plataforma |
|---|---|
| `linux-x64` | Linux x64 (glibc) |
| `linux-arm64` | Linux ARM64 (Raspberry Pi 64 bits, Graviton) |
| `linux-musl-x64` | Alpine Linux |
| `win-x64` | Windows x64 |
| `win-arm64` | Windows ARM |
| `osx-x64` | macOS Intel |
| `osx-arm64` | macOS Apple Silicon |

### Imágenes de contenedor (sin Dockerfile)

```bash
dotnet publish -t:PublishContainer
dotnet publish -t:PublishContainer -p:ContainerImageName=miapp -p:ContainerImageTag=1.0.0
dotnet publish -t:PublishContainer -p:ContainerRegistry=ghcr.io -p:ContainerRepository=usuario/miapp
dotnet publish -t:PublishContainer -p:ContainerArchiveOutputPath=./miapp.tar.gz   # Sin daemon
```

> Funciona con Docker o Podman (`DOTNET_CONTAINER_RUNTIME=podman` si hay ambos).

### Paquetes NuGet

```bash
dotnet pack -c Release                                   # Genera bin/Release/*.nupkg
dotnet pack -c Release -o ./nupkgs -p:PackageVersion=1.0.0
dotnet pack --include-symbols -p:SymbolPackageFormat=snupkg

# Subir a NuGet.org
dotnet nuget push nupkgs/MiLib.1.0.0.nupkg \
    --api-key $NUGET_API_KEY --source https://api.nuget.org/v3/index.json

# Subir a un feed configurado
dotnet nuget push nupkgs/*.nupkg --source MiFeed

# Retirar (unlist) una versión
dotnet nuget delete MiLib 1.0.0 --api-key $NUGET_API_KEY --source https://api.nuget.org/v3/index.json
```

Metadatos mínimos en el `.csproj`:

```xml
<PropertyGroup>
  <PackageId>MiLib</PackageId>
  <Version>1.0.0</Version>
  <Authors>santhurt</Authors>
  <Description>Descripción de la biblioteca</Description>
  <PackageLicenseExpression>MIT</PackageLicenseExpression>
  <PackageReadmeFile>README.md</PackageReadmeFile>
</PropertyGroup>
```

---

## 8. Gestión de herramientas y workloads

### Herramientas globales vs locales

| Tipo | Ámbito | Se guarda en | Se invoca con |
|---|---|---|---|
| Global (`-g`) | Usuario | `~/.dotnet/tools/` | `nombre-herramienta` |
| Local | Repositorio | `.config/dotnet-tools.json` | `dotnet nombre-herramienta` |
| Ruta (`--tool-path`) | Carpeta concreta | La carpeta indicada | `ruta/nombre-herramienta` |

> Para usar herramientas globales, añade `~/.dotnet/tools` al `PATH`:
> `echo 'export PATH="$PATH:$HOME/.dotnet/tools"' >> ~/.bashrc`

### Herramientas globales

```bash
dotnet tool install -g dotnet-ef                  # Instalar
dotnet tool install -g dotnet-ef --version 9.0.0  # Versión concreta
dotnet tool list -g                               # Listar
dotnet tool update -g dotnet-ef                   # Actualizar
dotnet tool update -g --all                       # Actualizar todas (.NET 9+)
dotnet tool uninstall -g dotnet-ef                # Desinstalar
dotnet tool search ef                             # Buscar en NuGet.org
```

### Herramientas locales (por repositorio)

```bash
dotnet new tool-manifest                   # Crea .config/dotnet-tools.json
dotnet tool install dotnet-ef              # Instala y registra en el manifiesto
dotnet tool list                           # Listar locales
dotnet tool run dotnet-ef migrations list  # Ejecutar (forma explícita)
dotnet ef migrations list                  # Ejecutar (forma corta)
dotnet tool restore                        # Instalar todo lo del manifiesto (tras clonar)
```

> Recomendado: herramientas locales para todo lo que use el proyecto,
> así cada desarrollador y el CI usan la misma versión.

### Ejecutar sin instalar (.NET 10+)

```bash
dotnet tool exec dotnetsay "Hola"          # Descarga temporal y ejecuta
dnx dotnetsay "Hola"                       # Atajo equivalente
```

### Herramientas útiles

| Paquete | Comando | Uso |
|---|---|---|
| `dotnet-ef` | `dotnet ef` | Migraciones de Entity Framework Core |
| `dotnet-outdated-tool` | `dotnet outdated` | Actualización interactiva de paquetes |
| `dotnet-reportgenerator-globaltool` | `reportgenerator` | Informes HTML de cobertura |
| `dotnet-counters` | `dotnet-counters` | Métricas en vivo (CPU, GC, peticiones) |
| `dotnet-trace` | `dotnet-trace` | Trazas de rendimiento |
| `dotnet-dump` | `dotnet-dump` | Volcados de memoria y análisis |
| `dotnet-gcdump` | `dotnet-gcdump` | Volcados del heap gestionado |
| `csharpier` | `dotnet csharpier` | Formateador de código opinado |
| `dotnet-script` | `dotnet script` | Scripts C# (`.csx`) y REPL |

### Workloads (cargas de trabajo opcionales del SDK)

```bash
dotnet workload search                     # Ver workloads disponibles
dotnet workload list                       # Ver instalados
dotnet workload install wasm-tools         # Instalar (p. ej. Blazor WASM AOT)
dotnet workload install maui-android       # MAUI para Android
dotnet workload update                     # Actualizar
dotnet workload restore                    # Instalar los que necesita el proyecto
dotnet workload repair                     # Reparar instalación
dotnet workload uninstall wasm-tools
```

> En instalaciones de sistema (`/usr/lib64/dotnet`) los workloads suelen requerir `sudo`.

---

## 9. Información y diagnóstico del entorno

### Información del SDK y runtimes

| Acción | Comando |
|---|---|
| Info completa del entorno | `dotnet --info` |
| Versión del SDK en uso | `dotnet --version` |
| SDKs instalados | `dotnet --list-sdks` |
| Runtimes instalados | `dotnet --list-runtimes` |
| Comprobar actualizaciones | `dotnet sdk check` |
| Salida de diagnóstico de cualquier comando | `dotnet -d <comando>` |
| Ayuda de un comando | `dotnet <comando> --help` |
| Abrir documentación web | `dotnet help <comando>` |

### Fijar versión del SDK (`global.json`)

```bash
dotnet new globaljson --sdk-version 9.0.100 --roll-forward latestFeature
```

```json
{
  "sdk": {
    "version": "9.0.100",
    "rollForward": "latestFeature"
  }
}
```

| `rollForward` | Comportamiento |
|---|---|
| `patch` | Solo parches del mismo *feature band* (9.0.1xx) |
| `latestFeature` | Último *feature band* de la misma versión menor |
| `latestMinor` | Última versión menor del mismo mayor |
| `latestMajor` | La versión más alta instalada |
| `disable` | Exactamente la versión indicada |

> `dotnet --version` dentro de la carpeta muestra qué SDK se resuelve.

### Variables de entorno útiles

| Variable | Efecto |
|---|---|
| `DOTNET_CLI_TELEMETRY_OPTOUT=1` | Desactivar telemetría |
| `DOTNET_NOLOGO=1` | Ocultar banner de bienvenida |
| `DOTNET_ROOT` | Ruta de instalación de .NET |
| `DOTNET_ENVIRONMENT` / `ASPNETCORE_ENVIRONMENT` | Entorno (`Development`, `Production`…) |
| `NUGET_PACKAGES` | Ruta alternativa para la caché global de paquetes |
| `DOTNET_CLI_UI_LANGUAGE` | Idioma del CLI (`es`, `en`) |
| `DOTNET_gcServer=1` | Forzar GC de servidor |

### Certificados HTTPS de desarrollo

```bash
dotnet dev-certs https                     # Crear certificado
dotnet dev-certs https --check             # Comprobar si existe
dotnet dev-certs https --clean             # Eliminar
dotnet dev-certs https --trust             # Confiar (en Linux solo para algunos navegadores/distros)
dotnet dev-certs https -ep ./cert.pfx -p clave   # Exportar
```

### Diagnóstico de aplicaciones en ejecución

```bash
dotnet tool install -g dotnet-counters dotnet-trace dotnet-dump

dotnet-counters ps                                    # Listar procesos .NET
dotnet-counters monitor -p <PID>                      # Métricas en vivo
dotnet-counters monitor -n MiApp --counters System.Runtime,Microsoft.AspNetCore.Hosting

dotnet-trace collect -p <PID> --duration 00:00:30     # Traza de 30 s (.nettrace)
dotnet-trace convert trace.nettrace --format Speedscope   # Ver en speedscope.app

dotnet-dump collect -p <PID>                          # Volcado de memoria
dotnet-dump analyze core_xxxx                         # Analizar (comandos: dumpheap -stat, clrstack, threads)
```

### Diagnóstico de compilación

```bash
dotnet build -v diag > build.log            # Log detallado de MSBuild
dotnet build -bl                            # Log binario msbuild.binlog
dotnet nuget why <Paquete>                  # Origen de una dependencia transitiva
dotnet msbuild -preprocess:proyecto.xml     # Ver proyecto con todos los imports
```

---

## 10. Solución de problemas

| Problema | Solución |
|---|---|
| `command not found: dotnet-ef` (herramienta global) | Añadir `~/.dotnet/tools` al `PATH` |
| `A compatible .NET SDK was not found` | Revisar `global.json` o instalar el SDK indicado (`dotnet --list-sdks`) |
| `Could not execute because the specified command or file was not found` | Herramienta local sin restaurar: `dotnet tool restore` |
| Errores raros tras cambiar de rama | `dotnet clean` y borrar `bin/` y `obj/`: `find . -type d \( -name bin -o -name obj \) -exec rm -rf {} +` |
| Paquetes corruptos o versión inesperada | `dotnet nuget locals all --clear && dotnet restore --force` |
| `NU1301: Unable to load the service index` | Feed inaccesible o credenciales caducadas: `dotnet nuget list source` |
| `NU1605: Detected package downgrade` | Alinear versiones (mejor con `Directory.Packages.props`) |
| `MSB3021: Unable to copy file` / archivo bloqueado | Parar la app y `dotnet build-server shutdown` |
| Certificado HTTPS no confiable | `dotnet dev-certs https --clean && dotnet dev-certs https --trust` |
| Conflicto entre SDK de Microsoft y de la distro | Usar una sola fuente (dnf **o** `dotnet-install.sh`) y comprobar `DOTNET_ROOT` |
| `dotnet watch` no detecta cambios | `DOTNET_USE_POLLING_FILE_WATCHER=1 dotnet watch` |
| Los tests no aparecen | Comprobar `Microsoft.NET.Test.Sdk` y el adaptador (`xunit.runner.visualstudio`) en el `.csproj` |

### Instalación en Fedora

```bash
sudo dnf install dotnet-sdk-10.0          # SDK (incluye runtime)
sudo dnf install dotnet-runtime-10.0      # Solo runtime
sudo dnf install aspnetcore-runtime-10.0  # Runtime de ASP.NET Core
```

Alternativa con el script oficial (instalación en `~/.dotnet`, sin root):

```bash
curl -sSL https://dot.net/v1/dotnet-install.sh -o dotnet-install.sh
chmod +x dotnet-install.sh
./dotnet-install.sh --channel 10.0
export DOTNET_ROOT="$HOME/.dotnet" PATH="$PATH:$HOME/.dotnet:$HOME/.dotnet/tools"
```
