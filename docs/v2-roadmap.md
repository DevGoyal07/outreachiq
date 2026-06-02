# Automated Sponsor Outreach CRM — Roadmap & Future Work

## What shipped (v1)

A Slack-triggered outreach pipeline that takes a single command — `/enrich <domain>` — and returns a scored, drafted, logged, human-approved sponsorship lead, end to end.

**Flow:** Slack slash command → company enrichment (Apollo.io) → AI lead scoring HIGH/MEDIUM/LOW (Groq, Llama 3.3 70B) → AI-drafted personalized cold email → logged to Google Sheets → posted to Slack with Approve / Regenerate buttons → human reviews → on approval, a Gmail draft is created and the lead is marked Approved.

**Built on:** n8n (orchestration), Apollo.io (enrichment), Groq (LLM inference), Slack (Block Kit + Interactivity), Google Sheets + Gmail APIs.

**Design principle:** human-in-the-loop. The system proposes; a person approves. The AI never auto-sends. A single configuration node generalizes the tool to any organization or event, so it is not hardcoded to one use case.

---

## Roadmap (v2)

Each item below was identified during the v1 build and consciously deferred. The reasons are noted honestly — most were cost or scope decisions, not technical blockers.

### 1. Hybrid LLM routing
**What:** Route each task to the model best suited to it — Groq (fast, low-cost) for lead scoring, a stronger generation model for email drafting.
**Why deferred:** v1 standardized on one provider for simplicity and stability. Testing surfaced a clear quality/cost tradeoff: Groq is more than sufficient for classification but writes flatter prose than a top-tier generation model.
**Impact:** Higher-quality drafts where it matters (the customer-facing output) while keeping inference cheap and fast on the high-volume classification step. Also halves per-provider call volume, reducing rate-limit exposure.

### 2. Automated contact discovery
**What:** Resolve the right recipient (e.g. campus-recruiting or partnerships lead) at the target company and address the draft to them directly.
**Why deferred:** The people-search and email-reveal APIs (Apollo, Hunter) are gated behind paid tiers. For a pilot, this cost was not justified. v1 keeps recipient selection as a human step at approval — consistent with the human-in-the-loop design.
**Impact:** Removes the one remaining manual step. Best implemented as a paid upgrade path once outreach volume justifies the spend.

### 3. Always-on hosting
**What:** Migrate from local execution with a development tunnel to a hosted, always-on instance with a stable endpoint.
**Why deferred:** v1 was built and validated locally for fast iteration. Hosting is a deployment concern, correctly tackled after the pipeline was proven.
**Impact:** The pipeline runs independently of any single machine, making it genuinely usable by a team rather than a single operator.

### 4. Regeneration limits and acceptance metrics
**What:** Cap regenerations per lead and track the Approved-vs-Regenerated ratio as a product metric.
**Why deferred:** The status field (Pending / Approved / Regenerated) is already logged in v1; the analytics layer on top of it was out of scope for the core loop.
**Impact:** Turns the approval flow into a measurable signal — an "AI acceptance rate" that quantifies draft quality and trust over time, and informs prompt iteration.

### 5. Payload and token efficiency
**What:** Further trim the data passed to the LLM at each step.
**Why deferred:** Partially implemented — a trimming step already reduces the enrichment payload by roughly 90% before scoring.
**Impact:** Lower cost per lead and reduced rate-limit risk at scale.

### 6. Multi-profile configuration
**What:** Extend the single configuration node to support multiple sender profiles (different organizations, events, or campaigns) selectable at runtime.
**Why deferred:** v1 generalizes to one configured org at a time, which covered the use case.
**Impact:** One deployment serves multiple teams or campaigns without duplication.

---

## What this roadmap demonstrates

- **Prioritization under constraints** — shipped a complete, working core and deferred enhancements deliberately, with stated reasoning.
- **Cost-aware build-vs-buy judgment** — evaluated paid data/inference services and chose where spending was and wasn't justified for a pilot.
- **Systems thinking** — identified provider tradeoffs and the right task-to-tool routing before committing to a rebuild.
- **Product safety and trust** — kept a human in the loop on every outbound action by design, not by omission.
