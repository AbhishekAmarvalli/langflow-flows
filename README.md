# Langflow flows

Two importable Langflow flow files in `flows/`. Each is a complete graph — import, set a model key, run.

```
Chat Input ──► Agent ──► Chat Output
                 ▲   ▲
      URL tool ──┘   └── Web Search tool (+ Calculator / Current Date inside the Agent)
```

| File | What it does |
| --- | --- |
| `flows/paid-internship-scout.json` | Screens internships for legitimacy and real pay: URL or web search in, verdict + quoted red flags + tracker JSON out |
| `flows/ai-research-brief-agent.json` | Reads a page and writes a structured research brief |

---

## 1. Paid Internship Scout

`flows/paid-internship-scout.json`

Built around one requirement: **no internship that charges you money, and no "paid" internship that doesn't actually pay.**

### Import
Langflow → **Projects** → **Upload a flow** → pick the JSON (or drag it onto the window).

### Setup (one step)
Agent node → **Language Model** → provider → model → API key.

**Recommended:** provider **Google Generative AI**, model a Gemini **Flash** (2.5 Flash or whatever Flash appears in your list), key from [aistudio.google.com](https://aistudio.google.com/apikey) — free tier, no card. OpenAI or Anthropic work equally well if you already have those keys.

The **web search tool needs no key** (it uses DuckDuckGo / Google News / RSS), so this flow has exactly one credential.

### Two ways to use it
1. **Paste a posting URL** into the Playground → it fetches that page and judges it.
2. **Type a request** like `paid remote backend internship` → it searches, fetches the top hits, and screens them in a table.

### The rules it enforces
Configured for: **tech field, remote worldwide, paid monthly by the company with an offer letter/contract.**

**Auto-SCAM** (score ≤ 25) when the posting shows any of:
- a fee of any kind from you — registration, training, kit, certificate, "security deposit"
- payment to a personal UPI/bank account or phone number
- no verifiable company footprint (no domain, no LinkedIn page, no findable staff)
- contact only via Gmail/Outlook/WhatsApp
- "guaranteed placement" or income promises
- Aadhaar/PAN/originals before an offer letter
- implausibly high pay for a vague role

**Paid bar** — all four needed for LEGITIMATE:
1. pay comes from the employing company, not a third-party "partner"
2. monthly pay backed by an offer letter / internship agreement / contract ("stipend discussed later" fails)
3. non-zero; amount may be negotiable
4. a real role with a stack or deliverables, a team or mentor, remote/onsite stated

**Scoring:** start at 100, subtract per signal (fee −40, no footprint −25, Gmail-only −20, guarantee claims −25, ID/bank data −25, vague-rich-role −20, pay unstated −15, unpaid trial −15, stale posting −10). **80–100 LEGITIMATE · 40–79 NEEDS CHECK · under 40 SCAM.**

Not auto-SCAM, deliberately: an unpaid probation period, or pay tied to performance with no base — those land on NEEDS CHECK with the reason spelled out.

### Output you get
1. **Verdict** — LEGITIMATE / NEEDS CHECK / SCAM, score /100, the deciding evidence
2. **Postings screened** — table: Company | Role | Pay | Remote | Verdict | Score (linked to the posting)
3. **Red flags** — each with the *exact quoted line* from the posting (≤160 chars); fabricated quotes are forbidden by the prompt
4. **Ask before applying** — 3–6 questions tailored to that posting
5. **Application message** — only for postings that clear the bar
6. **Tracker JSON** — array with `company, role, url, pay_amount, pay_currency, pay_frequency, contract_documented, remote, contact, verdict, score, red_flags`

### Customizing
Everything routes through the **Candidate profile** block at the top of the Agent's `Agent Instructions`: field, cities vs remote, seniority, and a pay floor. Add `At least 20,000 INR/month` there and the scoring follows. The rest of the prompt (the disqualifier list, the scoring table, the output shape) is plain prose you can edit in the same field.

---

## 2. AI Research Brief Agent

`flows/ai-research-brief-agent.json`

Same wiring, different job: paste a question or URL, get a fixed-shape brief (TL;DR, key findings, open questions, sources). Setup is identical — one model key. Useful if you want the simpler of the two graphs to start from, or want research rather than screening.

---

## Notes on how these were built

- Both flows were generated from Langflow's official **Simple Agent** starter export, so node payloads, output names, input types and edge handles are byte-identical to what Langflow itself writes (1.13.x schema). The ~200 KB of embedded per-node component source is stripped; built-in components resolve from Langflow's registry on load.
- Tool wiring uses the *Toolset* output (`component_as_tool`) into the Agent's `Tools` input, so URL and Web Search act as callable tools (and both can feed the same `Tools` handle).
- **Verified:** JSON validity, node/edge integrity, every edge's source output name + `output_types` and target field name + `input_types`, single tool-handle fan-in of 2, prompt section tokens present, and canvas layout without overlapping nodes.
- **Not verified:** neither flow has been executed against a live Langflow instance (no Python/Langflow in this environment). Behaviour depends on the model you select — the screening prompt assumes a model that follows long instructions well, so a small local model will be less reliable at the quoted-evidence rule.
