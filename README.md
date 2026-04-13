# ForecastPilot

> AI Revenue Forecasting Agent — powered by Claude (`claude-opus-4-6`). Give it your user base and an AI feature description. It runs an 8-step bottom-up methodology and returns three revenue scenarios in under 2 minutes.

## Try it live

**[→ ramjoshionline.github.io/forecastpilot](https://ramjoshionline.github.io/forecastpilot)**

No installation. No sign-up. Runs entirely in the browser.

---

## How to use it (3 steps)

### Step 1 — Get an Anthropic API key

Go to [console.anthropic.com](https://console.anthropic.com) → sign up or log in → API Keys → Create Key.

Your key looks like: `sk-ant-api03-...`

> **Privacy note:** Your key is stored only in your browser's `sessionStorage` for the duration of the tab. It is sent directly to the Anthropic API and nowhere else. When you close the tab, it's gone.

### Step 2 — Enter your key on the gate screen

When you open the live URL, you'll see this screen:

```
┌─────────────────────────────────┐
│  ForecastPilot                  │
│  AI REVENUE AGENT               │
│                                 │
│  Anthropic API Key              │
│  [ sk-ant-...                 ] │
│                                 │
│  [ Launch ForecastPilot →     ] │
└─────────────────────────────────┘
```

Paste your API key and click **Launch ForecastPilot →**.

### Step 3 — Run your first forecast

Type (or paste) two things into the chat:
1. Your **total user base** — a number (e.g. "50,000 users")
2. Your **AI feature description** — what it does and who uses it

**Example input to try:**
```
50,000 users. AI writing assistant embedded in a B2B project management 
tool — helps PMs draft project briefs, status updates, and stakeholder comms.
```

Hit **Enter** and watch the agent work through all 8 steps in real time.

---

## What you get back

ForecastPilot returns a full structured forecast report with:

| Section | What it contains |
|---|---|
| Executive summary | One-paragraph bottom line |
| SOM derivation | TAM → SAM → SOM with filter percentages |
| Adoption curve | Month-by-month ramp for 24 months |
| Usage segmentation | Light / Moderate / Heavy user profiles |
| Revenue model | Subscription upgrades + on-demand overages |
| Retention model | Cohort curve with long-term floor |
| **Scenario table** | **Conservative / Base / Optimistic at Month 6, 12, 24** |
| Triangulation | 3 sanity checks vs. public benchmarks |
| Assumption log | Every inferred value labelled with confidence |
| Confidence rating | High / Medium / Low with reasoning |

At the end, a **Download JSON** button appears — click it to save the full forecast as a structured `.json` file for use in spreadsheets or dashboards.

---

## The 8-step methodology

The agent follows a rigorous bottom-up framework — no hand-waving:

1. **Derive SOM from TAM** — filters by plan, region, role relevance, compliance
2. **Select adoption curve** — B2B SaaS benchmarks + workflow embeddedness adjustments
3. **Segment usage profiles** — Light / Moderate / Heavy tiers, blended average
4. **Model revenue paths** — subscription tier upgrades vs. on-demand overages
5. **Apply cohort retention** — retention curve with long-term floor (25–40%)
6. **Generate 3 scenarios** — Conservative / Base / Optimistic, pricing held constant
7. **Triangulation checks** — revenue share %, gross margin, ARPU vs. Copilot / Firefly / Autodesk
8. **Forecast + confidence rating** — full assumption log, recalibration triggers

---

## Tech stack

| Layer | What's used |
|---|---|
| Model | `claude-opus-4-6` via Anthropic API |
| Streaming | Server-sent events (SSE), real-time token streaming |
| Frontend | Vanilla HTML / CSS / JS — zero dependencies, zero build step |
| Hosting | GitHub Pages (static) |
| API key | Browser `sessionStorage` only — never hits a third-party server |

---

## Run it locally

No server needed — just open the file:

```bash
git clone https://github.com/ramjoshionline/forecastpilot.git
cd forecastpilot
open index.html   # macOS
# or: start index.html (Windows)
# or: xdg-open index.html (Linux)
```

Enter your API key when prompted. That's it.

---

## Self-host with a server-side API key

If you want the API key kept server-side (e.g. for a team deployment), use the Node.js version in `server/`:

```bash
cd server
npm install
cp .env.example .env
# Edit .env → add your ANTHROPIC_API_KEY
node server.js
# Open http://localhost:3000
```

---

## Project structure

```
forecastpilot/
├── index.html          # Static single-file app (GitHub Pages version)
├── _config.yml         # GitHub Pages config
├── system_prompt.txt   # The 8-step agent methodology in plain text
├── README.md
└── server/             # Node.js/Express version (server-side API key)
    ├── server.js
    ├── public/
    │   └── index.html
    ├── .env.example
    └── package.json
```

---

## Questions or issues

Open an issue on GitHub or reach out on [LinkedIn](https://linkedin.com/in/ramjoshi).
