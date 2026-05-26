# [Project Name TBD] — Evidence-Aware Co-Pilot for Employee Relations Investigations

**Hackathon:** Sea × OpenAI Codex Hackathon — Singapore, 6 June 2026
**Track:** Deep Domain AI
**Team size:** 4 (2 engineers, 2 business — including 1 with hands-on ER experience)

---

## 1. The Problem — From an ER Manager's Desk

Employee Relations (ER) is the discipline of investigating and resolving conduct, grievance, and policy-breach matters inside a company. ER specialists act as the internal lawyer, investigator, and arbiter — but with a wider remit than a lawyer's: they balance protecting the company from legal and reputational risk *and* preserving a workplace where employees feel safe and treated fairly.

By the time a case reaches the ER manager, three things are usually already broken:

1. **The complaint is messy — in format, in content, and in handoff.** Complaints arrive in inconsistent formats: a forwarded email, a hotline note, a manager's escalation. Facts, opinions, and emotional context are tangled. Specific dates, names, and channels are often missing — and worse, they can shift over time. Complainants don't have perfect memory; sometimes details mix or change as the case progresses, and sometimes a party adjusts their account in a way that's advantageous to them. The intake channel itself adds noise: not every complaint is filed through the right channel the first time. A phone call may come in first, then a follow-up email arrives with a slightly different framing. The first message receiver (often an HR colleague, not the ER manager) shares their interpretation of the verbal context as a side note. By the time the case reaches the ER manager, the same incident has been narrated, re-narrated, and partially reconstructed across multiple voices — each one introducing its own interpretive drift.
2. **Patience has run out.** Complainants and managers typically try to handle disputes themselves for days or weeks before escalation. By the time ER opens the case, the parties expect a fast, fair resolution — but the investigator is starting from zero.
3. **Evidence is the bottleneck.** ER cannot judge between accounts without evidence — and without it, every decision invites legal risk, perceived bias, and employee distrust. Yet deciding what evidence to ask for, from whom, by when — and tracking what's collected versus what's still missing — is handled manually. Critical evidence can be missed during review (human error, investigator perspective, or experience gaps), and tracking itself can drift, because investigations typically run for weeks or months.

Compounding this:

- **Institutional knowledge lives in senior ER people's heads.** "How did we handle a similar case last year?" is a question with no system to ask. New or junior ER staff re-invent the approach each time. Consistency — a legal defense — is fragile.
- **Each case is unique.** ER cases are about people. The same allegation type can require different evidence, different witnesses, different sensitivity. Templates fail; pure judgment doesn't scale.
- **Confidentiality and accuracy are non-negotiable.** ER decisions affect careers and livelihoods. AI tools that hallucinate, expose data, or hide their reasoning are unusable in this domain — no matter how impressive the demo.

Legal tech has built tools for lawyers. ER work is wider than legal work, and almost no tooling reflects how ER is actually done.

## 2. The Build — What We're Making

An evidence-aware co-pilot that helps the ER manager:

- **Make sense of a messy complaint** within minutes — extract parties, assign investigative roles, pull HR data, flag conflicts, score urgency.
- **Decompose the complaint into atomic claims** — discrete factual assertions that can each be independently evidenced.
- **Generate an Evidence Matrix** — for every claim: what evidence would prove it, what would disprove it, who could be a witness, what's time-sensitive, and which policy clause it maps to.
- **Ingest investigation artifacts** as they arrive — interview reports, follow-up emails, screenshots, voice memos, witness statements, photos, video — and update the Evidence Matrix live, flagging contradictions, gaps, and new claims.
- **Surface what the ER manager should review** — never decide. Every flag is grounded in a specific artifact and policy clause. Ungrounded outputs are visibly flagged.

**Product thesis:** The ER manager remains the decision-maker and supervisor throughout. The agent's job is to ensure no evidence gets missed, no contradiction goes unnoticed, no analogous past case is forgotten, and no claim is judged on insufficient grounds.

## 3. What This Is *Not*

To stay sharp on scope and honest with judges:

- **Not a decision-maker.** No outcome recommendations. No "guilty / not guilty." No automated discipline. Outputs are gaps, questions, contradictions, and citations.
- **Not a legal advisor.** We deliberately scope out labour-law interpretation for this build. Policy and past practice are the focus.
- **Not a multimodal reasoning engine.** Multimodal artifacts (audio, video, image) are ingested and converted to text/captions. All reasoning happens on the text representation, with the original artifact preserved and linked. This boundary is stated explicitly to judges.
- **Not a confidentiality-naive tool.** Architectural choices (redaction layer, citation-grounded outputs, human-in-the-loop checkpoints, no decision automation) directly address the domain's confidentiality and hallucination concerns.

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
                                                                actions
