# Defect Reports — Quality API
**Black Box Test | April 2026**

---

## Oversigt

| ID | Titel | Endpoint | Severity | Status |
|---|---|---|---|---|
| DEF-001 | Grade off-by-one: score=60 returnerer "F" | /api/grade | High | New |
| DEF-002 | Rabat off-by-one: amount=100 giver 0% | /api/discount | High | New |
| DEF-003 | Sensor-gennemsnit: integer division fejl | /api/sensor-average | High | New |
| DEF-004 | Username lowercaser uventet | /api/username | Medium | New |
| DEF-005 | Username whitespace-only returnerer tom streng | /api/username | Medium | New |
| DEF-006 | Booking: 6 pladser godkendes fejlagtigt | /api/booking | High | New |

---

## DEF-001 — Grade off-by-one: score=60 returnerer "F" i stedet for "D"

| Felt | Indhold |
|---|---|
| Defect ID | DEF-001 |
| Titel | Grade-grænse off-by-one: score=60 returnerer "F" i stedet for "D" |
| Severity | High |
| Priority | High |
| API endpoint / Modul | `/api/grade` \| `QualityService::calculateGrade()` |
| Preconditions | Backend kører lokalt på port 8080. Score-intervallet 0–100 forventes validt. |
| Steps to Reproduce | 1. Åbn browserklienten<br>2. Indtast score = `60`<br>3. Klik "Kald /api/grade"<br>4. Observér resultatet |
| Expected Result | `"D"` — score 60 er på grænsen for D-kategorien (60–69) |
| Actual Result | `"F"` — score 60 falder igennem alle betingelser og rammer default-returværdien |
| Test Type | Black box / Grænseværdi-analyse |
| Driftstilstand / Konfiguration | Default |
| Environment | Chrome, Windows 11, april 2026 |
| Reporter | Ruben |
| Status | New |

**Root cause:** Kode: `if (score > 60) return "D";` — brug af `>` i stedet for `>=`. Score 60 er IKKE > 60.

---

## DEF-002 — Rabatgrænse off-by-one: amount=100 giver 0% rabat i stedet for 10%

| Felt | Indhold |
|---|---|
| Defect ID | DEF-002 |
| Titel | Rabatgrænse off-by-one: amount=100 giver 0% rabat i stedet for 10% |
| Severity | High |
| Priority | High |
| API endpoint / Modul | `/api/discount` \| `QualityService::calculateDiscount()` |
| Preconditions | Backend kører lokalt. Rabatlogik: amount ≥ 100 → 10%, amount ≥ 500 → 20%, amount ≥ 1000 → 30%. |
| Steps to Reproduce | 1. Åbn browserklienten<br>2. Sæt amount = `100`, loyal = false, coupon = tom, production = false, hour = `12`<br>3. Klik "Kald /api/discount"<br>4. Observér result-værdien |
| Expected Result | `10` — amount = 100 ligger præcis på grænsen for første rabatinterval |
| Actual Result | `0` — betingelsen `amount > 100` er ikke opfyldt for nøjagtigt 100 |
| Test Type | Black box / Grænseværdi-analyse |
| Driftstilstand / Konfiguration | loyal_customer = false, coupon = ingen, production_mode = false, hour = 12 |
| Environment | Chrome, Windows 11, april 2026 |
| Reporter | Ruben |
| Status | New |

**Root cause:** Kode: `if (request.amount > 100) discount = 10;` — brug af strengt `>` i stedet for `>=`.

---

## DEF-003 — Sensor-gennemsnit returnerer forkert værdi pga. integer division

| Felt | Indhold |
|---|---|
| Defect ID | DEF-003 |
| Titel | Sensor-gennemsnit returnerer forkert værdi pga. integer division |
| Severity | High |
| Priority | High |
| API endpoint / Modul | `/api/sensor-average` \| `QualityService::calculateSensorAverage()` |
| Preconditions | Backend kører. Input er en CSV-liste af heltal. |
| Steps to Reproduce | 1. Åbn browserklienten<br>2. Indtast values = `1,2` i sensorfeltet<br>3. Klik "Kald /api/sensor-average"<br>4. Observér result |
| Expected Result | `1.5` — sum=3, antal=2, 3÷2 = 1.5 |
| Actual Result | `1.0` — division sker som heltalsdivision inden cast til double |
| Test Type | Black box / Ækvivalenspartitionering |
| Driftstilstand / Konfiguration | Ingen speciel konfiguration |
| Environment | Chrome, Windows 11, april 2026 |
| Reporter | Ruben |
| Status | New |

