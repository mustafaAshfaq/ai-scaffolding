---

name: adversarial-review

description: Multi-model adversarial review of code, architecture, analysis, technical documents, or any content where rigorous critique matters. Three specialized AI models (Opus, Sonnet, and GPT-5.6 Terra) independently review the content, then each critiques the other reviewers' findings, and a majority-consensus (2/3) synthesis drives the final updated content. Use this skill whenever the user wants a thorough review of code, system design, architecture decisions, technical writing, analysis documents, PRDs, or similar artifacts — especially when they say things like "adversarial review", "red team this", "get multiple perspectives", "rigorous review", "devil's advocate review", "review this code/design/analysis", or "critique this". Invoke this skill even if the user just pastes code or a document and asks what's wrong with it, what can be improved, or to critique it.

---
 
# Adversarial Review
 
Three independent expert reviewers analyze the content, then critique each other's findings. A majority-consensus synthesis (2/3 agreement) drives the final output. Minority dissents are surfaced to the user.
 
## When to use this
 
Any content where correctness, safety, quality, or soundness matters:

- **Code**: bugs, security vulnerabilities, logic errors, performance, API design

- **Architecture**: design flaws, scalability concerns, coupling, missing failure modes

- **Analysis / reports**: logical gaps, unsupported claims, missing edge cases, bias

- **Technical documents / PRDs**: ambiguity, contradictions, missing requirements
 
---
 
## Workflow Overview
 
```

Phase 1: Independent Reviews     (3 agents in parallel — Opus, Sonnet, GPT-5.6 Terra)

           ↓

Phase 2: Cross-Critique          (each agent critiques the other two reviews)

           ↓

Phase 3: Consensus Synthesis     (map findings to unanimous / majority / minority)

           ↓

Phase 4: Apply & Present         (update content + present diff + surface dissents)

```
 
---
 
## Step 1 — Understand the content
 
Before spawning any agents, determine:

- **What is it?** (code, architecture doc, analysis, etc.)

- **Where is it?** Paste in chat, or file path(s)?

- **Context**: Ask the user for any context that would help reviewers (e.g., "this is a backend service", "we're concerned about security", "it's in production"). If none is provided, proceed — the reviewers will infer from the content.

- **Focus areas**: If the user has specific concerns, note them. Pass them to reviewers as optional hints, not constraints — reviewers should still surface anything they find important.
 
If the content is in a file, read it now so you can pass the full text to subagents. If it's multiple files, read all of them.
 
---
 
## Phase 1: Independent Reviews
 
Spawn **three subagents in parallel** in the same response — one per model. Do not wait for one to finish before spawning the others.
 
Pass each subagent:

- The full content text

- The content type (code/architecture/analysis/etc.)

- Any user-provided context or focus areas

- The path to `agents/independent_reviewer.md` (read and follow it)

- A save path for the review output: `<workspace>/phase1/<reviewer-name>/review.json`
 
**Models to use:**

| Reviewer | Model |

|----------|-------|

| Opus     | `claude opus 4.8` |

| Sonnet   | `claude sonnet 5` |

| Codex    | `gpt-5.6 Terra` |
 
**Workspace location**: Create a workspace at a sensible path, e.g., `<cwd>/adversarial-review-<timestamp>/`. Create it before spawning agents.
 
While Phase 1 runs, tell the user what's happening and roughly what to expect next.
 
---
 
## Phase 2: Cross-Critique
 
Once all three Phase 1 reviews are complete, spawn **three more subagents in parallel** — one per reviewer.
 
Each critic receives:

- The original content

- All three Phase 1 reviews (the two from the other reviewers, and their own for reference)

- Instructions from `agents/cross_critic.md`

- Which reviewer they are (opus/sonnet/codex)

- Save path: `<workspace>/phase2/<reviewer-name>/critique.json`
 
Use the same models as Phase 1 (Opus critic → `claude opus 4.8`, etc.)
 
---
 
## Phase 3: Consensus Synthesis
 
Once all Phase 2 critiques are complete, **you** (the main agent) perform the synthesis — no subagent needed for this step, it's an analytical task.
 
Read all 6 outputs (3 reviews + 3 critiques). Follow the synthesis instructions in `agents/consensus_synthesizer.md`.
 
The core task: for every distinct finding raised across all reviews and critiques, determine:

- **Unanimous** (3/3 agree): All three raised or endorsed it

- **Majority** (2/3 agree): Two raised or endorsed it; one didn't or explicitly disagreed

- **Minority** (1/3): Only one raised it; the others didn't endorse it
 
A finding is "endorsed" if a critic agreed with it in Phase 2, or did not explicitly challenge it.
 
Produce a `synthesis.json` saved to `<workspace>/synthesis.json`. See `agents/consensus_synthesizer.md` for the exact structure.
 
---
 
## Phase 4: Apply & Present
 
Apply **unanimous and majority findings** to produce a revised version of the content. Minority findings are not applied but are surfaced.
 
For **code**: produce a corrected version with inline comments where changes were made explaining what was changed and why. If changes are extensive, produce a diff summary.
 
For **documents / analysis**: produce a revised version with tracked-changes-style annotations (mark additions, deletions, rewrites) or produce a clean revised version plus a changelog.
 
**Present the results as:**
 
1. **Executive Summary** — overall verdict (unanimous verdict across reviewers), key themes

2. **Applied Changes** — what was changed and the consensus justification (majority or unanimous)

3. **Minority Dissents** — findings only 1 reviewer raised; present them to the user to decide whether to act on them

4. **Reviewer Agreement Map** — a compact table showing which findings each reviewer raised/endorsed (helps the user see where the real disagreements are)

5. **Revised Content** — the updated artifact
 
If the content is a file, write the revised version back to disk (as a new file with `-reviewed` suffix, or update in-place if the user requests).
 
---
 
## Handling disagreements
 
When two reviewers flag something and one doesn't, briefly note the dissenter's silence or stated disagreement. Don't hide it — the user deserves to know the confidence level.
 
When all three disagree (e.g., one says "add caching", one says "don't cache yet", one says "cache differently"), do not apply the change. Surface all three perspectives with "Contested — no consensus" so the user can decide.
 
---
 
## Tips for quality
 
- Reviewers may use different terminology for the same issue. During synthesis, merge conceptually identical findings even if worded differently.

- Severity matters: a finding that one reviewer calls "critical" and another calls "major" should be treated as majority-endorsed and applied at the higher severity.

- Don't over-apply: "suggestion" level findings with only 2/3 agreement should be presented as optional improvements, not mandatory changes.

- Preserve the author's intent: when rewriting content, try to keep the original voice and structure. The goal is to improve, not replace.

- Document rationale: when making changes based on reviewer feedback, briefly document why each change was made. This helps future reviewers understand the reasoning and maintains transparency.