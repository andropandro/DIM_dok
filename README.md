# DIMService - Digital Stämpeltjänst

## Översikt

DIMService är en Windows Service-applikation som tillhandahåller HTTP-baserade API:er för att skapa digitala säkerhetsstämplar för dokumentklassificering. Tjänsten är specialiserad på att generera stämplar enligt svenska offentlighets- och sekretesslagen (OSL) och stödjer olika sekretessklassificeringar och säkerhetsskyddsklasser.

## Arkitektur

### Huvudkomponenter

```
DIMService/
├── Program.cs              - Applikationens startpunkt och värdkonfiguration
├── Worker.cs               - Bakgrundstjänstlogik och infrastrukturhantering
├── HttpServer.cs           - HTTP-server och API-endpoints
├── KonfigData.cs           - Konfigurationsdatastrukturer
├── KonfigHanterare.cs      - Konfigurationshantering
├── Repository.cs           - Globala konstanter och sökvägar
├── StampelInfo.cs          - Stämpeldatastruktur
├── StampelRetur.cs         - Stämpelreturstruktur
├── ErrorLog.cs             - Loggningshantering
└── Json/
    └── config DIMService.json - Standardkonfiguration
```

### Teknisk Stack

- **Framework**: .NET 8.0 (Windows-specifik)
- **Arkitektur**: Windows Service med HTTP-listener
- **Bildhantering**: System.Drawing.Common, SVG.NET
- **Hosting**: Microsoft.Extensions.Hosting.WindowsServices
- **Konfiguration**: JSON-baserad med hot-reload-stöd
- **Loggning**: Microsoft.Extensions.Logging med EventLog-integration

## Komponentbeskrivning

### Program.cs
**Syfte**: Applikationens huvudklass och konfiguration av värdtjänsten.

**Funktionalitet**:
- Konfigurerar IHostBuilder för Windows Service-integration
- Ställer in loggning (EventLog och Console)
- Registrerar Worker som hosted service
- Hanterar dependency injection-containern

### Worker.cs
**Syfte**: Huvudarbetarklass som hanterar tjänstens livscykel.

**Funktionalitet**:
- Initierar HTTP-servern vid uppstart
- Hanterar konfigurationsladdning
- Skapar och underhåller filsystemsstruktur
- Rensar temporära filer vid uppstart
- Migrerar konfigurationsfiler till rätt kataloger

**Filsystemsstruktur**:
```
%ALLUSERSPROFILE%/DIM/
├── Config/         - Konfigurationsfiler
├── Temp/          - Temporära stämpelfiler
└── Fellogg/       - Felloggar
```

### HttpServer.cs
**Syfte**: HTTP-server som exponerar REST API:er för stämpelgenerering.

**Teknisk implementation**:
- Använder HttpListener för HTTP-hantering
- CORS-stöd för webbapplikationsintegration
- Endast GET-requests stöds
- Asynkron hantering av inkommande requests
- Comprehensive felhantering och loggning

**Stämpelgenereringsengine**:
- Stödjer tre bildformat: EMF (Enhanced Metafile), PNG, SVG
- Vektorbaserad rendering med Graphics2D
- Skalbar arkitektur för flera samtidiga stämplar
- Automatisk storleksanpassning baserat på innehåll
- Metadata-inbäddning enligt olika standarder

### KonfigData.cs
**Syfte**: Definierar datastrukturer för konfigurationshantering.

**Klasser**:
- `KonfigData`: Huvudkonfiguration (port, format, sekretess, standardvärden)
- `SekretessData`: Sekretessklassificeringar med paragrafer och SSK
- `KonfigRad`: Grundläggande kod-text-par
- `Defaultvarde`: Standardvärden med numeriska värden

### StampelInfo.cs & StampelRetur.cs
**Syfte**: Datastrukturer för stämpelhantering.

**StampelInfo innehåller**:
- Sekretessnivå och rubrik
- Paragraftext enligt OSL
- Datum och verksamhetsinformation
- Arbetshandlingsflaggor

**StampelRetur innehåller**:
- Genererade bilddata (MemoryStream)
- Metafile-objekt för vektordata
- SVG-dokument för skalbar vektorgrafik
- Filsökvägar för temporära filer

## API-Dokumentation

