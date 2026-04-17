# FED MACRO TERMINAL

Interaktivt makroøkonomisk dashboard med data fra FRED (Federal Reserve Economic Data).

Live demo: https://vjanrikard.github.io/fed_macro_terminal/

## Arkitektur

FRED API støtter ikke CORS fra nettlesere — kall direkte fra frontend vil alltid bli blokkert. Løsningen er en **snapshot-strategi**:

```
GitHub Actions (daglig kl. 06:00 UTC)
  → fetch_snapshot.py henter alle serier server-side
  → lagrer data/fred_snapshot.json
  → deployer til gh-pages
Browser
  → laster data/fred_snapshot.json (same-origin, ingen CORS-begrensning) ✓
```

## Oppsett

### 1. Få FRED API-nøkkel (gratis)
Registrer deg på https://fred.stlouisfed.org/docs/api/api_key.html

### 2. Legg til API-nøkkel som GitHub Secret
- Gå til repo → Settings → Secrets and variables → Actions
- Klikk "New repository secret"
- Name: `FRED_API_KEY`
- Value: din API-nøkkel

### 3. GitHub Pages oppsett
- Gå til repo → Settings → Pages
- Source: Deploy from branch → `gh-pages` → / (root)

### 4. Push til main
GitHub Actions bygger automatisk, henter FRED-snapshot og deployer til GitHub Pages.

Hvis du ikke allerede har koblet prosjektet til GitHub, gjør dette én gang lokalt:

```powershell
cd C:\Users\vevan\OneDrive\GitHub\code\claude\Fed_Macro_Terminal
git branch -M main
git remote add origin https://github.com/vjanrikard/fed_macro_terminal.git
git push -u origin main
```

Hvis du allerede har laget en lokal commit og bare vil pushe den til GitHub nå:

```powershell
cd C:\Users\vevan\OneDrive\GitHub\code\claude\Fed_Macro_Terminal
git branch -M main
git remote add origin https://github.com/vjanrikard/fed_macro_terminal.git
git push -u origin main
```

Hvis `origin` allerede er satt opp, holder det med:

```powershell
cd C:\Users\vevan\OneDrive\GitHub\code\claude\Fed_Macro_Terminal
git branch -M main
git push -u origin main
```

Når du senere har gjort endringer og vil publisere dem:

```powershell
cd C:\Users\vevan\OneDrive\GitHub\code\claude\Fed_Macro_Terminal
git add .
git commit -m "Beskriv endringen"
git push
```

Etter push:
- GitHub Actions kjører workflowen på `main`
- snapshot bygges på nytt
- GitHub Pages oppdateres fra `gh-pages`

Hvis `origin` allerede finnes, bruk kun:

```powershell
git branch -M main
git push -u origin main
```

Hvis push blir avvist fordi GitHub-repoet allerede har innhold:

```powershell
cd C:\Users\vevan\OneDrive\GitHub\code\claude\Fed_Macro_Terminal
git branch -M main
git fetch origin
git pull origin main --allow-unrelated-histories
git push -u origin main
```

Hvis `git pull` lager merge-konflikter:
- løs konfliktene i filene
- kjør `git add .`
- kjør `git commit`
- kjør `git push`

Unngå `git push --force` med mindre du bevisst vil overskrive det som allerede ligger på GitHub.

### 5. Sjekk GitHub Actions og Pages

Etter at du har kjørt `git push`, kan du sjekke deploy slik:

1. Gå til repoet på GitHub
2. Åpne fanen `Actions`
3. Se etter workflowen `Build and Deploy to GitHub Pages`
4. Vent til workflowen er grønn (`Success`)
5. Gå deretter til `Settings` → `Pages`
6. Bekreft at siden publiseres fra `gh-pages`

Hvis workflowen feiler:
- åpne kjøringen i `Actions`
- les feilmeldingen i steget som feilet
- rett feilen lokalt
- commit og push på nytt

Når deploy er ferdig, skal siden være oppdatert her:

```text
https://vjanrikard.github.io/fed_macro_terminal/
```

## Lokalt
```bash
cp .env.example .env
# Legg inn din nøkkel i .env
npm install
npm start
# Åpne http://localhost:3000
```

Hvis du vil lagre `.env` utenfor repoet (anbefalt), sett sti via miljøvariabel før start:

```powershell
$env:FED_ENV_FILE = "C:\path\til\din\.env"
npm start
```

Eksempel innhold i ekstern `.env`:

```env
FRED_API_KEY=din_ekte_fred_api_nokkel
```

`server.js` proxyer FRED API server-side, slik at lokal utvikling fungerer uten snapshotfil.

## Datafiler

| Fil | Beskrivelse |
|---|---|
| `index.template.html` | Kildekode — API-nøkkel er erstatningsvariabel `__FRED_API_KEY__` |
| `index.html` | Bygget av GitHub Actions — ikke rediger direkte |
| `fetch_snapshot.py` | Henter alle FRED-serier og skriver `data/fred_snapshot.json` |
| `data/fred_snapshot.json` | Generert under deploy — ikke i kildekode |

## Datakilder

Alle data fra FRED API — Federal Reserve Bank of St. Louis. Snapshot oppdateres daglig.

| Indikator | Serie-ID | Frekvens |
|---|---|---|
| Fed Funds Rate | FEDFUNDS | Månedlig |
| CPI / Core CPI | CPIAUCSL / CPILFESL | Månedlig |
| PCE / Core PCE | PCEPI / PCEPILFE | Månedlig |
| Nonfarm Payrolls | PAYEMS | Månedlig |
| Arbeidsledighet | UNRATE | Månedlig |
| Real GDP | GDPC1 | Kvartalsvis |
| Industriproduksjon | INDPRO | Månedlig |
| Treasury Yields (1M–30Y) | DGS1MO–DGS30 | Daglig |
| 2Y-10Y Yield Spread | T10Y2Y | Daglig |
| HY Credit Spreads | BAMLH0A0HYM2 | Daglig |
| Fed Balance Sheet | WALCL | Ukentlig |
| M2 Money Supply | M2SL | Ukentlig |
| JOLTS Job Openings | JTSJOL | Månedlig |
| Arbeidsmarkedsdeltakelse | CIVPART | Månedlig |
| National Financial Conditions | NFCI | Ukentlig |
