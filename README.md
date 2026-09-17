# Scheme Navigator

A full-stack prototype for helping applicants find relevant concessional credit schemes and understand what to do next.

**Live demo:** https://scheme-navigator-teal.vercel.app/



---

## What it does

Scheme Navigator takes a small set of applicant details and walks through a practical application flow:

1. Enter income, project/course cost, and education status.
2. Match the applicant against the current scheme rules.
3. Show the matched scheme and the reason for the result.
4. Estimate loan amount and EMI.
5. Generate a document checklist.
6. Show relevant channel partners and distance from a demo location.
7. Let an admin update a scheme rule without deleting the previous version.
8. Re-check saved applicants after a policy change and identify eligibility changes.

The main idea behind the project is that **the rules engine makes the eligibility decision; AI is only used to explain an already-made decision in plain language.**

---

## Why I built it

Government and concessional-credit schemes can be difficult to navigate when eligibility rules, documentation, and application channels are spread across different sources.

Instead of treating the problem as a generic chatbot, this prototype focuses on a more structured workflow: keep the scheme rules in a database, evaluate them deterministically, preserve rule versions, and make policy changes traceable.

---

## Tech stack

**Frontend**
- React
- Vite
- CSS

**Backend**
- Node.js
- Express

**Database**
- Supabase / PostgreSQL

**Optional explanation layer**
- Groq API
- Falls back to a local template when no API key is configured

---

## Architecture

```text
React + Vite
     |
     | /api
     v
Node.js + Express
     |
     +--> Rules Engine ----------> scheme_rules
     |
     +--> Recommendation --------> applicants
     |
     +--> Calculator
     |
     +--> Partner Locator -------> channel_partners
     |
     +--> Admin / Policy Scanner
     |
     +--> Optional Groq explanation
```

The important boundary is:

```text
RULES DECIDE
AI EXPLAINS
```

The recommendation itself does not depend on an LLM.

---

## Project structure

```text
scheme-navigator/
├── backend/
│   ├── lib/
│   │   ├── explain.js
│   │   ├── rulesEngine.js
│   │   └── supabase.js
│   ├── routes/
│   │   ├── admin.js
│   │   ├── calculate.js
│   │   ├── partners.js
│   │   └── recommend.js
│   ├── schema.sql
│   ├── server.js
│   ├── package.json
│   └── .env.example
│
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   ├── App.jsx
│   │   ├── api.js
│   │   ├── main.jsx
│   │   └── styles.css
│   ├── index.html
│   ├── package.json
│   └── vite.config.js
│
├── .gitignore
└── README.md
```

---

## Run locally

### 1. Clone the repository

```bash
git clone <your-repository-url>
cd scheme-navigator
```

### 2. Set up Supabase

Create a Supabase project and run:

```text
backend/schema.sql
```

in the Supabase SQL Editor.

The SQL creates the tables and inserts demo data for schemes, partners, and applicants.

### 3. Start the backend

```bash
cd backend
npm install
cp .env.example .env
```

Add your Supabase values to `.env`:

```env
SUPABASE_URL=your-project-url
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key
GROQ_API_KEY=
PORT=4000
```

Then run:

```bash
npm start
```

The API runs at:

```text
http://localhost:4000
```

Health check:

```text
http://localhost:4000/api/health
```

### 4. Start the frontend

Open a second terminal:

```bash
cd frontend
npm install
npm run dev
```

The frontend runs at:

```text
http://localhost:5173
```

Vite proxies `/api` requests to the backend.

---

## API routes

| Method | Route | Purpose |
|---|---|---|
| GET | `/api/health` | API health check |
| POST | `/api/recommend` | Match an applicant to current scheme rules |
| POST | `/api/calculate` | Calculate loan amount and estimated EMI |
| GET | `/api/partners` | Find relevant channel partners |
| GET | `/api/admin/schemes` | Read current scheme rules |
| POST | `/api/admin/schemes/:schemeId/new-version` | Create a new rule version |
| POST | `/api/admin/scan` | Re-check saved applicants after policy changes |

---

## Versioned policy rules

Scheme rules are stored with a version number instead of simply overwriting the current value.

For example:

```text
Term Loan — v1
Income cap: ₹5,00,000

        ↓ policy update

Term Loan — v2
Income cap: ₹6,00,000
```

The previous version remains available. The admin scanner can then compare an applicant's stored eligibility with the result under the current rules.

Possible scanner results are:

- `newly_eligible`
- `no_longer_eligible`
- `no_change`

---

## What is implemented vs. demo data

I want this distinction to be clear rather than presenting prototype data as a production integration.

| Area | Current status |
|---|---|
| Rules engine | Implemented and deterministic |
| Supabase persistence | Implemented |
| Rule versioning | Implemented |
| Policy-change scanner | Implemented |
| EMI calculation | Implemented using standard amortization math |
| AI explanation | Optional; Groq call with local fallback |
| Partner filtering | Implemented |
| Partner NPA / fund utilization | **Demo data** |
| Partner coordinates | **Demo coordinates** |
| Live government scheme feed | **Not connected yet** |
| Authentication / admin authorization | **Not implemented yet** |

The project should be treated as a working prototype rather than a production government-service integration.

---

## A few current limitations

- `educationStatus` is collected and stored but is not yet part of the core eligibility calculation.
- The what-if slider currently uses the same recommendation endpoint as a normal submission, so preview requests can create applicant records.
- The calculator currently assumes a 36-month repayment period for the estimate.
- Partner NPA and fund-utilization values are seeded demo values.
- Partner distance is calculated using demo coordinates rather than a live maps/location service.
- Admin routes currently have no authentication layer.

These are known prototype limitations, not hidden assumptions.

---

## Demo flow

### Applicant

```text
Applicant details
      ↓
Scheme recommendation
      ↓
Financial estimate
      ↓
Document checklist
      ↓
Channel partners
```

### Admin

```text
Current scheme rule
      ↓
Create new version
      ↓
Keep old version
      ↓
Re-run saved applicants
      ↓
See who changed eligibility
```

---

## Environment variables

Never commit real credentials.

Required for the backend:

```env
SUPABASE_URL=
SUPABASE_SERVICE_ROLE_KEY=
```

Optional:

```env
GROQ_API_KEY=
```

The repository contains `.env.example`, while `.env` is ignored by Git.

---

## Notes for reviewers

The most important code paths to look at are:

- `backend/lib/rulesEngine.js` — deterministic matching logic
- `backend/routes/recommend.js` — applicant recommendation flow
- `backend/routes/admin.js` — versioning and policy-change scanner
- `backend/schema.sql` — database model and demo seed data
- `frontend/src/App.jsx` — application flow
- `frontend/src/components/AdminPanel.jsx` — policy management UI

The project is intentionally small enough to trace from the UI to the database without a large framework layer in between.

---

## License

MIT. See `LICENSE`.
