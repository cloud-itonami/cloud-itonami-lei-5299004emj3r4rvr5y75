# cloud-itonami-lei-5299004emj3r4rvr5y75

> **Independent third-party archive/analysis. Not affiliated with, endorsed by, or sponsored by 東京電力ホールディングス株式会社.**

This repository archives the publicly published site Terms of Use (サイトのご利用にあたって) of
**東京電力ホールディングス株式会社** (Tokyo Electric Power Company Holdings, Incorporated), with
source-url and retrieval-date provenance, per ADR-2607110300 (`cloud-itonami-lei-corporate-tos-catalog`,
`com-junkawasaki/root`). The archive itself (`blueprint.edn`, `NOTICE`, `80-data/public/
tos.journal.edn`) is unchanged by anything below — it stays a read-only reference archive; this
repo does not act, propose, or execute anything on the company's behalf using the archive data itself.

**As of 2026-07-24 this repo ALSO carries a governed actor layered on top of that
archive** (`src/tosmonitor/*`, ToSMonitor-LLM ⊣ ToSArchiveGovernor), added as part of a
10-repo validation batch extending the pilot on
`cloud-itonami-lei-2572ibtt8cczw6au4141` (P&G)
([ADR-2607241900](https://github.com/com-junkawasaki/root/blob/main/90-docs/adr/2607241900-cloud-itonami-lei-tos-monitor-actor-pilot.edn),
[ADR-2607242000](https://github.com/com-junkawasaki/root/blob/main/90-docs/adr/2607242000-cloud-itonami-lei-tos-monitor-actor-batch10.edn),
repo-local record: [`docs/adr/0001-tos-monitor-actor.md`](docs/adr/0001-tos-monitor-actor.md)).
This does NOT change the archive's own read-only nature, and it does NOT extend to
every other `cloud-itonami-lei-*` repository — most repos in this family remain plain
archives exactly as ADR-2607110300 describes. See `## Governed actor` below.

Part of the **worldwide-scope extension** of the cloud-itonami-lei catalog (acquired 2026-07-19 under
the owner-directed /loop "全世界の法人情報を取得して、連携統合", batch JP-UTIL-1), filling the catalog's
prior JP-electric-utility coverage gap.

## Company identity

- **Legal name**: 東京電力ホールディングス株式会社 (Tokyo Electric Power Company Holdings, Incorporated)
- **LEI (ISO 17442)**: [5299004EMJ3R4RVR5Y75](https://search.gleif.org/#/record/5299004EMJ3R4RVR5Y75) (GLEIF entity-verified; `registration.status` LAPSED — real LEI, not annually renewed, same convention as the isic-6492 Discover Financial entry)
- **Jurisdiction**: JP
- **Website**: https://www.tepco.co.jp
- **Ticker**: 9501 (TSE Prime)
- **ISIC Rev.5**: 3510 (electric power transmission/distribution)

## Contents

- `80-data/public/tos.journal.edn` — EDN quad-log of the archived site Terms of Use.
- `NOTICE` — copyright/attribution statement for the archived third-party text.
- `blueprint.edn` — machine-readable company identity record.

## Governed actor

**ToSMonitor-LLM ⊣ ToSArchiveGovernor**, built on this workspace's
[`langgraph-clj`](https://github.com/com-junkawasaki/langgraph-clj) StateGraph runtime,
modeled directly on
[`cloud-itonami-commitment-ledger`](https://github.com/cloud-itonami/cloud-itonami-commitment-ledger)'s
Store/Registry/Advisor/Governor/Phase/Operation/Sim shape (byte-identical code to the
pilot and every other repo in this batch except `tosmonitor.store`'s company/baseline
data).

Given a candidate ToS/legal-document snapshot, the actor proposes whether it materially
diverges from the archived baseline above, and drafts a change summary — but **it never
writes to the archive itself.** `tosmonitor.store`'s `commit-record!` only writes to
this actor's own Store (an in-process ledger), never to `80-data/public/
tos.journal.edn`. A human archivist decides whether to fold a proposal into the real
archive.

**Single actuation, `:tos/change-proposal`, never autonomous.** Every run — whether the
candidate matches the baseline or diverges from it — carries `:stake :actuation/
archive-update` and is permanently excluded from every rollout phase's auto-commit set.
Six HARD governor checks, each independently re-verified rather than taken on the
advisor's self-report:

| Check | Guards against |
|---|---|
| `grounding-violations` | a cited excerpt not actually present in the candidate's own text (hallucinated quotes) |
| `provenance-incomplete-violations` | a candidate missing full-text/source-url/retrieved-at/sha256 |
| `sha256-mismatch-violations` | a candidate's self-reported sha256 not matching its own content (ground-truth recompute, `:clj`-only in V1) |
| `retrieved-at-not-advancing-violations` | proposing a change from input staler than what is already archived |
| `doc-type-unknown-violations` | a doc-type outside this archive family's own observed vocabulary |
| `source-domain-mismatch-violations` | this actor's own distinctive check — a source-url that doesn't belong to the archived company (misattribution risk specific to an LEI-keyed independent archive; note: this check's base-domain matching is a naive last-two-labels heuristic that does not correctly handle two-part TLDs like `.co.jp` for DISTINGUISHING two different `.co.jp` companies from each other — see `tosmonitor.registry`/`tosmonitor.store` ns docstrings. It correctly matches this repo's own `www.tepco.co.jp` website against its own `www.tepco.co.jp` source-url) |

```bash
clojure -M:dev:run     # clean lifecycle + all six HARD-hold checks + a phase-0 hold + a MemStore->DatomicStore swap
clojure -M:dev:test    # governor contract · phase invariants · store parity · advisor smoke
clojure -M:lint        # clj-kondo (errors fail; CI mirrors this)
```

`clojure -M:dev:run` and the test suite always use the deterministic mock-advisor — no
live fetch of the company's current ToS page and no live `kotoba-server`/CACAO publish
happen anywhere in this actor. `tosmonitor.advisor/llm-advisor` exists as a written,
swappable seam but is not invoked. See
[`docs/adr/0001-tos-monitor-actor.md`](docs/adr/0001-tos-monitor-actor.md) for the full
design record.

## Design rationale

See ADR-2607110300 and the worldwide-scope extension ledger (`2607110300-cloud-itonami-lei-corporate-tos-catalog.worldwide-progress.edn`) in `com-junkawasaki/root` (`90-docs/adr/`).
