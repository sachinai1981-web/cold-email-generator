---
phase: quick
plan: 260315-qji
type: execute
wave: 1
depends_on: []
files_modified:
  - /Volumes/ORICO/Claude/gsd-practice/cold-email-generator.html
autonomous: true
requirements:
  - Add optional AI generation path using Anthropic API (claude-haiku-4-5-20251001)
  - API key stored in localStorage, loaded on page init
  - Falls back to template engine when no key present
  - Output format identical to current template output
  - Single HTML file, no build step, no external dependencies

must_haves:
  truths:
    - "User can paste API key into a password field and it persists across page refresh"
    - "With a valid API key, clicking Generate calls Claude API and renders 3 variants"
    - "Without an API key, clicking Generate uses the existing template engine unchanged"
    - "On API error, the button re-enables and an inline error message explains what went wrong"
    - "Resetting the form does not clear the stored API key"
  artifacts:
    - path: "/Volumes/ORICO/Claude/gsd-practice/cold-email-generator.html"
      provides: "Single-file tool with AI + template dual-path generation"
      contains: "generateWithAI, generateWithTemplates, localStorage.setItem('anthropic_api_key')"
  key_links:
    - from: "inp-api-key input"
      to: "localStorage"
      via: "input event listener → localStorage.setItem"
      pattern: "localStorage.setItem.*anthropic_api_key"
    - from: "generate()"
      to: "generateWithAI() or generateWithTemplates()"
      via: "localStorage.getItem('anthropic_api_key') branch"
      pattern: "localStorage.getItem.*anthropic_api_key"
    - from: "generateWithAI()"
      to: "api.anthropic.com/v1/messages"
      via: "fetch with required CORS header"
      pattern: "anthropic-dangerous-direct-browser-access"
    - from: "API JSON response"
      to: "renderVariant()"
      via: "JSON.parse → parsed.direct / parsed.curious / parsed.story"
      pattern: "JSON.parse.*renderVariant"
---

<objective>
Add an optional AI generation path to cold-email-generator.html. When the user supplies an Anthropic API key, clicking Generate calls claude-haiku-4-5-20251001 instead of the JS template engine. The API key is stored in localStorage and loaded automatically on page init. When no key is present, the tool behaves exactly as it does today. Output format (three cards, subject + body) is identical between both paths.

Purpose: Upgrade a useful offline tool to produce genuinely personalized, AI-written cold emails when the user has an API key — while preserving the instant, offline fallback for key-free use.
Output: Modified cold-email-generator.html with AI Mode section, dual generate() path, error handling, and loading state.
</objective>

<execution_context>
@/Users/sachinrai/.claude/get-shit-done/workflows/execute-plan.md
@/Users/sachinrai/.claude/get-shit-done/templates/summary.md
</execution_context>

<context>
File to modify: /Volumes/ORICO/Claude/gsd-practice/cold-email-generator.html

<interfaces>
<!-- Existing JS surface the executor will modify. Extracted from the source file. -->

Existing IIFE structure (lines 203–337):
- FIELDS array — 4 input field IDs
- directTemplate(name, company, pain, value) → { subject, body }
- curiousTemplate(name, company, pain, value) → { subject, body }
- storyTemplate(name, company, pain, value) → { subject, body }
- getFieldValues() → [name, company, pain, value]
- validate() → boolean (adds .error class on blank fields)
- clearErrors()
- renderVariant(cardId, { subject, body }) — uses textContent, XSS-safe
- showCopied(btn), copyToClipboard(text)
- generate() — synchronous, calls 3 templates then shows #results
- resetForm() — clears fields + hides #results
- Event wiring at bottom: btn-generate click, btn-reset click, Enter key, copy buttons

DOM anchor points:
- #btn-generate — the Generate button (line 172)
- #btn-reset — Reset button (line 173)
- #results — output container (line 176)
- input[type=text] selector used for keydown/input listeners (line 329)

CSS classes available:
- .card — dark bordered card style
- .secondary — ghost button style
- input[type=text] — styled inputs (password type will need same styles)
</interfaces>

Research findings (from 260315-qji-RESEARCH.md):
- API endpoint: POST https://api.anthropic.com/v1/messages
- Required headers: Content-Type, x-api-key, anthropic-version: 2023-06-01, anthropic-dangerous-direct-browser-access: true
- Model: claude-haiku-4-5-20251001
- max_tokens: 600 (sufficient for 3 short emails)
- Response text: data.content[0].text
- Request JSON format and prompt template defined in research file
- localStorage key: 'anthropic_api_key'
- Error handling: 401 → invalid key message, 429 → rate limit message, JSON parse fail → fallback to templates
</context>

<tasks>

