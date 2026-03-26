# Report121

**AI-powered sales intelligence platform for B2B teams.**

Report121 generates personalized company assessments that help salespeople start conversations with prospects by sharing relevant insights — before any commercial contact.

Instead of cold outreach, the seller approaches the lead already equipped with a data-driven report about their business.

> The assessment doesn't close the sale — it opens the right door.

🔗 **Live product:** [report121.com](https://report121.com)

---

## Demo

[![Watch the demo](https://img.youtube.com/vi/gA581Yu5lX4/maxresdefault.jpg)](https://youtu.be/gA581Yu5lX4)

---

## How It Works

1. The **Client** (paying company) accesses the dashboard and inputs a target company.
2. The system collects public data via APIs (website content, news, CNPJ registry, blog, YouTube).
3. An AI analyzes the data and generates a report with scores, strengths, improvement areas, and recommendations.
4. The seller discovers decision-makers (via lead discovery APIs) and shares the report link.
5. The prospect receives a personalized assessment that demonstrates value before any sales conversation.

---

## Assessment Types

### Digital Readiness (Prontidão Digital)
Analyzes the digital signals a company transmits to potential clients who research it before accepting a meeting.

**Four signals evaluated:**
| Signal | Weight | What it measures |
|--------|--------|-----------------|
| Value Proposition Clarity | 40% | Can a visitor understand in 10 seconds what the company does, for whom, and what problem it solves? |
| Case Studies | 20% | Does the company show concrete proof of results with real numbers and testimonials? |
| Client Portfolio | 20% | Are client logos, names, and sectors visible and recognizable? |
| External Perception | 20% | What appears when someone Googles the company? Authority, content, media presence? |

### Marketing Digital
Analyzes the company's educational content maturity across blog and YouTube Shorts.

### Tax Reform Risk (Reforma Tributária)
Analyzes the impact of Brazil's tax reform (IVA: IBS/CBS) on a specific company using public registry data.

---

## Architecture

```
Dashboard → AssessmentType (JSON config) → GenericPipeline
                                               ├── Collectors (site_content, web_search, blog, youtube, cnpj, trigger_events)
                                               ├── Analyzer (Claude or ChatGPT)
                                               └── Generic template (HTML)
```

**Key design decision:** Creating a new assessment type requires only filling out a JSON config in the Admin panel. No code changes. No deployment.

### GenericPipeline Flow

```
1. Read config from AssessmentType
2. collect_data() → runs each collector defined in config
3. analyze() → sends collected data to AI
4. calculate_scores() → computes scores (weighted_sum or ai_scored)
5. generate_report_data() → assembles data for the template
```

### Collector Agents

| Agent | What it does | Integration |
|-------|-------------|-------------|
| `collect_site_content` | Extracts text from website pages | Exa |
| `collect_web_search` | Searches Google for mentions and news | Serper |
| `collect_blog` | Finds and analyzes blog content | Serper + Exa |
| `collect_youtube` | Extracts YouTube Shorts data | Selenium |
| `collect_cnpj` | Retrieves company registry data (Brazil) | Brasil API |
| `collect_trigger_events` | Finds recent news about the company or industry | Serper + Exa + Claude |

### Trigger Events

The system searches for recent news (last 90 days) about the target company. If nothing is found, it falls back to industry-level news. This provides salespeople with a timely reason to reach out — not just a generic report.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | Django 5.x |
| Async Tasks | Celery + Redis |
| Database | PostgreSQL |
| Hosting | AWS Lightsail |
| File Storage | AWS S3 |
| Frontend | Django Templates + HTMX |
| AI | Claude (Anthropic) + ChatGPT (OpenAI) |
| Data APIs | Serper, Exa, Snovio, Hunter.io, Brasil API |
| PDF Generation | WeasyPrint |
| Monitoring | Sentry |
| SSL | Let's Encrypt (Certbot) |

---

## Project Structure

```
report121/
├── config/                      # Django settings (base/dev/prod)
├── apps/
│   ├── accounts/                # Users and authentication
│   ├── clients/                 # Paying clients and plans
│   ├── companies/               # Target companies
│   ├── leads/                   # Decision-makers / contacts
│   ├── assessments/             # Core: types, executions, pipelines
│   ├── reports/                 # Public reports (HTML)
│   ├── integrations/            # External API clients
│   ├── dashboard/               # Client web interface
│   ├── discovery/               # Lead discovery (multi-API waterfall)
│   └── agents/                  # Collection and analysis agents
│       ├── collectors/          # BlogCollector, YouTubeCollector, etc.
│       └── analyzers/           # AIAnalyzerAgent
├── templates/
├── static/
└── docs/
```

---

## Data Model

```
Plan ──1:N──► Client ──1:N──► User
                │
                ├──1:N──► Company ──1:N──► Lead
                │              │
                └──1:N──► AssessmentType
                               │
                               └──1:N──► Assessment ──1:N──► Execution
                                              │
                                              └──1:1──► Report ──1:N──► ReportView
```

---

## Key Technical Decisions

| Decision | Rationale |
|----------|-----------|
| Django over FastAPI | Needed admin panel, ORM, and templates — not just an API |
| Templates + HTMX over React/Vue | Simpler for MVP, enough interactivity, less complexity |
| PostgreSQL over MySQL | Better JSON support (native JSONField), Python ecosystem standard |
| GenericPipeline over per-type pipelines | One pipeline serves all assessment types via JSON config |
| Selenium for YouTube | YouTube has no public API for Shorts data |
| Claude + ChatGPT (configurable) | Claude for analysis, ChatGPT for baselines — selectable per assessment |
| Multi-API lead discovery | Waterfall enrichment (Hunter + Snovio + Skrapp) with cross-reference scoring |

---

## About

Built by [Cassio Politi](https://www.linkedin.com/in/cassiopoliti/) — AI-Powered SaaS Builder.

Report121 is a product by [Tracto](https://www.tracto.com.br/).

---

## Status

✅ In production at [report121.com](https://report121.com)
