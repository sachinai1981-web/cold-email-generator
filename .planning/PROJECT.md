# Cold Email Body Generator

## What This Is

A single standalone HTML file that generates cold email bodies. The user enters a prospect name, company, pain point, and their value proposition — and instantly gets 3 email body options in distinct tones (direct, curious, story-led), each under 100 words. No backend, no API, works in any browser.

## Core Value

Salespeople get 3 ready-to-use, personalized cold email bodies in seconds — without staring at a blank page.

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] User can enter: prospect name, company name, pain point, and value proposition
- [ ] Page generates 3 email body variants: Direct, Curious, and Story-Led
- [ ] Each generated email body is under 100 words
- [ ] Each email body has a one-click copy button
- [ ] Generation is template-based (no API, no backend, works offline)
- [ ] Everything lives in a single HTML file

### Out of Scope

- AI/LLM API calls — template-based is deliberate, keeps it fast and offline
- Subject line generation — body only
- Save/history functionality — single session use
- Multiple value props or A/B testing beyond the 3 tones

## Context

- User is a sales professional (Freshworks) who sends cold emails regularly
- Tool is for personal use — no auth, no backend needed
- Existing files in the directory are unrelated HTML projects (cocktails, recipes, etc.)
- Design should match the quality of other standalone HTML files in the repo

## Constraints

- **Tech stack**: Single HTML file — no frameworks, no build step, vanilla JS + CSS
- **Tone accuracy**: Each of the 3 templates must feel genuinely different — not just reworded
- **Word count**: Hard cap of 100 words per generated body (enforce via template design)

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|----|
| Template-based generation | No API dependency, instant, offline-capable | — Pending |
| Single HTML file | Matches existing project pattern, zero setup | — Pending |

---
*Last updated: 2026-03-14 after initialization*
