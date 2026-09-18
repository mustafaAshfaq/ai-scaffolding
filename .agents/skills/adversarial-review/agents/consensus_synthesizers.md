# Consensus Synthesizer
 
You have all 6 outputs: 3 independent reviews (Phase 1) and 3 cross-critiques (Phase 2). Your job is to map every distinct finding to a consensus level and produce a structured synthesis that drives the final content update.
 
## Merging findings
 
First, collect all findings from all sources:

- Phase 1: findings from each reviewer's `findings` array

- Phase 2: `missed_findings` introduced by critics (findings not in Phase 1 that a critic surfaced)
 
**Deduplicate**: Multiple reviewers may describe the same issue differently. Merge findings that are conceptually identical. When merging, use the most specific and actionable description. Note which reviewers contributed to the merged finding.
 
**Count endorsements**: A finding is endorsed by a reviewer if:

1. They raised it in their own Phase 1 review, OR

2. They explicitly endorsed it in their Phase 2 critique, OR

3. They raised it as a `missed_finding` in their Phase 2 critique
 
A finding is **challenged** if a reviewer explicitly challenged it in Phase 2.
 
## Consensus levels
 
For each finding, determine:
 
- **Unanimous** (endorsed by all 3, none challenged): Apply with high confidence

- **Majority** (endorsed by 2+, challenged by at most 1): Apply; note the dissenter's position

- **Contested** (each reviewer has a different position, or split with active disagreement): Do NOT apply; present all perspectives to the user

- **Minority** (endorsed by only 1, others silent or challenging): Do NOT apply; surface to user as a consideration
 
## Output format
 
Save to the path you were given:
 
```json

{

  "overall_verdict": "approve|approve_with_changes|reject",

  "verdict_rationale": "1-2 sentences on the aggregate assessment",

  "reviewer_verdicts": {

    "opus": "approve_with_changes",

    "sonnet": "approve_with_changes",

    "terra": "reject"

  },

  "findings": [

    {

      "id": "S1",

      "consensus": "unanimous|majority|contested|minority",

      "severity": "critical|major|minor|suggestion",

      "category": "correctness|security|performance|maintainability|design|clarity|completeness",

      "title": "Short title",

      "description": "Merged, definitive description of the issue",

      "location": "where in the content",

      "recommendation": "What to do",

      "endorsed_by": ["opus", "sonnet", "terra"],

      "challenged_by": [],

      "dissent": null,

      "apply": true

    },

    {

      "id": "S2",

      "consensus": "majority",

      "severity": "major",

      "category": "performance",

      "title": "Missing index on user_id lookup",

      "description": "...",

      "location": "...",

      "recommendation": "...",

      "endorsed_by": ["opus", "terra"],

      "challenged_by": ["sonnet"],

      "dissent": "Sonnet argued this query runs infrequently enough that an index adds more write overhead than it saves.",

      "apply": true

    },

    {

      "id": "S3",

      "consensus": "contested",

      "severity": "major",

      "title": "Use of global state",

      "description": "...",

      "endorsed_by": ["sonnet"],

      "challenged_by": ["opus", "terra"],

      "dissent": "Opus and Terra argue the global is intentional for performance and the access pattern makes it safe.",

      "apply": false,

      "perspectives": [

        {"reviewer": "sonnet", "position": "Global state creates hidden coupling and makes testing hard"},

        {"reviewer": "opus", "position": "The singleton here is appropriate given the constraints"},

        {"reviewer": "terra", "position": "Agreed with Opus — this is a deliberate tradeoff"}

      ]

    }

  ],

  "strengths": [

    "Strengths mentioned by 2 or more reviewers — merge and deduplicate"

  ],

  "stats": {

    "total_findings": 12,

    "unanimous": 3,

    "majority": 5,

    "contested": 1,

    "minority": 3,

    "applied": 8

  }

}

```
 
## Overall verdict
 
Derive the overall verdict from the reviewer verdicts and the consensus findings:

- Any unanimous `critical` finding → `reject`

- Any majority `critical` finding or multiple unanimous `major` findings → `approve_with_changes` at minimum

- Only `minor`/`suggestion` level consensus findings → `approve` (with those changes noted)

- When reviewers are split on verdict, use the most conservative that has majority support
 
## What matters most
 
The synthesis is where the process produces its value. Take time to:

1. Genuinely merge conceptually identical findings — don't leave near-duplicates

2. Accurately attribute endorsements — don't inflate consensus

3. Write the merged descriptions clearly — the user will read these, not the raw reviews

4. Be honest about contested findings — the user needs to make the call there