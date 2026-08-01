<div align="center">

# 📈 Finance Filing Triage

**An AI agent that opens EDGAR, finds a filer's annual reports, and reports what was filed and when — then films itself doing it.**

[![CI](https://github.com/coasty-ai/coasty-finance-filing-triage/actions/workflows/ci.yml/badge.svg)](https://github.com/coasty-ai/coasty-finance-filing-triage/actions/workflows/ci.yml)
[![Node](https://img.shields.io/badge/node-%E2%89%A520.11-339933?logo=node.js&logoColor=white)](https://nodejs.org)
[![Dependencies](https://img.shields.io/badge/dependencies-0-brightgreen)](package.json)
[![Runs offline](https://img.shields.io/badge/runs%20offline-%240.00-blue)](#try-it-in-30-seconds)
[![License](https://img.shields.io/badge/license-MIT-black)](LICENSE)

<img src="media/demo.gif" alt="Demo clip rendered by the bundled offline mock through the frame-capture pipeline" width="820">

<sub>The clip above is rendered by the **bundled offline mock**, so a fresh clone has a hero with no key and no spend. Run `npm run demo` against live Coasty and the same pipeline rebuilds it from the run's own model-input frames — the exact images the model saw.</sub>

</div>

---

- **Zero dependencies.** No `npm install`, no lockfile, no supply chain — pure Node built-ins.
- **Runs offline for $0.** No API key, no account. A bundled in-process mock runs the full agent loop on a fresh clone.
- **The demo video renders itself.** The frames come straight out of the run — against live Coasty they are the model's own input frames, so there is no storyboard that can drift.

## What this is

A complete, runnable [Coasty](https://coasty.ai) computer-use automation for **SEC filing full-text triage**. It gives an AI agent one goal in plain English, and the agent drives a real browser on a real cloud desktop to accomplish it — no selectors, no scraping rules, no DOM parsing to maintain.

**Zero dependencies. Runs offline for $0 on a fresh clone. ~$0.90 to run for real.**

```
"Go to https://www.sec.gov/cgi-bin/browse-edgar?action=getcompany and use
 EDGAR's company search to find the filer MICROSOFT CORP, then narrow the
 results to annual reports on form 10-K. Working only from the first page
 of results EDGAR returns, report three things: how many filings are listed
 on that page, the exact form type and the filing date of the most recent
 one, and that filing's accession number copied exactly as EDGAR displays
 it. Report those three values and stop — do not open any other filer and
 do not page past the first page of results."
```

That prompt *is* the automation. When the site redesigns, the prompt still works.

That property is the whole reason to reach for a computer-use agent here. Filing triage — *has this issuer filed yet, what form was it, on what date, under what accession number* — is the unglamorous first step in front of covenant monitoring, comparables research, KYC refreshes and disclosure surveillance. A scraper does it by encoding one team's reading of one page's markup, and then quietly returns the wrong row the day that markup changes. This automation encodes the **question** instead: find the filer, look at the annual reports, tell me the count, the form type, the date and the accession number. A human analyst given that sentence copes with a redesigned page without being retrained, and so does the agent.

The same prompt style transfers to the parts of this workflow where scraping is not an option at all — regulator portals, exchange notice boards and investor-relations sites that publish to humans and expose no API to anyone.

## Try it in 30 seconds

No API key. No account. No install. No spend.

```bash
git clone https://github.com/coasty-ai/coasty-finance-filing-triage
cd coasty-finance-filing-triage
npm start
```

That boots a bundled offline mock in-process and runs the whole agent loop against it. Then render the demo video from the run's own frames:

```bash
npm run demo     # needs ffmpeg; writes media/demo.mp4 + demo.gif + poster.jpg
```

Check your setup any time with `npm run doctor`.

## Run it for real

```bash
export COASTY_API_KEY=sk-coasty-test-...      # sandbox keys never bill
export COASTY_BASE_URL=https://coasty.ai/v1
export COASTY_ALLOW_LIVE=1                     # destination consent
npm start -- --live --confirm-cost-cents 120   # cost consent
```

Both consents are required and they are deliberately separate. A live key alone will not spend; a base URL alone will not spend. See [Safety](#safety).

| | |
|---|---|
| Expected cost | **90¢** (18 steps × 5 credits) |
| Worst case | **120¢** (24-step cap) |
| Model-input frames | **free** |
| Machine runtime | Coasty provisions and destroys its own VM |

`npm run estimate` prints this before anything runs.

EDGAR is a public system and this automation reads it the way a member of the public does: no key, no login, no cookie. It stays on the first page of results by instruction, so one run is one filer and a bounded amount of reading.

## How it works

```
POST /v1/tasks                          Coasty provisions its own ephemeral VM,
                                        drives the agent, and destroys the VM
GET  /v1/runs/{id}                      poll to a terminal state
GET  /v1/runs/{id}/screenshots          the exact frames the model saw — free
GET  /v1/runs/{id}/events               per-step narration (SSE)
ffmpeg                                  frames → demo.mp4 + demo.gif + poster
```

The demo video is a **byproduct of running the automation**, not a separate artifact to author and keep in sync. There is no storyboard, no HTML mock, and nothing that can drift from reality — if the agent did something different, the video shows something different.

That matters more than usual in this vertical: the frames are the audit trail. When the agent reports an accession number, the screenshots show the page it read it off, which is the difference between an answer you can put in front of a reviewer and an answer you have to take on faith.

Verification is intrinsic and runs without a human watching:

```
✓ frames captured              18 frames
✓ frame count matches steps    18 frames vs 18 steps
✓ not all frames degraded      0 degraded
✓ frames are distinct          18/18 unique
✓ duration matches pacing      12.00s vs 12.00s expected
✓ stream width correct         1280x720
✓ video is non-trivial         360 packets
```

## Safety

This repo is built so that **accidental spend is structurally impossible**, not merely discouraged:

- **Fail-closed destination.** An unset `COASTY_BASE_URL` resolves to the bundled offline mock. Production is never a default.
- **Two independent consents.** `COASTY_ALLOW_LIVE=1` authorises the *destination*; `--confirm-cost-cents N` authorises the *cost*, and N must equal the server-computed worst case exactly.
- **Idempotency by default.** The submit key is derived from the prompt, so a retried submit returns the original run instead of provisioning a second machine.
- **A hard cap per unit.** A worst case above `capCents` in [`automation.json`](automation.json) is refused before any request is made.
- **No credentials, ever.** This automation targets a public site. Nothing here reads a password, a token, or a cookie.

## Project layout

```
automation.json      the entire unit definition — prompt, target, budget, caps
src/client.mjs       Coasty client: fail-closed target, retry, idempotency
src/capture.mjs      model-input frames → mp4/gif/poster, with sanity checks
src/cli.mjs          run · demo · estimate
tools/mock.mjs       the bundled offline Coasty (real 1280×720 PNG frames)
tools/doctor.mjs     preflight
test/                36 tests, zero dependencies, fully offline
```

Adding a new automation is one `automation.json` and one prompt — `src/` never forks. See [AGENTS.md](AGENTS.md) for the authoring contract used by Claude Code and Codex.

## Tests

```bash
npm test     # node --test, no install, no network, no key
```

## Related

Part of the **Coasty automation catalog** — computer-use automations across 12 industries. See [the index](https://github.com/coasty-ai) for retail, healthcare, legal, logistics, energy, public sector, HR, education, manufacturing, nonprofit and e-commerce.

- [Coasty docs](https://coasty.ai/docs) · [API reference](https://coasty.ai/docs/llms.txt)
- [computer-use-cookbook](https://github.com/coasty-ai/computer-use-cookbook) — the API, by endpoint, in 4 languages
- [open-cowork](https://github.com/coasty-ai/open-cowork) — the open-source AI coworker

## License

MIT © Coasty
