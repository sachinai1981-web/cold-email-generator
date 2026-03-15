# Phase 1: Build - Research

**Researched:** 2026-03-14
**Domain:** Vanilla HTML/CSS/JS single-file tool — template-based text generation, clipboard API, form state management
**Confidence:** HIGH

---

## Summary

This phase builds a single self-contained HTML file: a cold email body generator for salespeople. The user enters four fields (prospect name, company, pain point, value prop), clicks Generate, and gets three email body variants in distinct tones (Direct, Curious, Story-Led). No API, no build step, no backend — everything runs in the browser from a local file.

The project already has a direct predecessor in the same repo: `subject-line-scorer.html`. That file establishes the complete design language (dark theme, `#0f1117` background, `#6366f1` indigo accent, card components, copy-to-clipboard pattern) and the JavaScript architecture (template pool, click-to-copy with `navigator.clipboard`, DOM rendering via `createElement`). The new tool should match this aesthetic exactly — users of the scorer will immediately recognise the new tool as part of the same suite.

The core engineering challenge is NOT the JS mechanics (those are solved by the predecessor). It is writing three email body templates that genuinely feel tonally different under 100 words — this is the product quality gate and requires careful copywriting, not more code.

**Primary recommendation:** Model the HTML/CSS/JS structure directly on `subject-line-scorer.html`. The main creative work is the three tone templates; the main technical work is word-count enforcement and the copy button per variant.

---

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| INP-01 | User can enter a prospect's first name or full name | Text input, same pattern as `g-name` in scorer |
| INP-02 | User can enter a prospect's company name | Text input, same pattern as `g-company` in scorer |
| INP-03 | User can enter a pain point the prospect is likely experiencing | Text input or textarea, same pattern as `g-pain` in scorer |
| INP-04 | User can enter their value proposition | Text input, same pattern as `g-value` in scorer |
| INP-05 | User can click a single Generate button to produce all 3 variants at once | Single `button.full` calling one JS function, same pattern as scorer |
| OUT-01 | Page displays a Direct-tone email body | Template function returns string; rendered into a card |
| OUT-02 | Page displays a Curious-tone email body | Same rendering path as OUT-01, different template |
| OUT-03 | Page displays a Story-Led-tone email body | Same rendering path as OUT-01, different template |
| OUT-04 | Each email body is under 100 words (enforced by template design) | Template structure keeps body to 3–4 short sentences; word count verified in template, not truncation |
| OUT-05 | Each email body has a one-click copy-to-clipboard button | `navigator.clipboard.writeText()` — same API as scorer |
| OUT-06 | Copy button provides visual confirmation ("Copied!" feedback) | Button text swap + timeout, identical to scorer's `copyLine()` |
| UX-01 | Form clears or resets easily for a new prospect | Reset button calls `form.reset()` or manually clears each input and hides results section |
| UX-02 | Each tone variant is visually labelled (Direct / Curious / Story-Led) | `card-label` / uppercase badge pattern, same as scorer |
| UX-03 | Layout is clean and usable on a laptop screen | Inherit scorer's `max-width: 640px` centered layout |
| TECH-01 | Entire tool is a single self-contained HTML file | No external JS, inline `<style>` and `<script>` blocks only; Google Fonts CDN link is acceptable |
| TECH-02 | Generation is template-based — no API calls, works fully offline | String interpolation into static template strings in JS |
| TECH-03 | Word count is enforced by template structure, not truncation | Templates designed to stay under 100 words by construction; a `wc()` helper can assert count during development |
</phase_requirements>

---

## Standard Stack

### Core

| Technology | Version | Purpose | Why Standard |
|------------|---------|---------|--------------|
| HTML5 | — | Document structure | Requirement: single HTML file |
| Vanilla CSS (inline `<style>`) | — | Styling | No build step; predecessor uses this pattern |
| Vanilla JS (inline `<script>`) | — | Template generation, DOM manipulation | No framework needed; predecessor proves this is sufficient |
| `navigator.clipboard` API | Browser-native (widely supported) | Copy to clipboard | Already proven in `subject-line-scorer.html`; no polyfill needed for modern browsers |

### Supporting

