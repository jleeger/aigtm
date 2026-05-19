# AIGTM Platform Architecture Document
**Version 1.1 — May 2026**
*Changelog from v1.0: (1) Execution mode classification added to all modules. (2) Workspace / BCO-instance hierarchy added to support multi-tenancy and multi-product customers. (3) Connector Registry defined as a fourth platform layer. (4) Reach and Sell blocks updated to reflect agentic-first execution model.*

---

## PART ONE: GOVERNING LOGIC

### The Feed-Forward Principle

The platform's organizing logic is feed-forward: each stage produces the inputs that make the next stage better. Weak upstream work compounds into downstream failure — bad market research produces a bad ICP, which produces messaging that doesn't resonate, which makes demand generation expensive, which pushes pressure onto sellers who compensate with discounting. The platform makes this dependency chain visible, auditable, and progressively improvable.

This has three operational implications:

**1. Upstream quality gates downstream quality.** Every module exposes the confidence level of its inputs and outputs. A module operating on manually-entered context produces lower-confidence output than one receiving structured upstream module output. This is surfaced to the user — not hidden.

**2. The platform improves with use.** Each cycle through the platform enriches the Business Context Object (BCO) — the central data schema every module reads from and writes to. A business that has run through the full platform once enters its second cycle with compounding advantages.

**3. The platform is its own first customer.** Every module is validated against AIGTM's own GTM problem before it is offered to paying customers. This is the reality-contact gate that prevents the platform from being built in a vacuum.

---

### The Execution Mode Principle

Every module has a defined default execution mode. This is a first-class design attribute, not an afterthought.

**Autonomous** — the module runs without human intervention. It executes, monitors, optimizes, and reports. The human sees a dashboard and can override, but no approval is required for the module to operate.

**Human-gated** — the module produces a recommendation set or structured output and waits for human confirmation before downstream modules use it as input. Appropriate for decisions that carry identity or strategic weight (brand, pricing, channel selection).

**Configurable** — the module's execution mode is set by the user at onboarding or in settings. The platform supports both autonomous and human-in-the-loop modes and the user chooses. Appropriate for outreach, content publishing, and sales motions where preferences vary by business type.

The execution mode is specified in every module definition below and in every Module Specification Document. It determines the agent's loop structure, the approval gate logic, and the dashboard surface.

---

### The Four-Layer Architecture

The platform has four distinct layers that interact through the Business Context Object:

**Layer 1 — The Universal Module Layer**
The 19 GTM functions (Modules 00–18), each implemented as an AI-native agentic module. Domain-agnostic. Every business that uses the platform, regardless of industry, runs through these same modules. The execution mode of each module is defined at the spec level.

**Layer 2 — The Domain Data Layer**
Pre-ingested, continuously updated external datasets that give specific solution-sets immediate entry-point value. BDaaS has the federal contracting dataset (SAM.gov / USASpending.gov). Future solution-sets may have legal contract databases, labor market data, etc. Domain data accelerates onboarding and enables modules to operate with higher confidence on less manual input.

**Layer 3 — The Business Context Object (BCO)**
The central schema that all modules read from and write to. Every piece of information the platform knows about a business lives here. Fields are populated by upstream module outputs, buyer-entered context, or domain data inference. Each field carries a confidence score and a provenance tag. The BCO is scoped to a single product/business line. Multiple BCO instances exist within a Workspace.

**Layer 4 — The Connector Registry**
The standardized integration layer between the platform and external services. Every module that needs to read from or write to an external system (social channels, ad platforms, CRMs, databases, communication tools) does so through the Connector Registry — never directly. The Registry exposes four methods per connector: authenticate, read, write, sync. This means adding a new integration is a connector build, not a module rebuild.

**Tier 1 connectors (built at Foundation phase):**
- Google Sheets — default data store for users without a CRM
- Stripe — payments, close events, subscription management
- SMTP / SendGrid — email outreach (M12) and comms (M08)
- Google Analytics / GA4 — web signal ingestion for behavioral discovery (M13)

**Tier 2 connectors (built as solution-sets require):**
- LinkedIn API — B2B outreach (M12)
- Meta Ads API — autonomous ad placement (M08)
- Google Ads API — autonomous ad placement (M08)
- Twilio / Bland.ai — AI voice agent outreach (M12)
- HubSpot — CRM sync for enterprise customers
- Salesforce — CRM sync for enterprise customers

