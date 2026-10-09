# Adaptive Skill Loop — QA Bug Reporting
*Independent concept inspired by Speexx Talk™ · Prototype for learning-design demonstration. Simulated AI analysis; no Speexx integration.*

## 1. Executive Summary

Adaptive Skill Loop is a small interactive prototype. It visualises one learning cycle: a language difficulty that recurs across job-specific practice is logged in a skill profile, triggers a short targeted practice, and is then reassessed in a new work scenario. A QA Engineer explaining bugs to developers is a good test case. The task is frequent, high-stakes and concrete, and it has an observable structure (steps → actual → expected). Its typical B1–B2 errors (3rd-person -s, past simple, do/does) cluster into one clearly teachable micro-skill. The MVP is one HTML page: one practice scenario, pre-scripted feedback that separates a *recurring pattern* from *in-the-moment* suggestions, a three-skill profile, a real 5-item drill, and one reassessment scenario. The key design rule: completing practice never erases historical observations. Only success in a new context changes the status, and even then only to "Improving". The prototype deliberately does **not** analyse text, use AI or speech, store data, model proficiency or connect to any Speexx system. It demonstrates a learning-design idea, not a technical claim.

## 2. Product / Learning Design Brief

| | |
|---|---|
| **Audience** | Speexx PM, Product Designer, LXD / Learning Science, L&D, hiring manager. They have 30–90 seconds. |
| **Learner persona** | Anna, QA Engineer, English B1–B2, international software team. |
| **User story** | As a QA engineer, I want my repeated language slips turned into one clear, short practice, so that my bug explanations become accurate in the next real conversation, not just in this one. |
| **Learning objective** | Anna can describe reproducible software behaviour and past reproduction steps with accurate verb forms (present simple 3rd person, past simple, *doesn't*) in a spoken or written bug report. |
| **Loop** | Job-specific practice → Feedback → Recurring pattern → Microlearning → Reassessment → continue monitoring. |
| **Rationale** | Feedback is most useful when the learner can tell a one-off slip from a pattern. Spaced, targeted retrieval practice works better than broad review, and transfer has to be checked in a *new* context, not inside the drill. |
| **Thesis** | From isolated feedback to a continuous skill loop. |

## 3. Speexx Context & Positioning

Speexx publicly describes Speexx Talk™ as an AI-powered tool for realistic, job-relevant conversations, adapted to CEFR level and professional context. It gives instant feedback on grammar, word order, pronunciation, clarity and spelling, asks follow-up questions, and provides session summaries and personalised learning suggestions. Speexx also describes personalised content matching across a library of 30,000+ micro-learning activities, and for IT specifically: individual development paths, IT-specific content and job-focused language practice.

This concept therefore **does not** claim a missing Speexx capability, does not claim to invent personalisation, and does not reflect Speexx's internal architecture. It's an **independent visualisation of one specific loop**, recurring error → skill profile → targeted practice → contextual reassessment, used to show learning-design thinking about *how* such a loop could be made visible and honest to the learner. There's no Speexx logo, UI, palette or integration. The taxonomy IDs belong to this prototype.

**Design question:** How might recurring language patterns from job-specific practice become targeted learning priorities — and then be checked again in a new work scenario?

## 4. User Flow & State Transitions

1. **INITIAL**: hero with the thesis, design question, loop diagram and *Try the 2-minute demo*.
2. **→ SCENARIO_ACTIVE**: the checkout-bug scenario appears. The learner card shows `EN-QA-GRAM-001` with **6** prior observations. The user types a response or clicks *Use sample response*. An empty submission shows an inline error.
3. **→ FEEDBACK_SHOWN** (*Analyse response — simulated*): pre-scripted feedback appears, with a positive opening and 3 cards. Card 1 is a **Recurring pattern** with the counter **6 → 7** and 5 corrections. Cards 2–3 are **In-the-moment** suggestions (vocabulary; Steps/Actual/Expected template). One issue observation is appended.
4. **→ PROFILE_UPDATED** (*Update learning profile*): the profile shows 3 skills. GRAM: 7 / 0 / *Priority practice*, marked "was 6 · +1". VOC: 3 / 1 / *Developing*. COM: 2 / 2 / *Monitor*. A recommendation card explains *why* this activity was recommended.
5. **→ DRILL_ACTIVE** (*Start microlearning*): 5 items with immediate ✓/✗ feedback and explanations, and progress *Question n/5*.
6. **→ DRILL_COMPLETE**: *Practice complete · Your score X/5*. `practiceCompleted = true`, status *Practice completed · Ready to reassess*, **issueCount stays 7**, with a note explaining why. If the score is under 4/5, *Retry practice* is offered but never blocks the flow.
7. **→ REASSESSMENT_ACTIVE** (*Try a new QA scenario*): the file-upload scenario, a textarea, and *Use improved sample response*.
8. **→ REASSESSMENT_COMPLETE** (*Reassess skill — simulated*): *Improvement detected in this sample.* Feedback, an optional refinement, and a before/after table: issueCount 7 → 7 (unchanged), recentSuccesses **0 → 1**, status → **Improving**. One success observation is appended. Then the status message, the final loop (Practice ✓ → Feedback ✓ → Targeted microlearning ✓ → Reassessment ✓ → Continue monitoring), and *Restart demo*.
9. **Restart → INITIAL**: profile, observations, inputs, quiz state, score, visibility and statuses are all rebuilt from the original data.

The sticky header progress bar (Practice · Feedback · Profile · Microlearning · Reassessment) shows the current and completed steps throughout.

## 5. Data Model

| Entity | Key fields | Role |
|---|---|---|
| **Skill Library** | `id`, `language`, `domain`, `category`, `name`, `practiceId`, `equivalentOf` | Defines a micro-skill and its practice activity. DE equivalents show the taxonomy scales across languages. |
| **Learner Profile** | per skill: `issueCount`, `recentSuccesses`, `practiceCompleted`, `lastPractisedAt`, `status` | Learner-level skill state, aggregated over sessions. |
| **Observation Record** | `id`, `learnerId`, `scenarioId`, `skillId`, `type` (issue/success), `pattern`, `example`, `correction`, `timestamp`, `confidence`, `simulated` | One concrete piece of evidence. Records are append-only and never deleted. |
| **Practice Activity** | `id`, `skillId`, `title`, `meta`, `description`, `passThreshold`, `items[] {stem, options, answer, feedback}` | Targeted microlearning linked to one skill. |
| **Scenario** | `id`, `type` (practice/reassessment), `title`, `context`, `colleaguePrompt`, `sampleResponse`, `simulatedFeedback` / `simulatedResult`, `targetSkillId` | A job situation plus its pre-scripted outcome. |

The full sample is in [`data.json`](data.json): 6 skills (3 EN + 3 DE), 3 profile entries, 6 seed observations, 2 scenarios, 1 practice activity and illustrative `statusRules`.

## 6. UI/UX Specification

- **Layout**: max-width 1160 px, generous whitespace. Two columns for the scenario (learner card | conversation), feedback (recurring card | in-the-moment cards) and profile (skills | recommendation). Below 860 px everything is one column. Below 560 px, buttons are full-width and correction pairs stack. No horizontal scroll (verified at 360 px).
- **Palette**: neutral B2B tokens. Background #F7F8FC, text #172033, accent #5B5BD6, success #16794B, warning #B54708, error #B42318. System font stack. No external dependencies.
- **Components**: sticky progress stepper, hero and loop diagram, learner card, chat bubble, textarea, primary/secondary buttons, feedback cards with *Recurring* / *In-the-moment* chips, a 6 → 7 counter, skill cards with status badges, recommendation card, 5-item drill, score ring, reassessment before/after table, final loop chips.
- **States**: completed steps lock their buttons. Sections fade in and scroll into view, and focus moves to the new heading. Each step carries a one-line "Why:" caption explaining the learning logic.
- **Accessibility**: semantic sections, landmarks and headings. `<label>` on both textareas. Native `<button>`s for all options. Visible focus outlines. `aria-live` on errors, drill feedback, profile and results. `aria-current="step"` in the stepper. Status always shown as text plus a symbol, never colour alone. `prefers-reduced-motion` is respected.

## 7. Rise 360 vs. Web

The core idea is *state*: counters that change, a history that persists through practice, and statuses that move only on evidence. A custom single-page web prototype shows that transparently and lets the reviewer see every transition. Rise 360 would be the better tool for the microlearning piece itself: a polished, branded lesson with Knowledge Check blocks, published to the web or an LMS and embedded via iframe in place of the built-in drill. Rise doesn't naturally expose the cross-session profile logic this concept is about, so a Rise embed is an **optional Phase 2**, not an MVP dependency (see README → *Optional Phase 2*).

## 8. Deliverables

| Deliverable | Format | Purpose | Required/Optional |
|---|---|---|---|
| `index.html` | Single HTML file (embedded CSS / JS / JSON) | Working prototype that opens with a double-click | Required |
| `data.json` | JSON | Readable reference of the data model: skills (EN + DE), profile, observations, scenarios, drill | Required |
| `README.md` | Markdown | Concept, disclaimer, run, flow, what's simulated, files, editing, publishing | Required |
| `BRIEF.md` | Markdown | This product / learning-design brief | Required |
| `screenshots/01-scenario.png`, `02-feedback-profile.png`, `03-microlearning-reassessment.png` | PNG, 1280 px | Visual preview for an email or portfolio. Reproducible via `?jump=` URLs. | Required |
| `loom-script.md` | Markdown | ≈55 s English voice-over with screen cues | Required |
| Rise 360 integration note | README section | How to replace the drill with a Rise embed later | Optional |

## 9. Implementation Plan (3–4 days)

- **Day 1**: learning flow, taxonomy, all copy, and the JSON data model.
- **Day 2**: HTML/CSS/JS, the state machine and transitions, and the drill mechanics.
- **Day 3**: responsive polish, accessibility pass, QA of every transition and Restart, screenshots, README and Loom script.
- **Day 4 (optional)**: visual refinement, Rise microlearning embed, German variant of the flow, deployment polish (GitHub Pages / Netlify).

## 10. Acceptance Checklist

- [x] A single page works without a backend (double-click `index.html`).
- [x] Exactly one practice scenario (checkout) and one reassessment scenario (file upload).
- [x] The user can type text or insert a sample response.
- [x] Analysis is labelled "simulated", with an explicit pre-scripted note.
- [x] The first analysis changes the GRAM issueCount from 6 to 7.
- [x] The Skill Profile shows three skills.
- [x] The 5-item multiple-choice drill has immediate feedback, progress and a score. Retry is shown under 4/5 and doesn't block.
- [x] Completing the drill does not reduce issueCount (it stays 7).
- [x] Reassessment changes recentSuccesses from 0 to 1 and the status to *Improving*.
- [x] Restart fully resets the demo.
- [x] "Why:" captions explain the learning logic at every step.
- [x] Desktop and mobile layouts are usable, with no horizontal scroll.
- [x] No claim of a Speexx integration or partnership. A not-affiliated note is in the footer.
- [x] "Independent concept inspired by Speexx Talk™" is visible in the hero and footer.
- [x] No real AI, API or ASR, no tracking, no external requests.