### Övergripande API-beteende
- **Bas-URL**: `http://localhost:[konfigurerad port]/`
- **HTTP-metod**: Endast GET stöds
- **CORS**: Aktiverat för alla domäner
- **Svarformat**: JSON eller plaintext beroende på endpoint

### Administrativa Endpoints

#### `/isrunning`
**Syfte**: Hälsokontroll för tjänsten
**Svar**: `"true"` som plaintext

#### `/config`
**Syfte**: Returnerar aktuell konfiguration
**Svar**: JSON-objekt med komplett konfiguration

### Informationsendpoints

#### `/fellogg` / `/felloggtext`
**Syfte**: Hämtar senaste felloggar
**Svar**: JSON-array eller formaterad text med felloggar

#### `/format` / `/formattext`
**Syfte**: Listar tillgängliga bildformat
**Svar**: JSON-array eller text med formatinformation

#### `/defaultvarden` / `/defaultvardentext`
**Syfte**: Returnerar systemets standardvärden
**Svar**: JSON-array eller text med standardkonfiguration

#### `/paragrafer` / `/paragrafertext`
**Parametrar**: `sekretess` (obligatorisk)
**Syfte**: Hämtar paragrafer för angiven sekretessnivå
**Svar**: JSON-array eller text med paragrafdata

#### `/ssk` / `/ssktext`
**Parametrar**: `sekretess` (standard: "Hemlig")
**Syfte**: Hämtar säkerhetsskyddsklasser för sekretessnivå
**Svar**: JSON-array eller text med SSK-data

### Stämpelgenereringsendpoints

#### `/stampel` / `/stampelurl`
**Syfte**: Genererar kompletta säkerhetsstämplar

**Obligatoriska parametrar per sekretessnivå**:
- `sekretess`: Sekretesskod (t.ex. "Hemlig", "SK")
- `paragrafer`: Kommaseparerade paragrafkoder

**Obligatoriska för SK-sekretess**:
- `ssk`: Säkerhetsskyddsklassifikation

**Valfria parametrar**:
- `datum`: Datum (standard: dagens datum)
- `arbetshandling`: Flagga för arbetshandling
- `bredd`: Stämpelbredd i pixlar
- `hojd`: Stämpelhöjd i pixlar
- `orientering`: "h" (horisontell) eller "v" (vertikal)
- `vhtnamn`: Verksamhetsnamn
- `format`: Bildformat ("emf", "png", "svg")

**Returvärden**:
- `/stampel`: Binär bilddata
- `/stampelurl`: Filsökväg till sparad fil

#### `/stampelhanvisning` / `/stampelhanvisningurl`
**Syfte**: Genererar hänvisningsstämplar för flersidiga dokument

**Parametrar**:
- `ssk`: Säkerhetsskyddsklassifikation (obligatorisk)
- `bredd`, `hojd`, `format`: Som för `/stampel`

## Stämpelgenereringsprocess

### Teknisk Implementation

#### Vektorbaserad Rendering (EMF/SVG)
1. **EMF-generering**: Använder Windows GDI+ Metafile-API
2. **SVG-generering**: Använder SVG.NET-biblioteket
3. **Gemensam renderingspipeline**: Textlayout, färghantering, ramskapande

#### Rasterbild-rendering (PNG)
1. Genererar först EMF-vektorbild
2. Renderar till Bitmap med anti-aliasing
3. Exporterar som PNG med optimal komprimering

#### Metadata-hantering
Alla stämplar får automatisk metadata enligt följande standarder:

**PNG-metadata** (iTXt-chunks):
- `data-creator`: Applikationsnamn
- `data-title`: Stämpelrubrik
- `data-description`: Kombinerad beskrivning
- `data-created`: Skapandetidpunkt
- `data-query`: Original-querysträng

**EMF-metadata** (Comment Records + Property Items):
- XML-formaterad metadata i EMF-kommentarer
- XMP-metadata för verktygskompatibilitet
- Property Items för Windows-integration

**SVG-metadata** (Standard SVG-element):
- Dublin Core-metadata
- Custom namespace för DIMService-specifik data
- W3C-kompatibel metadatastruktur

### Stämpeltyper