**Tier 3 connectors (vertical-specific):**
- SAM.gov API — BDaaS domain data layer
- USASpending.gov API — BDaaS domain data layer

---

### Workspace & Multi-Tenancy Architecture

The BCO as designed is a single-product schema. To support multiple customers on the platform and one customer with multiple products, the BCO is nested inside a Workspace hierarchy.

```
Platform
└── Workspace (one per paying customer)
    ├── owner (user account, billing, settings)
    ├── connectors (Connector Registry config for this Workspace)
    ├── execution_preferences (default modes per module)
    └── products [
          BCO_instance_1,   // product / business line 1
          BCO_instance_2,   // product / business line 2
          BCO_instance_N    // ...
        ]
```

**A Workspace** is the top-level container owned by a user account. It holds billing, Connector Registry configuration, and execution preferences that apply across all products in the Workspace.

**A BCO instance** is a complete, independent BCO for one product or business line. All 19 modules operate on a single BCO instance at a time. The person from the lunch meeting creates one Workspace, adds three BCO instances (one per web-app), and the platform runs parallel GTM motions for each.

**Cross-product intelligence** (identifying ICP overlap, suggesting shared campaigns, etc.) is a future capability. The Workspace container makes it possible without rebuilding. Do not implement now — do not foreclose it.

**Platform-level isolation:** Each Workspace is fully isolated. No BCO data crosses between customers. This is non-negotiable for trust and compliance.

---

### Context Onboarding Modes

Every solution-set entry point operates in one of three context modes, transparently indicated to the user:

**Mode 1 — Domain-accelerated onboarding.** A pre-ingested domain dataset exists. The buyer's self-identification keys into that dataset. The platform populates the BCO automatically and asks targeted clarifying questions only where the dataset is incomplete.

**Mode 2 — Agentic instruction set entry.** No domain dataset. The buyer enters business context in free-form or structured input. The orchestrating agent interrogates the input, identifies gaps, and requests targeted clarification.

**Mode 3 — Full upstream output.** The buyer has run upstream modules. Downstream modules receive structured BCO fields with high confidence scores.

---

## PART TWO: THE MODULE MAP

### Execution Mode Key
- **[A]** Autonomous — runs without human approval
- **[H]** Human-gated — produces output, waits for confirmation
- **[C]** Configurable — user sets mode at onboarding or in settings

---

### MODULE 00 — Business Model Clarity
**Execution mode: [H]** Human-gated. This is the generative root — it requires owner confirmation before any downstream work begins.
**Phase:** Pre-discovery
**Typical owner:** Founder / Owner / CEO

**What it produces:** The market sentence and right-to-win statement that give every downstream module its targeting logic.

**Inputs:** Founder/owner input, existing documents (pitch deck, website, prior research — optional)

**Agent actions:**
- Conduct structured intake interview (conversational or form-based)
- Generate the Market Sentence: "We help [specific person] achieve [specific outcome] without [specific friction]."
- Generate the Right-to-Win Statement
- Generate the Constraint Map
- Present to owner for confirmation or revision
- Populate BCO on confirmation

**BCO fields written:** `business.*`, `offer.*`, `market_sentence`, `right_to_win_statement`, `constraint_map`

**Feed-forward output to:** Module 01 (Market Research)

---

### MODULE 01 — Market Research
**Execution mode: [A]** Autonomous. Runs on confirmation of Module 00 output.
**Phase:** Discover

**What it produces:** Complete market view — TAM/SAM/SOM, growth, buying dynamics, regulatory context, adjacent trends.

**Inputs from BCO:** `market_sentence`, `offer.target_person`, `constraint_map`
**External sources:** Web research (Tavily/Perplexity), industry databases

**Agent actions:**
- Size TAM/SAM/SOM using market sentence as targeting brief
- Research growth trajectory, buying dynamics, regulatory context, adjacent trends
- Source and cite all figures
- Populate BCO

**BCO fields written:** `market.*`

**Feed-forward output to:** Module 02, Module 04

---

### MODULE 02 — Competitive Intelligence
**Execution mode: [A]** Autonomous.
**Phase:** Discover

**What it produces:** Complete competitor map and uncontested space identification.

**Inputs from BCO:** `market_sentence`, `market.buying_dynamics`, `offer.description`
**External sources:** Web research, competitor sites, review platforms, job postings, funding databases

