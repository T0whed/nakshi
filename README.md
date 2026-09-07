# Nakshi · নকশি

**Digital Product Passports for Bangladesh's garment exports.**

Nakshi reads the documents a garment factory already produces — purchase orders, mill
invoices, transaction certificates, utility bills, payroll sheets, in Bangla and English —
and assembles them into a verified EU Digital Product Passport for every order. Along the
way it runs a reconciliation engine that tests each claim against the evidence meant to
back it, and catches the ones that don't add up before the goods ship.

**Live demo → https://t0whed.github.io/nakshi/**

> A *nakshi kantha* records a story in its stitches: who made it, from what, over how long.
> A garment leaving Chattogram should be able to do the same.


---

## The problem

Ready-made garments are **80.6% of Bangladesh's exports** — Tk 4.76 lakh crore
(US$38.7bn) in FY2025-26 — and roughly half of that goes to the EU. Three deadlines are
closing on the same factory floor:

| When | What |
|---|---|
| 24 Nov 2026 | LDC graduation (3-year deferral backed by ECOSOC, awaiting a UNGA vote) |
| Late 2027 | EU ESPR textile delegated act — defines the passport's mandatory fields |
| ~2029 | Passport obligation begins, roughly 18 months after adoption |
| End 2029 | EBA duty-free access ends; apparel drops to 9.6% GSP or full MFN |

Two things follow. First, competing on price stops working. Second, the EU will require a
passport containing material composition, supplier identity, processing sites, energy
figures and chemical-management records — **none of which lives in a European brand's ERP.**
It lives in a Bangladeshi factory's filing cabinet, which is why the cost and the liability
get pushed down the chain to the supplier.

Meanwhile the same factory already maintains amfori BSCI, Sedex SMETA, Higg FEM, SLCP,
WRAP and a stack of bespoke buyer questionnaires. Identical facts, different forms, re-keyed
by hand **fifteen or more times a year**.

So the data a passport needs is already being collected. It is just being collected badly,
redundantly, and in a form no machine can read.

## The approach

No factory should have to adopt a new data discipline. The documents already exist, so the
engine goes to them.

```
  scans, photos,        ┌──────────┐   ┌──────────┐   ┌─────────────┐   ┌────────┐
  PDFs, spreadsheets ──▶│  Ingest  │──▶│ Extract  │──▶│  Reconcile  │──▶│  Emit  │
  (Bangla + English)    └──────────┘   └──────────┘   └─────────────┘   └────────┘
                         layout-aware   VLM + per-     BOM graph +       JSON-LD +
                         OCR            field conf.    claim testing     GS1 QR
```

**Ingest** — whatever the factory has: scanned certificates, phone photos of paper bills,
buyer PDFs, mill spreadsheets. Layout-aware OCR handling Bangla, English and Bangla numerals.

**Extract** — a vision-language model pulls typed fields with a confidence score on each.
Below 0.85 routes to a human, so review effort lands where the model is actually unsure.

**Reconcile** — the part that matters. Build the bill-of-materials graph (garment →
component → material → supplier → certificate), then test every claim against the evidence
supposed to support it.

**Emit** — a passport record with a GS1 Digital Link QR, plus auto-filled Higg FEM / SLCP /
SMETA responses from the same single source of truth.

Extraction is becoming a commodity. Reconciliation is not — knowing which *combinations* of
facts constitute a defect is domain knowledge, and it's what a vendor in Brussels can't write.

## The checks

| Check | Tests | Why it stops a shipment |
|---|---|---|
| Certificate validity | Validity windows against the on-board date | A label claim is unsupported the day its certificate lapses |
| Mass balance | Claimed recycled/organic content vs. certified volume | The gap is an unsubstantiated environmental claim |
| Consumption | Fabric invoiced vs. order qty × weight × wastage | A large gap means over-invoicing or parallel production |
| Site disclosure | Sites resolved from documents vs. approved-facility list | Undisclosed subcontracting loses orders outright |
| Tier depth | How far up the chain the documents reach | Stopping at Tier 2 blocks the passport and due diligence |
| Field completeness | Populated fields vs. anticipated ESPR set | Missing fields can't be back-filled after shipping |
| Energy intensity | Metered kWh ÷ units produced | A verified number sells; an estimated one is a liability |

## Running it

No build step, no dependencies, no server.

```bash
git clone https://github.com/<T0whed>/nakshi.git
cd nakshi
open index.html          # macOS   (Linux: xdg-open · Windows: start)
```

Or serve it, if you'd rather:

```bash
python3 -m http.server 8000   # → http://localhost:8000
```

## Try this

The sample order deliberately opens **failing, at 26/100, with three blockers.** A demo that
opens green teaches nothing. Then:

