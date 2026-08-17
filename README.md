# Recruiting Script

A CLI tool for managing personalized recruiting outreach campaigns. It resolves professional email addresses for user-supplied contacts, generates tailored messages, and creates Gmail drafts for manual review and sending.

## Key Principles

- **Draft-only** -- creates Gmail drafts but never sends automatically
- **No discovery** -- only processes contacts you explicitly provide
- **Credit-conscious** -- preflight checks against your Hunter monthly quota before making API calls
- **Privacy-aware** -- redacts secrets in logs, discards personal data, no tracking pixels

## How It Works

1. **Configure** your candidate profile (name, school, experience, resume, etc.)
2. **Import** contacts via CSV, JSON, or interactive input (name + company required)
3. **Create a campaign** -- networking (any audience) or post-application (recruiters only)
4. **Resolve emails** -- matches companies against the local pattern registry, falls back to Hunter Email Finder for uncertain patterns, and verifies all addresses via Hunter
5. **Generate messages** -- audience-specific templates with evidence-based personalization
6. **Preview and approve** -- review every message before any draft is created
7. **Create Gmail drafts** -- one draft per approved contact, with optional resume attachment

## Campaign Types

| Type | Audience | Purpose |
|------|----------|---------|
| Networking | Recruiters, hiring managers, employees | Explore opportunities, request conversations |
| Post-application | Recruiters only | Follow up on a specific application |

## Project Structure

```
data/                  Company email pattern registry (129 companies)
Specs/                 Requirement specifications
  recruiting-campaign-management.md
  professional-email-resolution.md
  recruiting-message-generation.md
  gmail-draft-creation.md
  outreach-safety-controls.md
```

## External Services

- **[Hunter](https://hunter.io)** -- Domain Finder, Email Finder, and Email Verifier for address resolution
- **Gmail API** -- OAuth 2.0 draft creation via `gmail.compose` scope

## Safety Controls

- Max 3 contacts per company per rolling 7-day window
- Max 10 new outreach drafts per day
- Duplicate and suppression list enforcement
- Canadian outreach requires reviewable provenance and opt-out footer
- Only verified professional addresses are eligible for drafts
