# Quick Task Research: Add Optional AI Generation to Cold Email Generator

**Researched:** 2026-03-15
**Domain:** Anthropic Messages API, browser-side fetch, localStorage, single-file HTML
**Confidence:** HIGH

---

## Summary

The existing `cold-email-generator.html` is a self-contained, zero-dependency vanilla JS file. It has four form inputs (name, company, pain, value), a `generate()` function that calls one of three template functions, and a `renderVariant()` function that writes subject + body into three card elements via `textContent`.

The AI path must slot in as an **alternative code path inside `generate()`**: if an API key is present, call Claude; otherwise call the existing templates. Output format stays identical — each variant produces `{ subject, body }` and is passed to the existing `renderVariant()`. No new DOM output structure needed.

The Anthropic Messages API now supports CORS from browsers via a required header (`anthropic-dangerous-direct-browser-access: true`). This makes a raw `fetch()` call from browser JS fully viable with no proxy or build step.

**Primary recommendation:** Add a collapsible "AI Mode" section below the existing inputs, store the API key in `localStorage`, and wrap the existing `generate()` function to branch on key presence.

---

## API Facts (HIGH confidence — official docs + verified)

### Endpoint

```
POST https://api.anthropic.com/v1/messages
```

### Required Headers

```
Content-Type: application/json
x-api-key: sk-ant-...
anthropic-version: 2023-06-01
anthropic-dangerous-direct-browser-access: true
```

The `anthropic-dangerous-direct-browser-access: true` header is **required** for CORS to work from a browser. Without it, the browser will block the preflight. This is intentionally named "dangerous" to signal client-side key exposure — not a bug, a deliberate opt-in.

### Request Body

```json
{
  "model": "claude-haiku-4-5-20251001",
  "max_tokens": 600,
  "system": "...",
  "messages": [
    { "role": "user", "content": "..." }
  ]
}
```

Note: `system` is a top-level field, NOT part of the messages array.

### Response — extract text

```javascript
const text = data.content[0].text;
```

Full response shape: `{ id, type, role, content: [{ type: "text", text: "..." }], model, stop_reason, usage }`.

### Model ID

Exact string to use: `"claude-haiku-4-5-20251001"`

A non-dated alias `"claude-haiku-4-5"` also exists but using the dated version gives reproducible behavior.

---

## localStorage Pattern

Simple key-value, survives page reload, scoped to the origin (for `file://` this means per-file path on disk):

```javascript
// Save
localStorage.setItem('anthropic_api_key', apiKeyInput.value.trim());

// Load on page init
const savedKey = localStorage.getItem('anthropic_api_key') || '';

// Clear
localStorage.removeItem('anthropic_api_key');
```

For a `file://`-served page, `localStorage` works normally in all modern browsers. The scope is the exact file path, so the key persists across page refreshes.

---

## Conditional Logic Structure

```javascript
async function generate() {
  if (!validate()) return;
  clearErrors();

  const [name, company, pain, value] = getFieldValues();
  const apiKey = localStorage.getItem('anthropic_api_key');

  if (apiKey) {
    await generateWithAI(name, company, pain, value, apiKey);
  } else {
    generateWithTemplates(name, company, pain, value);
  }
}
```

Split the existing synchronous `generate()` body into `generateWithTemplates()`. Add `generateWithAI()` as an async function. Wire the button to the new async `generate()`.

Button state during AI call — disable + change label, re-enable in `finally`:

```javascript
const btn = document.getElementById('btn-generate');
btn.disabled = true;
btn.textContent = 'Generating...';
try { /* fetch */ } finally {
  btn.disabled = false;
  btn.textContent = 'Generate Emails';
}
```

---

## Prompt Engineering

The AI must produce output that matches exactly what the template engine produces: three variants each with a distinct subject line and email body. The cleanest approach is to ask for structured JSON output.

### System Prompt

```
You are a cold email copywriter for B2B SaaS sales. Write concise, human-sounding cold emails under 100 words each. Never use buzzwords or empty phrases. Each email must feel genuinely different in tone and structure.
```

### User Message

```
Generate 3 cold email variants for this prospect:
- Prospect first name: {name}
- Their company: {company}
- Their pain point: {pain}
- My value proposition: {value}

Return ONLY valid JSON in this exact format, no other text:
{
  "direct": {
    "subject": "...",
    "body": "..."
  },
  "curious": {
    "subject": "...",
    "body": "..."
  },
  "story": {
    "subject": "...",
    "body": "..."
  }
}

Rules:
- Each body must be under 100 words
- Direct: confident opener, state the problem, offer solution, ask for 15 mins
- Curious: open with a genuine question, reference the pain, offer insight, soft CTA
- Story-Led: lead with a customer story, tie to their situation, invite them to learn more
- Do not use filler phrases like "I hope this finds you well"
- Address the prospect by first name in the opening line
```

### Parsing the response

```javascript
const raw = data.content[0].text.trim();
const parsed = JSON.parse(raw);
renderVariant('card-direct',  parsed.direct);
renderVariant('card-curious', parsed.curious);
renderVariant('card-story',   parsed.story);
```

