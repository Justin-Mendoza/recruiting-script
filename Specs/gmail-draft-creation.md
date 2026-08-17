## ADDED Requirements

### Requirement: Personal Gmail OAuth authentication
The system SHALL authenticate a personal Gmail account with OAuth 2.0 using the narrowest Google scope that supports draft creation. It SHALL explain that `gmail.compose` also technically authorizes sending even though the application exposes no send behavior.

#### Scenario: First-time authorization
- **GIVEN** no valid Gmail token exists
- **WHEN** the user connects Gmail
- **THEN** the system completes an interactive OAuth flow, displays the requested scope, and stores the token using OS-appropriate protected local storage

#### Scenario: Authorization is revoked
- **GIVEN** Gmail authorization is expired or revoked and cannot be refreshed
- **WHEN** draft creation is requested
- **THEN** the system creates no further drafts and asks the user to reconnect

### Requirement: Draft-only application behavior
The system SHALL implement Gmail draft creation and update but MUST NOT implement or invoke Gmail message-send or draft-send operations. The CLI MUST NOT expose a send command.

#### Scenario: Create an approved draft
- **GIVEN** a provider-verified professional address and an approved message revision
- **WHEN** the user confirms draft creation
- **THEN** the system creates a Gmail draft and records its identifier

#### Scenario: Automatic sending is requested
- **GIVEN** the user invokes the CLI
- **WHEN** the user attempts to request sending
- **THEN** no send operation is available and no Gmail send endpoint is called

### Requirement: Batch draft creation with failure isolation
The system SHALL create one Gmail draft per approved contact, report progress per contact, and continue after a single-contact failure when Gmail remains usable.

#### Scenario: Create a successful batch
- **GIVEN** six approved contacts and valid Gmail authorization
- **WHEN** batch draft creation runs
- **THEN** the system attempts six individual draft creations and reports each result

#### Scenario: One draft is rejected
- **GIVEN** Gmail rejects one item-specific payload in a batch
- **WHEN** creation continues
- **THEN** the failed item remains approved-but-not-created while unrelated items may complete

#### Scenario: Authorization fails mid-batch
- **GIVEN** Gmail authorization becomes unusable during a batch
- **WHEN** the failure is detected
- **THEN** the system stops new attempts, preserves completed draft references, and reports unattempted contacts

### Requirement: Idempotent draft creation
The system SHALL derive and persist an idempotency key from campaign, contact, application when applicable, approved message revision, and attachment digest.

#### Scenario: Retry a completed batch
- **GIVEN** some batch items already created drafts
- **WHEN** the user retries the batch
- **THEN** the system skips completed idempotency keys and attempts only unresolved items

#### Scenario: Materially revised message
- **GIVEN** a previous draft exists and the user approves materially changed content
- **WHEN** draft creation is requested
- **THEN** the system clearly offers to update the known draft or create a new revision

### Requirement: Standards-compliant MIME draft
The system SHALL construct a UTF-8 RFC 5322/MIME message with one To recipient, no Cc or Bcc, a sanitized subject, plain-text body, and optional PDF résumé.

#### Scenario: Construct a draft with résumé
- **GIVEN** approved text and a valid résumé
- **WHEN** MIME construction runs
- **THEN** the payload contains the exact approved text and correctly typed PDF attachment with no other recipients

### Requirement: No mailbox-content dependency
The initial release SHALL NOT read inbox or sent-message bodies. Follow-up eligibility and outcomes SHALL depend on locally recorded user actions.

#### Scenario: Show a possible follow-up
- **GIVEN** a user-recorded sent date and no recorded reply or suppression
- **WHEN** history is reviewed
- **THEN** the system may show that manual follow-up is due without reading Gmail or creating it automatically
