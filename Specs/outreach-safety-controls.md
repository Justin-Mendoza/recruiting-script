## ADDED Requirements

### Requirement: Verified professional-address drafting
The system MUST create drafts only for company-domain professional addresses with a current valid Hunter verification result. Unverified, invalid, unknown, disposable, accept-all, consumer-domain, and personal addresses SHALL be blocked.

#### Scenario: Inferred address lacks verification
- **GIVEN** the local registry produced an address but Hunter has not returned a current valid result
- **WHEN** message or draft eligibility is evaluated
- **THEN** the system blocks that contact from approval and Gmail draft creation

#### Scenario: User supplies an exact professional address
- **GIVEN** the user supplies an exact professional address with provenance
- **WHEN** eligibility is evaluated
- **THEN** the system requires Hunter verification before accepting it while retaining the supplied evidence

### Requirement: Conservative outreach limits
The system SHALL default to at most three draft-created or user-recorded-sent contacts per company in a rolling seven-day period and ten new outreach drafts per local day. Lower limits SHALL be allowed; higher limits SHALL require an explicit persistent override and warning.

#### Scenario: Company limit reached
- **GIVEN** three contacts at one company already count toward the rolling window
- **WHEN** another contact is approved
- **THEN** the system blocks that contact by default and explains the limit

#### Scenario: Daily limit reached mid-batch
- **GIVEN** a batch would exceed the local daily limit
- **WHEN** eligibility is calculated
- **THEN** the system identifies which contacts are eligible now and leaves the remainder approved but unscheduled for draft creation

### Requirement: Duplicate prevention
The system SHALL detect duplicate normalized input contacts, duplicate resolved email addresses, prior outreach to the same address, and repeated outreach for the same application before approval.

#### Scenario: Two names resolve to one mailbox
- **GIVEN** two imported records resolve to the same normalized email address
- **WHEN** batch validation runs
- **THEN** the system blocks both pending user correction rather than choosing one automatically

#### Scenario: Contact was already approached
- **GIVEN** history shows outreach to an address for the same application
- **WHEN** the address appears in a new batch
- **THEN** the system blocks a new cold-introduction draft and shows the existing history

### Requirement: Durable suppression list
The system SHALL maintain suppression entries for opt-outs, declines, hard bounces, invalid addresses, user blocks, and corrected registry data. Suppression SHALL take precedence over imports, Hunter results, registry inference, and limit overrides.

#### Scenario: Suppressed address is re-imported
- **GIVEN** an address is suppressed
- **WHEN** a later supplied person resolves to it
- **THEN** the system excludes it without offering an override

#### Scenario: User records a hard bounce
- **GIVEN** the user reports that a registry-derived and Hunter-verified address hard-bounced
- **WHEN** the outcome is saved
- **THEN** the system suppresses that address, flags the company's pattern for review, and warns before using the pattern for new contacts

### Requirement: Provenance review for Canadian outreach
For a Canadian recipient or unknown geography, the system SHALL require reviewable contact-input provenance, registry or Hunter resolution provenance, role relevance, sender identification, and a configurable no-further-contact sentence before drafting. These controls SHALL be presented as risk reduction, not legal assurance.

#### Scenario: Canadian contact lacks provenance
- **GIVEN** a Canadian contact lacks reviewable sourcing or address evidence
- **WHEN** approval is requested
- **THEN** the system blocks the contact until evidence is recorded or the contact is removed

#### Scenario: Canadian footer is omitted
- **GIVEN** an otherwise eligible Canadian message omits the configured no-further-contact sentence
- **WHEN** the user reviews it
- **THEN** the system warns the user and requires explicit per-message acknowledgement

### Requirement: Safe secrets and personal data
The system MUST NOT place Hunter API keys, OAuth tokens, raw contact imports, résumé contents, or local outreach records in logs or repository-tracked configuration. Diagnostics SHALL redact credentials and email addresses by default.

#### Scenario: Network error contains sensitive data
- **GIVEN** a Hunter or Gmail error contains headers or an address
- **WHEN** it is logged
- **THEN** the system redacts secrets and identifying address details

### Requirement: No engagement surveillance
The system MUST NOT add tracking pixels, link trackers, read receipts, or hidden remote content to drafts.

#### Scenario: Render MIME content
- **GIVEN** an approved message
- **WHEN** MIME is constructed
- **THEN** it contains no tracking or hidden remote-resource markup
