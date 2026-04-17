# FED MACRO TERMINAL

Interaktivt makroøkonomisk dashboard med data fra FRED (Federal Reserve Economic Data).

Live demo: https://vjanrikard.github.io/fed_macro_terminal/

## Arkitektur

FRED API støtter ikke CORS fra nettlesere — direkte kall fra frontend blokkeres alltid. Løsningen er en **snapshot-strategi**:

```
GitHub Actions – deploy.yml (hver time)
  → fetch_snapshot.py henter alle serier server-side
  → lagrer data/fred_snapshot.json
  → committer tilbake til main

GitHub Actions – pages-deploy.yml (ved push til main)
  → injiserer FRED_API_KEY i index.html fra index.template.html
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

```bash
git add .
git commit -m "Beskriv endringen"
git push
```

Etter push:
- `pages-deploy.yml` bygger og deployer `index.html` til `gh-pages`
- `deploy.yml` kjører hvert time og oppdaterer snapshotdata

### 5. Sjekk GitHub Actions og Pages

1. Gå til repoet → fanen `Actions`
2. Se etter `Build and Deploy Pages` — vent til den er grønn
3. Bekreft under Settings → Pages at siden publiseres fra `gh-pages`

Siden er tilgjengelig på:
```
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

Anbefalt: lagre `.env` utenfor repoet og pek til den via `FED_ENV_FILE`:

```powershell
$env:FED_ENV_FILE = "C:\Users\vevan\OneDrive\Secrets\fred.env"
npm start
```

`server.js` proxyer FRED API server-side, slik at lokal utvikling fungerer uten snapshotfil.

## Datafiler

| Fil | Beskrivelse |
|---|---|
| `index.template.html` | Kildekode — API-nøkkel er erstatningsvariabel `__FRED_API_KEY__` |
| `index.html` | Bygget av GitHub Actions — ikke rediger direkte |
| `fetch_snapshot.py` | Henter alle FRED-serier og skriver `data/fred_snapshot.json` |
| `data/fred_snapshot.json` | Oppdateres automatisk hvert time |

## Datakilder

Alle data fra FRED API — Federal Reserve Bank of St. Louis. Snapshot oppdateres hvert time.

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