#### HEMLIG-stämplar
- Rubrik med horisontellt streck
- Paragraftext under strecket
- Datum och verksamhetsinformation
- Röd färg (#ED1C24)

#### SK-stämplar (Säkerhetsskyddad)
- Paragraftext överst
- Sekretessklassificering
- Samma färgschema och layout

#### Hänvisningsstämplar
- Kompakt format för flersidiga dokument
- SSK-klassificering
- "Se sid 1" hänvisning

## Konfigurationshantering

### Konfigurationsfil (JSON)
```json
{
  "Lyssnarport": 8080,
  "Format": [...],
  "Sekretess": [...],
  "Defaultvarden": [...]
}
```

### Hot-reload-stöd
- Konfigurationsändringar detekteras automatiskt
- Ingen omstart av tjänsten krävs
- Felhantering vid ogiltiga konfigurationer

### Standardvärden
- Bredd: 302 pixlar
- Höjd: 182 pixlar (stämpel) / 66 pixlar (hänvisning)
- Format: EMF
- Orientering: Horisontell

## Säkerhet och Compliance

### Säkerhetsaspekter
- Endast lokala anslutningar (localhost)
- Ingen autentisering (förutsätter säker nätverksmiljö)
- Minimal attack surface (endast GET-requests)
- Validering av alla inparametrar

### Compliance
- Följer svenska offentlighets- och sekretesslagen (OSL)
- Stöder standardiserade sekretessklassificeringar
- Metadata-spårbarhet för dokument

## Felsökning och Underhåll

### Loggning
- **EventLog**: Systemfel och viktiga händelser
- **Console**: Utvecklingsloggning
- **Fellogg-filer**: Detaljerad JSON-baserad felspårning

### Vanliga Problem
1. **Port redan upptagen**: Ändra `Lyssnarport` i konfiguration
2. **Git-versionering misslyckas**: Kontrollera Git-tillgänglighet i PATH
3. **Stämpelgenerering misslyckas**: Kontrollera diskutrymme i temp-katalog

### Prestanda
- **Minneshantering**: Automatisk cleanup av temporära filer
- **Samtidiga requests**: Asynkron hantering utan blocking
- **Bildoptimering**: Skalbar vektorrendering innan rasterkonvertering

## Byggprocess och Deployment

### Versionering
- Automatisk versionshantering baserat på Git-commits
- Format: `Major.Minor.CommitCount`
- MSBuild-integration för CI/CD

### Build-konfiguration
```xml
<Target Name="GetGitCommitCount" BeforeTargets="GetAssemblyVersion">
  <Exec Command="git rev-list --count HEAD" ConsoleToMSBuild="true" IgnoreExitCode="false">
    <Output TaskParameter="ConsoleOutput" PropertyName="GitCommitCount" />
  </Exec>
  <PropertyGroup>
    <Version>$(VersionPrefix).$(GitCommitCount)</Version>
  </PropertyGroup>
</Target>
```

### Systemkrav
- Windows 10/11 eller Windows Server 2019+
- .NET 8.0 Runtime
- Git (för automatisk versionering)
- Administratörsbehörighet för Windows Service-installation

## Tekniska Begränsningar

### Plattformsbegränsningar
- **Windows-specifik**: System.Drawing.Common kräver Windows
- **GDI+-beroende**: EMF-generering kräver Windows GDI+
- **Service-integration**: Windows Service-specifik hosting

### Prestandabegränsningar
- **Minnesförbrukning**: Stämpelgenerering håller bilddata i minnet
- **Samtidiga användare**: Begränsas av HttpListener-prestanda
- **Filsystemsberoende**: Temporära filer kräver diskutrymme

### Funktionella Begränsningar
- **Endast GET-requests**: Ingen POST/PUT-support
- **Lokal binding**: Endast localhost-anslutningar
- **Statisk konfiguration**: Viss konfiguration kräver omstart

## Framtida Utveckling

### Föreslagna Förbättringar
1. **REST API-expansion**: Stöd för POST-requests och JSON-payloads
2. **Autentisering**: JWT eller API-key-baserad säkerhet
3. **Dockerisering**: Containeriserad deployment
4. **Databas-integration**: Persistering av stämpeldata
5. **Batch-processing**: Samtidig generering av flera stämplar

### Teknisk Skuld
- Beroende av System.Drawing.Common (legacy)
- Hårdkodade färgvärden
- Begränsad felhantering för nätverksfel
- Saknad integrationstest-täckning 
