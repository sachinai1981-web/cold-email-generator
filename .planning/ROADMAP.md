# Roadmap: Cold Email Body Generator

## Overview

This project is a single HTML file. All 17 requirements converge on one deliverable: a working tool that accepts four inputs and produces three ready-to-use email bodies. There is no meaningful delivery boundary to split — the tool either works or it doesn't. One phase, one ship.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [x] **Phase 1: Build** - Complete cold email body generator — inputs, template engine, 3 tone variants, copy buttons, and UX polish (completed 2026-03-15)

## Phase Details

### Phase 1: Build
**Goal**: A salesperson can open the HTML file in any browser, enter prospect details, and instantly get 3 ready-to-use cold email bodies they can copy with one click
**Depends on**: Nothing (first phase)
**Requirements**: INP-01, INP-02, INP-03, INP-04, INP-05, OUT-01, OUT-02, OUT-03, OUT-04, OUT-05, OUT-06, UX-01, UX-02, UX-03, TECH-01, TECH-02, TECH-03
**Success Criteria** (what must be TRUE):
  1. User fills in prospect name, company, pain point, and value prop, clicks Generate, and all 3 email bodies appear immediately with no errors
  2. Each of the 3 variants (Direct, Curious, Story-Led) reads as a genuinely different tone — not just reworded copy
  3. Every generated email body is under 100 words by design (template structure, not truncation)
  4. User clicks the copy button on any variant and the text lands on their clipboard; the button shows "Copied!" confirmation
  5. User can clear the form and generate for a new prospect without refreshing the page
**Plans**: 2 plans

Plans:
- [ ] 01-01-PLAN.md — Build cold-email-generator.html (HTML structure, CSS, JS template engine, copy/reset)
- [ ] 01-02-PLAN.md — Human verification: run full smoke checklist, confirm tone quality and clipboard behaviour

## Progress

**Execution Order:**
Phases execute in numeric order: 1

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Build | 2/2 | Complete   | 2026-03-15 |
