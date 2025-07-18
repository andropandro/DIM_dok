# DIM - HTTP Endpoints Teknikerguide
## Digital Informationsmarkering - API-endpoints för tekniker

![Platform](https://img.shields.io/badge/platform-Windows%2010%20%7C%20Windows%2011-blue)
![HTTP API](https://img.shields.io/badge/HTTP%20API-REST--like-green)
![Port](https://img.shields.io/badge/port-5001%20%28default%29-orange)
![Method](https://img.shields.io/badge/method-GET%20only-yellow)
![Response](https://img.shields.io/badge/response-JSON%20%7C%20Text-purple)
![Developer](https://img.shields.io/badge/developer-Dynalab%20AB-lightblue)

### Dokumentversion
- **Version:** 1.0
- **Datum:** 2025-01-02
- **Målgrupp:** Tekniker och systemadministratörer
- **Syfte:** HTTP API-endpoints för drift, övervakning och felsökning

## Innehållsförteckning

- [1. Översikt](#1-översikt)
- [2. Systemstatus och övervakning](#2-systemstatus-och-övervakning)
- [3. Konfigurationsinformation](#3-konfigurationsinformation)
- [4. Stämpelgenerering](#4-stämpelgenerering)
- [5. Felsökning och diagnostik](#5-felsökning-och-diagnostik)
- [6. Säkerhet och bästa praxis](#6-säkerhet-och-bästa-praxis)
- [7. Support och vidare läsning](#7-support-och-vidare-läsning)

---

## 1. Översikt

**För dig som tekniker:** Detta dokument beskriver alla tillgängliga HTTP-endpoints i DIMService som du kan använda för systemövervakning, felsökning, konfigurationskontroll och problemdiagnostik. Dessa endpoints är dina viktigaste verktyg för att förstå och underhålla DIM-systemet.

### 1.1 Vad är DIM HTTP API?

**Systemets HTTP-gränssnitt:**
DIMService exponerar ett HTTP API på port 5001 (konfigurerbart) som tillhandahåller access till systemets interna status, konfiguration och funktionalitet. Detta API är **inte** avsett för slutanvändare utan för **teknisk diagnostik och systemintegration**.

#### Varför är HTTP API:et viktigt för tekniker?

**🔍 Systemövervakning och diagnostik:**
- **Realtidsstatus** - Kontrollera om tjänsten svarar
- **Felloggar** - Hämta aktuella felmeddelanden direkt från systemet
- **Konfigurationskontroll** - Verifiera att inställningar laddats korrekt
- **Funktionalitettest** - Testa stämpelgenerering utan GUI

**🛠️ Praktiska fördelar för drift:**
- **Skriptbar övervakning** - Automatisera hälsokontroller
- **Fjärrdiagnostik** - Felsök från annan maskin (med rätt nätverksinställningar)
- **Integrationstestning** - Verifiera att systemet fungerar som förväntat
- **Snabb problemlösning** - Direkt access till system-status utan GUI

**⚠️ Säkerhetsaspekter att förstå:**
- **Endast GET-requests** tillåts - inga systemändringar via API
- **Localhost-only** som standard - extern access kräver konfigurationsändringar
- **Ingen autentisering** - förlita dig på nätverkssegmentering för säkerhet
- **Känslig information** - vissa endpoints kan visa konfigurationsdata

### 1.2 Bas-URL och åtkomst

**Standard HTTP-adress:**
```
http://localhost:5001/
```

**Viktiga tekniska detaljer:**
- **Port:** Konfigurerbar via `Lyssnarport` i `config DIMService.json`
- **Protokoll:** HTTP endast (inte HTTPS)
- **Metoder:** Endast GET-requests stöds
- **Encoding:** UTF-8 för alla svar
- **CORS:** Aktiverat för cross-origin requests

#### Hur du testar åtkomst:

**🌐 Via webbläsare:**
```
http://localhost:5001/isrunning
```
Svar: `true` (om tjänsten körs)

**💻 Via PowerShell:**
```powershell
Invoke-RestMethod -Uri "http://localhost:5001/isrunning"
```

**🔧 Via curl (om installerat):**
```cmd
curl http://localhost:5001/isrunning
```

### 1.3 Endpoint-kategorier

**För strukturerad förståelse** är endpoints indelade i tre huvudkategorier:

#### 🚦 Systemstatus och övervakning
- Kontrollera tjänstestatus
- Hämta felloggar för diagnostik
- Övervaka systemhälsa

#### ⚙️ Konfigurationsinformation  
- Verifiera laddad konfiguration
- Kontrollera standardvärden
- Validera säkerhetsklassificeringar

#### 🏷️ Stämpelgenerering
- Testa stämpelfunktionalitet
- Generera test-stämplar
- Validera bildformat

---

## 2. Systemstatus och övervakning

**Kritiska endpoints för daglig drift:** Dessa endpoints hjälper dig övervaka systemets hälsa och identifiera problem snabbt.

### 2.1 `/isrunning` - Tjänstestatus

**Syfte:** Enklaste sättet att kontrollera om DIMService körs och svarar på HTTP-requests.

**URL:** `http://localhost:5001/isrunning`
**Metod:** GET
**Parametrar:** Inga

#### Svar:
```
true
```

#### Användning för tekniker:

**✅ Framgångsrik användning:**
- **Övervakningsskript** - Automatisk kontroll av tjänstestatus
- **Nätverksdiagnostik** - Verifiera att port 5001 är tillgänglig
- **Starttester** - Kontrollera att tjänsten svarat efter omstart
- **Load balancer health checks** - För miljöer med flera instanser

**🔧 Praktiska exempel:**

**PowerShell-övervakningsskript:**
```powershell
# Enkel hälsokontroll
try {
    $response = Invoke-RestMethod -Uri "http://localhost:5001/isrunning" -TimeoutSec 10
    if ($response -eq "true") {
        Write-Host "DIMService: OK" -ForegroundColor Green
    } else {
        Write-Host "DIMService: Oväntat svar - $response" -ForegroundColor Yellow
    }
} catch {
    Write-Host "DIMService: OFFLINE - $($_.Exception.Message)" -ForegroundColor Red
}
```

**Batch-fil för snabbkontroll:**
```cmd
@echo off
curl -s http://localhost:5001/isrunning
if %ERRORLEVEL% EQU 0 (
    echo DIMService: Tillgänglig
) else (
    echo DIMService: Ej tillgänglig - kontrollera tjänst
)
```

### 2.2 `/fellogg` och `/felloggtext` - Felloggar

**Syfte:** Hämta de senaste felloggarna direkt från systemet för snabb problemdiagnostik.

**URL:** 
- `http://localhost:5001/fellogg` (JSON-format)
- `http://localhost:5001/felloggtext` (läsbar text)

**Metod:** GET
**Parametrar:** Inga

#### Svar för `/fellogg` (JSON):
```json
[
  {
    "Tidpunkt": "2024-12-19T10:30:00",
    "ExceptionTyp": "FileNotFoundException", 
    "Meddelande": "Kunde inte hitta config-fil: C:\\ProgramData\\DIM\\Config\\config DIMService.json",
    "StackTrace": "..."
  }
]
```

#### Svar för `/felloggtext` (läsbar):
```
Tidpunkt: 2024-12-19 10:30:00
ExceptionTyp: FileNotFoundException
Meddelande: Kunde inte hitta config-fil: C:\ProgramData\DIM\Config\config DIMService.json
StackTrace: ...
--------------------------------------------------
```

#### Användning för tekniker:

**🔍 Snabb problemdiagnostik:**
- **Identifiera orsak** till tjänsteproblem utan att gå till loggfiler
- **Realtidsövervakning** av fel som uppstår
- **Fjärrdiagnostik** utan att logga in på servern
- **Trendanalys** av återkommande problem

**⚠️ Vad loggas:**
- **Konfigurationsfel** (felaktig JSON, saknade filer)
- **HTTP-fel** (requests som misslyckas)
- **Stämpelgenereringsfel** (bildgenereringsfel)
- **Systemfel** (filsystemsproblem, minnesbrist)

**🔧 Praktiska exempel:**

**PowerShell för felanalys:**
```powershell
# Hämta och analysera senaste fel
$errors = Invoke-RestMethod -Uri "http://localhost:5001/fellogg"
if ($errors.Count -gt 0) {
    Write-Host "Senaste fel:" -ForegroundColor Yellow
    $latest = $errors | Sort-Object Tidpunkt -Descending | Select-Object -First 1
    Write-Host "Typ: $($latest.ExceptionTyp)" -ForegroundColor Red
    Write-Host "Meddelande: $($latest.Meddelande)" -ForegroundColor Red
    Write-Host "Tid: $($latest.Tidpunkt)" -ForegroundColor Red
} else {
    Write-Host "Inga fel registrerade" -ForegroundColor Green
}
```

**Kontinuerlig övervakning:**
```powershell
# Övervaka fel var 30:e sekund
while ($true) {
    $errors = Invoke-RestMethod -Uri "http://localhost:5001/fellogg"
    $count = $errors.Count
    Write-Host "$(Get-Date): $count fel i loggen" -ForegroundColor $(if($count -eq 0){"Green"}else{"Yellow"})
    Start-Sleep 30
}
```

---

## 3. Konfigurationsinformation

**Verifiera systemkonfiguration:** Dessa endpoints hjälper dig kontrollera att konfigurationen laddats korrekt och förstå systemets aktuella inställningar.

### 3.1 `/config` - Komplett konfiguration

**Syfte:** Hämta hela den aktuella konfigurationen som systemet använder.

**URL:** `http://localhost:5001/config`
**Metod:** GET
**Parametrar:** Inga

#### Svar (förkortat exempel):
```json
{
  "Lyssnarport": 5001,
  "Defaultvarden": [
    {
      "Kod": "VerksamhetNamn",
      "Text": "Statens Fastighetsverk\r\n(SFV)",
      "Varde": 0
    }
  ],
  "Format": [
    {
      "Kod": "svg",
      "Text": "SVG-vektor"
    }
  ],
  "Sekretess": [
    {
      "Kod": "Hemlig",
      "Text": "HEMLIG",
      "Paragrafer": [...],
      "SSK": [...]
    }
  ]
}
```

#### Användning för tekniker:

**✅ Konfigurationsvalidering:**
- **Verifiera** att ändringar i JSON-filen faktiskt laddats
- **Kontrollera** aktuell portkonfiguration
- **Validera** säkerhetsklassificeringar och paragrafer
- **Felsöka** varför vissa inställningar inte fungerar som förväntat

**📋 Vad du kan kontrollera:**
- **Lyssnarport** - Vilken port tjänsten faktiskt använder
- **VerksamhetNamn** - Organisationens namn som visas på stämplar
- **Säkerhetsklassificeringar** - Alla tillgängliga sekretessnivåer
- **Bildformat** - Vilka format som stöds
- **Standardstorlekar** - Default-dimensioner för stämplar

**🔧 Praktiska exempel:**

**Kontrollera laddad port:**
```powershell
$config = Invoke-RestMethod -Uri "http://localhost:5001/config"
Write-Host "Tjänsten lyssnar på port: $($config.Lyssnarport)"
```

**Verifiera organisationsnamn:**
```powershell
$config = Invoke-RestMethod -Uri "http://localhost:5001/config"
$orgName = ($config.Defaultvarden | Where-Object {$_.Kod -eq "VerksamhetNamn"}).Text
Write-Host "Organisationsnamn: $orgName"
```

**Lista alla säkerhetsklassificeringar:**
```powershell
$config = Invoke-RestMethod -Uri "http://localhost:5001/config"
Write-Host "Tillgängliga säkerhetsklassificeringar:"
$config.Sekretess | ForEach-Object { Write-Host "- $($_.Kod): $($_.Text)" }
```

### 3.2 `/defaultvarden` och `/defaultvardentext` - Standardvärden

**Syfte:** Hämta standardinställningar som används för stämpelgenerering.

**URL:**
- `http://localhost:5001/defaultvarden` (JSON)
- `http://localhost:5001/defaultvardentext` (text)

#### Användning för tekniker:
- **Kontrollera standardstorlekar** för stämplar
- **Verifiera organisationsnamn** som visas på stämplar
- **Valideera standardformat** för bildgenerering

### 3.3 `/format` och `/formattext` - Bildformat

**Syfte:** Lista alla bildformat som systemet stöder.

**URL:**
- `http://localhost:5001/format` (JSON)
- `http://localhost:5001/formattext` (text)

#### Svar exempel:
```json
[
  {
    "Kod": "png",
    "Text": "PNG-bild"
  },
  {
    "Kod": "svg", 
    "Text": "SVG-vektor"
  },
  {
    "Kod": "emf",
    "Text": "Windows Metafile"
  }
]
```

#### Användning för tekniker:
- **Verifiera** vilka format som kan användas i stämpelgenerering
- **Kontrollera** att alla förväntade format är konfigurerade
- **Felsöka** problem med specifika bildformat

### 3.4 `/paragrafer` - OSL-paragrafer

**Syfte:** Hämta tillgängliga paragrafer för en specifik säkerhetsklassificering.

**URL:** `http://localhost:5001/paragrafer?sekretess=[sekretessKod]`
**Parametrar:** 
- `sekretess` (obligatorisk) - Sekretesskod (t.ex. "Hemlig", "SK")

#### Exempel:
```
http://localhost:5001/paragrafer?sekretess=Hemlig
```

#### Användning för tekniker:
- **Validera** att korrekta OSL-paragrafer är konfigurerade
- **Kontrollera** juridisk efterlevnad
- **Felsöka** varför vissa paragrafer inte visas i stämplar

### 3.5 `/ssk` - Säkerhetsskyddsklassificeringar

**Syfte:** Hämta tillgängliga SSK-nivåer för SK-klassificerade dokument.

**URL:** `http://localhost:5001/ssk?sekretess=[sekretessKod]`
**Parametrar:**
- `sekretess` (valfri) - Standard: "Hemlig"

#### Användning för tekniker:
- **Kontrollera** SSK-nivåer för SK-klassificerad information
- **Validera** att alla organisationsspecifika SSK-nivåer finns

---

## 4. Stämpelgenerering

**Testa funktionalitet:** Dessa endpoints låter dig testa stämpelgenerering direkt utan att använda DIM-klienten.

### 4.1 `/stampel` och `/stampelurl` - Säkerhetsstämplar

**Syfte:** Generera säkerhetsstämplar baserat på angivna parametrar.

**URL:** 
- `http://localhost:5001/stampel` - Returnerar bilddata
- `http://localhost:5001/stampelurl` - Returnerar filsökväg

**Obligatoriska parametrar:**
- `sekretess` - Sekretesskod (t.ex. "Hemlig", "SK") 
- `paragrafer` - Kommaseparerade paragrafkoder
- `ssk` - Säkerhetsskyddsklassificering (för SK-sekretess)

**Valfria parametrar:**
- `datum` - Datum (standard: dagens datum)
- `bredd` - Bredd i pixlar (standard: 302)
- `hojd` - Höjd i pixlar (standard: 182)
- `format` - Bildformat (standard: png)
- `orientering` - "h" för horisontell, "v" för vertikal
- `vhtnamn` - Verksamhetsnamn (override)

#### Exempel för tekniker:

**Testa grundläggande stämpelgenerering:**
```
http://localhost:5001/stampelurl?sekretess=Hemlig&paragrafer=21kap8p,21kap1p&datum=2024-12-19&format=png
```

**Testa SK-stämpel:**
```
http://localhost:5001/stampelurl?sekretess=SK&paragrafer=21kap8p&ssk=Kvalificerat&datum=2024-12-19&format=svg
```

#### Användning för tekniker:

**🧪 Funktionstestning:**
- **Verifiera** att stämpelgenerering fungerar efter konfigurationsändringar
- **Testa** olika parameterkombinationer
- **Validera** att alla bildformat genereras korrekt
- **Kontrollera** att metadata inkluderas i stämplar

**🔧 Automatiserade tester:**

**PowerShell-test av stämpelgenerering:**
```powershell
# Test grundläggande stämpelgenerering
$testUrl = "http://localhost:5001/stampelurl?sekretess=Hemlig&paragrafer=21kap8p&format=png"
try {
    $response = Invoke-RestMethod -Uri $testUrl
    Write-Host "Stämpelgenerering: OK - $response" -ForegroundColor Green
} catch {
    Write-Host "Stämpelgenerering: FEL - $($_.Exception.Message)" -ForegroundColor Red
}
```

### 4.2 `/stampelhanvisning` och `/stampelhanvisningurl` - Hänvisningsstämplar

**Syfte:** Generera kompakta hänvisningsstämplar för flersidiga dokument.

**URL:**
- `http://localhost:5001/stampelhanvisning` - Returnerar bilddata
- `http://localhost:5001/stampelhanvisningurl` - Returnerar filsökväg

**Obligatoriska parametrar:**
- `ssk` - Säkerhetsskyddsklassificering

**Valfria parametrar:**
- `bredd`, `hojd`, `format` - Samma som för vanliga stämplar

#### Exempel:
```
http://localhost:5001/stampelhanvisningurl?ssk=Kvalificerat&format=svg
```

---

## 5. Felsökning och diagnostik

### 5.1 Vanliga problem och lösningar

#### 🚨 Tjänsten svarar inte på HTTP-requests

**Symptom:** Timeout eller "connection refused" fel

**Kontrollera:**
1. **Tjänststatus:** `Get-Service DIMService`
2. **Portkonfiguration:** Verifiera `Lyssnarport` i config
3. **Brandvägg:** Kontrollera att port 5001 är tillåten lokalt
4. **Processanvändning:** Kontrollera att inget annat använder porten

**Diagnostik:**
```powershell
# Kontrollera om port används
netstat -an | findstr :5001

# Kontrollera tjänstestatus
Get-Service DIMService | Format-List *

# Testa grundläggande anslutning
Test-NetConnection -ComputerName localhost -Port 5001
```

#### 🔧 Endpoints returnerar fel

**Vanliga felmeddelanden:**
- `"Sekretess ej funnen"` - Felaktig sekretesskod i parameter
- `"Internt serverfel"` - Problem med konfiguration eller filsystem
- `"Ingen fellogg finns"` - Inga fel har inträffat ännu

**Lösningssteg:**
1. **Kontrollera parametrar** - Använd `/config` för att verifiera giltiga värden
2. **Validera konfiguration** - Kontrollera JSON-syntax
3. **Kontrollera filbehörigheter** - NetworkService måste kunna läsa/skriva
4. **Granska felloggar** - Använd `/fellogg` för detaljerad information

### 5.2 Monitoring och automation

#### Komplett hälsokontrollskript:

```powershell
# DIM System Health Check
Write-Host "=== DIM System Health Check ===" -ForegroundColor Cyan

# 1. Tjänststatus
$service = Get-Service -Name "DIMService" -ErrorAction SilentlyContinue
if ($service) {
    Write-Host "Tjänst status: $($service.Status)" -ForegroundColor $(if($service.Status -eq "Running"){"Green"}else{"Red"})
} else {
    Write-Host "Tjänst status: EJ INSTALLERAD" -ForegroundColor Red
}

# 2. HTTP API-tillgänglighet
try {
    $response = Invoke-RestMethod -Uri "http://localhost:5001/isrunning" -TimeoutSec 5
    Write-Host "HTTP API: OK" -ForegroundColor Green
} catch {
    Write-Host "HTTP API: OFFLINE" -ForegroundColor Red
}

# 3. Antal fel i logg
try {
    $errors = Invoke-RestMethod -Uri "http://localhost:5001/fellogg" -TimeoutSec 5
    $errorCount = $errors.Count
    Write-Host "Felloggar: $errorCount fel" -ForegroundColor $(if($errorCount -eq 0){"Green"}elseif($errorCount -lt 5){"Yellow"}else{"Red"})
} catch {
    Write-Host "Felloggar: Kunde inte hämta" -ForegroundColor Yellow
}

# 4. Konfigurationsvalidering
try {
    $config = Invoke-RestMethod -Uri "http://localhost:5001/config" -TimeoutSec 5
    Write-Host "Konfiguration: OK (Port: $($config.Lyssnarport))" -ForegroundColor Green
} catch {
    Write-Host "Konfiguration: FEL" -ForegroundColor Red
}

# 5. Stämpelgenerering test
try {
    $testResult = Invoke-RestMethod -Uri "http://localhost:5001/stampelurl?sekretess=Hemlig&paragrafer=21kap8p&format=png" -TimeoutSec 10
    Write-Host "Stämpelgenerering: OK" -ForegroundColor Green
} catch {
    Write-Host "Stämpelgenerering: FEL" -ForegroundColor Red
}

Write-Host "=== Health Check Complete ===" -ForegroundColor Cyan
```

---

## 6. Säkerhet och bästa praxis

### 6.1 Säkerhetsöverväganden

**🔒 Networksäkerhet:**
- **Standard:** Endast localhost-access (127.0.0.1)
- **Ingen autentisering** - API:et litar på nätverkssegmentering
- **Känslig data** - Konfiguration kan innehålla organisationsspecifik information
- **Loggdata** - Felloggar kan exponera systemsökvägar

**⚠️ Varningar för tekniker:**
- **Dokumentera** eventuella förändringar av nätverksinställningar
- **Begränsa** åtkomst till endast nödvändiga system
- **Övervaka** användning av endpoints i produktionsmiljö
- **Backup** kritiska konfigurationer innan ändringar

### 6.2 Prestanda och begränsningar

**📊 Prestandaaspekter:**
- **Samtidiga requests:** HTTP-servern hanterar en request åt gången
- **Stämpelgenerering:** Kan ta 1-3 sekunder beroende på komplexitet
- **Minnespåverkan:** Bildgenerering kräver temporärt minne
- **Filsystemsaccess:** Vissa endpoints läser från disk

**💡 Rekommendationer:**
- **Cache-resultat** i egna övervakningsskript
- **Använd timeout** på alla HTTP-requests (5-10 sekunder)
- **Undvik** kontinuerliga stämpelgenereringstester i produktion
- **Övervaka** diskutrymme för temporära filer

---

## 7. Support och vidare läsning

### Relaterad dokumentation:
- [`README.md`](../README.md) - Allmän systemöversikt
- [`docs/DIM_Konfigurationsfiler_Tekniker.md`](DIM_Konfigurationsfiler_Tekniker.md) - Konfigurationshantering
- [`docs/DIM_MSI_Leveransdokumentation_Tekniker.md`](DIM_MSI_Leveransdokumentation_Tekniker.md) - Installation och deployment

### Support:
- **Utvecklare:** Dynalab AB    www.dynalab.se
- **API-version:** Se `/config` endpoint för aktuell systemversion

**Vid API-problem, inkludera:**
- URL som användes
- Parametrar som skickades
- Felmeddelande från `/fellogg`
- DIMService version och konfiguration
