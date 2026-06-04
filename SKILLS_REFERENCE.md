# CSK Skills Reference

Complete reference for the **14 skills** in the Company Skill Kit (CSK). They sit on **one universal
5-gate spine** (`G1 ALIGN → G2 DEFINE → G3 BUILD&VERIFY → G4 SHIP`; `G5 OPERATE` deferred) and split
into **3 archetypes**: 8 artifact + 4 execution + 2 utility.

> **CLI owns structure & lifecycle; skills own content.** Artifact skills do not ship a `template.md` —
> they call `csk new` to scaffold a **doc-as-folder**, fill its `FILL[...]` regions, then gate with
> `csk validate`. Lifecycle fields (`status` / `version` / `created` / `updated` / `change_log`) are
> written only by the CLI. See `README.md` and the parent repo's `.project/docs/csk-suite.md`.

---

## Artifact skills (8) — produce a doc-as-folder via `csk new`

| # | Skill | Gate | Folder | Consolidates (legacy swallowed) | Output |
|---|-------|------|--------|---------------------------------|--------|
| 1 | `csk-brief` | G1 ALIGN | `g1-align/` | lean-canvas, vision-board, feature-catalog, roadmap, stakeholder-interview, jtbd, persona, journey-map, user-research | Product Brief — problem, users, scope, value |
| 2 | `csk-spec` | G2 DEFINE | `g2-define/` | prd, srs, fsd, user-story, feature-catalog | Spec — FR/NFR, stories + Given-When-Then AC |
| 3 | `csk-architecture` | G2 DEFINE | `g2-define/` | adr, c4-l1, c4-l2, erd, openapi | Architecture — C4, ADRs, API/data contracts (`--export-contract`) |
| 4 | `csk-design` | G2 DEFINE | `g2-define/` | wireframe, wireflow, design-system, brand | Design — layout, flow, tokens, brand |
| 5 | `csk-backlog` | G3 BUILD&VERIFY | `g3-build/` | wbs, raci, risk-register, sprint-planning, sprint-backlog | Backlog — work breakdown, sprints, risks |
| 6 | `csk-test` | G3 BUILD&VERIFY | `g3-build/` | test-strategy, test-plan, test-cases, rtm (seed) | Test pack — strategy, plan, cases |
| 7 | `csk-bug` | G3 BUILD&VERIFY | `g3-build/` | bug-report | Bug record — S1–S4 / P1–P4, repro, env (atomic; single flat file) |
| 8 | `csk-release` | G4 SHIP | `g4-ship/` | uat | Release pack — UAT plan, cases, sign-off |

---

## Execution skills (4) — drive work; **no persisted artifact**

| # | Skill | Purpose |
|---|-------|---------|
| 9 | `csk-scout` | Fully ephemeral codebase scouting / context gathering — produces no artifact. |
| 10 | `csk-code` | Implement a story; the commit links back to the story (RTM lineage). |
| 11 | `csk-debug` | Root-cause investigation before a fix. |
| 12 | `csk-bugfix` | Apply + verify a fix; keep the commit↔story link. |

Execution skills **skip `csk new`** (per the `<csk-rule>` archetype clause) but still keep the
commit↔story link and the validate gate-exit.

---

## Utility skills (2) — used at every gate

| # | Skill | Purpose | Invocation |
|---|-------|---------|-----------|
| 13 | `csk-conduct` | Front-door router + cross-gate workflow conductor (search / discover / workflow branches). `--advisory` returns an execution plan; `--execute` spawns owner agents and gates via `csk validate`. | `csk-conduct [--advisory\|--execute]` |
| 14 | `csk-validate` | Pre-flight check that an artifact will pass the CLI's `csk validate` gate (parts exist, no residual `FILL`, `source_skill_version` stamped, approved backed by `change_log`). | `csk-validate --skill csk-{type} {file\|doc-dir}` |

> **Moved to the CLI:** the old `csk-revise` skill is now `csk revise`; the old per-skill `csk-rtm` is the
> computed `csk rtm --view` / `csk trace`. `csk migrate` is **deferred** (greenfield has nothing to
> migrate) and is not shipped.

---

## Authoring an artifact

Artifact skills are thin content-fillers over the CLI lifecycle:

```bash
# 1. CLI scaffolds the doc-as-folder (canonical frontmatter, part stubs, FILL[...] regions):
csk new --type spec --slug checkout --title "Checkout Spec"   # → g2-define/spec-checkout/

# 2. The skill interviews you and fills the FILL[...] regions (content + semantics).

# 3. Gate at exit:
csk validate g2-define/spec-checkout
csk status g2-define/spec-checkout approved      # validate runs inline on → approved

# 4. Edit later (never overwrite in place):
csk revise g2-define/spec-checkout --type content --reason "add export FR"
```

Type → gate: `brief→g1-align`, `spec/architecture/design→g2-define`, `backlog/test/bug→g3-build`,
`release→g4-ship`.

---

## Using skills in Claude Code

After `csk install`, skills are available as slash commands. A typical gate-by-gate flow:

```
G1 ALIGN          /csk-brief
G2 DEFINE         /csk-spec   /csk-architecture   /csk-design
G3 BUILD&VERIFY   /csk-backlog   /csk-scout → /csk-code   /csk-test   /csk-bug → /csk-debug → /csk-bugfix
G4 SHIP           /csk-release
anytime           /csk-conduct  (what next?)   /csk-validate --skill csk-spec <doc>
```

---

## Canonical frontmatter (CLI-owned)

Every artifact main node carries this schema; lifecycle fields are written only by the CLI:

```yaml
---
id: SPEC-checkout             # prefix = type: BRIEF, SPEC, ARCH, DESIGN, BACKLOG, TEST, BUG, RELEASE
type: spec
title: "Checkout Spec"
status: draft                 # draft → review → approved → superseded|archived (rejected branch)
version: 1.0.0                # bumped by `csk revise`
owners: [csk-po]
source_skill: csk-spec
source_skill_version: 1.0.0   # stamped by `csk new`; validate FAILs if absent
created: 2026-06-04T00:00:00Z
updated: 2026-06-04T00:00:00Z
derives_from: [BRIEF-checkout]   # upstream → feeds `csk trace` / `csk rtm`
parts: [requirements, stories, acceptance]
change_log:
  - { version: 1.0.0, date: 2026-06-04, type: initial, reason: initial release }
---
```

`change_log` types: `initial | breaking | content | wording | typo | restore | status`.

---

## Traceability — computed (no hand-maintained RTM)

```bash
csk trace SPEC-checkout            # upstream + downstream lineage of one artifact
csk trace link --story US-001 --commit a1b2c3d   # record a commit↔story execution link (csk-code/csk-bugfix)
csk rtm --view coverage            # lineage degree per artifact
csk rtm --view impact              # transitive downstream blast radius
csk rtm --view code                # story↔commit execution links
csk db rebuild && csk db verify    # rebuild index; CI gate (drift / broken-refs)
csk broken-refs                    # standalone broken-ref check
```

---

## Related documentation

- **README.md** — installation, commands, and the 5-gate workflow guide
- Parent repo `.project/docs/csk-suite.md` (+ `.vi.md`) — the full CSK model (spine, archetypes, state
  machine, `<csk-rule>`, doc-as-folder, CLI reference)