1. Push **GRS certificate expiry** past the ship date → the first blocker clears.
2. Raise **rPET covered by GRS certificate** to `5040` *or* lower the recycled claim to `32` →
   the mass-balance failure resolves. Either fix is legitimate and the engine accepts both.
3. Tick both chain checkboxes → the passport reaches 14/14 fields, score 100, *Export-ready*.
4. Now raise **order quantity** to `60000` without touching the fabric invoice → the
   consumption check catches the discrepancy.

## How the code is organised

Everything is `index.html` — markup, styles and logic in one file, deliberately, so it runs
from a file:// URL with nothing installed.

| What | Where |
|---|---|
| The engine | `runChecks(s)` — each check pushes one `{k, t, m, w}` object onto `r` |
| Scoring | `score(rs)` — `fail = -20`, `warn = -7`, floor at 0 |
| Regulatory countdowns | the `MILESTONES` array |
| Sample order defaults | `value="..."` on the inputs in the *Extracted values* panel |
| Rendering | `render()` — reads state, runs checks, repaints everything |

Adding an eighth check is about ten lines:

```js
// inside runChecks(s), before the return
const ratio = s.grsKg / (s.fabKg || 1);
r.push(ratio <= 1 ? {
  k: "pass",
  t: "Certified volume within fabric total",
  m: "Certified rPET does not exceed the fabric it is claimed against.",
  w: `certified <b>${fmt(s.grsKg)} kg</b> ≤ fabric <b>${fmt(s.fabKg)} kg</b>`
} : {
  k: "fail",
  t: "Certified volume exceeds total fabric",
  m: "The certificate covers more material than the order contains — a transcription "
   + "error, or a certificate being reused across orders.",
  w: `excess <b>${fmt(s.grsKg - s.fabKg)} kg</b>`
});
```

The score, the tally, the passport record and the consumer view all update from that
automatically. `k` is `"pass"`, `"warn"` or `"fail"`; `t` is the title, `m` the explanation,
`w` the evidence line (HTML permitted).

## Status, and what's actually real

This is a demo built to prove the reconciliation concept, not a production system. Being
precise about the line:

**Genuinely running** — the engine and all seven checks, the mass-balance and consumption
arithmetic, certificate-validity logic, readiness scoring, the supply-chain graph, the
passport record, the framework field mapping, the GS1 QR, the regulatory countdowns.
Everything recomputes live from the inputs.

**Sample data** — the six source documents are a synthetic order. Meghna Apparels, Shitalpur
Textile Mills, Rupganj Dyeing and Nordlys Retail are invented companies; no real firm's data
appears anywhere. The extraction stage shows the pipeline's *output format* against that
fixed sample rather than running a model in the page.

**Not built yet** — the OCR/VLM pipeline, persistence, multi-order state, auth, the buyer
portal, the actual questionnaire exports.

## Roadmap

- **Q4 2026** — three pilot factories in Gazipur and Narayanganj. One metric: compliance-officer
  hours per buyer audit, measured before and after.
- **2027** — Bangla document corpus at scale, supplier graph across shared mills, BGMEA
  partnership. Fifty paying factories on audit-prep value alone.
- **Late 2027** — the delegated act publishes and fixes the field set. Because the graph is
  already built, conformance is a mapping exercise, not a data-collection project. This
  timing is the bet.
- **2028–29** — buyer-side product; expansion to Vietnam, India, Türkiye, Cambodia.

## Sources

- [BGMEA — Export Performance](https://www.bgmea.com.bd/page/Export_Performance) — export values and share of total exports
- [The Daily Star — EU GSP rules and Bangladesh's RMG](https://www.thedailystar.net/business/economy/news/eu-gsp-rules-and-bangladeshs-rmg-4218431) — post-graduation tariff scenarios
- [The Daily Star — LDC graduation deferral awaits UNGA nod](https://www.thedailystar.net/news/bangladesh/news/ldc-graduation-deferral-awaits-unga-nod-4230371) — graduation date and ECOSOC deferral
- [DPP Timeline 2026–2030](https://passportcraft.com/insights/dpp-timeline-2026-2030-every-deadline) — delegated act and obligation dates
- [Intertek — EU Ecodesign & Digital Product Passport](https://www.intertek.com/blog/2025/05-28-eu-ecodesign-digital-product-passport/) — ESPR scope and sequencing

## Context

Built for the SUST selection for the National AI Business Summit 2026, Dhaka.

## Contact

[Towheduzzaman] · <towheduzzamantowhed@gmail.com> · Shahjalal University of Science and Technology, Sylhet

## Licence

MIT — see [LICENSE](LICENSE).