<task type="auto">
  <name>Task 1: Add AI Mode UI section and CSS</name>
  <files>/Volumes/ORICO/Claude/gsd-practice/cold-email-generator.html</files>
  <action>
    Make two additions to the HTML:

    1. CSS — In the &lt;style&gt; block, add input[type=password] to the existing input[type=text] rule so it inherits the same styling. Also add styles for #ai-error (font-size:12px, color:#ef4444, margin-top:8px, display:none) and for the ai-badge span (.ai-badge: font-size:11px, color:#10b981, font-weight:600, margin-left:8px).

    2. HTML — Between the #inp-value input and the #btn-generate button (i.e. after line 170, before line 172), insert:

    ```html
    <details id="ai-settings" style="margin-top:20px;">
      <summary style="cursor:pointer; font-size:12px; color:#94a3b8; font-weight:600; list-style:none; display:flex; align-items:center; gap:6px;">
        <span>&#9654;</span> AI Mode <span style="font-weight:400; color:#64748b;">(optional)</span>
      </summary>
      <div style="margin-top:12px;">
        <label for="inp-api-key" style="margin-top:0;">Anthropic API Key</label>
        <input type="password" id="inp-api-key" placeholder="sk-ant-...">
        <p style="font-size:11px; color:#64748b; margin-top:6px; line-height:1.5;">
          Stored in localStorage only. Sent exclusively to api.anthropic.com. Visible in browser DevTools.
        </p>
      </div>
    </details>
    <p id="ai-error"></p>
    ```

    The #ai-error paragraph sits between the &lt;/details&gt; and the Generate button — it will be shown/hidden by JS.
  </action>
  <verify>Open cold-email-generator.html in browser. A collapsed "AI Mode (optional)" disclosure element appears between the Value Prop field and the Generate button. Expanding it shows a password-masked input field and the disclaimer text. The input inherits the same dark styling as the other text inputs.</verify>
  <done>AI Mode section renders correctly; password input styled consistently; #ai-error element present in DOM.</done>
</task>

