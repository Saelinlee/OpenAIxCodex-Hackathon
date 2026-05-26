# [Project Name TBD] — Evidence-Aware Co-Pilot for Employee Relations Investigations

**Hackathon:** Sea × OpenAI Codex Hackathon — Singapore, 6 June 2026
**Track:** Deep Domain AI
**Team:** 4 (2 engineers, 2 business — one with hands-on ER experience)

---

## 1. The Problem — From an ER Manager's Desk

Employee Relations (ER) is the discipline of investigating and resolving conduct, grievance, and policy-breach matters inside a company. ER specialists act as internal investigator and arbiter — but with a wider remit than a lawyer's: they protect the company from legal and reputational risk *and* preserve a workplace where employees feel safely and fairly treated.

By the time a case reaches the ER manager, three things are usually broken:

**The complaint is messy.** Complaints arrive as forwarded emails, hotline notes, or manager escalations. Facts, opinions, and emotion are tangled. Dates, names, and channels are often missing — and they shift over time. Complainants don't have perfect memory; sometimes details mix, and sometimes a party adjusts their account in a way that suits them. The intake channel adds noise: a phone call comes in first, then a follow-up email arrives with slightly different framing. The first receiver — often an HR colleague, not the ER manager — adds their interpretation of the verbal context. By the time the case lands on the ER manager's desk, the same incident has been narrated, re-narrated, and partially reconstructed across multiple voices, each introducing interpretive drift.

**Patience has run out.** Complainants and managers typically try to handle disputes themselves for days or weeks before escalation. The parties expect fast resolution; the investigator is starting from zero.

**Evidence is the bottleneck.** ER cannot judge between two accounts without evidence. Without evidence, every decision is exposed to legal risk, perceived bias, and employee distrust. Yet figuring out *what evidence to ask for, from whom, by when* — and tracking what's been collected versus what's still missing — is done largely from memory.

Compounding this:

- **Institutional knowledge lives in senior ER people's heads.** "How did we handle a similar case last year?" has no system to ask. Junior ER staff re-invent each approach. Consistency — a legal defense — is fragile.
- **Each case is unique.** ER cases are about people. The same allegation type can require different evidence, different witnesses, different sensitivity. Templates fail; pure judgment doesn't scale.
- **Confidentiality and accuracy are non-negotiable.** ER decisions affect careers. AI tools that hallucinate, expose data, or hide their reasoning are unusable here — no matter how impressive the demo.

Legal tech serves lawyers. ER work is wider than legal work, and almost no tooling reflects how ER is actually done.

## 2. Scope — What This Is *Not*

Stated up front, because the boundary is the product:

- **Not a decision-maker.** No outcome recommendations. No "guilty / not guilty." No automated discipline. Outputs are gaps, questions, contradictions, and citations.
- **Not a legal advisor.** Labour-law interpretation is explicitly out of scope. Policy and past practice are the focus.
- **Not a multimodal reasoning engine.** Audio, video, and image artifacts are ingested and converted to text or captions. All reasoning happens on the text representation; the original artifact is preserved and linked. This boundary is stated to judges, not hidden.
- **Not confidentiality-naive.** The architecture answers the domain's constraints directly: redaction at ingest, citation-grounded outputs, human-in-the-loop checkpoints, and zero decision automation.

**Architectural principle: no decisions, only gaps and citations.** Every output is either (a) a structured piece of evidence linked to a specific artifact and policy clause, or (b) a flag that requires the ER manager's attention. The agent never concludes. This is what makes the tool usable in a domain where AI tools usually aren't.

## 3. The Build — What We're Making

An evidence-aware co-pilot that helps the ER manager:

- **Make sense of a messy complaint** within minutes — extract parties, assign investigative roles, pull HR data, flag conflicts, score urgency.
- **Decompose the complaint into atomic claims** — discrete factual assertions that can each be independently evidenced.
- **Generate an Evidence Matrix** — for every claim: what evidence would prove it, what would disprove it, who could be a witness, what's time-sensitive, and which policy clause it maps to.
- **Ingest investigation artifacts as they arrive** — interview reports, follow-up emails, screenshots, voice memos, witness statements, photos, video — and update the Evidence Matrix live, flagging contradictions, gaps, and newly discovered claims.
- **Surface what the ER manager should review** — never decide. Every flag is grounded in a specific artifact and policy clause. Ungrounded outputs are visibly flagged.