**Agent actions:**
- Identify all meaningful competitors (direct, indirect, status quo)
- Map each across offering, positioning, pricing, distribution, share, moat
- Identify uncontested space
- Produce competitive positioning map
- Populate BCO

**BCO fields written:** `competitive.*`

**Feed-forward output to:** Module 03, Module 04, Module 05

---

### MODULE 03 — Idea / Outcome Validation
**Execution mode: [H]** Human-gated. This is the feed-forward correction gate — it confirms or revises Module 00 before Define work begins.
**Phase:** Discover (closing gate)

**What it produces:** Validated verdict on the Module 00 hypothesis against real research.

**Inputs from BCO:** All Module 00, 01, 02 outputs.

**Agent actions:**
- Cross-reference market sentence against market evidence
- Validate or revise right-to-win statement against competitive intelligence
- If misalignment: flag specific issues and trigger Module 00 revision
- If confirmed: generate Discover Phase Summary and clear Define phase
- Populate BCO

**BCO fields written:** `validation.*`

**Feed-forward output to:** Module 04. If revision triggered: returns to Module 00.

---

### MODULE 04 — ICP & Customer Avatar Definition
**Execution mode: [H]** Human-gated. ICP definition carries strategic weight and requires owner confirmation.
**Phase:** Define

**What it produces:** Precise ICP profiles — firmographic, behavioral, psychographic — plus the three-axis motivation map, qualification scorecard, and anti-ICP.

**Inputs from BCO:** `market_sentence`, `market.buying_dynamics`, `competitive.uncontested_space`, `validation.discover_phase_summary`

**Agent actions:**
- Define primary ICP (firmographic, behavioral, psychographic)
- Map motivators to three-axis model (risk avoidance, gain, expansion)
- Define top 5 reasons to buy / not buy / fears
- Define anti-ICP
- Build qualification scorecard
- Identify dominant decision tension
- Populate BCO on confirmation

**BCO fields written:** `ICP.*`

**Feed-forward output to:** Module 05, 08, 12

---

### MODULE 05 — Positioning & Messaging Architecture
**Execution mode: [H]** Human-gated. Messaging is the strategic backbone — owner must confirm before it propagates to all downstream modules.
**Phase:** Define

**What it produces:** Single ownable market position plus full message hierarchy, language library (vector-indexed), and objection response map.

**Inputs from BCO:** `ICP.*`, `competitive.positioning_map`, `competitive.uncontested_space`

**Agent actions:**
- Define category and primary value proposition (all three motivation axes)
- Build message hierarchy: category → differentiation → proof points → objection responses
- Build language library from ICP-sourced phrases (vector-indexed for downstream retrieval)
- Build objection response map (objection → root fear → counter-message → proof requirement)
- Populate BCO on confirmation

**BCO fields written:** `messaging.*`

**Feed-forward output to:** All Reach and Sell modules.

---

### MODULE 06 — Pricing & Packaging Strategy
**Execution mode: [H]** Human-gated. Commercial model requires owner confirmation.
**Phase:** Define

**What it produces:** Commercial model — pricing structure, tiers with upgrade triggers, commercial terms.

**Inputs from BCO:** `ICP.*`, `market.buying_dynamics`, `competitive.player_map`, `offer.description`

**Agent actions:**
- Model value capture options vs. competitive alternatives and buyer WTP
- Define pricing structure and tier feature gates
- Define commercial terms that enable the sales motion
- Populate BCO on confirmation

**BCO fields written:** `pricing.*`

**Feed-forward output to:** Module 07, 11, 15

---

### MODULE 07 — Channel & Route-to-Market Strategy
**Execution mode: [H]** Human-gated. Channel selection determines the entire outreach and sales architecture.
**Phase:** Define

**What it produces:** Clear channel decision matched to ICP buying behavior, plus entry motion definition.

**Inputs from BCO:** `ICP.*`, `market.buying_dynamics`, `pricing.structure`, `constraint_map`

**Agent actions:**
- Evaluate channel options against ICP buying behavior
- Select primary channel(s) with explicit rationale
- Define entry motion: buyer's first experience and path to purchase
- Define the sales motion selector for M13/M14/M15: online-automated | hybrid | enterprise-human
- Populate BCO on confirmation

**BCO fields written:** `channel.*`
**New BCO field:** `channel.sales_motion_selector` — determines execution mode of M13, M14, M15

**Feed-forward output to:** Module 08, 09, 12

---

### MODULE 08 — Demand Generation & Content Marketing
**Execution mode: [C]** Configurable — autonomous by default, human-approval mode available.
**Phase:** Reach

