# ForecastPilot

**An AI agent that turns two inputs — a user base and a feature description — into a structured, three-scenario AI revenue forecast in under two minutes.**

Built to demonstrate how a well-scoped methodology, encoded as agent instructions, can replace hours of analyst work without sacrificing rigour.

🔗 **[Try it live → ramjoshionline.github.io/forecastpilot](https://ramjoshionline.github.io/forecastpilot)**  
No sign-up. No installation. Runs entirely in the browser with your own Anthropic API key.

---

## Why this exists

Most AI revenue forecasts fail in one of two ways: they're too optimistic (using TAM instead of SOM, ignoring retention floors, skipping validation checks) or they're too resource-heavy to run regularly (requiring a dedicated analyst, a spreadsheet model, and a strategy offsite).

ForecastPilot is an attempt to solve both problems by encoding a rigorous bottom-up methodology directly into an agent's reasoning instructions. The agent doesn't improvise — it follows a fixed 8-step pipeline, states every assumption it makes, and validates its output against three independent benchmarks before returning a result.

The design question I was answering: *what does it look like when a PM encodes their mental model into an agent, rather than into a spreadsheet?*

---

## What it does

Give ForecastPilot two inputs:

| Input | Required? | Example |
|---|---|---|
| Total user base (TAM) | ✅ | `50,000 users` |
| AI feature description | ✅ | `AI writing assistant for PMs — drafts briefs and status updates inline` |
| Plan / tier breakdown | Optional | `60% free, 30% pro, 10% enterprise` |
| Forecast horizon | Optional | `24 months` |
| Region / compliance notes | Optional | `EU-heavy, HIPAA constraints apply` |
| Existing adoption data | Optional | `8% adopted in closed beta` |

It returns a full structured report with:
- Executive summary
- SOM derivation (TAM → SAM → SOM with explicit filter percentages)
- Month-by-month adoption curve (24 months)
- Usage segmentation (Light / Moderate / Heavy profiles)
- Revenue model (subscription upgrades + on-demand overages)
- Cohort retention model with long-term floor
- **3-scenario forecast table** (Conservative / Base / Optimistic at Month 6, 12, 24)
- Triangulation & validation against public benchmarks
- Full assumption log with confidence labels
- Confidence rating (High / Medium / Low)
- **Downloadable PDF report**

---

## Agent architecture

This section explains the design decisions behind ForecastPilot as an AI agent — not just what it does, but *how it's built* and *why*.

### Agent type: Single-turn analytical agent

ForecastPilot is a **single-turn, zero-memory, instruction-following agent**. It receives a structured input, reasons through a fixed pipeline, and returns a complete output in one pass.

This was a deliberate design choice. Revenue forecasting is a bounded, well-defined task with a known methodology. A multi-turn conversational agent would introduce unnecessary ambiguity and latency. The single-turn pattern is faster, more predictable, and easier to evaluate.

The agent can ask up to 3 clarifying questions before proceeding if inputs are ambiguous — but this is bounded. After 3 exchanges (or if inputs are clear), it must proceed with stated assumptions. This prevents the agent from stalling and forces it to make its reasoning explicit.

---

### The system prompt as the orchestration layer

There is no external orchestration framework (no LangChain, no LlamaIndex, no AutoGen). **The system prompt is the orchestration layer.**

This is intentional. The methodology is the core IP. Encoding it in a system prompt rather than in code means:
- The reasoning steps are readable, auditable, and version-controlled in plain text (`system_prompt.txt`)
- The agent's behaviour can be updated without changing application code
- The prompt itself serves as documentation of the methodology

The system prompt defines an ordered 8-step pipeline. The model is instructed to work through every step sequentially and to label each assumption as `[PROVIDED]` or `[ASSUMED]`. This is a form of **chain-of-thought prompting** — forcing the model to externalise its reasoning rather than jump to a conclusion.

```
Step 1 → Market sizing      (TAM → SAM → SOM)
Step 2 → Adoption modelling (benchmark curve + adjustments)
Step 3 → Usage segmentation (Light / Moderate / Heavy profiles)
Step 4 → Revenue path mapping (upgrades vs. overages)
Step 5 → Retention modelling (cohort curve + floor)
Step 6 → Scenario generation (Conservative / Base / Optimistic)
Step 7 → Triangulation & validation (3 independent checks)
Step 8 → Confidence rating + assumption log
```

The model cannot skip steps or reorder them. Each step's output feeds the next.

---

### Memory architecture: stateless by design

ForecastPilot has **no persistent memory**. Each forecast is an independent, self-contained inference call.

This is a product decision, not a technical limitation. Revenue forecasts carry sensitive business data. A stateless architecture means:
- No forecast data is retained after the session
- No cross-contamination between runs
- The user retains full control of their data

The API key is stored in browser `localStorage` for convenience (so users don't re-enter it on every visit), but all forecast inputs and outputs live only in the browser tab for the duration of the session.

If persistent memory were added in a future version, it would be for **recalibration** — storing actuals against forecasted values to improve assumption accuracy over time. That's the natural next step for a production deployment.

---

### Tools and capabilities

ForecastPilot uses a minimal tool set by design:

| Capability | How it's implemented | Why |
|---|---|---|
| **Structured reasoning** | Chain-of-thought via system prompt | Forces sequential, auditable steps |
| **Benchmark lookup** | Encoded in system prompt (static) | GitHub Copilot, Adobe Firefly, Autodesk AI ARPU ranges |
| **Streaming output** | Anthropic SSE streaming API | Shows reasoning steps as they're generated — builds trust |
| **PDF generation** | `html2pdf.js` (client-side) | Zero server dependency; works fully in-browser |
| **Markdown rendering** | `marked.js` (client-side) | Formats the report output cleanly |

There is no web search tool, no code interpreter, and no external API calls beyond the Anthropic inference endpoint. This is deliberate scope control — the agent does one thing and does it well.

A future version could add a **web search tool** to pull live ARPU benchmarks (rather than using static values from the system prompt), and a **code interpreter** to run the scenario math externally and validate the model's arithmetic. Both would increase accuracy without changing the core architecture.

---

### The UI as an agent transparency layer

The **Analysis Pipeline** view — the 8-step progress tracker visible during generation — is not just a loading screen. It's an **agent transparency mechanism**.

Each step lights up as the model generates tokens that match that step's reasoning pattern. This serves two product goals:

1. **Trust**: Users can see the agent is following the methodology, not free-associating
2. **Debuggability**: If a forecast looks wrong, users can trace it back to the step where the reasoning diverged

This is an underused pattern in AI products. Most agents hide their reasoning entirely. ForecastPilot surfaces it because the methodology *is the value proposition* — if users can't see the steps, they can't trust the output.

---

### Why claude-opus-4-6 specifically

ForecastPilot uses `claude-opus-4-6` rather than a faster/cheaper model because:

1. **Quantitative reasoning**: The scenario math requires accurately applying percentages across compounding retention curves and multiple user cohorts. Smaller models make systematic errors in multi-step numerical reasoning.
2. **Instruction following**: The 8-step pipeline requires strict sequential adherence. Claude Opus follows complex structured instructions more reliably than smaller models.
3. **Assumption quality**: The agent must infer defensible assumptions from minimal inputs. This requires world knowledge about B2B SaaS adoption patterns that smaller models handle less accurately.

For a production deployment, a two-tier approach would make sense: Sonnet for initial input parsing and clarification questions, Opus for the full 8-step reasoning pass.

---

## The 8-step methodology in detail

### Step 1 — Market sizing (TAM → SAM → SOM)

The agent applies three successive filters to arrive at the Serviceable Obtainable Market — the realistic eligible user pool for forecasting:

- **TAM → SAM**: Removes users on plans/regions/product versions without AI access
- **SAM → SOM**: Further filters by role relevance, account health, and compliance barriers

Every filter percentage is stated explicitly as `[ASSUMED]` with reasoning. This prevents the forecast from silently inflating its own eligible base.

### Step 2 — Adoption curve

Rather than using a single adoption rate, the agent builds a **month-by-month curve** for 24 months, anchored to B2B SaaS AI benchmarks:

| Milestone | Benchmark range |
|---|---|
| Month 3 | 10–15% of SOM |
| Month 12 | 20–30% of SOM |
| Month 36 | 40–60% of SOM |

The curve is then adjusted based on four factors: workflow embeddedness, onboarding quality, feature gating, and market buzz. Each factor has a defined directional impact (+/- percentage points), making the adjustment auditable.

### Step 3 — Usage segmentation

The agent defines three behavioural tiers rather than using a flat average:

| Profile | Population | Behaviour |
|---|---|---|
| Light | Bottom 50% | Minor tasks, low frequency |
| Moderate | 50th–85th percentile | Weekly workflows |
| Heavy | Top 15% | Daily core production use |

A **blended weighted average** is then computed across tiers. This matters because heavy users often drive a disproportionate share of consumption — a flat average would understate infrastructure cost and overstate margin.

### Step 4 — Revenue path mapping

Consumption is split across two monetisation paths:

- **Path A** (Subscription upgrades): Users whose usage exceeds their tier allowance, modelled as upgrades × incremental subscription revenue
- **Path B** (On-demand overages): Excess consumption billed at overage rates

This two-path model captures the real dynamics of usage-based pricing, where different user segments monetise differently.

### Step 5 — Cohort retention

The agent applies a retention curve rather than a flat churn rate:

| Month | Retention range |
|---|---|
| Month 1 | 100% |
| Month 2 | 60–70% |
| Month 3 | 45–55% |
| Month 6 | 35–45% |
| Month 12+ | 25–40% (floor) |

Active users each month = new adopters + surviving users from all prior cohorts. This compounding model is more accurate than simple monthly churn for features with high initial drop-off but stable long-term engaged cores.

### Step 6 — Three scenarios

Pricing is held constant across all three scenarios. Only demand-side variables are varied:

| Driver | Conservative | Base | Optimistic |
|---|---|---|---|
| Adoption rate | −40% of base | Base | +60% of base |
| Blended usage | −33% of base | Base | +50% of base |
| New customer growth | Pipeline low-end | Mid-point | Upside |

**Why hold pricing constant?** Pricing is a decision, not an assumption. Varying it in scenarios conflates strategy with forecasting and makes the output less actionable.

### Step 7 — Triangulation & validation

The bottom-up number is validated against three independent checks:

1. **Revenue share check**: AI revenue as % of total product revenue. Expected: 3–8% in year 1, 10–20% by year 3. Flags if outside range.
2. **Unit economics check**: Implied gross margin at forecast volume. Should be >60% at scale.
3. **ARPU benchmark check**: Revenue per eligible user vs. GitHub Copilot (~$19/user/month), Adobe Firefly (embedded in $55+/month Creative Cloud), Autodesk AI (embedded in $300+/year plans). Flags if >2× higher or lower than closest comp.

If any check diverges significantly from expectations, the agent flags it explicitly and identifies which upstream assumption is likely driving it.

### Step 8 — Confidence rating + assumption log

Every assumption made during the forecast is logged in a structured table:

| Assumption | Value used | Source | Confidence |
|---|---|---|---|
| SAM filter % | 28% excluded | [ASSUMED] — legacy plan estimate | Medium |
| Month 12 adoption | 22% | [ASSUMED] — embedded workflow adj. | Medium |
| Retention floor | 32% | [ASSUMED] — B2B SaaS benchmark | Low |

The overall confidence rating (High / Medium / Low) reflects the proportion of key assumptions that were provided vs. inferred. This gives the reader a calibrated sense of how much to trust the output — and what data collection would most improve it.

---

## Tech stack

| Layer | What's used | Decision rationale |
|---|---|---|
| **AI model** | `claude-opus-4-6` | Best-in-class multi-step reasoning and instruction following |
| **Orchestration** | System prompt (plain text) | Methodology as code — readable, auditable, version-controlled |
| **Streaming** | Anthropic SSE API | Real-time transparency of agent reasoning steps |
| **Frontend** | Vanilla HTML/CSS/JS | Zero build step, zero dependencies, works as a static file |
| **PDF export** | `html2pdf.js` (client-side) | No server needed; report generated locally in the browser |
| **Hosting** | GitHub Pages | Zero infrastructure; auto-deploys on push |
| **API key storage** | Browser `localStorage` | Persists across sessions; never sent to any third-party server |

---

## What this is not

- **Not a general-purpose AI assistant.** ForecastPilot does one thing. Scope control is a feature.
- **Not a replacement for a finance team.** It's a starting point — a rigorous first draft that a CFO or VP of Product can stress-test, not sign off on blindly.
- **Not a black box.** Every assumption is visible. Every step is traceable. The methodology is open-source.

---

## What's next

If this were a production product, the natural next steps would be:

1. **Actuals tracking loop**: Store forecast vs. actuals to recalibrate assumptions over time — turning the agent from a one-shot tool into a learning system
2. **Live benchmark retrieval**: Replace static ARPU benchmarks with a web search tool call to pull current public data
3. **Team mode**: Server-side API key + auth layer so product and finance teams can share forecasts without each managing their own key
4. **Evaluation harness**: A test suite of known-outcome scenarios to measure forecast accuracy and catch regression when the system prompt is updated

---


