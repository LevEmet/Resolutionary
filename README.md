# Resolutionary
AI-guided conflict resolution for family disputes. Two parties, two private channels, one neutral Guide — nothing crosses without itemized checkbox consent. Not mediation; an educational self-help tool that produces court-ready points of agreement.

# Resolutionator

**Reach agreement without the war.**

Resolutionator is an AI-guided conflict resolution app for family disputes — divorce, custody, parenting time, child support, and property division. Two parties who cannot (or should not) negotiate face-to-face each talk privately with a neutral AI "Guide," which carries proposals between them using an explicit, itemized consent system. Nothing crosses from one side to the other unless the person it came from approved that exact item with a checkbox.

It is an educational self-help tool, **not mediation, not legal advice, not therapy**, designed and trained by a licensed marriage & family therapist and trained family law mediator. The output is a plain-English *Points of Agreement* summary the parties can take to attorney review and their state's free self-help court forms.

---

## How it works

```
Party A ──(private channel)──┐                     ┌──(private channel)── Party B
                             │      THE GUIDE      │
                             │  (one neutral AI    │
                             │   engine, two       │
                             │   conversations)    │
                             └──── SHARED BOARD ───┘
                              only checkbox-approved
                              content, visible to both
```

The design is **asynchronous shuttle resolution**: the parties never interact directly and never need to be online at the same time.

Three data stores with strict walls:

| Store | Contents | Who sees it |
|---|---|---|
| A's private channel | Everything Party A tells the Guide | A + the Guide only |
| B's private channel | Everything Party B tells the Guide | B + the Guide only |
| Shared board | Only checkbox-approved items | Both parties |

The Guide sees both channels (like a real shuttle facilitator) but can only *reveal* what was consented — and the printed summary is built exclusively from the shared board, so it can never contain anything that wasn't approved by both clicks.

## Core flow