| Technology | Version | Purpose | When to Use |
|------------|---------|---------|-------------|
| Google Fonts CDN (`Inter` or system font stack) | — | Typography | Optional — the scorer uses system font stack (`-apple-system, BlinkMacSystemFont, 'Segoe UI'`); stick with that unless you want Inter explicitly |
| CSS `animation: fadeIn` keyframe | — | Reveal results smoothly | Same `.fade-in` class the scorer uses |

### Alternatives Considered

| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| Vanilla JS string templates | React/Vue component | React adds 40KB, requires a build step — violates TECH-01 |
| `navigator.clipboard` | `document.execCommand('copy')` | execCommand is deprecated; clipboard API is standard and already used in scorer |
| Inline CSS | External stylesheet | External file violates TECH-01 single-file requirement |

**Installation:** None — no npm, no build. Open the file in a browser.

---

## Architecture Patterns

### File Structure (single file)

```
cold-email-generator.html
├── <head>
│   ├── <meta charset>, viewport
│   └── <style>  ← all CSS inline
└── <body>
    ├── .header           ← title + subtitle
    ├── .card (form)      ← 4 inputs + Generate button + Reset button
    ├── #results          ← hidden until generate(); contains 3 variant cards
    │   ├── .variant-card#direct
    │   ├── .variant-card#curious
    │   └── .variant-card#story
    └── <script>          ← all JS inline
        ├── TEMPLATES object (3 tone functions)
        ├── generate()
        ├── renderVariant()
        ├── copyBody()
        └── resetForm()
```

### Pattern 1: Template Function Per Tone

**What:** Each tone is a pure function `(name, company, pain, value) => string` that returns a pre-written, personalized email body. The function interpolates inputs into a fixed structure.

**When to use:** Always — this is TECH-02 (no API, template-based). The function owns the word budget.

**Example:**
```javascript
// Direct tone — ~60–70 words, no fluff, CTA in last sentence
function directTemplate(name, company, pain, value) {
  return `Hi ${name},

I noticed ${company} is likely dealing with ${pain}. Most teams I speak with are losing hours a week to this.

${value} — and we typically help teams like yours fix it in under 30 days.

Worth a quick 15-minute call this week?`;
}

// Curious tone — opens with a question, builds intrigue
function curiousTemplate(name, company, pain, value) {
  return `Hi ${name},

Quick question — is ${company} actively working on ${pain} right now, or is it on the backburner?

I ask because we've helped similar teams cut that problem in half using ${value}.

Happy to share what worked — would that be useful?`;
}

// Story-Led tone — micro-story opener, connects to them
function storyTemplate(name, company, pain, value) {
  return `Hi ${name},

A sales leader at a company like ${company} came to us 6 months ago struggling with exactly ${pain}.

After trying ${value}, their team turned it around in one quarter.

I'd love to share the specifics — is that worth 15 minutes?`;
}
```

### Pattern 2: Render Results into DOM (no innerHTML for user data — use textContent)

**What:** Build result cards by setting `textContent` on elements, not concatenating user input into `innerHTML`. This avoids XSS if the tool is ever served over HTTP.

**When to use:** Whenever rendering user-provided input into the DOM.

**Example:**
```javascript
// Source: MDN Web Docs — DOM manipulation best practice
function renderVariant(elementId, label, body) {
  const card = document.getElementById(elementId);
  card.querySelector('.variant-label').textContent = label;
  card.querySelector('.variant-body').textContent = body;
  card.querySelector('.copy-btn').dataset.body = body;
}
```

### Pattern 3: Copy-to-Clipboard with Visual Confirmation

**What:** `navigator.clipboard.writeText(text)` returns a Promise. On resolve, swap button label to "Copied!", then restore after 1500ms. Identical to `copyLine()` in the scorer.

**Example:**
```javascript
// Derived from subject-line-scorer.html copyLine() — already proven pattern
function copyBody(btn) {
  const text = btn.dataset.body;
  navigator.clipboard.writeText(text).then(() => {
    const orig = btn.textContent;
    btn.textContent = 'Copied!';
    btn.classList.add('copied');
    setTimeout(() => {
      btn.textContent = orig;
      btn.classList.remove('copied');
    }, 1500);
  });
}
```

### Pattern 4: Show/Hide Results Section

**What:** Results `div` starts with `display:none`. After generate(), set `display:block` and add `.fade-in` class. Reset hides it again.

**Example:**
```javascript
function generate() {
  // ... validate inputs, build bodies ...
  const results = document.getElementById('results');
  results.style.display = 'block';
  results.classList.add('fade-in');
  results.scrollIntoView({ behavior: 'smooth', block: 'nearest' });
}

