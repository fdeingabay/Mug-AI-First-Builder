# KB Restaurador

## Propósito
Herramienta de escritorio offline para que un restaurador cargue su criterio de identificación de procesos fotográficos (típico / posible / excluyente sobre 11 facetas), partiendo de datos del Graphics Atlas de IPI.
Mide con un conjunto de control de obras ya dictaminadas si el esquema reproduce sus dictámenes. Spec completa: `PRD-002.md`.

## Stack
- Windows 11. Se entrega como `.exe` portable: sin instalador ni permisos de administrador.
- .NET 10 (LTS) · C# · WPF · xUnit.
- SQLite vía `Microsoft.Data.Sqlite` (trae su propio SQLite, no depende del sistema). Modo WAL. Una transacción por cada afirmación guardada (RNF-05, AC-05).
- La carpeta de datos junto al `.exe` guarda la base SQLite y los recursos crudos de cada instantánea con su hash (RF-04). Se ubica con `AppContext.BaseDirectory`; no usar `Assembly.Location`, que en single-file viene vacío.
- Núcleo: `HttpClient` (HTTP), AngleSharp (HTML), `System.Text.Json` (JSON), `SHA256` (hash de recursos).
- Layout:
  - `src/KbRestaurador.Core/`: biblioteca con ingesta, parseo, base y regla de RF-14. No referencia WPF.
  - `src/KbRestaurador.App/`: interfaz WPF.
  - `tests/KbRestaurador.Core.Tests/`: xUnit. Fixtures en `tests/KbRestaurador.Core.Tests/Fixtures/`.

## Cómo correr
Desarrollar en Windows nativo (PowerShell), no en WSL: WPF solo corre en Windows.
Prerrequisitos: .NET 10 SDK (Visual Studio o Rider opcionales).

```powershell
dotnet restore                                      # instalar dependencias
dotnet run --project src/KbRestaurador.App          # levantar en desarrollo
dotnet test                                         # tests (sin red)
dotnet publish src/KbRestaurador.App -c Release -r win-x64 --self-contained -p:PublishSingleFile=true -o publish/
                                                    # .exe portable en publish/ (con las DLL nativas al lado)
```

## Qué NO hacer
- No marcar nada como **excluyente** a partir de los datos del Atlas, aunque ningún espécimen tenga ese valor. Lo excluyente lo carga solo el restaurador; un valor no observado es **sin datos** (RF-09, R-03, R-06).
- No normalizar valores en silencio (mayúsculas, espacios, typos). El parseo no pierde ni inventa valores, y las variantes las arbitra el restaurador (RF-05, RF-06).
- No guardar datos fuera de la carpeta junto al `.exe` (ni `%APPDATA%`/`Environment.SpecialFolder`, ni el registro). Respaldar debe ser copiar esa carpeta (RNF-04).
- No agregar features fuera del PRD (`PRD-002.md`). Lo que no está en sus RF/RNF, o figura en "Fuera de Alcance", no se implementa.