**What it produces:** A continuously running content and distribution engine. Calendar, assets, ad placement, A/B testing, metrics tracking, and optimization — all executed agentically.

**Inputs from BCO:** `ICP.*`, `messaging.*`, `channel.stack`
**Connector Registry:** Meta Ads API, Google Ads API, SMTP/SendGrid, social platform APIs

**Agent actions:**
- Generate content calendar derived from motivation map and language library
- Generate content assets (long-form, short-form, email sequences, ad copy)
- Tag every asset to the motivational element it addresses
- Place ads via Connector Registry (Meta, Google) — **fully autonomous by default**
- Execute distribution across configured channels
- Monitor metrics: CTR, conversion rate, CPL, ROAS per asset and per channel
- Run A/B tests: always generate 2+ variants, promote winners, retire losers
- Feed engagement data back to BCO (enriches ICP and M13 behavioral signals)
- In human-approval mode: queue assets for review before publishing

**BCO fields written:** `demand_gen.*`
**BCO feedback loop:** Engagement data written to `ICP.behavioral_signals` — refines ICP over time

**Feed-forward output to:** Module 09, 10, 12

---

### MODULE 09 — Brand & Communications
**Execution mode: [H]** Human-gated. Brand identity is always a human decision.
**Phase:** Reach

**What it produces:** A structured recommendation set — brand identity pillars, visual identity guidelines, communications framework, website architecture — for human review and confirmation.

**Inputs from BCO:** `messaging.category_definition`, `messaging.primary_value_proposition`, `ICP.primary`, `channel.primary`

**Agent actions:**
- Derive brand identity pillars from positioning (not invented)
- Generate visual identity guidelines (type, color, spatial system) matched to ICP aesthetic
- Build communications framework: what the brand says, how, what it never says
- Generate website architecture from ICP buying journey (not product feature hierarchy)
- Present as structured recommendation document for owner confirmation
- On confirmation: populate BCO and unlock downstream modules that reference brand

**BCO fields written:** `brand.*`

**Feed-forward output to:** Module 10, 11

---

### MODULE 10 — Partner & Ecosystem Development
**Execution mode: [A]** Autonomous — runs as a background intelligence layer, not a discrete human-facing deliverable.
**Phase:** Reach

**What it produces:** A continuously maintained partner intelligence feed that enriches M08 (demand gen), M12 (pipeline), and M17 (expansion) rather than a standalone output.

**Inputs from BCO:** `ICP.*`, `market.buying_dynamics`, `competitive.player_map`, `channel.stack`

**Agent actions:**
- Continuously identify and score partner candidates (technology, services, channel, referral)
- Monitor partner activity and signal changes (new offerings, funding, market moves)
- Surface relevant partner opportunities to M08, M12, M17 as enrichment signals
- Maintain partner candidate list and activation playbook in BCO

**BCO fields written:** `partners.*`

**Embedded in:** M08 (co-marketing opportunities), M12 (referral pipeline), M17 (expansion via partner channel)

---

### MODULE 11 — Sales Enablement
**Execution mode: [H]** Human-facing output layer. Produced agentically, consumed by humans.
**Phase:** Sell

**What it produces:** Discovery framework, objection playbook, competitive battle cards, demo framework, and full sales playbook — all derived from upstream BCO outputs, not invented by sellers.

**Inputs from BCO:** Full Define phase outputs (M04–M07), full Reach phase outputs (M08–M10)
**Cold-start protocol (Mode 2):** Structured intake captures minimum BCO fields if upstream modules haven't run. Output confidence is displayed.

**Agent actions:**
- Generate discovery framework (surfaces real motivational drivers, not stated needs)
- Generate objection handling playbook (each objection linked to root fear via three-axis model)
- Generate competitive battle cards (from M02 competitive intelligence)
- Generate demo/proof-of-value framework
- Compile full sales playbook
- Populate BCO

**BCO fields written:** `enablement.*`

**Feed-forward output to:** Module 12, 13

---

### MODULE 12 — Pipeline Generation
**Execution mode: [C]** Configurable — fully autonomous by default; channel selection and approval gates configurable per channel.
**Phase:** Sell

**What it produces:** A scored, qualified prospect pipeline generated and worked autonomously across the user's chosen outreach channels.

