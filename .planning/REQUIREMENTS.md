# Requirements: Cold Email Body Generator

**Defined:** 2026-03-14
**Core Value:** Salespeople get 3 ready-to-use, personalized cold email bodies in seconds — without staring at a blank page.

## v1 Requirements

### Inputs

- [ ] **INP-01**: User can enter a prospect's first name or full name
- [ ] **INP-02**: User can enter a prospect's company name
- [ ] **INP-03**: User can enter a pain point the prospect is likely experiencing
- [ ] **INP-04**: User can enter their value proposition
- [ ] **INP-05**: User can click a single Generate button to produce all 3 variants at once

### Output

- [ ] **OUT-01**: Page displays a Direct-tone email body using the entered inputs
- [ ] **OUT-02**: Page displays a Curious-tone email body using the entered inputs
- [ ] **OUT-03**: Page displays a Story-Led-tone email body using the entered inputs
- [ ] **OUT-04**: Each email body is under 100 words (enforced by template design)
- [ ] **OUT-05**: Each email body has a one-click copy-to-clipboard button
- [ ] **OUT-06**: Copy button provides visual confirmation (e.g. "Copied!" feedback)

### UX

- [ ] **UX-01**: Form clears or resets easily so the user can generate for a new prospect
- [ ] **UX-02**: Each tone variant is visually labelled (Direct / Curious / Story-Led)
- [ ] **UX-03**: Layout is clean and usable on a laptop screen

### Technical

- [ ] **TECH-01**: Entire tool is a single self-contained HTML file (no external dependencies except fonts/CSS CDN)
- [ ] **TECH-02**: Generation is template-based — no API calls, works fully offline
- [ ] **TECH-03**: Word count is enforced by template structure, not truncation

## v2 Requirements

### Enhancements

- **ENH-01**: AI-powered generation via Claude API for more natural variation
- **ENH-02**: Subject line generation alongside body
- **ENH-03**: Save / copy history across sessions
- **ENH-04**: Custom tone editor (add your own tone templates)

## Out of Scope

| Feature | Reason |
|---------|--------|
| Subject line generation | Body only — keeps scope tight for v1 |
| AI/LLM API calls | Template-based is deliberate — offline, instant, zero cost |
| Save/history | Single session use — no backend needed |
| Mobile layout | Laptop/desktop use case only for v1 |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| INP-01 | Phase 1 | Pending |
| INP-02 | Phase 1 | Pending |
| INP-03 | Phase 1 | Pending |
| INP-04 | Phase 1 | Pending |
| INP-05 | Phase 1 | Pending |
| OUT-01 | Phase 1 | Pending |
| OUT-02 | Phase 1 | Pending |
| OUT-03 | Phase 1 | Pending |
| OUT-04 | Phase 1 | Pending |
| OUT-05 | Phase 1 | Pending |
| OUT-06 | Phase 1 | Pending |
| UX-01 | Phase 1 | Pending |
| UX-02 | Phase 1 | Pending |
| UX-03 | Phase 1 | Pending |
| TECH-01 | Phase 1 | Pending |
| TECH-02 | Phase 1 | Pending |
| TECH-03 | Phase 1 | Pending |

**Coverage:**
- v1 requirements: 17 total
- Mapped to phases: 17
- Unmapped: 0 ✓

---
*Requirements defined: 2026-03-14*
*Last updated: 2026-03-14 after initial definition*