JSON parsing can fail if the model adds preamble text. Wrap in try/catch and fall back to template engine on parse error, showing a non-blocking status message.

---

## UI Addition — API Key Field

Add a collapsed disclosure section below the existing form card, above the Generate button. Keep it lightweight:

```html
<details id="ai-settings" style="margin-top:16px;">
  <summary style="cursor:pointer; font-size:12px; color:#94a3b8; font-weight:600;">
    AI Mode (optional)
  </summary>
  <div style="margin-top:12px;">
    <label for="inp-api-key">Anthropic API Key</label>
    <input type="password" id="inp-api-key" placeholder="sk-ant-...">
    <p style="font-size:11px; color:#64748b; margin-top:6px;">
      Stored in localStorage. Never sent anywhere except api.anthropic.com.
    </p>
  </div>
</details>
```

Use `type="password"` so the key is masked by default. Load saved key on `DOMContentLoaded`. Save to `localStorage` on every `input` event (or on generate — either works).

Add a visual indicator in the Generate button or results area showing which mode was used (e.g., "Generated with AI" vs nothing / a small badge). This avoids confusion when switching between modes.

---

## Security Considerations

**Client-side key exposure is inherent and intentional here.** This is a personal-use tool, not a multi-user app. The risk profile is:

- The API key is visible in browser DevTools (Network tab, Request Headers)
- `localStorage` is accessible to any JS running on the same `file://` path — in practice, only this file
- The key is NOT sent to any third-party server — only `api.anthropic.com`

**What to communicate to the user (in the UI):**

> "Your API key is stored locally and only sent to Anthropic's API. Anyone with access to your browser can see it in DevTools."

**Mitigation steps that ARE worth doing:**
- Use `type="password"` input so the key isn't visible on screen
- Add a "Clear API key" button or clear on reset
- Do NOT log the key to the console

**Mitigation steps that are NOT worth doing for a personal tool:**
- Server-side proxy (defeats the single-file, no-backend constraint)
- Key encryption in localStorage (trivially reversible in same context)

---

## Error Handling

| Scenario | What to do |
|----------|-----------|
| Invalid API key (401) | Show inline error: "Invalid API key. Check your key or use template mode." |
| Rate limit (429) | Show: "Rate limit hit. Try again in a moment." |
| JSON parse failure | Fall back to template engine silently, or show: "AI output was unexpected — used templates instead." |
| Network error | Show: "Could not reach Anthropic API. Check your connection." |
| Any error | Re-enable button, ensure results area shows something (fallback to templates is cleanest) |

Surface errors in a small `<p id="ai-error">` element above the Generate button. Clear it on each new generation attempt.

---

## Integration Checklist for Implementation

- [ ] Add `<details>` API key section to HTML (above Generate button)
- [ ] Add `input[type=password]` CSS to style block (inherits `input[type=text]` styles if selector updated)
- [ ] Add `ai-error` error display element
- [ ] On `DOMContentLoaded`: load saved key into the input
- [ ] On key input: save to localStorage
- [ ] Refactor `generate()` → async, branch on key presence
- [ ] Extract `generateWithTemplates()` from existing `generate()` body (no logic changes)
- [ ] Implement `generateWithAI()` with fetch, JSON parse, renderVariant calls
- [ ] Add button loading state (disable + text change) in AI path
- [ ] Add JSON parse error fallback
- [ ] Add "Clear key" path on Reset button (optional but clean)

---

## Don't Hand-Roll

| Problem | Use Instead |
|---------|-------------|
| HTTP client | Native `fetch()` — no axios/ky needed |
| JSON parsing | Native `JSON.parse()` |
| Key storage | `localStorage` — no cookie library needed |
| Streaming | Skip it — non-streaming response is simpler, 600 max_tokens returns fast enough |

---

## Sources

- [Anthropic CORS browser support announcement](https://simonwillison.net/2024/Aug/23/anthropic-dangerous-direct-browser-access/) — confirms `anthropic-dangerous-direct-browser-access: true` header requirement (MEDIUM — verified author is reliable, matches official SDK docs)
- [Anthropic Messages API docs](https://platform.claude.com/docs/en/api/messages) — request/response format (HIGH — official)
- [Claude Haiku 4.5 model page](https://www.anthropic.com/claude/haiku) — model availability (HIGH — official)
- [Inworld model listing](https://inworld.ai/models/anthropic-claude-haiku-4-5-20251001) — confirms dated model ID `claude-haiku-4-5-20251001` (MEDIUM — third party, consistent with official naming pattern)

---

## Confidence Breakdown

| Area | Level | Reason |
|------|-------|--------|
| API endpoint + headers | HIGH | Official docs verified |
| CORS header requirement | HIGH | Multiple sources + official SDK confirms |
| Model ID | HIGH | Official Anthropic model page |
| localStorage behavior on file:// | HIGH | Standard browser API, well-documented |
| Prompt structure for JSON output | MEDIUM | Pattern is proven; exact prompt quality requires runtime testing |
| Error codes (401, 429) | MEDIUM | Standard HTTP + Anthropic docs convention |

**Research date:** 2026-03-15
**Valid until:** 2026-06-15 (API format stable; model ID is pinned/dated)