**Inputs from BCO:** `ICP.*`, `messaging.*`, `channel.*`, `enablement.discovery_framework`
**Domain data layer:** If configured (SAM.gov for BDaaS, LinkedIn for B2B, etc.)
**Connector Registry:** LinkedIn API, SMTP/SendGrid, Twilio/Bland.ai (voice), Meta/Google Ads

**Outreach channels (user-selectable at Workspace level):**
- Email sequences — autonomous generation, personalization, send, track, follow-up
- AI voice agent — autonomous outbound calling via Twilio/Bland.ai
- LinkedIn — autonomous connection requests and message sequences
- Paid acquisition — feeds intent signals from M08 ad engagement into pipeline
- Inbound qualification — autonomously qualifies form fills, demo requests, trial signups

**Agent actions:**
- Build target list from ICP qualification scorecard applied to available data sources
- Score each prospect
- Generate personalized outreach per channel using language library as source of truth
- Execute sequences autonomously on configured channels
- Monitor engagement signals across all channels
- Qualify prospects based on engagement behavior and ICP scorecard
- Feed engagement data back to BCO (refines ICP behavioral signals)
- Pass qualified prospects to Module 13
- Populate BCO: lead records, outreach events, qualification status

**BCO fields written:** `pipeline.*`
**BCO feedback loop:** Engagement data enriches `ICP.behavioral_signals`

**Feed-forward output to:** Module 13

---

### MODULE 13 — Discovery & Qualification
**Execution mode: [C]** Configurable — mode set by `channel.sales_motion_selector` from Module 07.

- **Online-automated:** Discovery is behavioral signal analysis. No human meeting. The agent reads web analytics, email engagement, ad click patterns, and trial usage to infer motivators, budget reality, and decision timeline. Output is a deal signal profile, not a meeting summary.
- **Hybrid:** Agent prepares pre-meeting brief and post-meeting capture. Human conducts the conversation.
- **Enterprise-human:** Full human-conducted discovery with agentic support (brief, question set, capture, qualification scoring).

**Phase:** Sell

**What it produces:** Deal signal profile — real motivators (inferred or stated), decision structure, budget reality, timeline — that determines whether and how to advance.

**Inputs from BCO:** `pipeline.qualified_leads`, `enablement.discovery_framework`, `ICP.motivation_map`, `ICP.behavioral_signals`
**Connector Registry:** Google Analytics, web signal APIs, CRM (if connected)

**Agent actions (online-automated mode):**
- Ingest behavioral signals: pages visited, time on site, content downloaded, email opens/clicks, ad interactions, trial usage patterns
- Map signals to three-axis motivation model
- Infer budget reality from company data (firmographic enrichment)
- Infer timeline from engagement recency and velocity
- Score deal and output deal signal profile
- Populate BCO deal fields

**Agent actions (hybrid / enterprise-human mode):**
- Generate pre-meeting brief (account research, contact profiles, known signals)
- Generate discovery question set tailored to this prospect
- Capture and structure post-meeting output
- Apply qualification criteria
- Populate BCO deal fields

**BCO fields written:** `deal.[id].*`

**Feed-forward output to:** Module 14

---

### MODULE 14 — Solution Development & Proof of Value
**Execution mode: [C]** Configurable — mode set by `channel.sales_motion_selector`.

- **Online-automated:** Proof is delivered autonomously — matched case studies surfaced, ROI calculator pre-populated with prospect's data, free trial activated, onboarding sequence triggered.
- **Hybrid / Enterprise-human:** Solution configured with agentic support; proof narrative generated for human delivery; mutual close plan built and tracked.

**Phase:** Sell

**What it produces:** A proof-of-value delivery matched to the buyer's motivational profile and the configured sales motion.

**Inputs from BCO:** `deal.[id].*`, `enablement.demo_framework`, `messaging.objection_response_map`

**Agent actions (online-automated):**
- Match prospect's behavioral signal profile to existing case study library
- Surface highest-relevance proof assets autonomously (email, in-app, retargeting)
- Pre-populate ROI calculator with prospect's firmographic data
- Trigger free trial or demo activation sequence
- Monitor proof engagement and feed signals back to deal record

**Agent actions (hybrid / enterprise-human):**
- Configure solution to buyer's specific motivational profile
- Generate proof-of-value narrative
- Build mutual close plan (reverse-timed from stated timeline)
- Track plan progress

**BCO fields written:** `deal.[id].solution_configuration`, `deal.[id].proof_of_value_narrative`, `deal.[id].mutual_close_plan`

