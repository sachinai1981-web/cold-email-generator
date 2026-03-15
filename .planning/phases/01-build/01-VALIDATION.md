---
phase: 1
slug: build
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-14
---

# Phase 1 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | None — single HTML file, no test runner |
| **Config file** | none — Wave 0 installs smoke test checklist |
| **Quick run command** | Open `cold-email-generator.html` in Chrome, fill all 4 fields, click Generate |
| **Full suite command** | Run full manual smoke checklist (all rows below) |
| **Estimated runtime** | ~2 minutes (manual) |

---

## Sampling Rate

- **After every task commit:** Open file in Chrome, generate one output, verify all 3 variants appear and copy works
- **After every plan wave:** Run full manual smoke checklist
- **Before `/gsd:verify-work`:** All smoke checks must pass
- **Max feedback latency:** ~2 minutes

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| 1-01-01 | 01 | 1 | INP-01–05 | Manual smoke | Fill all 4 fields, click Generate — verify 3 variants appear | ❌ W0 | ⬜ pending |
| 1-01-02 | 01 | 1 | OUT-01 | Manual smoke | Verify "Direct" card renders with prospect name | ❌ W0 | ⬜ pending |
| 1-01-03 | 01 | 1 | OUT-02 | Manual smoke | Verify "Curious" card renders with prospect name | ❌ W0 | ⬜ pending |
| 1-01-04 | 01 | 1 | OUT-03 | Manual smoke | Verify "Story-Led" card renders with prospect name | ❌ W0 | ⬜ pending |
| 1-01-05 | 01 | 1 | OUT-04 | Template assertion | `wc(directTemplate(...)) < 100` in browser console | ❌ W0 | ⬜ pending |
| 1-01-06 | 01 | 1 | OUT-05 | Manual smoke | Click copy on each variant, paste into text editor | ❌ W0 | ⬜ pending |
| 1-01-07 | 01 | 1 | OUT-06 | Manual smoke | Verify button label changes to "Copied!" for 1.5s | ❌ W0 | ⬜ pending |
| 1-01-08 | 01 | 1 | UX-01 | Manual smoke | Fill form, generate, click Reset — verify inputs cleared and results hidden | ❌ W0 | ⬜ pending |
| 1-01-09 | 01 | 1 | UX-02 | Visual inspection | Labels "Direct", "Curious", "Story-Led" visible on rendered cards | ❌ W0 | ⬜ pending |
| 1-01-10 | 01 | 1 | UX-03 | Visual inspection | No overflow, readable at 1280px viewport | ❌ W0 | ⬜ pending |
| 1-01-11 | 01 | 1 | TECH-01 | File check | File is self-contained: no external JS imports, all CSS/JS inline | ❌ W0 | ⬜ pending |
| 1-01-12 | 01 | 1 | TECH-02 | Network tab | DevTools Network: 0 requests on Generate | ❌ W0 | ⬜ pending |
| 1-01-13 | 01 | 1 | TECH-03 | Template review | Read template source — each template ≤ 100 words by construction | ❌ W0 | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

- [ ] `cold-email-generator.html` — the file itself does not exist yet (entire output of this phase)
- [ ] No automated test framework needed — manual smoke checklist above is the validation mechanism

*Note: All verification for this phase is manual. The project has no existing test infrastructure and requirements specify a single zero-dependency HTML file. Playwright could automate this in a future phase.*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| All 3 email variants appear on Generate | INP-05, OUT-01–03 | Single HTML file, no test runner | Open file in Chrome, fill fields, click Generate, confirm 3 cards appear |
| Each tone reads distinctly different | OUT-01–03 | Craft judgment — no automated tone checker | Read Direct, Curious, Story-Led outputs; confirm opener structure differs (statement vs. question vs. story) |
| Every body under 100 words | OUT-04, TECH-03 | Templates by design; manual count | Run `wc(body)` in browser console for each variant |
| Copy lands on clipboard | OUT-05, OUT-06 | Clipboard API; needs real browser | Click copy, paste into text editor, confirm text matches |
| Reset clears all state | UX-01 | DOM state — needs visual confirm | Generate, click Reset, verify inputs empty and results hidden |
| Works fully offline | TECH-02 | Network isolation test | Open file with no internet, generate — confirm no network errors |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 120s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