**Product thesis:** The ER manager remains the decision-maker and supervisor throughout. The agent's job is to ensure no evidence gets missed, no contradiction goes unnoticed, no analogous past case is forgotten, and no claim is judged on insufficient grounds.

### Worked Example — Atomic Claim Decomposition

This is the moat. To show what it actually looks like:

**Raw complaint (excerpt):** *"Last few weeks have been awful. During the team's Friday standup on the 12th, my manager Alex made a comment about my accent in front of everyone — and it wasn't the first time. When I raised it with him on Slack later that day, he told me I was being oversensitive and that this would affect my mid-year review. I've barely been included in project discussions since."*

**Atomic claims extracted:**

| # | Claim | Channel | Time | Witness type |
|---|-------|---------|------|--------------|
| C1 | Alex made a comment about complainant's accent during the 12th Friday standup | Verbal, in meeting | Specific date | Meeting attendees |
| C2 | Similar accent-related comments occurred prior to the 12th | Verbal | Unspecified, recurring | Prior meeting attendees / DM history |
| C3 | Alex told complainant they were "oversensitive" via Slack on the 12th | Slack DM | Same day | Slack export |
| C4 | Alex implied a negative review consequence in the same Slack exchange | Slack DM | Same day | Slack export |
| C5 | Complainant has been excluded from project discussions since the 12th | Behavioural pattern | Ongoing | Calendar invites, project channels, team members |

Each claim has a distinct evidence path, a distinct witness pool, and maps to a distinct policy clause (discrimination, retaliation, exclusion). Templates can't do this. This is where Codex earns its keep.

## 4. Pipeline — How It Works

```
                    ┌────────────────────────────────────────────┐
                    │            ER MANAGER (DECIDER)            │
                    │   Reviews, edits, approves at every stage  │
                    └────────────────────────────────────────────┘
                                       ▲
                                       │ review / edit / approve
                                       │
   ┌────────────┐    ┌─────────────┐   ▼   ┌──────────────┐    ┌──────────────┐
   │  STAGE 1   │ ─► │   STAGE 2   │ ────► │   STAGE 3    │ ─► │   STAGE 4    │
   │  INTAKE &  │    │  EVIDENCE   │       │ INGEST &     │    │ SUFFICIENCY  │
   │  CLAIM     │    │  MATRIX     │       │ UPDATE LOOP  │    │ REVIEW &     │
   │  DECOMP.   │    │  GENERATION │       │ (multimodal) │    │ ACTION LAYER │
   └────────────┘    └─────────────┘       └──────────────┘    └──────────────┘
        │                  │                     │                    │
        ▼                  ▼                     ▼                    ▼
   Case Opening      Evidence Matrix      Updated Matrix +      Status report +
   Card              (per-claim plan)     flagged reviews       MCP-triggered
                                          ◀── DEMO CENTRE ──▶   actions
```

### Stage 1 — Intake & Claim Decomposition

**Input:** Raw complaint (forwarded email, hotline note, manager escalation).

**The system:**
- Extracts named parties, dates, channels, alleged conduct.
- Cross-references each person against the mocked HRIS — name, email, department, title, tenure, reporting line, prior ER involvement.
- Assigns investigative roles: Complainant, Respondent, Witness, Bystander, Decision-maker-with-conflict.
- Runs conflict checks (does respondent report into the ER chain? Does a proposed witness report to the respondent?).
- Classifies allegation type against a fixed taxonomy (8–10 categories: harassment, discrimination, misconduct, performance dispute, retaliation, policy breach, interpersonal conflict, etc.).
- Scores urgency from duration since first incident, escalation count, retaliation/safety/legal-exposure markers, seniority of parties, witness-availability decay.
- Decomposes the complaint into atomic claims — each claim is one independently evidenceable assertion, with channel, time, and likely witness type tagged.