**Feed-forward output to:** Module 15

---

### MODULE 15 — Commercial Negotiation & Close
**Execution mode: [C]** Configurable — mode set by `channel.sales_motion_selector`.

- **Online-automated:** Close is a Stripe checkout event. The module monitors intent signals (pricing page visits, trial-to-paid conversion triggers), executes conversion sequences, and records the close event. No human involvement required.
- **Hybrid / Enterprise-human:** Human-led negotiation with agentic document generation, term tracking, and contract automation.

**Phase:** Sell

**What it produces:** A signed agreement or completed transaction — executed at the quality level appropriate to the sales motion.

**Inputs from BCO:** `deal.[id].*`, `pricing.commercial_terms`, `pricing.tiers`
**Connector Registry:** Stripe (online close), DocuSign or equivalent (enterprise contract)

**Agent actions (online-automated):**
- Monitor intent signals: pricing page visits, feature upgrade clicks, trial expiry approach
- Execute conversion sequence at optimal trigger point
- Process payment via Stripe Connector
- Record close event to BCO
- Trigger Module 16 (onboarding)

**Agent actions (hybrid / enterprise-human):**
- Track negotiation positions and concession history
- Flag risk terms and escalation triggers
- Generate counter-proposals within pricing guardrails
- Automate contract generation and routing
- Record close event to BCO on signature

**BCO fields written:** `deal.[id].negotiation_record`, `deal.[id].final_commercial_terms`, `deal.[id].close_date`

**Feed-forward output to:** Module 16

---

### MODULE 16 — Implementation & Onboarding
**Execution mode: [A]** Autonomous — triggered immediately on close event.
**Phase:** Sustain

**What it produces:** The customer at first value as fast as possible. The 48-hour rule applies.

**Inputs from BCO:** `deal.[id].*`, `ICP.motivation_map`, `customer.[id].*`
**Connector Registry:** Email (SMTP/SendGrid), in-app messaging

**Agent actions:**
- Trigger onboarding sequence immediately on close event
- Generate implementation plan derived from customer's motivational profile
- Execute onboarding communications proactively (before customer asks)
- Track progress to first value milestone
- Populate BCO

**BCO fields written:** `customer.[id].implementation_plan`, `customer.[id].first_value_date`, `customer.[id].onboarding_status`

**Feed-forward output to:** Module 17

---

### MODULE 17 — Customer Success & Expansion
**Execution mode: [A]** Autonomous health monitoring with configurable expansion outreach.
**Phase:** Sustain

**What it produces:** Continuous customer health monitoring, success evidence, and expansion trigger identification.

**Inputs from BCO:** `customer.[id].*`, `ICP.motivation_map`, `pricing.tiers`
**Connector Registry:** Product analytics, email, CRM (if connected)

**Agent actions:**
- Monitor health signals against three-axis motivation model
- Generate success evidence dossier: before/after, artifact, time-to-result
- Identify expansion trigger: when usage naturally points to next tier or product
- Identify referral and reference opportunities
- Populate BCO

**BCO fields written:** `customer.[id].health_score`, `customer.[id].success_evidence`, `customer.[id].expansion_trigger`, `customer.[id].referral_status`

**Feed-forward output to:** Module 18

---

### MODULE 18 — Renewal & Customer Advocacy
**Execution mode: [A]** Autonomous — renewal is a scheduled event, not a surprise conversation.
**Phase:** Sustain

**What it produces:** Secured renewal and active advocacy assets. Closes the feed-forward loop by writing real customer evidence back to M00 and M05.

**Inputs from BCO:** `customer.[id].*`
**Connector Registry:** Email, Stripe (renewal billing)

**Agent actions:**
- Execute renewal sequence on schedule
- Generate case study / success story asset (auto-drafted from success evidence dossier)
- Activate customer reference and referral programs
- Write success evidence back to BCO: updates `messaging.proof_points` and `right_to_win_statement`
- Populate BCO advocacy fields

**BCO fields written:** `customer.[id].renewal_status`, `customer.[id].advocacy_assets`
**BCO feedback loop — loop closes here:** `messaging.proof_points` and `right_to_win_statement` updated with real customer evidence. Platform intelligence compounds.

---

## PART THREE: THE BUSINESS CONTEXT OBJECT (BCO)

### BCO Design Principles