1. **Case creation** — the initiating party picks a case type (divorce with dependents, divorce without dependents, or modifying an existing agreement) and the issues to resolve. Required issues (custody, placement, child support, finances in kid cases) are locked in automatically. Three secret links are generated: Party A, Party B, and the Owner dashboard.
2. **Safety screening** — each party is screened privately (privacy, violence/fear, coercion) before seeing *anything* the other party shared. The shared board stays locked per-party until their screen completes. DV disclosures trigger resources, a flag to the owner, and a proceed-at-own-choice path with DV-aware coaching.
3. **Intake and resolution order** — the Guide teaches the dependency order once (parenting time → child support via the state's official calculator → assets & maintenance) and holds it gently. Out-of-order requests are noted and parked; insistence gets one time-cost warning, then autonomy wins.
4. **Consent-gated shuttling** — anything a party wants carried is itemized into checkbox consent cards (one term per item). Approved items post to the shared board **by the server**, not by the model — approved content cannot silently fail to appear.
5. **Response cards** — incoming proposals arrive as per-item checkboxes. Checked = agreed (recorded automatically, since both parties have now approved it). Unchecked = the Guide works it as a counter.
6. **Endgame** — when every issue is agreed, the Guide runs a closing sequence: print the Points of Agreement (with a 48-hour signature window and attestations), optional attorney review, and hand-off to the state's official self-help filing forms.

## Enforcement architecture

Everything the product *depends on* is enforced in code, not left to model instructions:

| Guarantee | Mechanism |
|---|---|
| Parties can't see each other's channel | Server-side role-scoped state; the other channel is never sent to the browser |
| Nothing crosses without consent | Board posts happen only via checkbox approval; unconsented attempts are blocked and flagged to the owner with an override button |
| Screening before content | Board locked per-party until the Guide signals screening complete |
| No round ends without a consent card | Server detects hand-off language with no pending card and forces a corrected reply |
| Health records never transit | Playbook hard rule + no file upload feature exists, by design (HIPAA / 42 CFR Part 2) |
| Account numbers / SSNs never cross | Playbook hard rule (production adds a regex scrubber before storage) |
| Duplicate consent cards | Server dedupes identical re-asks |

## The playbook

The Guide's entire personality, ethics, and technique live in one editable text document (`playbook.js`) — eleven hard rules (neutrality, confidentiality, consent, safety screening, no legal advice, impasse honesty, one-question-at-a-time, data protection, distress protocol) plus a technique layer (bridge ladders, verify-before-carry, discrepancy radar, income standards, child-focused conflict teaching, safe-exchange options, resolution ordering) and a growing toolbox of real-world instruments (Soberlink, limited releases of information, graduated schedules, civil standby exchanges, right of first refusal).

The owner dashboard has a live playbook editor per case; `playbook.js` is the master used for new cases. The playbook is the product — the app around it is plumbing.

## Features

- **Per-party credits** — each party pays separately; time is metered only while actively engaged (idle tabs are free). At zero, that party's link goes dormant and the other party sees a "not linked" notice with an *Add credits for them* sponsor button.
- **Voice** — five curated voices (OpenAI TTS), tap-to-talk transcription with review-before-send, and a hands-free live mode with pause / replay / speed controls. Both API keys stay server-side.
- **Owner dashboard** — flag review queue (safety, impasses, discrepancies, blocked posts with override), read-only view of both channels, live playbook editor, per-party balances, and estimated API cost vs. revenue per case.
- **Print summary** — agreed points, open items, disclaimers, and dual attestation blocks ("I sign freely, without the other party present"), built solely from mutually consented content.

## Quick start

Requirements: [Node.js](https://nodejs.org) 18+ and an [Anthropic API key](https://console.anthropic.com). An [OpenAI API key](https://platform.openai.com) is optional (voice features).

```bash
git clone <this repo>
cd resolutionator
npm install

# create .env
cat > .env <<'EOF'
ANTHROPIC_API_KEY=sk-ant-your-key
OPENAI_API_KEY=sk-your-openai-key
EOF

npm start
# → http://localhost:3000
```

To play both sides yourself: create a case, open the Party A link in a normal window, the Party B link in a private/incognito window, and the Owner link in a third tab.

## Configuration

| Env var | Default | Purpose |
|---|---|---|
| `ANTHROPIC_API_KEY` | — | Required. The Guide's brain. |
| `OPENAI_API_KEY` | — | Optional. Enables voice (TTS + transcription). |
| `RESOLUTIONATOR_MODEL` | `claude-sonnet-4-5` | Claude model for the Guide |
| `RESOLUTIONATOR_TTS_MODEL` | `gpt-4o-mini-tts` | Text-to-speech model |
| `RESOLUTIONATOR_STT_MODEL` | `whisper-1` | Transcription model |
| `RESOLUTIONATOR_START_MINUTES` | `60` | Each party's opening time balance (test mode) |
| `RESOLUTIONATOR_RATE` | `25` | Displayed price per hour |
| `PORT` | `3000` | Server port |

## Project structure

```
resolutionator/
├── server.js          # Express server: state, consent enforcement, voice proxy, credits
├── playbook.js        # The Guide's constitution — master default for new cases
├── public/
│   ├── index.html     # Landing page
│   ├── create.html    # Case creation wizard (case types + issue logic)
│   ├── party.html     # Party view: private channel, board, cards, voice, credits
│   └── owner.html     # Owner dashboard: flags, playbook editor, economics
├── data.json          # All case data (created at runtime; delete to wipe)
└── .env               # API keys (never committed)
```

## Status & roadmap

**This is a local test build.** Access is by unguessable link only — no logins, no HTTPS, plain-JSON storage. Do not put real clients on it in this form.

Planned before real users: account signup with email verification, hosted deployment behind HTTPS (Cloudflare-fronted), Stripe for the credit system, email/SMS "something's waiting" notifications (never content), the 10-minute practice sandbox as a marketing funnel, data-retention and encryption policy, and a code-level PII scrubber. Later: business mediation category, co-parenting subscription tier, B2B API for courts and human mediators.

## Disclaimers

Resolutionator is an educational self-help communication tool. It is not mediation, does not provide legal advice, does not create any professional relationship, and produces no binding agreements — terms concerning custody, placement, or support bind no one until approved by a court. Users in crisis should call or text **988** (Suicide & Crisis Lifeline) or call **911**. Domestic violence resources: **1-800-799-7233** (National DV Hotline, or text START to 88788).

## License

Copyright © 2026. All rights reserved. Not licensed for reuse or redistribution.

