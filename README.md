# Sentry Intel

**An open-source intelligence dashboard for exploring defence and security reports.**

Sentry Intel collects public news, turns reports into scored alerts, and gives users a single place to read summaries, sources, timelines, and suggested response actions. It was built by **Last Minute Legends** for Zinnovatio 4.0's AI for Defence, Aerospace and National Security track.

> **Demo notice:** The bundled articles and summaries are illustrative sample data, not verified live intelligence. Treat alerts, scores, summaries, citations, and response suggestions as starting points for human review—not operational advice.

## What it does

- **Dashboard:** Browse alerts by severity and inspect their summaries, citations, timelines, and response playbooks.
- **Manual news ingestion:** Fetch recent Google News India defence and security RSS results, normalize and deduplicate them, and generate alerts. If the feed is unavailable or empty, ingestion uses bundled sample articles.
- **Threat scoring:** Assign a 0–100 score and severity using explainable keyword and source-count heuristics. This is not a trained threat classifier.
- **Ask Sentry and morning brief:** Retrieve relevant alerts using keyword, entity, location, and recency signals, then generate an answer or brief. Optional LLM providers are used where configured; extractive generation provides a fallback.
- **Faithfulness audit:** Check selected named entities and citations in the bundled summaries against the bundled articles. This check does not prove that summaries are factually correct or free of hallucinations.

## How it works

```text
Google News RSS ─┐
                 ├─> manual ingestion ─> scoring + summarization ─> alert storage
Sample articles ─┘                                      │                 │
                                               sources + timeline     dashboard / Ask Sentry
```

The feed supplies headlines and snippets; it does not necessarily supply full article text. Google News URLs may lead through its news service. Ingestion is **manual**, not scheduled. Sample articles can be used when live fetching fails, and the API reports which source was selected. A successful request alone does not prove articles were saved: check its `processed`, `skipped`, `failed`, and `errors` fields.

## Get started

### Requirements

- Node.js **24** and npm. The project scripts use Node's TypeScript stripping support.
- Internet access to install packages and fetch live news. The bundled sample dashboard can be used without a news connection once dependencies are installed.

### Install and run

```bash
git clone https://github.com/dhruvsharma-0001/sentry_intel.git
cd sentry_intel
npm install
npm run dev
```

Open [http://localhost:3000](http://localhost:3000). No API keys or cloud database are required for the local demo. Initially, the app reads the bundled sample alerts.

### Trigger ingestion

With the app running, use another terminal:

```bash
curl -X POST http://localhost:3000/api/ingest
```

The response identifies `source` as `live` or `fallback` and includes processing counts and per-article errors. Refresh the dashboard to load the latest stored alerts. The sample articles may already have alerts, so a fallback run can legitimately skip them.

### Optional configuration

Copy `.env.example` to `.env.local` and add only the values you need. Restart the server after changing environment variables. Never commit real keys.

| Variable | Purpose |
| --- | --- |
| `GROQ_API_KEY` | Optional Groq key for summaries and Ask Sentry. |
| `OPENAI_API_KEY` | Optional OpenAI key; used after Groq if available. |
| `NEXT_PUBLIC_SUPABASE_URL` | Supabase project URL, if using Supabase storage. |
| `SUPABASE_SERVICE_ROLE_KEY` | Server-only Supabase service-role key. Keep it secret. |

`GROK_API_KEY` is also accepted as a Groq key when it starts with `gsk_`. The other placeholders in `.env.example` are not required for this flow. Without an LLM key, the app uses extractive fallbacks.

## Storage options

**Local demo:** When neither the Supabase URL nor service-role key is configured, the app uses bundled sample data and writes changes to `data/local-db.json`. The file is ignored by Git. This is a single-machine, single-process demo store; it does not coordinate concurrent writers and is unsuitable for serverless or durable shared deployments.

**Supabase:** Configure both the URL and service-role key and apply the SQL in `supabase/migrations/20261029021500_init_sentry_intel.sql` to your project. Partial configuration is treated as an error. Supabase integration and the end-to-end live pipeline still need verification in the target environment. The Supabase schema stores a narrower alert representation than the local demo; the dashboard and Ask Sentry integration should be checked before relying on this mode.

## Commands

| Command | Purpose |
| --- | --- |
| `npm run dev` | Start the development server. |
| `npm run build` | Build the production app and check TypeScript. |
| `npm start` | Start the production server after building. |
| `npm run test:ingestion` | Run ingestion and scoring tests with mocked feed responses. |
| `npm run verify-faithfulness` | Check bundled summaries against bundled source data. |
| `npm run generate-summaries` | Regenerate cached summaries; review generated data changes before committing. |

## API

| Endpoint | Purpose |
| --- | --- |
| `GET /api/alerts` | List stored alerts. |
| `GET /api/alerts/:id` | Get an alert by ID. |
| `GET /api/stats` | Get alert counts by severity. |
| `POST /api/ingest` | Fetch news or use fallback articles, then create alerts. |
| `POST /api/ask` | Ask a question with `{ "question": "..." }`, or request a brief with `{ "action": "morning_brief" }`. |

These endpoints are intended for the demo and do not provide an authentication system. Secure them before exposing a deployment publicly.

## Project layout

```text
app/                    Dashboard and API routes
lib/ingest.ts           Google News RSS ingestion
lib/summarize.ts        Summaries, heuristic scoring, and alert creation
lib/ask.ts              Alert retrieval and question answering
lib/playbooks.ts        Suggested response actions
lib/local-db.ts         Local demo persistence
lib/supabase.ts         Storage interface and Supabase integration
data/                   Bundled sample articles and summaries
scripts/                Summary generation and faithfulness audit
supabase/migrations/    Supabase schema
tests/                  Ingestion and scoring tests
```

## Limitations and responsible use

- Live RSS access depends on Google News and network availability; the feed has not been verified in every deployment environment.
- Keyword-based scoring and retrieval can miss context, misclassify reports, or retrieve irrelevant results. Source counts do not independently verify agreement or source independence.
- LLM output can contain mistakes. The faithfulness audit checks a limited set of properties; it is not a guarantee of accuracy.
- Response suggestions are general guidance. Verify sources and follow instructions from relevant authorities.
- This project uses public information and illustrative samples. It is not a classified intelligence system or a replacement for analysts.

Built by **Last Minute Legends** for Zinnovatio 4.0.