**Every field has a confidence score (0–100) and a provenance tag:**
- `module_generated` — populated by an upstream module's agent output
- `manually_entered` — populated by buyer input
- `domain_inferred` — populated from a pre-ingested domain dataset
- `verified` — confirmed by real-world evidence (customer outcomes, market data)

**Confidence degrades gracefully.** A module receiving a low-confidence field produces lower-confidence output and surfaces this to the user.

**The BCO is auditable.** Every field has a history log.

**The BCO is the moat.** After 12–18 months of use, the BCO knows a business better than any generic tool.

### Top-Level Schema (v1.1 — with Workspace hierarchy)

```
Workspace {
  id
  owner_id
  billing_status
  execution_preferences { default_mode_per_module{} }
  connectors { [connector_id: config] }

  products [
    BCO {
      product_id
      product_name

      // Module 00
      business { name, category, stage, website }
      offer { description, target_person, outcome, friction_removed }
      market_sentence { value, confidence, provenance }
      right_to_win_statement { value, confidence }
      constraint_map { resource[], regulatory[], competitive[] }

      // Module 01
      market { TAM, SAM, SOM, growth_trajectory, buying_dynamics,
               regulatory_context, adjacent_trends[] }

      // Module 02
      competitive { player_map[], uncontested_space, moat_analysis,
                    positioning_map }

      // Module 03
      validation { market_sentence_confidence, right_to_win_confidence,
                   discover_phase_verdict, discover_phase_summary }

      // Module 04
      ICP { primary{}, anti_profile{}, motivation_map{},
            reasons_to_buy[], reasons_not_to_buy[], fears_and_concerns[],
            dominant_decision_tension, qualification_scorecard{},
            behavioral_signals[] }          // enriched by M08 + M12 engagement data

      // Module 05
      messaging { category_definition, primary_value_proposition,
                  message_hierarchy{}, language_library[vector],
                  objection_response_map[], proof_points[] }

      // Module 06
      pricing { structure, tiers[], commercial_terms, willingness_to_pay_range }

      // Module 07
      channel { primary, entry_motion, stack[], resource_allocation,
                sales_motion_selector }     // online-automated | hybrid | enterprise-human

      // Module 08
      demand_gen { content_calendar, content_assets[], distribution_logic,
                   ab_test_framework, ad_campaigns[], metrics{} }

      // Module 09
      brand { identity_pillars[], visual_identity, communications_framework,
              website_architecture }

      // Module 10
      partners { category_map, candidate_list[], activation_playbook }

      // Module 11
      enablement { discovery_framework, objection_playbook, battle_cards[],
                   demo_framework, sales_playbook }

      // Module 12
      pipeline { target_list[], outreach_events[], qualified_leads[],
                 channel_performance{} }

      // Modules 13-15 (deal-level, repeating)
      deals [ {
        id, motivators{}, decision_structure, budget_reality, timeline,
        qualification_verdict, behavioral_signal_profile{},
        solution_configuration, proof_of_value_narrative,
        mutual_close_plan, negotiation_record,
        final_commercial_terms, close_date, close_type  // stripe_checkout | signed_contract
      } ]

      // Modules 16-18 (customer-level, repeating)
      customers [ {
        id, implementation_plan, first_value_date, onboarding_status,
        health_score, success_evidence, expansion_trigger,
        referral_status, renewal_status, advocacy_assets
      } ]
    }
  ]
}
```

---

## PART FOUR: AGENT ARCHITECTURE

### Orchestration Model

Single orchestrator + specialized functional agents. The Orchestrator manages BCO state, sequences module activation, routes tasks to functional agents, surfaces confidence signals, and manages execution mode logic (autonomous loop vs. human-gate wait state).

**Execution mode management:**
- **Autonomous modules** run in a continuous loop. The Orchestrator schedules runs, monitors outputs, and feeds results to downstream modules.
- **Human-gated modules** enter a wait state after producing output. The Orchestrator holds downstream module activation until the user confirms.
- **Configurable modules** read the Workspace `execution_preferences` and the module-level setting to determine loop behavior.

### Connector Registry Architecture

The Connector Registry is a standardized abstraction layer. No module calls an external API directly.

```
Module → Orchestrator → Connector Registry → External Service
                              ↑
                    authenticate | read | write | sync
```

Each connector implements four methods:
- `authenticate(credentials)` — handles OAuth, API key, or token flow
- `read(params)` → structured data — pulls data from external service into BCO
- `write(data)` → confirmation — pushes BCO data or agent output to external service
- `sync(params)` → delta — keeps BCO and external service in alignment

