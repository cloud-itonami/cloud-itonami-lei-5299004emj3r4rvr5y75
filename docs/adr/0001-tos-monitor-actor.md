# ADR-0001: ToSMonitor-LLM ⊣ ToSArchiveGovernor -- a governed actor layered on this archive

- Status: Accepted (2026-07-24)
- Related: [`com-junkawasaki/root` ADR-2607110300](https://github.com/com-junkawasaki/root/blob/main/90-docs/adr/2607110300-cloud-itonami-lei-corporate-tos-catalog.edn)
  (the archive-only design this repo was created under -- unchanged by this
  ADR); [`com-junkawasaki/root` ADR-2607241900](https://github.com/com-junkawasaki/root/blob/main/90-docs/adr/2607241900-cloud-itonami-lei-tos-monitor-actor-pilot.edn)
  (the original 1-repo pilot, on `cloud-itonami-lei-2572ibtt8cczw6au4141`,
  P&G); [`com-junkawasaki/root` ADR-2607242000](https://github.com/com-junkawasaki/root/blob/main/90-docs/adr/2607242000-cloud-itonami-lei-tos-monitor-actor-batch10.edn)
  (the 10-repo validation batch this repo is part of -- deliberately
  includes this repo as the batch's only non-US/EU jurisdiction, to
  stress-test the domain-matching check against a `.co.jp` host).

## Context

This repository archives 東京電力ホールディングス株式会社 (TEPCO)'s publicly
published site Terms of Use, per ADR-2607110300 -- a read-only reference
archive. As part of a 10-repo validation batch extending the
`cloud-itonami-lei-2572ibtt8cczw6au4141` pilot (see ADR-2607241900/
ADR-2607242000 for the full fleet-level design rationale), this repo gains
a governed actor layer on top of the unchanged archive.

## Decision

Identical design and code to the pilot and every other repo in this batch
(`src/tosmonitor/{governor,phase,operation,registry,advisor}.cljc` are
byte-for-byte identical across all of them) -- see ADR-2607241900 for the
full rationale of each of the six HARD governor checks, the single
always-escalate `:tos/change-proposal` actuation, and the mock-advisor-only
scope. Only `tosmonitor.store`'s company/baseline demo data is specific to
this repo:

- **Company**: 東京電力ホールディングス株式会社 (Tokyo Electric Power Company
  Holdings), LEI 5299004EMJ3R4RVR5Y75, website `https://www.tepco.co.jp`.
- **Baseline provenance** (real, from this repo's own `80-data/public/
  tos.journal.edn`): source-url `https://www.tepco.co.jp/legal/`,
  retrieved-at `2026-07-19T07:41:07Z`, doc-type `:terms-of-service`.
- **Baseline full text**: a short, hand-written representative Japanese
  excerpt (not the real archived page), with a self-consistent SHA-256
  computed from that excerpt itself -- matching the pilot's own convention
  (ADR-2607241900).

**Known limitation exercised, not newly introduced**: `tosmonitor.registry/
source-domain-matches-company?` uses a naive last-two-labels base-domain
heuristic (documented in the pilot's own `tosmonitor.registry` ns
docstring as a V1 scope boundary: does not handle multi-part TLDs like
`.co.uk`/`.co.jp`). For THIS repo, website and source-url are both
`www.tepco.co.jp` -- the SAME host -- so the check's positive case is
unaffected. The limitation would only matter for distinguishing two
DIFFERENT `.co.jp` companies from each other, which this repo's own tests
do not need to exercise (the negative `source-domain-mismatch-violations`
test uses a clearly unrelated `.example` domain, not another `.co.jp`
site).

The archive-of-record (`80-data/public/tos.journal.edn`) is never touched;
`commit-record!` only writes to this actor's own Store.

## Consequences

Same as the pilot (ADR-2607241900) and the batch (ADR-2607242000), plus:
validates the actor pattern against a non-Latin-script company name/legal
text and a two-part-TLD domain, both firsts for this actor family.

## Run

```bash
kbb -M:dev:run     # walk a clean lifecycle + all six HARD-hold checks + a phase-0 hold + a backend swap
kbb -M:dev:test    # governor contract · phase invariants · store parity · advisor smoke
kbb -M:lint        # clj-kondo (errors fail; CI mirrors this)
```
