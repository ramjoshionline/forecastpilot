# ForecastPilot

AI Revenue Forecasting Agent — browser chat UI powered by Claude (`claude-opus-4-6`) via the Anthropic API. Applies an 8-step bottom-up forecasting methodology to generate 3-scenario AI revenue forecasts from minimal inputs.

## Live demo

**[forecastpilot on GitHub Pages →](https://ramjoshionline.github.io/forecastpilot)**

You'll need an Anthropic API key from [console.anthropic.com](https://console.anthropic.com). The key is stored in `sessionStorage` only and sent directly to the Anthropic API — never to any third-party server.

## How it works

Provide two inputs:
- **TAM** — your total user base (e.g. "100,000 users")
- **AI feature description** — what the feature does and who uses it

ForecastPilot runs the 8-step framework and returns:
- A structured markdown forecast report with 3 scenarios (Conservative / Base / Optimistic)
- A downloadable JSON file with all forecast data

## The 8-step methodology

1. Derive SOM from TAM — filters to Serviceable Obtainable Market
2. Select adoption curve — B2B SaaS benchmarks + adjustment factors
3. Segment usage profiles — Light / Moderate / Heavy tiers, blended average
4. Model revenue paths — subscription upgrades vs. on-demand overages
5. Apply cohort retention — retention curve with long-term floor
6. Generate 3 scenarios — Conservative / Base / Optimistic
7. Triangulation checks — revenue share, unit economics, ARPU benchmarks
8. Forecast + confidence rating — assumption log, recalibration triggers

## Tech stack

- **Model**: `claude-opus-4-6` via Anthropic API (streaming)
- **Frontend**: Vanilla HTML/CSS/JS — zero dependencies, zero build step
- **Hosting**: GitHub Pages (static) or Node.js/Express (server mode)

## Run locally (static — no server needed)

Just open `index.html` in a browser. Enter your Anthropic API key when prompted.

```bash
open index.html
```

## Run locally (server mode)

The `server/` directory contains a Node.js/Express version that keeps the API key server-side.

```bash
cd server
npm install
cp .env.example .env   # add your ANTHROPIC_API_KEY
node server.js
# open http://localhost:3000
```

## Project structure

```
forecastpilot/
  index.html          Static single-file app (GitHub Pages)
  _config.yml         GitHub Pages config
  system_prompt.txt   The 8-step agent methodology (reference)
  server/
    server.js         Express server (server-side API key)
    public/
      index.html      Server-mode UI
    .env.example
    package.json
  README.md
```
