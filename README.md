# Case Study Engine

Turns a closed client project's library (emails, transcripts, BRD, POs, change requests, tracker, code repo, raw metric exports, feedback, media) into a case study that a buyer believes and the client's finance and legal teams can sign off. Built for the PanScience Innovation Hackathon (31 Oct 2026) on the fictional Project Drishti library.

The hard part is not the writing. It is working out what was really promised, what was delivered, which numbers are true and what actually caused the results. So the engine is built around one rule: **no line leaves without a file behind it.**

## Run it

Requires Python 3.10+ with `matplotlib`, `Pillow` and `python-pptx` (`pip install -r requirements.txt`). The library is confidential and is **not** in this repo; put it at `library/Project_Drishti_Library/`.

```bash
python run_engine.py run --project projects/drishti
```

```bash
python -m unittest -v tests.test_engine
```

```bash
python run_engine.py gate --project projects/drishti
```

```bash
python run_engine.py publish --project projects/drishti
```

**Web demo (localhost):** `python webapp.py`, then open http://localhost:8765. Run the chain, browse the six outputs, type a claim to see the fact-checker accept or refuse it, and click through the approval gate.

`run` stops at `AWAITING_INTERNAL_REVIEW`. `gate` reads the signed decisions in `approvals/` (`internal_review.json`, then `client_decision.json`). `publish` refuses unless the gate says `RELEASED`. Add `--llm` to `run` (with `pip install anthropic` and `ANTHROPIC_API_KEY` set) to have Claude (`claude-opus-5-5`) do the extraction and drafting; the verifier, calculator and gate stay deterministic.

## The six outputs (in `projects/drishti/outputs/`)

| # | Output | File |
|---|---|---|
| 1 | Project story spine - dated, every line to file:line, tracker delays, data-quality issues | `01_story_spine.md` |
| 2 | Results sheet - definition, baseline, window, method, final number, contributors, source disagreements, working | `02_results_sheet.md` |
| 3 | Exposure report - identifiers, pattern hits, combination risks, media (incl. GPS found in a photo), contract rule and replacement | `03_exposure_report.md` |
| 4 | Case study - gatekeeper, user, buyer versions plus a forwardable champion note | `04_case_study.md` (traced version with sources: `work/04_case_study_traced.md`) |
| 5 | Media pack - cohort chart, gate chart with causes, redrawn architecture, one sales slide, interactive payback calculator | `05_media/` |
| 6 | Approval request and one-page approval sheet (options, every number's source, what was changed and why, reply-by date) | `06_approval/` |

Also: `00_library_index.md`, `work/evidence_ledger.json`, `work/04_refusals.md`, `work/run_log.txt`, `RELEASE_STATUS.json`.

## The chain

```
Librarian -> Extractor -> Historian -> Analyst -> Exposure officer -> Writer -> Fact-checker -> Media -> Approval clerk -> STOP
```

Each agent's job and instructions: [prompts/README.md](prompts/README.md). Model prompts: [prompts/extractor.md](prompts/extractor.md), [prompts/writer.md](prompts/writer.md).

- **Refuses untraceable claims.** Every evidence quote must appear verbatim in its file. Every case-study line must cite verified sources, every number in it must match those sources, it must not make a ruled-out claim (zero LTIs, 74% to 97%, benchmark savings, "halved") and must not leak an identifier.
- **Shows its working.** Every number is recomputed from raw CSVs by a declarative metric spec; the results sheet prints the sums and formula.
- **Stops for a person.** The pack is frozen by hash. Internal reviewer (not the author), then the client's Corporate Communications with Legal in copy. Edit the pack and the approval is void. Withdrawal is recorded and stops use.
- **Works on another project untouched.** Only `projects/<name>/project.json` (plus extraction and drafts) changes. `tests/fixtures/mini_project` is a different industry and library run by the same code.

## Repo map

```
run_engine.py            CLI
engine/core.py           library access, evidence ledger, run context
engine/agents/*.py       the nine agents
engine/gate.py           human approval gate
engine/llm.py            optional Claude client
prompts/                 agent instructions
projects/drishti/        config, evidence extraction, drafts, outputs
tests/                   9 checks + a second synthetic project
docs/                    process (one page), manual-edit log, prompt log, test output
pitch/                   3-slide pitch (pitch.pptx, make_pitch.py)
```

## Confidentiality

The library is fictional but must be treated as confidential (hackathon rules): it is git-ignored. The outputs in `projects/drishti/outputs/` include internal working files (story spine, exposure report) that name client people; keep this repository **private**.
