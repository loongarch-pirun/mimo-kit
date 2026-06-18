---
name: santa-method
description: "Multi-agent adversarial verification with convergence loop. Use when user says Santa Method, 双智能体查验, 双盲审查, santa check, check it twice."
origin: "Ronald Skelton - Founder, RapportScore.ai"
disable-model-invocation: true
---

# Santa Method

Multi-agent adversarial verification framework. Make a list, check it twice. If it's naughty, fix it until it's nice.

The core insight: a single agent reviewing its own output shares the same biases, knowledge gaps, and systematic errors that produced the output. Two independent reviewers with no shared context break this failure mode.

## When to Activate

Invoke this skill when:
- Output will be published, deployed, or consumed by end users
- Compliance, regulatory, or brand constraints must be enforced
- Code ships to production without human review
- Content accuracy matters (technical docs, educational material, customer-facing copy)
- Batch generation at scale where spot-checking misses systemic patterns
- Hallucination risk is elevated (claims, statistics, API references, legal language)

Do NOT use for internal drafts, exploratory research, or tasks with deterministic verification (use build/test/lint pipelines for those).

## Architecture

```
┌─────────────┐
│  GENERATOR   │  Phase 1: Make a List
│  (Agent A)   │  Produce the deliverable
└──────┬───────┘
       │ output
       ▼
┌──────────────────────────────┐
│     DUAL INDEPENDENT REVIEW   │  Phase 2: Check It Twice
│                                │
│  ┌───────────┐ ┌───────────┐  │  Two agents, same rubric,
│  │ Reviewer B │ │ Reviewer C │  │  no shared context
│  └─────┬─────┘ └─────┬─────┘  │
│        │              │        │
└────────┼──────────────┼────────┘
         │              │
         ▼              ▼
┌──────────────────────────────┐
│        VERDICT GATE           │  Phase 3: Naughty or Nice
│                                │
│  B passes AND C passes → NICE  │  Both must pass.
│  Otherwise → NAUGHTY           │  No exceptions.
└──────┬──────────────┬─────────┘
       │              │
    NICE           NAUGHTY
       │              │
       ▼              ▼
   [ SHIP ]    ┌─────────────┐
               │  FIX CYCLE   │  Phase 4: Fix Until Nice
               │              │
               │ iteration++  │  Collect all flags.
               │ if i > MAX:  │  Fix all issues.
               │   escalate   │  Re-run both reviewers.
               │ else:        │  Loop until convergence.
               │   goto Ph.2  │
               └──────────────┘
```

## Phase Details

### Phase 1: Make a List (Generate)

Execute the primary task. No changes to your normal generation workflow. Santa Method is a post-generation verification layer, not a generation strategy.

### Phase 2: Check It Twice (Independent Dual Review)

Spawn two review agents in parallel. Critical invariants:

1. **Context isolation** — neither reviewer sees the other's assessment
2. **Identical rubric** — both receive the same evaluation criteria
3. **Same inputs** — both receive the original spec AND the generated output
4. **Structured output** — each returns a typed verdict, not prose

The reviewer prompt: "You are an independent quality reviewer. You have NOT seen any other review of this output. Evaluate the output against EACH rubric criterion. For each: PASS (criterion fully met) or FAIL (specific issue found, cite the exact problem). Return structured JSON: verdict, checks, critical_issues, suggestions. Be rigorous. Your job is to find problems, not to approve."

### Rubric Design

The rubric is the most important input. Vague rubrics produce vague reviews. Every criterion must have an objective pass/fail condition.

| Criterion | Pass Condition | Failure Signal |
|-----------|---------------|----------------|
| Factual accuracy | All claims verifiable against source material or common knowledge | Invented statistics, wrong version numbers, nonexistent APIs |
| Hallucination-free | No fabricated entities, quotes, URLs, or references | Links to pages that don't exist, attributed quotes with no source |
| Completeness | Every requirement in the spec is addressed | Missing sections, skipped edge cases, incomplete coverage |
| Compliance | Passes all project-specific constraints | Banned terms used, tone violations, regulatory non-compliance |
| Internal consistency | No contradictions within the output | Section A says X, section B says not-X |
| Technical correctness | Code compiles/runs, algorithms are sound | Syntax errors, logic bugs, wrong complexity claims |

#### Domain-Specific Rubric Extensions