<task type="auto">
  <name>Task 2: Implement dual-path generate() with AI fetch logic</name>
  <files>/Volumes/ORICO/Claude/gsd-practice/cold-email-generator.html</files>
  <action>
    Modify the IIFE script block. All changes are inside the existing IIFE — do not break the closure.

    STEP A — Load saved key on init. After the FIELDS declaration (before the template functions), add:

    ```javascript
    const API_KEY_STORAGE = 'anthropic_api_key';
    ```

    STEP B — Rename existing generate() body into generateWithTemplates():

    ```javascript
    function generateWithTemplates(name, company, pain, value) {
      renderVariant('card-direct',  directTemplate(name, company, pain, value));
      renderVariant('card-curious', curiousTemplate(name, company, pain, value));
      renderVariant('card-story',   storyTemplate(name, company, pain, value));
    }
    ```

    STEP C — Add generateWithAI() as an async function:

    ```javascript
    async function generateWithAI(name, company, pain, value, apiKey) {
      const btn = document.getElementById('btn-generate');
      const errEl = document.getElementById('ai-error');
      errEl.style.display = 'none';
      btn.disabled = true;
      btn.textContent = 'Generating with AI...';

      try {
        const resp = await fetch('https://api.anthropic.com/v1/messages', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'x-api-key': apiKey,
            'anthropic-version': '2023-06-01',
            'anthropic-dangerous-direct-browser-access': 'true',
          },
          body: JSON.stringify({
            model: 'claude-haiku-4-5-20251001',
            max_tokens: 600,
            system: 'You are a cold email copywriter for B2B SaaS sales. Write concise, human-sounding cold emails under 100 words each. Never use buzzwords or empty phrases. Each email must feel genuinely different in tone and structure.',
            messages: [{
              role: 'user',
              content: `Generate 3 cold email variants for this prospect:\n- Prospect first name: ${name}\n- Their company: ${company}\n- Their pain point: ${pain}\n- My value proposition: ${value}\n\nReturn ONLY valid JSON in this exact format, no other text:\n{"direct":{"subject":"...","body":"..."},"curious":{"subject":"...","body":"..."},"story":{"subject":"...","body":"..."}}\n\nRules:\n- Each body must be under 100 words\n- Direct: confident opener, state the problem, offer solution, ask for 15 mins\n- Curious: open with a genuine question, reference the pain, offer insight, soft CTA\n- Story-Led: lead with a customer story, tie to their situation, invite them to learn more\n- Do not use filler phrases like "I hope this finds you well"\n- Address the prospect by first name in the opening line`,
            }],
          }),
        });

        if (resp.status === 401) {
          errEl.textContent = 'Invalid API key. Check your key or remove it to use template mode.';
          errEl.style.display = 'block';
          generateWithTemplates(name, company, pain, value);
          return;
        }
        if (resp.status === 429) {
          errEl.textContent = 'Rate limit reached. Try again in a moment.';
          errEl.style.display = 'block';
          generateWithTemplates(name, company, pain, value);
          return;
        }
        if (!resp.ok) {
          errEl.textContent = `API error (${resp.status}). Using template mode instead.`;
          errEl.style.display = 'block';
          generateWithTemplates(name, company, pain, value);
          return;
        }

        const data = await resp.json();
        const raw = data.content[0].text.trim();
        let parsed;
        try {
          parsed = JSON.parse(raw);
        } catch (_) {
          errEl.textContent = 'AI output was unexpected — used templates instead.';
          errEl.style.display = 'block';
          generateWithTemplates(name, company, pain, value);
          return;
        }

        renderVariant('card-direct',  parsed.direct);
        renderVariant('card-curious', parsed.curious);
        renderVariant('card-story',   parsed.story);

        // Show AI badge on first card label
        const firstLabel = document.querySelector('#card-direct .card-label');
        if (firstLabel && !firstLabel.querySelector('.ai-badge')) {
          const badge = document.createElement('span');
          badge.className = 'ai-badge';
          badge.textContent = '· AI';
          firstLabel.appendChild(badge);
        }

      } catch (err) {
        errEl.textContent = 'Could not reach Anthropic API. Check your connection.';
        errEl.style.display = 'block';
        generateWithTemplates(name, company, pain, value);
      } finally {
        btn.disabled = false;
        btn.textContent = 'Generate Emails';
      }
    }
    ```

    STEP D — Replace the existing generate() with an async version that branches:

    ```javascript
    async function generate() {
      if (!validate()) return;
      clearErrors();

      const [name, company, pain, value] = getFieldValues();
      const apiKey = localStorage.getItem(API_KEY_STORAGE);

      if (apiKey) {
        await generateWithAI(name, company, pain, value, apiKey);
      } else {
        generateWithTemplates(name, company, pain, value);
      }

      const results = document.getElementById('results');
      results.style.display = 'block';
      results.classList.remove('fade-in');
      void results.offsetWidth;
      results.classList.add('fade-in');
      results.scrollIntoView({ behavior: 'smooth', block: 'nearest' });
    }
    ```

    STEP E — Add localStorage wiring and init in the event wiring section (before or after existing event listeners):

    ```javascript
    // API key persistence
    const apiKeyInput = document.getElementById('inp-api-key');
    const savedKey = localStorage.getItem(API_KEY_STORAGE) || '';
    if (savedKey) apiKeyInput.value = savedKey;
    apiKeyInput.addEventListener('input', () => {
      const v = apiKeyInput.value.trim();
      if (v) localStorage.setItem(API_KEY_STORAGE, v);
      else localStorage.removeItem(API_KEY_STORAGE);
    });
    ```

    Note: Do NOT clear the API key in resetForm(). The key persists across resets by design — the user deliberately stored it.

    Note: The existing event listener `document.getElementById('btn-generate').addEventListener('click', generate)` remains unchanged — the async generate() is a drop-in replacement.

    Note: The Enter key listener uses `input[type=text]` selector — this correctly excludes the password input, which is intentional (pressing Enter in a password field should not trigger generation without the user clicking the button).
  </action>
  <verify>
    Test path 1 (no API key): Open file in browser, fill all 4 fields, click Generate — 3 cards appear instantly via templates, no AI badge, no loading state.
    Test path 2 (with API key): Paste a valid Anthropic API key into the AI Mode field, reload — key is pre-filled. Fill 4 fields, click Generate — button shows "Generating with AI...", then 3 cards appear with AI-written content, AI badge visible on Direct card.
    Test path 3 (invalid key): Enter a fake key, click Generate — error message appears above Generate button, 3 template fallback cards render.
    Test path 4 (persistence): After saving key and reloading, the key is still pre-populated in the field.
  </verify>
  <done>Both generate paths work. AI path calls API and renders results. Template path unchanged. Errors surface inline with fallback to templates. API key persists across reload. Reset does not clear key.</done>
</task>

</tasks>

<verification>
Open /Volumes/ORICO/Claude/gsd-practice/cold-email-generator.html in a browser (file:// protocol):

1. With no API key — Generate produces 3 cards instantly, identical to before this change.
2. With a valid API key in the AI Mode field — Generate shows loading state, then produces 3 AI-written cards with "· AI" badge.
3. With an invalid API key — Generate shows an inline error and still produces 3 template cards (no broken state).
4. Reload after saving a key — password field is pre-filled.
5. Click Reset — form clears, results hide, API key field still populated.
</verification>

<success_criteria>
- AI Mode disclosure section renders between Value Prop and Generate button
- Password input styled identically to text inputs
- Valid API key → Claude API called, 3 AI cards rendered
- No API key → template engine used, behavior unchanged from before
- Invalid key / network error → inline error shown, template fallback rendered, button re-enabled
- API key persists in localStorage across page refresh
- Single HTML file, no new dependencies, no build step
</success_criteria>

<output>
After completion, create /Volumes/ORICO/Claude/gsd-practice/.planning/quick/260315-qji-add-optional-ai-generation-to-cold-email/260315-qji-SUMMARY.md
</output>
