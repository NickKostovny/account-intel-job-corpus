# account-intel-job-corpus

Stage 2 of 5 in Invert's account-intelligence pipeline. Job postings as a corpus for reconstructing each account's internal org structure, and the collectors that build it. Upstream: [account-intel-core](https://github.com/NickKostovny/account-intel-core). Downstream: the Jev layers and the briefs. Canonical code: [account-intelligence](https://github.com/NickKostovny/account-intelligence).

## Why this stage exists

Job postings started as buying signals (20 rows in `signals.csv`). A call on 2026-07-23 corrected that. Postings are a context corpus: the richest public source for an org's internal structure, because titles on LinkedIn are self-authored and flattened while postings are org-authored and have to say where a role sits in order to hire for it. The requisition that motivated the build ("Director, Process Data Science and Statistics") had its team and parent org mashed into one prose field, its tools column empty although the posting named Snowflake and Streamlit, and a Glassdoor re-host as its only URL. So `job_posting` stopped being a signal type and postings got their own tables.

## Two tiers, two collectors

**Tier 1, deterministic, no model.** Requisition index: req id, city, role title, capture window, then job family, seniority and site by rules. Zero fabrication risk.

**Tier 2, LLM, filtered subset.** Body text to team name and acronym, parent org, teams supported, tech stack, vocabulary, hiring manager, each with the verbatim line it came from. Python owns the body bytes, so quote verification is an exact substring test.

**Collector A, Wayback CDX** (`jobs_cdx.py`). Works for Phenom-style careers sites with city in the URL. Capture dates are first-capture dates, not posted dates.

**Collector B, Workday CXS JSON API** (`jobs_workday.py`). Workday has no archive, so it shows only what is open now. Tenants cannot be guessed; `--discover` finds them from the curated company domain.

No collector exists yet for SuccessFactors (Astellas, Bayer, BioNTech, Boehringer, Novo Nordisk) or Eightfold (Alnylam, AstraZeneca's newer board).

## Files

| File | Role |
|---|---|
| `ats_probe.py` | careers host and requisition URL pattern per account, to `data/careers.csv`. Cached under `data/cache/ats/`, about a minute per account. |
| `ats_census.py`, `careers_probe.py` | which ATS each account runs, robots, sitemaps, JSON-LD. |
| `jobs_cdx.py` | Tier-1 collector A. `--all`, `--account <aid>`, `--rollup` (share of mix plus peer baselines), `--resolve-only`. |
| `jobs_workday.py` | Tier-1 collector B. `--all`, `--discover <aid>`. Upserts the careers row so both collectors coexist (`workday_host`, `workday_site` columns). |
| `fetch_bodies.py` | body text to `data/cache/bodies/<posting_id>.txt`. Uses `aiq.patch()` under the lock. |
| `gen_org_wf.py` | emits `wf/org-<aid>.wf.js` with bodies embedded as `const DATA` for the Workflow tool. |
| `verify_org.py` | the verbatim gate. Exact substring check of every quote against the body it claims. Quarantines failures. Exits 1 below 95 percent. |
| `merge_org.py` | to `posting_extracts.csv` (tier-monotonic upsert) and `posting_quotes.csv` (append only). Unescapes HTML once. |
| `build_org.py` | to `org_units.csv`, `org_edges.csv`, `org_site_coverage.csv`. Raises rather than write an edge without a verified quote. Detects reporting-line cycles. |
| `orgmap.py` | layout math (layered Reingold-Tilford) and SVG for the org map in the brief. |
| `mcp/org_intel_server.py` | nine MCP tools returning structured records only: `list_accounts, list_org_units, get_org_unit, list_org_edges, get_provenance, query_postings, get_hiring_mix, list_gaps, get_vocabulary`. |
| `skills/org-query/SKILL.md` | the `/org-query` skill: one requisition, one account top-up, one site, Tier-1 refresh, explain a gap, discover a host. Install at `.claude/skills/org-query/`. |
| `scheduled-tasks/ai-jobs-index-biweekly/SKILL.md` | the armed biweekly Tier-1 refresh (cron `0 6 1,15 * *`), pure Python, no agents. Install at `~/.claude/scheduled-tasks/`. |
| `aiq.py` | shared library, copy of the canonical one. See `SHARED.md`. |

## Tables

Column lists are in `TABLES.md`. IDs are content-derived: `{aid}--jr-{req_id}` for a posting, `{aid}--ou-{slug}` for a unit.

## Run

```bash
python3 ats_probe.py                       # careers hosts (network, cached)
python3 jobs_cdx.py --all                  # Tier-1 A
python3 jobs_workday.py --all              # Tier-1 B
python3 jobs_cdx.py --rollup               # share of mix + peer baselines
python3 fetch_bodies.py --account <aid>    # bodies
python3 gen_org_wf.py --account <aid> --pending --shard 8 --max-agents 6 --out wf/org-<aid>.wf.js
#   Workflow({scriptPath: ".../wf/org-<aid>.wf.js"}) -> save .result to ../org_out_<date>.json
python3 verify_org.py ../org_out_<date>.json    # GATE. exits 1 below 95%
python3 merge_org.py ../org_out_<date>.json
python3 build_org.py
```

Network steps need the sandbox off. Never run two writers at once. Before any run: `ps aux | grep -E "fetch_bodies|jobs_"`.

MCP registration: `claude mcp add org-intel -- python3 "<abs>/mcp/org_intel_server.py"`.

## Verified numbers (2026-07-29 build; the 2026-09-10 sweep widened the index)

| | |
|---|---|
| postings | 5,301 (AstraZeneca 5,097, Biogen 204) |
| AZ site resolution | 1,037 exact, 102 alias, 2 metro, 420 ambiguous (correctly unresolved), 3,517 city not in sites |
| AZ Tier-2 admitted | 160; bodies fetched 74 usable, 26 unavailable (about 50 percent archive yield) |
| Biogen | 68 site-resolved, 15 admitted, 15 of 15 bodies |
| quote verification | 403 of 403 exact, 0 quarantined, 7 unsupported claims dropped |
| org units | 21 observed plus 68 named but unmapped |
| org edges | 98: 14 reports_to, 63 supports, 21 located_at |
| rollup | 89 rows, 23 with a bias flag, 0 flagged rows missing a bias note |

Real recovered structure at AstraZeneca: Integrated Bioanalysis (IBA) under Clinical Pharmacology and Safety Sciences; Bioinformatics under Oncology Data Science; Statistical Programming under Vaccines and Immune Therapies; Oncology Strategy, Business Development & Alliances (SBDA) under Oncology R&D; ML/AI Ops under Evinova; Commercial Reporting & Analytics (CRA) under GBS Commercial Operations.

## Rules

- Never skip `verify_org.py`. It is the only thing between a fabricated team name and a sales email.
- Human judgment lives in `org_overrides.json`. Read it, add to it, never regenerate it.
- Never present a raw per-quarter count as hiring volume. CDX timestamps are first-capture dates; a crawl burst looks like a hiring burst (AZ 2024Q2 showed 304 data reqs vs 12 to 41 elsewhere). Use `share`, `self_index`, then `peer_index` (share divided by the median share across accounts in the same quarter). `bias_flag` and `bias_note` travel with every rollup row.
- No tool in the MCP server accepts a natural-language question and no response has a summary field. The rep team rejected "ask a question, get a paragraph, paste it into an email". The internal customer who raised that objection has not signed off on the server; it is built so the objection lands on the interface rather than the data.

## Bugs fixed here, do not reintroduce

1. City collisions resolved to the wrong site silently (AZ has two Cambridges and two Södertäljes; 77 collisions across active accounts). `resolve_site()` returns `ambiguous` and never guesses.
2. Workday city slugs carry a state suffix (`Cambridge-MA`). `slug_to_key()` strips it.
3. Subsidiaries double-counted (New Haven CT is both AstraZeneca and Alexion; `careers.astrazeneca.com` serves Alexion reqs). `mirror_for()` records `mirror_account_id` and `mirror_basis` (`dual_registered` or `subsidiary_only`).
4. Interns slipped through Tier-2 admission via an ORG_CORE family. `HARD_EXCLUDE` always wins; `SOFT_EXCLUDE` can be overridden by an ORG_CORE family.
5. Concurrent writes destroyed rows, twice. A background `fetch_bodies.py` checkpointed a stale snapshot over a fresh Biogen ingest and wiped 203 rows; after the lock was added the still-running old process did it again because Python does not reload a running module. Every writer now holds `aiq.lock()` across the whole read-modify-write, `fetch_bodies.py` uses `aiq.patch()`, and `aiq.write()` refuses to shrink a table by more than 2 percent.
6. HTML entities double-escaped (`Oncology R&amp;amp;D`). `merge_org.py` unescapes once on ingest.
7. Claim-to-quote matching was too strict and dropped 96 Biogen claims that had passed the gate. `backed()` falls back to containment; the quote must still be verified and name the subject. Drops fell to 7.
8. A place-name mismatch zeroed a site (Biogen's spine says Durham, NC; Workday returns Research Triangle Park, NC for 39 reqs). `PLACE_ALIASES` in `aiq.py`, reverse-indexed into the alias bucket so it never outranks a real city match. `data/unmatched_cities.csv` ranks what still does not match (`needs_alias`, `missing_site`, `no_place`). It surfaced Biogen's Baar, Switzerland (17 reqs), a real site absent from the 604-site spine.
9. A reporting-line cycle deleted teams from the map (two Biogen postings each named the other as parent). `build_forest()` breaks the weakest-evidenced link and marks the node "POSTINGS DISAGREE ON REPORTING LINE". A conflicting reporting line is a finding, not something to resolve quietly.
10. The Workflow `args` parameter arrives undefined for sizeable payloads. Embed as `const DATA`.

## Honest limits

- Posted dates for the bulk of the archive are unrecoverable; volume over time is share of mix, permanently.
- Full-text extraction over a whole archive is a category error (about 15M input tokens for AZ alone). Tier 1 exists to avoid it.
- `hiring_manager` stays near-empty; most postings name nobody. Whether a req was filled is not in this data.
- The motivating PDSS requisition is unrecoverable: Glassdoor 403s behind a CAPTCHA, zero Wayback snapshots. The extraction agent found org claims about it in search snippets and refused to record them, because a snippet is not a page it read. That refusal is the anti-fabrication contract working.

## Open

- Collectors for SuccessFactors and Eightfold.
- Tier-2 org extraction beyond AstraZeneca and Biogen. Budget note from the Jev work: Claude Tier-2 is about $0.27 per posting; Jev selection fields cost cents, so triage with Jev and send only relations (reports_to, parent_org, supports) to Claude.
- Biogen's open reqs skew commercial; its bioprocess reqs are scientist level and fall below the Tier-2 seniority bar. Lower it with `fetch_bodies.py --min-score` if org structure matters more than budget.
- Arm the monthly Tier-2 scheduled task, prompt-gated for its first two cycles.
- Accounts still at `probe_status=needs_discovery`: `python3 ats_probe.py --report`.

## Data

Not in this repo. `data/`, `wf/` and `data/cache/` stay on Nick's Mac or travel as a zip.
