<div align="center">

# 🧠 AI Stack Auditor

### Multi-agent AI spend rationalization for startups & SMBs

*Discover, analyze, and optimize your company's AI software spend — automatically.*

[![Live Demo](https://img.shields.io/badge/demo-live-22D97A?style=for-the-badge)](https://ai-stack-auditor-koar.vercel.app/)
[![Watch Video](https://img.shields.io/badge/watch-demo_video-red?style=for-the-badge&logo=youtube)](https://youtu.be/bw-0zxUH4EY)
[![Built for Kaggle](https://img.shields.io/badge/Kaggle-AI_Agents_Capstone-20BEFF?style=for-the-badge&logo=kaggle)](https://www.kaggle.com/competitions/vibecoding-agents-capstone-project)

</div>

---

## The Problem

Companies are adopting AI software faster than they can govern it. Writing assistants,
coding copilots, meeting transcription tools, design generators — teams pick these up
independently, with no procurement process and no centralized visibility.

The result: **overlapping subscriptions** serving the same job under different brand
names, **hidden AI add-ons** quietly bundled into much larger SaaS bills, **underused
premium tiers**, and **renewals nobody's watching.**

Generic SaaS spend-management tools track *cost*. Almost none of them understand that
Jasper and Copy.ai serve the same job, or that a company might already be paying for
something elsewhere that makes a separate subscription redundant. **AI Stack Auditor is
built specifically to close that gap.**

---

## What It Does

Upload a company's invoices, spend CSVs, or a manually typed tool list, and a pipeline of
seven specialized agents works through it step by step — visualized live, not hidden
behind a single black-box answer:

| Agent | Role |
|---|---|
| 🔍 **Discovery** | Finds every AI tool and hidden AI add-on across uploaded invoices/CSVs |
| 🏷️ **Job-Mapping** | Classifies each tool by the business function it actually serves |
| ⚠️ **Waste Detection** | Flags duplicates, underused seats, overpriced tiers, renewal risk |
| 📈 **ROI Intelligence** | Scores business value vs. cost — so expensive-but-justified tools aren't flagged as waste |
| 💡 **Alternative Recommendation** | Proposes a leaner stack, cross-checked against ROI before ever suggesting a cancellation |
| 🧪 **Stack Simulator** | Sandbox to test hypothetical changes before committing to anything |
| ✅ **Action** | Compiles a manager-ready savings report — drafts only, nothing auto-executed |

**Every recommendation requires explicit human approval.** Nothing gets cancelled,
downgraded, or modified automatically — the system informs a decision, it doesn't make one.

---

## Why This Is Different

| | Generic SaaS spend tools | AI Stack Auditor |
|---|:---:|:---:|
| Tracks cost | ✅ | ✅ |
| Understands AI task-level functional overlap | ❌ | ✅ |
| Detects hidden AI add-ons inside larger SaaS bills | ❌ | ✅ |
| Weighs cost against business value (ROI), not cost alone | ❌ | ✅ |
| Lets users simulate changes before committing | ❌ | ✅ |
| Transparent, multi-agent reasoning (not a black box) | ❌ | ✅ |

---


## Architecture


```
┌───────────────────────┐      ┌──────────────────────────┐
│  React Frontend         │────▶│  FastAPI Agent Service     │
│  Tailwind + Framer Motion│◀────│  Discovery → Job-Mapping →  │
│  Dark/Light mode          │      │  Waste Detection ⇉ ROI →    │
└───────────┬─────────────┘      │  Recommendation → Action    │
            │                     └────────────┬─────────────┘
            ▼                                   ▼
   ┌──────────────────┐              ┌───────────────────────┐
   │  Supabase Auth      │              │  Gemini LLM Layer        │
   │  (sign up/in)        │              │  structured JSON output  │
   └──────────────────┘              └───────────────────────┘
            │
            ▼
   ┌──────────────────────────────────────────────────┐
   │  Supabase Postgres (Row-Level Security)              │
   │  businesses · tools · findings · roi_scores ·        │
   │  recommendations · reports                            │
   └──────────────────────────────────────────────────┘
```

Waste Detection and ROI Intelligence run **in parallel**, not sequentially — both consume
the same tool list independently, so a tool's cost and business value are always assessed
together rather than one gating the other.

---

## Tech Stack

**Frontend** — React · Tailwind CSS · Framer Motion
**Backend** — FastAPI (Python), orchestrating the agent pipeline
**LLM** — Gemini, with structured JSON output validation + retry-on-parse-failure
**Database & Auth** — Supabase (Postgres + built-in auth + Row-Level Security)
**File Parsing** — pandas, pdfplumber
**PDF Export** — jsPDF + html2canvas
**Deployment** — Vercel (frontend) · Render (backend)

---

## Getting Started

### Prerequisites
- Node.js 18+
- Python 3.10+
- A free [Supabase](https://supabase.com) project
- A free [Gemini API key](https://aistudio.google.com/apikey)

### 1. Clone the repo
```bash
git clone https://github.com/YOUR_USERNAME/ai-stack-auditor.git
cd ai-stack-auditor
```

### 2. Set up Supabase
Run the schema in `backend/supabase_schema.sql` inside your Supabase project's SQL Editor.
This creates `businesses`, `tools`, `findings`, `roi_scores`, `recommendations`, and
`reports` — all with Row-Level Security enabled, scoped to each authenticated user.

### 3. Configure environment variables

**`frontend/.env`**
```
VITE_SUPABASE_URL=https://xxxxxxxxxxxx.supabase.co
VITE_SUPABASE_ANON_KEY=your_publishable_or_anon_key
```

**`backend/.env`**
```
SUPABASE_URL=https://xxxxxxxxxxxx.supabase.co
SUPABASE_SERVICE_ROLE_KEY=your_secret_or_service_role_key
GEMINI_API_KEY=your_gemini_api_key
```

> ⚠️ Never commit `.env` files. The service role key bypasses Row-Level Security — it must
> stay server-side only.

### 4. Install & run

```bash
# Backend
cd backend
pip install -r requirements.txt --break-system-packages
uvicorn main:app --reload

# Frontend (in a separate terminal)
cd frontend
npm install
npm run dev
```

Visit `http://localhost:5173` — or click **"Try Demo (no account)"** to explore instantly
without setting up auth.

---

## Project Structure

```
ai-stack-auditor/
├── frontend/
│   ├── src/
│   │   ├── components/       # UI screens & the Agent Trace Panel
│   │   ├── lib/               # Supabase client
│   │   └── App.jsx
│   └── tailwind.config.js     # centralized color tokens for dark/light theming
├── backend/
│   ├── agents/                 # all 7 agents, one file each
│   ├── orchestrator.py         # pipeline coordination (parallel where possible)
│   ├── schemas.py              # Pydantic structured-output schemas
│   ├── knowledge_base/         # curated AI tool → category → price reference data
│   └── main.py
├── data/
│   └── demo_dataset.csv
└── docs/                       # PRD, TRD, and build documentation
```

---

## Design Principles

- **Every recommendation is a draft.** The system informs, it never auto-executes.
- **Every finding is traceable** — back to the source data, the agent that produced it,
  and a confidence score. No bare assertions.
- **Usage data is honestly labeled as estimated**, not presented as verified telemetry,
  since this MVP doesn't integrate live SSO/usage APIs.
- **Demo mode never touches the database** — the product can always be shown working,
  with zero live dependencies.

---

## Roadmap

- [ ] Direct SaaS integrations (no manual upload required)
- [ ] Real usage telemetry via SSO/API
- [ ] Renewal forecasting & policy guardrails
- [ ] Peer benchmarking against similar companies
- [ ] Prompt Optimization Agent — LLM token/cost efficiency *(different data source, different buyer — deliberately out of MVP scope)*
- [ ] AI Router Agent — cheapest-suitable-model recommendation *(same reasoning as above)*

---

## Built For

[Kaggle — AI Agents: Intensive Vibe Coding Capstone Project](https://www.kaggle.com/competitions/vibecoding-agents-capstone-project)
Track: **AI for Business**

---

## License

MIT — see [LICENSE](LICENSE) for details.

---

<div align="center">
<sub>Built with agents that reason transparently, act only with approval, and know the
difference between "expensive" and "wasteful."</sub>
</div>