**Root cause:** Kode: `return static_cast<double>(sum / values.size());` — cast sker EFTER heltalsdivision. Skal være: `return static_cast<double>(sum) / values.size();`

---

## DEF-004 — formatUsername lowercaser korrekt input og ødelægger store bogstaver

| Felt | Indhold |
|---|---|
| Defect ID | DEF-004 |
| Titel | formatUsername lowercaser korrekt input og ødelægger store bogstaver |
| Severity | Medium |
| Priority | Medium |
| API endpoint / Modul | `/api/username` \| `QualityService::formatUsername()` |
| Preconditions | Backend kører. Brugernavn-validering forventes at trimme whitespace og returnere det rensede navn. |
| Steps to Reproduce | 1. Åbn browserklienten<br>2. Indtast name = `  Alice  ` (med mellemrum)<br>3. Klik "Kald /api/username"<br>4. Observér result |
| Expected Result | `"Alice"` — whitespace trimmes, men store bogstaver bevares |
| Actual Result | `"alice"` — funktionen omdanner hele strengen til lowercase |
| Test Type | Black box / Funktionalitetstest |
| Driftstilstand / Konfiguration | Ingen speciel konfiguration |
| Environment | Chrome, Windows 11, april 2026 |
| Reporter | Ruben |
| Status | New |

**Root cause:** Kode anvender `std::tolower()` på alle tegn. Spec/test forventer kun trimning — ikke lowercase-konvertering.

---

## DEF-005 — formatUsername returnerer tom streng for whitespace-only input

| Felt | Indhold |
|---|---|
| Defect ID | DEF-005 |
| Titel | formatUsername returnerer tom streng for whitespace-only input |
| Severity | Medium |
| Priority | Medium |
| API endpoint / Modul | `/api/username` \| `QualityService::formatUsername()` |
| Preconditions | Backend kører. Tom streng returnerer "anonymous"; whitespace-only bør returnere "Ugyldig". |
| Steps to Reproduce | 1. Åbn browserklienten<br>2. Indtast name = `   ` (3 mellemrum)<br>3. Klik "Kald /api/username"<br>4. Observér result |
| Expected Result | `"Ugyldig"` eller `"anonymous"` — whitespace-only er ugyldigt input |
| Actual Result | `""` (tom streng) — funktionen trim()'er whitespace til tom streng og returnerer den |
| Test Type | Black box / Grænseværdi-analyse |
| Driftstilstand / Konfiguration | Ingen speciel konfiguration |
| Environment | Chrome, Windows 11, april 2026 |
| Reporter | Ruben |
| Status | New |

**Root cause:** Funktionen tjekker kun `if (name.empty())` FØR trim. Efter trim kan value være tom streng, som returneres direkte uden ekstra check.

---

## DEF-006 — Booking: 6 pladser godkendes fejlagtigt (off-by-one i seats-grænse)

| Felt | Indhold |
|---|---|
| Defect ID | DEF-006 |
| Titel | Booking: 6 pladser godkendes fejlagtigt (off-by-one i seats-grænse) |
| Severity | High |
| Priority | High |
| API endpoint / Modul | `/api/booking` \| `QualityService::canBookSeats()` |
| Preconditions | Backend kører. Maksimal kapacitet uden override antages at være 5 pladser. |
| Steps to Reproduce | 1. Åbn browserklienten<br>2. Sæt seats = `6`, override = false, maintenance = false, reserved = `0`<br>3. Klik "Kald /api/booking"<br>4. Observér result |
| Expected Result | `false` — 6 pladser overskrider kapaciteten og kræver safety override |
| Actual Result | `true` — 6 pladser godkendes, da betingelsen er `<= 6` |
| Test Type | Black box / Grænseværdi-analyse |
| Driftstilstand / Konfiguration | hasSafetyOverride = false, maintenanceMode = false, currentReservations = 0 |
| Environment | Chrome, Windows 11, april 2026 |
| Reporter | Ruben |
| Status | New |

**Root cause:** Kode: `if (request.requestedSeats <= 6) return true;` — brug af `<=` i stedet for `< 6`. Grænsen burde ekskludere 6.

Forklaring af CI/CD pipeline:
Pipelinen har to jobs. build-and-test bygger projektet og kører alle unit tests på en Ubuntu-server — fejler én test, stopper pipelinen. deploy-pages kører kun hvis tests er grønne og deployer web/-mappen til GitHub Pages.

Refleksion:
Manuel black box-test er god til at finde fejl fra en brugers perspektiv ved at teste grænseværdier uden at kende koden. Automatiseret CI/CD sikrer at de samme tests køres ved hvert push, så fejl fanges tidligt og man aldrig deployer en fejlende version. De to tilgange supplerer hinanden godt.