function resetForm() {
  ['inp-name','inp-company','inp-pain','inp-value'].forEach(id => {
    document.getElementById(id).value = '';
  });
  const results = document.getElementById('results');
  results.style.display = 'none';
  results.classList.remove('fade-in');
  document.getElementById('inp-name').focus();
}
```

### Anti-Patterns to Avoid

- **Using innerHTML with user input:** Interpolating `name` or `company` directly into `innerHTML` strings creates XSS exposure. Use `textContent` for user-derived strings.
- **Truncating to 100 words post-generation:** The requirement (TECH-03, OUT-04) says enforce via template design, not truncation. Templates must be written short. A `wc()` assertion during development catches drift.
- **Three templates that are the same tone:** The most common failure mode. Direct = declarative statement + CTA. Curious = question-led + intrigue. Story = narrative opener + single data point. These need genuinely different sentence structures, not just reworded copy.
- **Requiring all four fields to generate:** Gracefully handle partial input (e.g. `name || 'there'`, `company || 'your company'`) so the tool doesn't block on empty optional fields.

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Clipboard copy | Custom `execCommand` fallback | `navigator.clipboard.writeText()` | execCommand is deprecated since Chrome 70; clipboard API has 97%+ browser support as of 2026 |
| Word count | Complex NLP | Simple `str.trim().split(/\s+/).filter(Boolean).length` | Sufficient for < 200 word bodies; already pattern-proven in scorer's `wc()` helper |
| CSS animations | JS-based animation library | CSS `@keyframes fadeIn` + class toggle | Zero dependency, already in scorer |
| Form reset | Custom field-clearing loop | `form.reset()` OR individual `.value = ''` | Both work; `form.reset()` is one line if inputs are inside a `<form>` element |

**Key insight:** The scorer already solved every UI/UX mechanic this tool needs. The only genuinely new work is the three tone templates — invest creative energy there.

---

## Common Pitfalls

### Pitfall 1: Templates That Read as the Same Tone

**What goes wrong:** All three variants use similar sentence openers ("I noticed...", "I saw...", "I wanted to...") and the salesperson can't tell which is which without reading the label.

**Why it happens:** Focusing on the data interpolation and forgetting the structural tone differentiation.

**How to avoid:** Each tone needs a distinct OPENER structure:
- Direct: declarative statement of fact, no question, hard CTA
- Curious: opens WITH a question, withholds resolution, soft close
- Story: "A [role] at [type of company]..." micro-narrative opener

**Warning signs:** If you can swap the label from "Direct" to "Curious" and the body still feels appropriate, the templates aren't differentiated enough.

### Pitfall 2: Template Creep Over 100 Words

**What goes wrong:** Template feels good in isolation but word count is 110–120 words.

**Why it happens:** Adding qualifiers, softeners, or extra context sentences.

**How to avoid:** Write the template, count with `wc()`, cut until under 100. Aim for 65–80 words — under 100 is the ceiling, not the target.

**Warning signs:** More than 4 sentences in a template is a red flag.

### Pitfall 3: Clipboard API Fails Silently on `file://` Protocol

**What goes wrong:** `navigator.clipboard.writeText()` throws a `NotAllowedError` in some browsers when the page is opened as a local `file://` URL rather than `http://localhost`.

**Why it happens:** The Permissions Policy for clipboard write is restricted to secure origins. Chrome allows `file://` for clipboard in most versions, but Firefox may block it.

**How to avoid:** Test in Chrome first (most salespeople use Chrome). Add a `.catch()` handler that falls back to selecting the text in a temporary `<textarea>` so copy still works.

```javascript
navigator.clipboard.writeText(text).catch(() => {
  const ta = document.createElement('textarea');
  ta.value = text;
  document.body.appendChild(ta);
  ta.select();
  document.execCommand('copy');
  document.body.removeChild(ta);
});
```

**Warning signs:** Copy button shows "Copied!" but nothing lands on clipboard in Firefox.

### Pitfall 4: Generate Without Validation Produces Broken Output

**What goes wrong:** User clicks Generate with empty fields; template produces "Hi , I noticed  is dealing with ..." — looks broken.

**Why it happens:** No input guard before rendering.