Adding a new integration = building a new connector. No module code changes.

---

## PART FIVE: SOLUTION-SET CONFIGURATIONS

### Solution-Set 00 — AIGTM Full Stack
**Modules:** 00–18 (complete)
**ICP:** Founder / operator with full GTM authority
**Default execution mode:** Configurable per module (set at Workspace onboarding)
**Market sentence:** *We help founders build a complete, compounding go-to-market engine — without the cost of a full marketing and sales team.*

### Solution-Set 01 — BDaaS (Government Contracting)
**Modules:** 00, 04, 05, 12 (SAM.gov domain data), 13, 14, 15, 16
**ICP:** Small GovCon firm ($1M–$10M, 1–2 BD staff)
**Default execution mode:** Autonomous for M12 pipeline generation; Human-gated for M00, M04, M05
**Domain data layer:** SAM.gov + USASpending.gov
**Market sentence:** *We help small government contractors find and win better-matched federal opportunities — without a full BD team.*

### Solution-Set 02 — Online Business Accelerator (placeholder)
**Modules:** 00, 04, 05, 06, 08, 09, 12, 13 (automated), 15 (Stripe close)
**ICP:** Founder with an existing online product, no sales team, no desire to manage sales manually
**Default execution mode:** Fully autonomous for M08, M12, M13, M15
**Market sentence:** TBD — to be produced by running this solution-set through Module 00.
**Note:** This solution-set maps directly to the lunch meeting use case (web-app founder, 3 products, no sales experience, no desire to manage online sales effort). The sales motion selector is set to `online-automated` at M07.

---

## PART SIX: BUILD SEQUENCE

### Phase 0 — Foundation (Before Any Module)
1. BCO schema design and validation (all fields, Workspace hierarchy, confidence scoring, provenance tagging)
2. Orchestrator scaffold (BCO state management, module sequencing, confidence surfacing, execution mode management, cost tracking)
3. **[NEW v1.1] Workspace & BCO instance hierarchy** — Workspace container, multi-BCO-instance schema, isolation logic (F-06)
4. **[NEW v1.1] Connector Registry scaffold** — four-method interface, Tier 1 connectors: Google Sheets, Stripe, SMTP/SendGrid, Google Analytics (F-07)
5. Technology stack decisions + repo setup
6. Module 00 manual run (AIGTM as customer) — establish AIGTM BCO baseline

### Phases 1–5 — Module Build (unchanged from v1.0)
Build order: 00 → 01 → 02 → 03 → 04 → 05 → 06 → 07 → 08 → 09 → 10 → 11 → 12 → 13 → 14 → 15 → 16 → 17 → 18. S→B→V for each.

### Phase 6 — Solution-Set Packaging
SS-00 full-stack deploy. SS-01 BDaaS (SAM.gov domain data layer). SS-02 Online Business Accelerator (validate against lunch meeting use case as first external test).

---

## PART SEVEN: TECHNOLOGY CONSTRAINTS

**Locked decisions:**
- BCO schema: PostgreSQL + vector store (Pinecone or pgvector — decide before F-02)
- LLM: Claude API (primary)
- Web research: Tavily or Perplexity API
- Auth: Clerk
- Payments: Stripe (also Tier 1 connector)
- Connector Registry: custom-built (Python or TypeScript), not a third-party integration platform

**Deferred:**
- Frontend framework
- Backend framework
- Hosting / cloud provider
- Workflow orchestration (resolved at module build time if needed)

---

## PART EIGHT: QUALITY GATES

Every module must pass all six gates before it is considered complete:

1. **BCO integration:** Correctly reads from and writes to BCO schema including Workspace hierarchy?
2. **Feed-forward dependency:** Uses its upstream inputs?
3. **Confidence surfacing:** Displays output confidence and provenance to user?
4. **AIGTM validation:** Run against AIGTM's own GTM problem — useful output produced?
5. **Cold-start protocol:** Defined behavior when upstream modules haven't run?
6. **Economics tracking:** Logs LLM cost per run?

**New gate for autonomous modules (v1.1):**

7. **Execution loop integrity:** Does the autonomous loop have a defined run schedule, a failure/retry protocol, and a circuit breaker that halts execution and alerts the user if output quality degrades below threshold?

---

*End of AIGTM Platform Architecture Document v1.1*
*Next artifact: BCO Schema Specification (F-01)*
