# DIM - Digital Informationsmarkering

## Innehållsförteckning

- [Översikt för tekniker](#översikt-för-tekniker)
- [Systemkomponenter](#systemkomponenter)
- [Installation och systemkrav](#installation-och-systemkrav)
- [Konfiguration och drift](#konfiguration-och-drift)
- [Felsökning](#felsökning)
- [Avinstallation](#avinstallation)
- [Utökad teknisk dokumentation](#utökad-teknisk-dokumentation)

---

## Översikt för tekniker

DIM (Digital Informationsmarkering) är ett system för att skapa digitala säkerhetsstämplar för klassificering av dokument enligt svenska offentlighets- och sekretesslagen (OSL). Systemet är utvecklat för att möta svenska myndigheters behov av säker och standardiserad dokumentmarkering.

### Vad gör DIM?

DIM tillhandahåller en komplett lösning för att:
- **Klassificera dokument** enligt säkerhetsnivåer som "Hemlig", "Säkerhetsskyddsklassificerad (SK)" etc.
- **Skapa standardiserade stämplar** som följer juridiska krav för informationshantering
- **Märka dokument** med korrekta hänvisningar till relevanta paragrafer i OSL
- **Säkerställa spårbarhet** genom metadata i alla genererade markeringar

### Målgrupp för denna dokumentation

Denna README är **riktat till tekniker** som ansvarar för:
- ✅ Installation av DIM-systemet
- ✅ Konfiguration av tjänster och inställningar  
- ✅ Drift och underhåll av systemet
- ✅ Felsökning av tekniska problem
- ✅ Avinstallation vid behov

**Observera:** Detta dokument innehåller **inte** utvecklarorienterad information som kodflöden, API-detaljer eller programmeringsspecifikationer.

> **💡 Behöver du djupare teknisk information?**  
> För avancerad konfigurationshantering, MSI-installationsinformation och HTTP API-endpoints, se avsnittet [Utökad Teknisk Dokumentation](#utökad-teknisk-dokumentation) längre ner i detta dokument.

---

## Systemkomponenter

DIM består av två huvudkomponenter som levereras tillsammans i ett MSI-installationspaket:

### 1. DIMService (Windows-tjänst)
- **Funktion:** Backend-tjänst som hanterar stämpelgenerering
- **Typ:** Windows Service som körs automatiskt
- **Kommunikation:** HTTP API på konfigurerad port (standard: 5001)
- **Användarkonto:** NT AUTHORITY\NetworkService
- **Placering:** `C:\Program Files\DIM\DIMService\`

### 2. DIM (Klientapplikation)
- **Funktion:** Grafiskt användargränssnitt för manuell stämpelgenerering
- **Typ:** Windows Forms-applikation  
- **Kommunikation:** Ansluter till DIMService via HTTP
- **Användare:** Alla som behöver skapa dokumentstämplar
- **Placering:** `C:\Program Files\DIM\DIM\`

### Systemarkitektur
```
[DIM Klient] ←→ HTTP (port 5001) ←→ [DIMService] → [Stämpelfiler]
     ↓                                      ↓
[Användargränssnitt]                [Konfigurationsfiler]
```

---

## Installation och systemkrav

### Systemkrav
- **Operativsystem:** Windows 10 eller Windows 11 (x64)
- **Behörigheter:** Administratörsbehörighet för installation
- **Nätverk:** Port 5001 TCP måste vara tillgänglig lokalt
- **Diskutrymme:** Minst 200 MB ledigt utrymme

### Leveransformat
DIM levereras som en MSI-fil med namnet: `DIM_[version].msi`

Exempel: `DIM_1_0_0_354.msi`

### Installation
1. Högerklicka på MSI-filen och välj "Kör som administratör"
2. Följ installationsguiden
3. Både DIMService och DIM-klient installeras automatiskt
4. DIMService-tjänsten startas automatiskt
5. Genväg för DIM-klient skapas på skrivbordet

---

## Konfiguration och drift

### Konfigurationsfiler
Alla konfigurationsfiler placeras i: `C:\ProgramData\DIM\Config\`

- **`config DIMService.json`** - Tjänstens huvudkonfiguration
- **`config DIM.json`** - Klientens anslutningsinställningar

### Arbetskataloger
- **`C:\ProgramData\DIM\Logs\`** - Felloggar och händelseloggar
- **`C:\ProgramData\DIM\Temp\`** - Temporära filer för dokumentvisning

### Tjänsthantering
DIMService kan hanteras via Windows Tjänster:
- **Tjänstnamn:** DIMService
- **Starttyp:** Automatisk
- **Inloggning som:** NT AUTHORITY\NetworkService

### Kontroll av systemstatus
- **Tjänststatus:** Via Windows Tjänster eller `services.msc`
- **HTTP-tillgänglighet:** Öppna webbläsare och gå till `http://localhost:5001/isrunning`
- **Klientanslutning:** Starta DIM-applikationen - den visar anslutningsstatus

---

## Felsökning

### Vanliga problem

#### DIMService startar inte
1. Kontrollera Windows Event Log för felmeddelanden
2. Verifiera att port 5001 inte används av annan tjänst
3. Kontrollera behörigheter för NetworkService-kontot

#### DIM-klient kan inte ansluta
1. Verifiera att DIMService-tjänsten körs
2. Kontrollera brandväggsinställningar för port 5001
3. Validera konfigurationen i `config DIM.json`

#### Konfigurationsfel
1. Kontrollera JSON-syntax i konfigurationsfilerna
2. Jämför med standardkonfigurationer
3. Starta om DIMService efter konfigurationsändringar

### Loggfiler
- **Windows Event Log:** Systemfel och viktiga händelser
- **`C:\ProgramData\DIM\Logs\`** - Detaljerade felloggar från DIMService

---

## Avinstallation

### Via kontrollpanelen
1. Öppna "Program och funktioner" i Kontrollpanelen
2. Hitta "DIM" i listan
3. Högerklicka och välj "Avinstallera"
4. Följ avinstallationsguiden

### Via MSI (administratör)
```cmd
msiexec /x DIM_1_0_0_354.msi /qn
```

### Manuell rensning (vid behov)
Efter avinstallation kan följande kataloger rensas manuellt:
- `C:\ProgramData\DIM\` (konfiguration och loggar)
- Genvägar på skrivbord och i startmeny

---

## Utökad teknisk dokumentation

För djupgående information om specifika tekniska aspekter, se följande dokument:

### 📋 Konfigurationshantering
**Dokument:** [`docs/DIM_Konfigurationsfiler_Tekniker.md`](docs/DIM_Konfigurationsfiler_Tekniker.md)

**Innehåll:**
- Detaljerad genomgång av alla konfigurationsparametrar
- JSON-strukturer och exempel
- Hot-reload funktionalitet
- Säkerhetsklassificeringar och paragrafkonfiguration
- HTTP API-access till konfigurationsdata
- Valideringsregler och felsökning av konfiguration

**Använd detta dokument när du behöver:**
- Anpassa systemet för specifika organisationsbehov
- Konfigurera säkerhetsklassificeringar
- Felsöka konfigurationsrelaterade problem
- Förstå hur systemets inställningar fungerar

### 🚀 MSI-installation och leverans
**Dokument:** [`docs/DIM_MSI_Leveransdokumentation_Tekniker.md`](docs/DIM_MSI_Leveransdokumentation_Tekniker.md)

**Innehåll:**
- Kommandoradsinstallation med msiexec
- Silent installation för massutrollning
- Versionhantering och uppgraderingsprocesser
- Systemintegration och behörigheter
- Deployment-strategier för företagsmiljöer
- Avancerad felsökning av installationsproblem

**Använd detta dokument när du behöver:**
- Utföra automatiserad installation i större miljöer
- Planera systemutrollning
- Hantera versionsuppgraderingar
- Lösa komplexa installationsproblem
- Förstå MSI-paketets innehåll och beteende

### 🌐 HTTP API-endpoints
**Dokument:** [`docs/DIM_HTTP_Endpoints_Tekniker.md`](docs/DIM_HTTP_Endpoints_Tekniker.md)

**Innehåll:**
- Komplett guide till alla tillgängliga HTTP-endpoints
- Systemövervakning och diagnostik via API
- Automatiserade hälsokontroller och övervakningsskript
- Konfigurationskontroll och valideringstestning
- Stämpelgenerering via HTTP för integrationstester
- Felsökning av nätverks- och API-problem
- Säkerhetsaspekter och bästa praxis för API-användning

**Använd detta dokument när du behöver:**
- Övervaka systemhälsa och upptäcka problem proaktivt
- Felsöka HTTP-kommunikation mellan klient och tjänst
- Automatisera systemkontroller och drift
- Integrera DIM med andra system
- Verifiera konfiguration och funktionalitet programmatiskt
- Skapa egna övervaknings- och diagnostikverktyg

---

## Support och vidare läsning

### Utökad teknisk dokumentation:
För djupgående teknisk information, se dokumenten listade i avsnittet [Utökad teknisk dokumentation](#utökad-teknisk-dokumentation) ovan:
- **Konfigurationshantering** - Detaljerad JSON-konfiguration och API-validering
- **MSI-installation och leverans** - Avancerad installation och deployment
- **HTTP API-endpoints** - Systemövervakning och diagnostik via API

### Support:
- **Utvecklare:** Dynalab AB    www.dynalab.se
- **Systemdiagnostik:** Använd `http://localhost:5001/isrunning` för snabb statuskontroll

**Vid supportärenden, inkludera:**
- Operativsystemversion (Windows 10/11)
- DIM-version (från MSI-filnamn eller `/config` endpoint)
- Beskrivning av problemet och steg för att återskapa
- Relevanta loggfiler från `C:\ProgramData\DIM\Logs\`
- Resultat från `http://localhost:5001/fellogg` (om DIMService körs)