**How to avoid:** Either (a) require at minimum one field and gracefully default the rest (`name || 'there'`, `company || 'your company'`, etc.), or (b) check all four are non-empty and highlight the first empty field. Option (a) is more forgiving and matches the tool's quick-use purpose.

### Pitfall 5: Results Not Clearing on Reset

**What goes wrong:** User clicks Reset, types new prospect info, clicks Generate — but the old results flash briefly then update, or worse, old copy button has stale `dataset.body`.

**Why it happens:** Reset only clears inputs but not the results DOM state.

**How to avoid:** Reset function must hide results AND clear `dataset.body` on all copy buttons (or results will be rebuilt from scratch each generate call — just ensure `renderVariant()` always overwrites, never appends).

---

## Code Examples

Verified patterns from existing codebase (`subject-line-scorer.html`):

### Design Token Reference (from scorer)

```css
/* Source: /Users/sachinrai/subject-line-scorer.html — existing design system */
background: #0f1117;        /* page background */
#1e2130                     /* card background */
#2d3348                     /* card border */
#6366f1                     /* indigo accent (buttons, active states) */
#5254cc                     /* indigo hover */
#f8fafc                     /* heading text */
#e2e8f0                     /* body text */
#94a3b8                     /* secondary text */
#64748b                     /* muted text / labels */
#10b981                     /* green (success / "Copied!") */
#ef4444                     /* red (error) */
border-radius: 14px         /* cards */
border-radius: 8px          /* inputs, buttons */
font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif
```

### Card Component (from scorer)

```css
/* Source: subject-line-scorer.html */
.card {
  background: #1e2130;
  border: 1px solid #2d3348;
  border-radius: 14px;
  padding: 24px;
}
.card-label {
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: #64748b;
  margin-bottom: 10px;
  display: block;
}
```

### Button Styles (from scorer)

```css
/* Source: subject-line-scorer.html */
button {
  background: #6366f1;
  color: #fff;
  border: none;
  border-radius: 8px;
  padding: 11px 20px;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s, transform 0.1s;
}
button:hover  { background: #5254cc; }
button:active { transform: scale(0.97); }
button.full   { width: 100%; padding: 13px; font-size: 15px; }

button.secondary {
  background: transparent;
  border: 1px solid #2d3348;
  color: #94a3b8;
  font-size: 12px;
  padding: 6px 12px;
}
.copied { color: #10b981 !important; transition: color 0.2s; }
```

### Fade-In Animation (from scorer)

```css
/* Source: subject-line-scorer.html */
.fade-in { animation: fadeIn 0.3s ease; }
@keyframes fadeIn {
  from { opacity: 0; transform: translateY(6px); }
  to   { opacity: 1; transform: translateY(0); }
}
```

### Word Count Helper (from scorer)

```javascript
// Source: subject-line-scorer.html — function wc()
function wc(s) {
  return s.trim().split(/\s+/).filter(Boolean).length;
}
```

---

## State of the Art

| Old Approach | Current Approach | Impact |
|--------------|-----------------|--------|
| `document.execCommand('copy')` | `navigator.clipboard.writeText()` | execCommand deprecated; use Promise-based API |
| `display:none` → `display:block` toggle | Same — still idiomatic | No change needed for a simple show/hide |
| `var` declarations | `const` / `let` | Use const/let; already used in scorer |

**Deprecated/outdated:**
- `document.execCommand('copy')`: Deprecated. Use `navigator.clipboard` with `.catch()` fallback for file:// edge case.

---

## Open Questions

1. **File naming convention**
   - What we know: Other files in the repo are named `subject-line-scorer.html`, `cocktails.html`, `recipe-italian-gravy.html`
   - What's unclear: Should the new file be `cold-email-generator.html` or something shorter?
   - Recommendation: Use `cold-email-generator.html` — descriptive, matches the pattern

2. **Textarea vs. text input for pain point and value prop**
   - What we know: Pain point and value prop can be multi-word phrases (e.g. "scaling SDR headcount without increasing ramp time")
   - What's unclear: Whether a single `<input type="text">` is long enough for these fields or if a `<textarea>` is better UX
   - Recommendation: Use `<input type="text">` for consistency with scorer and to keep the form compact. The placeholder can show a long example to set expectations.

3. **Reset behaviour: clear inputs only vs. reset + focus**
   - What we know: UX-01 requires easy reset
   - What's unclear: Whether clicking Reset should scroll back to the top of the form
   - Recommendation: Clear inputs, hide results, call `.focus()` on the first field — same reset flow as any web form

