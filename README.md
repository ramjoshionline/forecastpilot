# ForecastPilot

AI Revenue Forecasting Agent — browser-based chat UI powered by claude-opus-4-6.

## Setup

1. Install dependencies
   ```
   npm install
   ```

2. Add your Anthropic API key
   ```
   cp .env.example .env
   # Edit .env and add your ANTHROPIC_API_KEY
   ```

3. Start the server
   ```
   node server.js
   ```

4. Open in browser
   ```
   http://localhost:3000
   ```

## Usage

Provide two inputs:
- **TAM** — your total user base (e.g. "100,000 users")
- **AI feature description** — what the feature does and who uses it

ForecastPilot will run the 8-step framework and return:
- A structured markdown forecast report with 3 scenarios (Conservative / Base / Optimistic)
- A downloadable JSON file with all forecast data

## Project structure

```
forecastpilot/
  server.js           Express server + streaming API endpoint
  system_prompt.txt   Agent instructions (the 8-step methodology)
  public/
    index.html        Chat UI
  .env                Your API key (not committed)
  .env.example        Template
  package.json
```