**Output:** A **Case Opening Card** (one screen) the ER manager reviews, edits, and approves.

### Stage 2 — Evidence Matrix Generation

**Input:** Approved Case Opening Card + atomic claims.

**For each atomic claim, the system generates:**
- **Supporting evidence types** that would prove the claim (Slack export, screenshot with metadata, contemporaneous notes, calendar invite log).
- **Counter-evidence types** that would disprove it (anti-confirmation-bias built in).
- **Candidate witnesses** mapped to real employees in the HRIS (for a public-incident claim, pulls attendees from the calendar event).
- **Time-sensitivity flags** (Slack DMs can be deleted — flag for immediate preservation request).
- **Policy clause mapping** with citation IDs.

**Output:** A **Case Evidence Matrix** — a structured, editable table that becomes the investigation plan.

### Stage 3 — Ingest & Update Loop *(Demo centre — the moment judges remember)*

**Input:** Investigation artifacts uploaded over time — interview reports, follow-up emails, screenshots, voice memos, witness statements, photos, video.

**The system:**
- **Transcribes/extracts:** Whisper for audio, OCR for screenshots, vision model for video keyframes and image captioning.
- **Links each artifact to one or more atomic claims.**
- **Updates Evidence Matrix status per claim:** Missing → Partial → Sufficient → Contradicted.
- **Flags for ER manager review:**
  - **New claims discovered** in interviews — auto-decompose, add to matrix.
  - **Contradictions** — witness statement conflicts with complainant timeline or with another artifact.
  - **Open gaps** — claim has no witness statement despite available witnesses; time-sensitive evidence not yet collected.
  - **Witness conflicts** — proposed witness reports to respondent (pressure risk).
- **Every flag cites** the artifact ID, the claim ID, and the policy clause that makes it relevant.

**Output:** A live-updating Evidence Matrix and a flagged-review queue. **This is what the judges should see moving on screen.**

### Stage 4 — Sufficiency Review & Action Layer

**Input:** ER manager triggers a sufficiency review.

**The system produces:**
- Per-claim status report (Sufficient / Partial / Missing / Contradicted) with supporting artifact list.
- Recommended next investigative steps to close remaining gaps.
- Risks of proceeding to judgment now — which claims would be undefensible.
- Precedent reference: analogous past cases with similar evidence profiles, and how they were handled. *Supporting feature. First to be cut if precedent corpus underdelivers.*