**Content/Marketing:**
- Brand voice adherence
- SEO requirements met (keyword density, meta tags, structure)
- No competitor trademark misuse
- CTA present and correctly linked

**Code:**
- Type safety (no any leaks, proper null handling)
- Error handling coverage
- Security (no secrets in code, input validation, injection prevention)
- Test coverage for new paths

**Compliance-Sensitive (regulated, legal, financial):**
- No outcome guarantees or unsubstantiated claims
- Required disclaimers present
- Approved terminology only
- Jurisdiction-appropriate language

### Phase 3: Naughty or Nice (Verdict Gate)

Both reviewers must pass. No partial credit. If only one reviewer catches an issue, that issue is real. The other reviewer's blind spot is exactly the failure mode Santa Method exists to eliminate. Pass → NICE (ship). Any FAIL → NAUGHTY (fix cycle).

### Phase 4: Fix Until Nice (Convergence Loop)

MAX_ITERATIONS = 3. Each round: collect all critical issues from both reviewers → fix agent repairs ONLY flagged issues (no refactoring, no unrequested changes) → re-run BOTH reviewers as fresh agents (no memory of previous round) → re-evaluate. Exhausted iterations → escalate to human. Critical: each review round uses fresh agents — no anchoring bias from prior context.

## Implementation Patterns

### Pattern A: Claude Code Subagents (Recommended)

Subagents provide true context isolation. Each reviewer is a separate process with no shared state. Use the Agent tool: Agent(description="Santa Review B", prompt="Review this output for quality... RUBRIC: ... OUTPUT: ..."). Spawn both in parallel for speed.

### Pattern B: Sequential Inline (Fallback)

When subagents aren't available, simulate isolation with explicit context resets: (1) Generate output. (2) New context: "You are Reviewer 1. Evaluate ONLY against this rubric. Find problems." (3) Record findings verbatim. (4) Clear context completely. (5) New context: "You are Reviewer 2..." (6) Compare, fix, repeat. The subagent pattern is strictly superior — inline simulation risks context bleed.

### Pattern C: Batch Sampling

For large batches (100+ items): (1) Run Santa on a random sample (10-15% of batch, minimum 5 items). (2) Categorize failures by type. (3) If systematic patterns emerge, apply targeted fixes to entire batch. (4) Re-sample and re-verify. (5) Continue until clean sample passes. Reduces cost to ~15-20% while catching >90% of systematic issues.

## Failure Modes and Mitigations

| Failure Mode | Symptom | Mitigation |
|-------------|---------|------------|
| Infinite loop | Reviewers keep finding new issues after fixes | Max iteration cap (3). Escalate. |
| Rubber stamping | Both reviewers pass everything | Adversarial prompt: "Your job is to find problems, not approve." |
| Subjective drift | Reviewers flag style preferences, not errors | Tight rubric with objective pass/fail criteria only |
| Fix regression | Fixing issue A introduces issue B | Fresh reviewers each round catch regressions |
| Reviewer agreement bias | Both reviewers miss the same thing | Mitigated by independence, not eliminated. For critical output, add third reviewer. |
| Cost explosion | Too many iterations on large outputs | Batch sampling pattern. Budget caps per verification cycle. |

## Integration with Other Skills

| Skill | Relationship |
|-------|-------------|
| Verification Loop | Use for deterministic checks (build, lint, test). Santa for semantic checks. Run verification-loop first, Santa second. |
| Eval Harness | Santa Method results feed eval metrics. Track pass@k across Santa runs. |
| Continuous Learning v2 | Santa findings become instincts. Repeated failures → learned behavior. |
| Strategic Compact | Run Santa BEFORE compacting. Don't lose review context mid-verification. |

## Metrics

- First-pass rate: % of outputs that pass Santa on round 1 (target: >70%)
- Mean iterations to convergence: average rounds to NICE (target: <1.5)
- Issue taxonomy: distribution of failure types
- Reviewer agreement: % of issues flagged by both vs. only one (low = rubric needs tightening)
- Escape rate: issues found post-ship that Santa should have caught (target: 0)

## Cost Analysis

Santa Method costs approximately 2-3x the token cost of generation alone per verification cycle. For most high-stakes output: Cost of Santa = (generation) + 2x(review per round) x (avg rounds). Cost of NOT Santa = reputation damage + correction effort + trust erosion. For batch operations, sampling reduces cost to ~15-20% while catching >90% of systematic issues.