```

### Stage 1 — Intake & Claim Decomposition

**Input:** Raw complaint (forwarded email, hotline note, manager escalation).

**The system:**
- Extracts named parties, dates, channels, alleged conduct.
- Cross-references each person against the mocked HRIS — pulls name, email, department, title, tenure, reporting line, prior ER involvement.
- Assigns investigative roles: Complainant, Respondent, Witness, Bystander, Decision-maker-with-conflict.
- Runs **conflict checks** (e.g., does respondent report to anyone in the ER chain? Is a proposed witness in the respondent's reporting line?).
- Classifies allegation type against a fixed taxonomy (8–10 categories: harassment, discrimination, misconduct, performance dispute, retaliation, policy breach, interpersonal conflict, etc.).
- **Scores urgency** — duration since first incident, escalation count, retaliation/safety/legal-exposure markers, seniority of parties, witness availability decay.
- **Decomposes the complaint into atomic claims** — each claim is one independently evidenceable assertion, with channel, time, and likely witness type tagged.

**Output:** A **Case Opening Card** (one screen) the ER manager reviews, edits, and approves.

### Stage 2 — Evidence Matrix Generation

**Input:** Approved Case Opening Card + atomic claims.

**For each atomic claim, the system generates:**
- **Supporting evidence types** that would prove the claim (e.g., Slack export, screenshot with metadata, contemporaneous notes, calendar invite log).
- **Counter-evidence types** that would disprove it (anti-confirmation-bias built in).
- **Candidate witnesses** mapped to real employees in the HRIS (e.g., for a public-incident claim, pulls attendees from the calendar event).
- **Time-sensitivity flags** (e.g., Slack DMs can be deleted — flag for immediate preservation request).
- **Policy clause mapping** with citation IDs.

**Output:** A **Case Evidence Matrix** — a structured, editable table that becomes the investigation plan.

### Stage 3 — Ingest & Update Loop

**Input:** Investigation artifacts uploaded over time — interview reports, follow-up emails, screenshots, voice memos, witness statements, photos, video.

**The system:**
- **Transcribes/extracts** (Whisper for audio, OCR for screenshots, vision model for video keyframes / image captioning).
- **Links each artifact to one or more atomic claims.**
- **Updates Evidence Matrix status per claim:** Missing → Partial → Sufficient → Contradicted.
- **Flags for ER manager review:**
  - **New claims discovered** in interviews — auto-decompose, add to matrix.
  - **Contradictions** — witness statement conflicts with complainant timeline, or with another artifact.
  - **Open gaps** — claim has no witness statement despite available witnesses; time-sensitive evidence not yet collected.
  - **Witness conflicts** — proposed witness reports to respondent (pressure risk).
- **Every flag cites** the artifact ID, the claim ID, and the policy clause that makes it relevant.

**Output:** A live-updating Evidence Matrix and a flagged-review queue.

### Stage 4 — Sufficiency Review & Action Layer

**Input:** ER manager triggers a sufficiency review.

**The system produces:**
- Per-claim status report (Sufficient / Partial / Missing / Contradicted) with the supporting artifact list.
- Recommended next investigative steps to close remaining gaps.
- Risks of proceeding to judgment now — which claims would be undefensible.
- Precedent reference: analogous past cases with similar evidence profiles, and how they were handled. (Supporting feature, not the headline.)

**MCP Action Layer (closing flourish):**
- **Gmail MCP** — draft initial outreach emails (complainant acknowledgment, respondent notification, witness invitations). Drafts only; ER manager sends.
- **Calendar MCP** — propose interview slots against attendee availability, hold tentative blocks.
- **Notes/Docs MCP** — auto-create the case folder structure (scoping memo, per-witness interview template, evidence log, decision log).

## 5. Demo Arc — The 3-Minute Story

Three time-jumped snapshots of the same case:

**0:00 – 0:45 — "Day 1: The complaint lands"**
A messy forwarded email arrives. In under 30 seconds on-screen:
- 4 atomic claims extracted.
- 6 people identified, roles assigned, HR data auto-filled.
- 1 conflict flag (proposed reviewer reports to respondent).
- Urgency score: High — 3 weeks of pre-escalation disputing detected.
- Evidence Matrix generated with witness names pulled from calendar.

**0:45 – 2:00 — "Day 3: Investigation underway"**
ER manager uploads interview transcript, 2 witness statements, Slack screenshots, a voice memo. The Evidence Matrix updates live:
- 1 new claim discovered from the respondent interview ("admitted one comment, disputed context").
- 1 contradiction flagged (witness B's timeline conflicts with complainant's by 2 hours).
- 2 claims move Missing → Partial → Sufficient.
- 1 witness conflict surfaced (proposed witness D reports to respondent).

**2:00 – 2:45 — "Day 7: Sufficiency review"**
- 3 of 4 claims now Sufficient; Claim 4 (retaliation causal link) still Partial.
- System surfaces 2 analogous past cases — both required additional timeline evidence to close the causal gap.
- Gmail drafts queued for the 2 still-uncovered witnesses. Calendar holds proposed. Case folder populated.

**Closing line:** *"The ER manager made every decision. The agent made sure no evidence got missed."*

## 6. Why This Wins on the Three Judging Dimensions

- **Problem framing.** A specific, painful, real workflow with named bottlenecks — drawn from lived experience, not a search-engine summary of "AI for HR."
- **Quality of build.** A working end-to-end pipeline with multimodal ingest, agentic loops, RAG over policy and past cases, and MCP integrations — not a single-prompt wrapper.
- **Depth of thinking.** Claim decomposition and the Evidence Matrix are domain-specific reasoning structures, not general-purpose LLM patterns. The "no decisions, only gaps and citations" architecture directly answers the domain's confidentiality and hallucination constraints.
- **Effective use of Codex.** Codex powers the agentic loop: claim decomposition, evidence-type reasoning, contradiction detection, artifact-to-claim linking, and the live-update orchestration across stages.

## 7. Team & Roles

| Role | Owner | Responsibilities |
|------|-------|------------------|
| **Reasoning core & agentic loop** | Engineer 1 (senior) | Claim decomposition prompts, Evidence Matrix generation, ingest-and-update loop, contradiction detection, RAG over policy and past cases. Owns Codex orchestration. |
| **App, UI, multimodal & MCP** | Engineer 2 | Frontend (one clean screen per stage), live-updating Evidence Matrix view, mocked HRIS, Whisper/OCR/vision integrations, MCP layer in the final hours. |
| **Domain rubric owner & demo lead** | Business 1 (ER background) | Allegation→evidence-type rubric, claim decomposition examples (10–15 worked), synthetic Employee Handbook, synthetic case corpus, the 3-snapshot demo script, live demo on stage. Domain authority during Q&A. |
| **Demo artifacts, pitch & judging alignment** | Business 2 | Drafts the demo artifacts (the email, interview transcripts, screenshots, voice memo scripts), owns pitch deck, judging-rubric alignment, runs co-demo, handles non-domain Q&A. |

## 8. Time Plan — Hackathon Day

| Time | Milestone |
|------|-----------|
| 09:30 – 10:00 | Kick-off, align on cut lines, finalize the 3-snapshot demo content using pre-drafted templates |
| 10:00 – 12:30 | Engineers stand up Stages 1–2 (intake + Evidence Matrix). Business team finalizes case corpus, policy doc, demo artifacts |
| 12:30 – 13:00 | Working lunch — first end-to-end dry run of Stages 1–2 |
| 13:00 – 15:30 | Engineers build Stage 3 (ingest + update loop, multimodal, contradictions). Business team drafts pitch + judging-rubric mapping |
| 15:30 – 16:30 | Stage 4 (sufficiency review) + MCP layer (Gmail, Calendar, Notes). **MCPs are a flourish — cut them if Stage 3 isn't solid by 15:30.** |
| 16:30 – 17:00 | Demo dry runs, pitch rehearsal, contingency for known failure modes |
| 17:00 | Code freeze |
| 17:15 – 19:30 | Round 1 judging — co-demo, domain depth on Q&A |
| 19:30 – | Finalist round (if shortlisted) |

## 9. Risks & Mitigations

| Risk | Mitigation |
|------|------------|
| Synthetic case corpus feels thin and judges see through it | Pre-draft template now; aim for 30–50 varied cases by noon on the day. If quality slips, cut precedent layer entirely and lean on policy + Evidence Matrix |
| MCP OAuth eats hours and breaks the demo | MCPs are scoped as a closing flourish; demo works without them. Hard cut at 16:30 if not landing |
| Multimodal ingest looks impressive but reasoning over it hallucinates | Reason only over text representations; preserve original artifact links; state the boundary explicitly in the pitch |
| Claim decomposition is the moat — if Codex prompts are weak, the whole story collapses | Pre-draft 10–15 worked decomposition examples for few-shot prompting and as eval set. Engineer 1 spends the first hour solely on this prompt |
| Confidentiality concern from judges | Architecture story is built in: redaction layer, citation-grounded outputs, human-in-the-loop, no decision automation. Make this a slide, not an afterthought |
| Demo overruns 3 minutes | Three discrete snapshots, hard-cut transitions, rehearsed handoffs between co-demoers |

## 10. Pre-Event Preparation — Templates to Pre-Draft

Per hackathon rules, no code may be submitted that was built before the event. These are **planning artifacts and synthetic content only** — not code:

1. **Allegation → Evidence Type Rubric** (the core domain artifact — the heart of Stage 2)
2. **Atomic Claim Decomposition examples** (10–15 worked examples for few-shot prompting and eval)
3. **Synthetic Employee Handbook excerpt** (~15 pages — only the clauses we'd actually cite)
4. **Past Case Corpus** (schema + 30–50 instances)
5. **Demo Artifacts** (the Day 1 email, Day 3 interview transcript, screenshots, voice memo script)

Drafting order, by leverage: **#1 → #2 → #5 → #3 → #4.**

---

*This README is a living document. Edit freely during the event as the build evolves.*
