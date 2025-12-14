# CI Pipeline

## Introduktion

I demonstrationen visades en CI-pipeline med Node.js och JavaScript. Detta dokument visar hur samma koncept och verktyg översätts till .NET. Principerna är identiska, det är verktygen som skiljer sig.

---

## Verktygsmappning: JavaScript → .NET

| JavaScript/Node.js | .NET Motsvarighet | Beskrivning |
|---|---|---|
| npm install | dotnet restore | Återställer NuGet-paket från .csproj |
| npm run build | dotnet build | Kompilerar projektet |
| npm test (Jest) | dotnet test (xUnit/NUnit) | Kör enhetstester |
| ESLint | dotnet format | Kontrollerar kodstil och formatering |
| npm audit | dotnet list package --vulnerable | Skannar efter sårbarheter i beroenden |
| Istanbul/Jest coverage | Coverlet + ReportGenerator | Genererar täckningsrapporter |
| package.json | .csproj / .sln | Projektfil med beroenden och konfiguration |

---

## Testramverk i .NET

I .NET finns tre huvudsakliga testramverk att välja mellan:

1. **xUnit** – Det modernaste och mest populära valet. Används av Microsoft för .NET Core själv.
2. **NUnit** – Etablerat ramverk med lång historik, porterat från Java's JUnit.
3. **MSTest** – Microsofts eget ramverk, djupt integrerat med Visual Studio.

**Rekommendation:** Börja med xUnit om ni inte har specifika krav.

---

## Pipeline-steg för .NET

### Steg 1: Setup och Restore

Motsvarar: `npm install`

1. `actions/checkout@v4` – Hämtar koden
2. `actions/setup-dotnet@v4` – Installerar .NET SDK
3. `dotnet restore` – Återställer NuGet-paket

### Steg 2: Kodstilskontroll

Motsvarar: `npm run lint (ESLint)`

```
dotnet format --verify-no-changes
```

Detta kommando kontrollerar att koden följer de kodstilsregler som definierats i `.editorconfig`.

### Steg 3: Kompilering

Motsvarar: `npm run build`

```
dotnet build --configuration Release --no-restore
```

Flaggan `--no-restore` hoppar över restore eftersom det redan gjorts.

### Steg 4: Enhetstester

Motsvarar: `npm test (Jest)`

```
dotnet test --no-build --configuration Release
```

Kör alla tester i solution. Testprojekt identifieras automatiskt via `<IsTestProject>true</IsTestProject>` i .csproj.

### Steg 5: Kodtäckning

Motsvarar: Jest coverage threshold

```
dotnet test --collect:"XPlat Code Coverage"
```

Använder Coverlet för att samla in täckningsdata. För att sätta en tröskel, lägg till i .csproj: `<Threshold>80</Threshold>`

### Steg 6: Säkerhetsskanning

Motsvarar: `npm audit`

```
dotnet list package --vulnerable --include-transitive
```

---

## Komplett GitHub Actions Workflow

Skapa filen som `.github/workflows/ci.yml` i ert repository.

```yaml
name: .NET CI Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - name: Setup .NET
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: 8.0.x
    - name: Restore dependencies
      run: dotnet restore
    - name: Check code formatting
      run: dotnet format --verify-no-changes
    - name: Build
      run: dotnet build --configuration Release --no-restore
    - name: Run tests with coverage
      run: dotnet test --no-build --configuration Release --collect:"XPlat Code Coverage"
    - name: Security scan
      run: dotnet list package --vulnerable --include-transitive
```

---

## Användbara kommandon lokalt

1. `dotnet new xunit -n MyProject.Tests` – Skapa nytt testprojekt
2. `dotnet add package coverlet.collector` – Lägg till coverage
3. `dotnet test --collect:"XPlat Code Coverage"` – Kör tester med täckning
4. `dotnet format` – Autoformatera koden
5. `dotnet build --warnaserror` – Bygg och behandla varningar som fel

---

## Tips för .NET-projekt

- **Skapa en .editorconfig-fil** i projektets rot för att definiera kodstilsregler som hela teamet följer.
- **Använd Directory.Build.props** för att centralisera konfiguration som gäller alla projekt i en solution.
- **Aktivera Nullable Reference Types** för att fånga null-relaterade buggar vid kompilering.
- **Sätt TreatWarningsAsErrors** till true i produktionsprojekt för striktare kodkvalitet.
- **Använd Roslyn Analyzers** för mer avancerad statisk kodanalys utöver basic formatting.

---

## Resurser

- Microsoft Learn: .NET Testing – docs.microsoft.com/dotnet/core/testing
- xUnit dokumentation – xunit.net
- Coverlet (kodtäckning) – github.com/coverlet-coverage/coverlet
- GitHub Actions for .NET – github.com/actions/setup-dotnet

