# Adaptive Skill Loop — QA Bug Reporting

**Independent concept inspired by Speexx Talk™.**
Prototype for learning-design demonstration. Simulated AI analysis; no Speexx integration.

A single-page interactive demo of one learning loop:

> recurring language observation → skill profile → targeted microlearning → reassessment in a new work scenario

shown with one QA Engineer (Anna, English B1–B2) explaining software bugs to a developer.

This is **not** a Speexx product, feature proposal or integration. It doesn't use Speexx branding, interface or data. The taxonomy IDs (`EN-QA-*`, `DE-QA-*`) belong to this prototype only.

---

## Open locally

Double-click `index.html`. You don't need npm, a server or an internet connection.

## Files

```
adaptive-skill-loop/
├── index.html          # the whole prototype: HTML + CSS + vanilla JS + embedded JSON data
├── data.json           # readable copy of the data model (same content as the embedded JSON)
├── README.md
├── BRIEF.md            # product / learning-design brief
├── loom-script.md      # 30–60 s voice-over script
└── screenshots/
    ├── 01-scenario.png
    ├── 02-feedback-profile.png
    └── 03-microlearning-reassessment.png
```

## Interaction flow (state machine)

| # | State | Trigger | What changes |
|---|---|---|---|
| 0 | `INITIAL` | page load | Hero only |
| 1 | `SCENARIO_ACTIVE` | **Try the 2-minute demo** | Checkout scenario, learner card (GRAM count = 6) |
| 2 | `FEEDBACK_SHOWN` | **Analyse response — simulated** (needs any text or the sample) | 3 feedback cards; `EN-QA-GRAM-001.issueCount` 6 → 7; new issue observation added |
| 3 | `PROFILE_UPDATED` | **Update learning profile** | Skill Profile (3 skills), GRAM status → *Priority practice*, recommendation card |
| 4 | `DRILL_ACTIVE` | **Start microlearning** | 5-item multiple choice, immediate feedback, progress 1/5 … 5/5 |
| 5 | `DRILL_COMPLETE` | answer item 5 → **See score** | Score X/5; `practiceCompleted = true`; status → *Practice completed · Ready to reassess*; **issueCount stays 7**; *Retry practice* if score < 4 (optional) |
| 6 | `REASSESSMENT_ACTIVE` | **Try a new QA scenario** | File-upload scenario |
| 7 | `REASSESSMENT_COMPLETE` | **Reassess skill — simulated** | `recentSuccesses` 0 → 1, status → *Improving*, success observation appended, final loop |
| — | → `INITIAL` | **Restart demo** | Every value, input, quiz answer, section and status is reset |

## What is simulated

- **All "analysis"**: the feedback and reassessment results are pre-scripted in the JSON. The typed text is never analysed. The UI says so next to both buttons.
- **The AI colleague**: a static prompt.
- **Profile counts and observations**: demo values. They're not psychometric scores, and "7 observations" is not a level.
- **What's real**: the five-item drill (it scores your actual answers) and the in-memory state transitions.

There's no AI, LLM/API call, speech/ASR, backend, database, authentication, analytics/tracking or external data, and nothing is stored after you close the tab.

## Change copy, or add a skill or scenario

All content lives in the `<script type="application/json" id="demo-data">` block near the end of `index.html`. `data.json` is a documented copy of it.

- **Copy**: edit the strings in `scenarios[*]` (title, context, prompt, sample response, feedback) or `practiceActivities[0]` (drill items).
- **New skill**: add an entry to `skillLibrary` and a matching key in `profile` (`issueCount`, `recentSuccesses`, `status`). Every key in `profile` is rendered as a skill card.
- **New drill item**: add to `practiceActivities[0].items`. Each item takes `stem` (use `___` for the gap), `options`, `answer` and `feedback`. The progress count and score adapt automatically.
- **New status**: add a label → style mapping in `STATUS_STYLE` in the script.
- The demo flow deliberately uses exactly one `practice` and one `reassessment` scenario.

If you edit `data.json`, paste it back into the embedded block, or run:

```bash
node -e "const f=require('fs');let h=f.readFileSync('index.html','utf8');const d=JSON.stringify(JSON.parse(f.readFileSync('data.json','utf8')),null,1);h=h.replace(/(<script type=\"application\/json\" id=\"demo-data\">)[\s\S]*?(<\/script>)/,'\$1\n'+d+'\n\$2');f.writeFileSync('index.html',h)"
```

## Publish as a static page

- **GitHub Pages**: push the folder to a repo, then go to Settings → Pages → deploy from branch (root). Share `https://<user>.github.io/<repo>/`.
- **Netlify**: drag and drop the folder at app.netlify.com/drop.
- **CodePen**: paste the CSS from `<style>` into the CSS panel and the body markup into the HTML panel, *including* the JSON `<script>` block. Paste the main `<script>` into the JS panel.
- Any static host works. (Glitch is not an option: it ended app hosting on 8 July 2025.)

## Screenshots

`screenshots/` contains three desktop captures (1280 px wide). To re-capture any state, open the page with a shortcut parameter. These run the flow with the sample responses and all-correct drill answers, and turn off animations:

| File | URL | Viewport |
|---|---|---|
| 01-scenario.png | `index.html?jump=scenario` | 1280 × 1000 |
| 02-feedback-profile.png | `index.html?jump=profile&from=sec-feedback` | 1280 × 1900 |
| 03-microlearning-reassessment.png | `index.html?jump=reassessed&from=sec-drill` | 1280 × 1750 |
| optional mobile | `index.html?jump=feedback` | 390 × 844, DevTools device mode |

`jump` accepts `scenario | feedback | profile | drill | score | reassessed`. `from` hides everything above that section, so the capture starts at the top.

Headless Chrome example:

```bash
chrome --headless=new --hide-scrollbars --window-size=1280,1000 --screenshot=01-scenario.png "file:///…/index.html?jump=scenario"
```

For a mobile capture, use DevTools device mode instead of headless. Headless Chrome on Windows enforces a minimum window width and crops the page.

## Optional Phase 2: Rise 360

The 5-item drill is self-contained, so you can swap it for a Rise 360 microlearning lesson (Knowledge Check blocks) without touching the loop logic:

1. Build the lesson in Rise and publish it to **Web**. Host the output folder statically.
2. In `renderDrill()`, render `<iframe src="<rise-url>" title="Verb Forms microlearning" style="width:100%;height:640px;border:0">` instead of the question markup, plus a **"I've finished the practice"** button that calls the existing completion logic (`nextItem` at the last item, or a small `completePractice()`).
3. Keep the profile update (`practiceCompleted = true`, issueCount unchanged) in this page. A web-published Rise iframe doesn't report scores back without LMS/xAPI tooling, so score display becomes optional.

The MVP doesn't depend on Rise.
