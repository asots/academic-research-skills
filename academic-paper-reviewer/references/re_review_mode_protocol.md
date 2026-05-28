# Re-Review Mode (Verification Review)

Re-review mode is the dedicated mode for Pipeline Stage 3', designed to **verify whether revisions address the first-round review comments**.

### How It Works

```
Input:
1. Original Revision Roadmap (Stage 3 output)
2. Revised manuscript
3. Response to Reviewers (optional)

Phase 0: Reads the Revision Roadmap, builds a checklist
Phase 1: EIC checks each item (other reviewers not activated)
Phase 2: Editorial Synthesis -> New Decision
```

### Verification Logic

```
For each item in the Revision Roadmap:

Priority 1 (Required):
  -> Check each item for corresponding changes in the revised manuscript
  -> Assess revision quality (FULLY_ADDRESSED / PARTIALLY_ADDRESSED / NOT_ADDRESSED / MADE_WORSE)
  -> All Priority 1 items must be FULLY_ADDRESSED for Accept

**Traceability Rule**: For each Priority 1 item, the reviewer MUST:
1. Read the author's claim from the Response to Reviewers
2. Navigate to the stated revision location in the manuscript
3. Independently verify the claim matches the actual change
4. If Author's Claim is empty or vague ("addressed as suggested"), mark Verified? as `🔍 Cannot verify` and flag in Quality Assessment
5. **(Kong A1 / v3.11)** If Schema 11 row carries `commitment_extracted` (non-empty list from `revision_coach_agent` Step 3.5), verify each commitment against the actual manuscript diff:
   - Walk each `(commitment_text, commitment_type, required_evidence_type)` triple.
   - Locate the claimed `revision_location` in the revised manuscript.
   - Classify per-commitment `fulfillment_status`:
     - `fulfilled` — the required_evidence_type is present and substantively addresses the commitment_text.
     - `partial` — evidence exists but does not fully address the commitment (e.g., experiment run on different dataset than asked).
     - `not-fulfilled` — required_evidence_type is absent and `unfulfilled_rationale` is empty.
     - `explicitly-rejected-with-rationale` — author provides one of the three valid rationale forms (done-elsewhere pointer / rejection reasons / future-work acknowledgment); not a failure.
   - For any commitment with status ∈ `{not-fulfilled}` lacking `unfulfilled_rationale`, surface a **`COMMITMENT_GAP`** entry in re-review output (advisory only, **not** a hard block — author retains final responsibility per `POSITIONING.md`).
   - This step is the verification analog of `revision_coach_agent` Step 3.5 (Kong A1). Per-commitment lifecycle gating is what closes the Kong §7.4.3 commitment-fulfillment gap.

Priority 2 (Suggested):
  -> Check each item
  -> At least 80% should have a response
  -> NOT_ADDRESSED items require author explanation

Priority 3 (Nice to Fix):
  -> Check but does not affect Decision
```

### New Issue Detection

```
In addition to checking old items, EIC also scans for:
- Whether content added during revision introduces new problems
- Whether newly added references are correct (but deep verification is left to Stage 4.5 integrity check)
- Whether revisions cause inconsistencies
```

### Socratic Guidance After Re-Review

```
If Re-Review Decision = Major Revision:
  -> Activate Residual Coaching (residual issue guidance)
  -> EIC guides user through Socratic dialogue:
    1. Gap analysis — "How many issues did the first round of revisions resolve? Why are the remaining ones hard to address?"
    2. Root cause diagnosis — "Is it insufficient evidence, unclear argumentation, or a structural problem?"
    3. Trade-off decisions — "Which ones can be marked as research limitations?"
    4. Action plan — Plan revision approach for each residual issue
  -> Maximum 5 rounds of dialogue
  -> User can say "just fix it" to skip guidance
```

### Re-Review Output Format

```markdown
# Verification Review Report

## Decision
[Accept / Minor Revision / Major Revision]

## Revision Response Checklist

### Priority 1 — Required Revisions

| # | Original Review Comment | Author's Claim | Response Status | Revision Location | Verified? | Quality Assessment |
|---|------------------------|---------------|-----------------|-------------------|-----------|-------------------|
| R1 | [Original text] | [What the author claims to have done in Response to Reviewers] | FULLY_ADDRESSED | Section X.X | ✅ Yes | Adequately addressed; newly added content effectively resolves the issue |
| R2 | [Original text] | [Author's stated change] | PARTIALLY_ADDRESSED | Section Y.Y | ⚠️ Partial | Partially addressed, but still missing [specific gap] |

### Priority 2 — Suggested Revisions

| # | Original Review Comment | Response Status | Notes |
|---|------------------------|-----------------|-------|
| S1 | [Original text] | FULLY_ADDRESSED | -- |
| S2 | [Original text] | NOT_ADDRESSED | Author explanation: [reason] |

### Priority 3 — Nice to Fix

| # | Original Review Comment | Response Status |
|---|------------------------|-----------------|
| N1 | [Original text] | FULLY_ADDRESSED |

## New Issues (Discovered During Revision)

| # | Type | Location | Description |
|---|------|----------|-------------|
| NEW-1 | [Type] | Section X.X | [Description] |

## Decision Rationale
[Rationale based on the checklist]

## Residual Issues (If Any)
[List unresolved items, suggest marking as Acknowledged Limitations]
```

## v3.6.2 sprint contract status

v3.6.2 introduces sprint contracts for `reviewer_full` and `reviewer_methodology_focus` only. A template for this mode will follow in a subsequent patch release. Until then, this mode runs without contract enforcement and retains its pre-v3.6.2 behaviour.
