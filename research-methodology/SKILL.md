---
name: research-methodology
description: Load the research-skill-graph methodology and 6-lens analytical framework. Auto-invoke when conducting structured research on any topic — Bitcoin, networking, homelab, privacy, web development, local AI, business, or any question requiring multi-angle analysis. Triggers on: "research X", "analyse X", "how does X work", "is X secure", "should I use X", "what happens if X", or any question where a structured multi-lens approach would produce better answers than a direct reply.
user-invocable: false
---

# Research Methodology

The research-skill-graph uses a **6-lens analytical framework** to analyse any question from fundamentally different angles. Each lens must RETHINK the question — not just add more information. The tension between lenses is where insight lives.

Vault root: `~/Git/research-skill-graph/`

---

## Step 0: Orient Before Starting

1. Check `research-log.md` for prior research that connects to this question
2. Check `knowledge/concepts.md` and `knowledge/data-points.md` for existing relevant knowledge
3. If already covered, synthesise from existing notes rather than starting fresh

---

## Step 1: Pick the Right Framework

Read [`methodology/research-frameworks.md`](~/Git/research-skill-graph/methodology/research-frameworks.md) to select the right approach:

| Question Type | Primary Lens Order |
|---|---|
| "How does X work?" | technical → historical → security → first-principles |
| "Is X true / secure?" | technical → security → contrarian → historical |
| "Why is X happening?" | historical → economic → technical → contrarian |
| "What should I use?" | technical → security → economic → contrarian → first-principles |
| "What happens if X?" | first-principles → technical → economic → security |

Also set research depth:
- **Level 1 (Quick Scan):** 3 lenses, top 5 sources, directional understanding
- **Level 2 (Standard):** all 6 lenses, 15–25 sources, informed position
- **Level 3 (Deep Dive):** all 6 lenses with sub-questions, 50+ sources, contradiction resolution required

---

## Step 2: Evaluate Sources Before Trusting Them

Read [`methodology/source-evaluation.md`](~/Git/research-skill-graph/methodology/source-evaluation.md) before collecting evidence.

Every source must be assigned a tier:
- **Tier 1** — RFCs, protocol specs, official docs, on-chain data, CVEs → use for hard claims
- **Tier 2** — Expert analysis, Bitcoin Optech, long-form practitioner posts → use for interpretation
- **Tier 3** — Community guides, experienced operators, Stack Exchange → use for angles and tips
- **Tier 4** — Tech news, Wikipedia → initial orientation only, always verify upstream
- **Tier 5** — Reddit, Twitter, anecdotes → signal detection only

Apply the red-flag checklist: no cited sources, vendor financial incentive, emotional language, cherry-picked benchmarks → downgrade by one tier.

---

## Step 3: Run All 6 Lenses

For each lens, read its file for the specific angle, core questions, and output format. Each lens MUST rethink the question independently — treat them as written by researchers who disagree with each other.

### Technical — [`lenses/technical.md`](~/Git/research-skill-graph/lenses/technical.md)
*What do the specs, data, and measurements actually say?*
Start with Tier 1 sources only. Quantify everything. Output: COMPONENT / SPECIFICATION / MEASURED / SOURCE / CAVEAT.

### Security — [`lenses/security.md`](~/Git/research-skill-graph/lenses/security.md)
*What's the attack surface? Who's the adversary? What breaks first?*
Check CVEs, hardening guides, trust boundaries, privilege escalation paths. Output: ASSET / THREAT MODEL / ATTACK VECTOR / SEVERITY / MITIGATION / RESIDUAL RISK.

### Economic — [`lenses/economic.md`](~/Git/research-skill-graph/lenses/economic.md)
*Follow the money. Who pays, who profits, what incentives drive behaviour?*
Check funding sources, lock-in mechanisms, TCO, supply chain control. Output: PRODUCT / BUSINESS MODEL / INCENTIVE ALIGNMENT / TOTAL COST / LOCK-IN RISK / SUSTAINABILITY.

### Historical — [`lenses/historical.md`](~/Git/research-skill-graph/lenses/historical.md)
*What patterns repeat? What's been tried before? What context is everyone forgetting?*
Trace lineage, find post-mortems, check mailing list archives. Output: PRECEDENT / WHEN / OUTCOME / PARALLEL / KEY DIFFERENCE / LESSON.

### Contrarian — [`lenses/contrarian.md`](~/Git/research-skill-graph/lenses/contrarian.md)
*What if the consensus is wrong? Who benefits from the current narrative?*
Write down the consensus explicitly first, then challenge it with the strongest possible counter-argument. Output: CONSENSUS / COUNTER-ARGUMENT / EVIDENCE FOR / EVIDENCE AGAINST / WHO BENEFITS / STRENGTH (1–5).

### First Principles — [`lenses/first-principles.md`](~/Git/research-skill-graph/lenses/first-principles.md)
*Forget everything. Rebuild from fundamental truths only.*
Define the actual problem in one sentence. List non-negotiable constraints. Output: PROBLEM / FUNDAMENTAL CONSTRAINTS / CURRENT APPROACH / FIRST PRINCIPLES APPROACH / GAP / TRADE-OFFS.

---

## Step 4: Handle Contradictions

Read [`methodology/contradiction-protocol.md`](~/Git/research-skill-graph/methodology/contradiction-protocol.md) when sources or lenses disagree.

Protocol:
1. Check if they're talking about the same thing (versions, configs, eras)
2. Find the root: different data, different interpretation, different scope, different threat model, different incentives
3. Document as: "Source A argues [X] because [data]. Source B argues [Y] because [data]. Disagreement stems from [root]. Under [C1], A is right. Under [C2], B is right."
4. If unresolvable → it IS a finding. Add to `research-log.md` as an open question.

Common productive tensions:
- Technical says "works as specified" / Security says "the spec itself is flawed"
- Economic says "free tool" / Contrarian says "you're the product"
- Historical says "this approach always fails" / First Principles says "constraints have changed"

---

## Step 5: Synthesise

Read [`methodology/synthesis-rules.md`](~/Git/research-skill-graph/methodology/synthesis-rules.md) before writing conclusions.

Process:
1. Write one paragraph per lens: main finding + confidence level + what surprised you
2. Agreement map: 4+ lenses pointing same direction → high confidence; 1–2 → hypothesis only
3. Tension map: disagreements between lenses ARE the insight — document both positions
4. Cross-lens insights: the best findings combine lenses (e.g. technical supports feature X, but security reveals it expands attack surface, which means contrarian's "premature" argument dominates)
5. Confidence calibration per major finding: CLAIM / EVIDENCE / CONFIDENCE (high/medium/low) / WHAT WOULD CHANGE MY MIND

Anti-patterns to avoid: confirmation bias, narrative fallacy, recency bias, authority bias, vendor bias, anchoring.

**Never present a single-lens finding as a conclusion.**

---

## Step 6: Record Everything

1. Write findings into `projects/<topic>/` subfolder
2. Update `knowledge/concepts.md` with new definitions and mental models
3. Update `knowledge/data-points.md` with hard numbers (value, source, date, context — all required)
4. Update the relevant `knowledge/<domain>/` notes
5. Log the completed project in `research-log.md`

Vault conventions:
- Australian English throughout
- Wikilinks: `[[filename]]` without path prefixes
- Frontmatter tags: lowercase kebab-case (`bitcoin-protocol`, `docker-compose`)
- Confidence levels: **high / medium / low** — no other scale
- Every data point must include: value, source, date, context