---

## Validation Architecture

### Test Framework

| Property | Value |
|----------|-------|
| Framework | None detected — this is a single HTML file with no test infrastructure |
| Config file | None — see Wave 0 |
| Quick run command | Open `cold-email-generator.html` in Chrome, run manual smoke test |
| Full suite command | Same — no automated test runner applicable for a zero-dependency HTML file |

**Note:** Automated unit testing of vanilla-JS HTML files is possible with Playwright or a simple Node test harness, but the project has no existing test infrastructure and the requirements do not specify one. The appropriate validation for this phase is a structured manual smoke test checklist.

### Phase Requirements → Test Map

| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| INP-01 through INP-05 | Form accepts inputs and Generate fires | Manual smoke | Open file, fill all 4 fields, click Generate | Wave 0 |
| OUT-01 | Direct email body appears | Manual smoke | Verify "Direct" card renders with prospect name | Wave 0 |
| OUT-02 | Curious email body appears | Manual smoke | Verify "Curious" card renders with prospect name | Wave 0 |
| OUT-03 | Story-Led email body appears | Manual smoke | Verify "Story-Led" card renders with prospect name | Wave 0 |
| OUT-04 | Each body under 100 words | Template design assertion | `wc(directTemplate(...)) < 100` in browser console | Wave 0 |
| OUT-05 | Copy button works | Manual smoke | Click copy on each variant, paste into text editor | Wave 0 |
| OUT-06 | "Copied!" confirmation appears | Manual smoke | Verify button label changes for 1.5 seconds | Wave 0 |
| UX-01 | Reset clears form and hides results | Manual smoke | Fill form, generate, click Reset, verify state | Wave 0 |
| UX-02 | Each tone visually labelled | Visual inspection | Labels visible on rendered cards | Wave 0 |
| UX-03 | Clean laptop layout | Visual inspection | No overflow, readable at 1280px viewport | Wave 0 |
| TECH-01 | Single self-contained HTML file | File check | `wc -l cold-email-generator.html` > 0, no external JS | Wave 0 |
| TECH-02 | No API calls, works offline | Network tab | Open DevTools Network, generate — 0 requests | Wave 0 |
| TECH-03 | Word count by template design | Template review | Read template source, count words manually | Wave 0 |

### Sampling Rate

- **Per task commit:** Open file in Chrome, generate one output, verify all 3 variants appear and copy works
- **Per wave merge:** Run full manual smoke checklist above
- **Phase gate:** All smoke checks pass before `/gsd:verify-work`

### Wave 0 Gaps

- [ ] `cold-email-generator.html` — the file itself does not exist yet (entire output of this phase)
- [ ] No test framework needed — manual smoke checklist above is the validation mechanism

---

## Sources

### Primary (HIGH confidence)

- `/Users/sachinrai/subject-line-scorer.html` — direct predecessor; design tokens, card CSS, button CSS, copy-to-clipboard implementation, fade-in animation, `wc()` helper all sourced from here
- `/Volumes/ORICO/Claude/gsd-practice/.planning/REQUIREMENTS.md` — authoritative requirements; all 17 IDs directly referenced
- MDN Web Docs — `navigator.clipboard.writeText()` API (browser-native, no version concerns)

### Secondary (MEDIUM confidence)

- `/Volumes/ORICO/Claude/gsd-practice/.planning/PROJECT.md` — confirmed constraints (single HTML file, template-based, vanilla JS)
- `/Volumes/ORICO/Claude/gsd-practice/.planning/STATE.md` — confirmed locked decisions (no API, single file)

### Tertiary (LOW confidence)

- Browser compatibility note on `navigator.clipboard` with `file://` protocol — based on general browser behaviour knowledge; recommend testing in target browser (Chrome) to confirm

---

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — confirmed by existing codebase (scorer), no external dependencies
- Architecture: HIGH — directly derived from predecessor file with identical requirements shape
- Pitfalls: HIGH for template tone differentiation and clipboard (observed in predecessor); MEDIUM for file:// clipboard edge case
- Template copywriting: MEDIUM — the three tone templates will need iteration; quality is a craft judgment

**Research date:** 2026-03-14
**Valid until:** Stable indefinitely — vanilla HTML/JS/CSS APIs do not change on short timescales