**MCP Action Layer (closing flourish — cut at 16:30 if Stage 3 isn't solid):**
- **Gmail MCP** — draft initial outreach emails (complainant acknowledgment, respondent notification, witness invitations). Drafts only; ER manager sends.
- **Calendar MCP** — propose interview slots against attendee availability, hold tentative blocks.
- **Notes/Docs MCP** — auto-create the case folder structure (scoping memo, per-witness interview template, evidence log, decision log).

## 5. How Codex Earns Its Keep

This is a Codex hackathon. The build is engineered around Codex's strengths in structured reasoning loops, not as a generic LLM wrapper.

- **Claim decomposition (Stage 1):** Codex turns unstructured complaint text into a typed list of atomic claims with channel, time, and witness-type tags. Few-shot prompted from 10–15 worked examples (see §10). This is the hardest single inference and the one most likely to fail silently — owned by Engineer 1 from the first hour.
- **Evidence-type reasoning (Stage 2):** For each claim, Codex generates supporting *and counter* evidence types, witness candidates, time-sensitivity flags, and policy citations. Structured-output prompting against the policy corpus.
- **Artifact-to-claim linking (Stage 3):** Codex reads each incoming artifact (or its text representation) and assigns it to one or more existing claims — or flags it as evidence of a new, undecomposed claim.
- **Contradiction detection (Stage 3):** Codex compares each new artifact against the existing evidence set per claim, surfacing conflicts in timeline, agency, or fact pattern. This is the live-update moment that makes Stage 3 the demo centre.
- **Sufficiency synthesis (Stage 4):** Codex composes per-claim status and recommended next steps, citing artifact IDs and policy clauses.

Each loop has a small held-out eval set so we know on the day whether prompts are landing — not "the demo worked once."

## 6. Demo Arc — The 3-Minute Story

Three time-jumped snapshots of the same case. The Day 3 snapshot is the visceral centre — the live-updating Evidence Matrix is what the audience sees moving.

**0:00 – 0:40 — "Day 1: The complaint lands"**
A messy forwarded email arrives. In under 30 seconds on screen:
- 5 atomic claims extracted from the example complaint.
- 6 people identified, roles assigned, HR data auto-filled from the mocked HRIS.
- 1 conflict flag (proposed reviewer reports to respondent).
- Urgency score: High — 3 weeks of pre-escalation disputing detected.
- Evidence Matrix generated with witness names pulled from calendar attendees.

**0:40 – 2:10 — "Day 3: Investigation underway"** *(the moment that wins the room)*
ER manager uploads, in sequence: an interview transcript, two witness statements, Slack screenshots, a voice memo. The Evidence Matrix updates **live** on screen as each artifact lands:
- 1 new claim discovered from the respondent interview ("admitted one comment, disputed context").
- 1 contradiction surfaced (witness B's timeline conflicts with complainant's by 2 hours).
- 2 claims move Missing → Partial → Sufficient.
- 1 witness conflict flagged (proposed witness D reports to respondent).

Every flag traces back to the artifact and the policy clause. No conclusions, only structured questions.

**2:10 – 2:45 — "Day 7: Sufficiency review"**
- 3 of 5 claims now Sufficient; the retaliation causal-link claim still Partial.
- System surfaces 2 analogous past cases — both required additional timeline evidence to close the causal gap.
- Gmail drafts queued for two still-uncovered witnesses. Calendar holds proposed. Case folder populated.

**Closing line:** *"The ER manager made every decision. The agent made sure no evidence got missed."*

## 7. Mapping to the Judging Rubric

Stated in the judges' language, not ours.

- **Problem framing.** A specific, painful, real workflow with named bottlenecks — drawn from a teammate's lived ER experience, not a search of "AI for HR." The interpretive-drift dynamic is documented from the inside.
- **Quality of build.** End-to-end pipeline: structured intake, RAG over a synthetic Employee Handbook, multimodal ingest, live-updating Evidence Matrix, contradiction detection, and an MCP action layer. Working demo across four stages.
- **Depth of thinking.** Atomic claim decomposition and the Evidence Matrix are domain-specific reasoning structures, not generic agent patterns. The "no decisions, only gaps and citations" architecture directly answers the confidentiality and hallucination concerns the domain demands.
- **Effective use of Codex.** Five distinct Codex loops with held-out eval sets — see §5.

## 8. Team & Roles

| Role | Owner | Responsibilities |
|------|-------|------------------|
| **Reasoning core & agentic loop** | Engineer 1 (senior) | Claim decomposition prompts, Evidence Matrix generation, ingest-and-update loop, contradiction detection, RAG over policy and past cases. Owns Codex orchestration. First hour spent solely on claim-decomposition prompt against the worked-example eval set. |
| **App, UI, multimodal & MCP** | Engineer 2 | Frontend (one clean screen per stage), live-updating Evidence Matrix view, mocked HRIS, Whisper/OCR/vision integrations, MCP layer in the final hours. |
| **Domain rubric owner & demo lead** | Business 1 (ER background) | Allegation→evidence-type rubric, claim decomposition examples, synthetic Employee Handbook, synthetic case corpus, the 3-snapshot demo script, live demo on stage. Domain authority during Q&A. |
| **Demo artifacts, pitch & rehearsal lead** | Business 2 | Owns the *physical* demo artifacts (the Day 1 email, interview transcripts, screenshots, voice memo scripts) — these must be convincing or Stage 3 falls flat. Owns pitch deck, judging-rubric mapping, time-keeping on the day, dry-run scheduling. Runs co-demo, handles non-domain Q&A. |

## 9. Time Plan — Hackathon Day

Real build time is ~5 hours after subtracting lunch, dry runs, and code freeze. The plan is built around explicit cut lines.

| Time | Milestone | Cut line |
|------|-----------|----------|
| 09:30 – 10:00 | Kick-off, align on cut lines, finalize the 3-snapshot demo content using pre-drafted templates | — |
| 10:00 – 12:30 | Engineers stand up Stages 1–2 (intake + Evidence Matrix). Business team finalizes case corpus, policy doc, demo artifacts | **By 12:30:** Stage 1 must produce a Case Opening Card from a synthetic complaint. If not, Stage 1 collapses to manual entry and we lead with Stage 2 |
| 12:30 – 13:00 | Working lunch — first end-to-end dry run of Stages 1–2 | — |
| 13:00 – 15:30 | Engineers build Stage 3 (ingest + update loop, multimodal, contradictions). Business team drafts pitch + judging-rubric mapping | **By 15:30:** the live-update demo must work on one artifact end-to-end. If not, drop video ingest and image captioning; keep audio + text only |
| 15:30 – 16:30 | Stage 4 (sufficiency review) + MCP layer (Gmail, Calendar, Notes) | **By 16:30:** if MCPs aren't authenticating cleanly, cut them entirely. Stage 4 produces a status report without external actions |
| 16:30 – 17:00 | Demo dry runs, pitch rehearsal, contingency for known failure modes | Final cut decisions locked |
| 17:00 | Code freeze | — |
| 17:15 – 19:30 | Round 1 judging — co-demo, domain depth on Q&A | — |
| 19:30 – | Finalist round (if shortlisted) | — |

## 10. Risks & Mitigations

Mitigations are commitments, not hedges.

| Risk | Mitigation |
|------|------------|
| Claim decomposition is the moat — if Codex prompts are weak, the whole story collapses | Engineer 1 spends the first hour solely on the claim-decomposition prompt against a 10–15 example eval set, pre-drafted before the event. No other work until this passes |
| Synthetic case corpus feels thin and judges see through it | Template pre-drafted; target 30–50 varied cases by noon. If quality slips, cut the precedent layer entirely and lean on policy + Evidence Matrix |
| MCP OAuth eats hours and breaks the demo | Hard cut at 16:30. Stage 4 demo works without MCPs — sufficiency report stands on its own |
| Multimodal ingest looks impressive but reasoning over it hallucinates | Reason only over text representations; preserve original artifact links; state the boundary explicitly in the pitch. If video keyframe captioning is unreliable by 15:30, drop video and demo with audio + screenshots only |
| HRIS mock is too sparse and the conflict check moment falls flat | Mocked HRIS schema and 30+ employee records pre-drafted before the event. Engineer 2 spends a budgeted hour making it convincing — sparse mocks kill the Day 1 snapshot |
| Confidentiality concern from judges | Architecture story is a slide, not an afterthought: redaction layer, citation-grounded outputs, human-in-the-loop, no decision automation |
| Demo overruns 3 minutes | Three discrete snapshots with hard-cut transitions, rehearsed handoffs between co-demoers, Business 2 time-keeps with a visible clock |
| Stage 3 live-update fails in front of judges | Pre-record the live-update sequence as a fallback video; ship the live demo first, fall back only if it fails |

## 11. Pre-Event Preparation — Templates to Pre-Draft

Per hackathon rules, no code may be submitted that was built before the event. These are **planning artifacts and synthetic content only** — not code:

1. **Allegation → Evidence Type Rubric** (the core domain artifact — the heart of Stage 2)
2. **Atomic Claim Decomposition examples** (10–15 worked examples for few-shot prompting and eval)
3. **Demo Artifacts** (the Day 1 email, Day 3 interview transcript, screenshots, voice memo script)
4. **Synthetic Employee Handbook excerpt** (~15 pages — only clauses we'd actually cite)
5. **Past Case Corpus** (schema + 30–50 instances)
6. **Mocked HRIS schema and records** (30+ employees with reporting lines, departments, prior ER involvement flags)

Drafting order, by leverage: **#1 → #2 → #3 → #6 → #4 → #5.**

---

*This README is a living document. Edit freely during the event as the build evolves.*
