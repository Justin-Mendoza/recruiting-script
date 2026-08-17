## ADDED Requirements

### Requirement: Audience-specific message generation
The system SHALL generate distinct message structures for recruiters, hiring managers, and employees using the candidate profile, campaign context, supplied title/category, and user-provided personalization evidence.

#### Scenario: Generate a recruiter post-application message
- **GIVEN** a post-application campaign and an eligible recruiter address
- **WHEN** a message is generated
- **THEN** it mentions the applied role, application timing, relevant candidate evidence, a specific company or role connection, and a request to review or route the application

#### Scenario: Generate an employee networking message
- **GIVEN** a networking campaign and an eligible employee address
- **WHEN** a message is generated
- **THEN** it asks for role-appropriate insight or a brief conversation rather than assuming the employee can review an application

### Requirement: Evidence-based personalization
Every message SHALL include at least one user-reviewed company, team, role, project, or recipient-specific reason for contact. Generated text MUST NOT invent facts about the recipient, company, candidate, or application.

#### Scenario: Personalization is absent
- **GIVEN** a supplied contact has no reviewed personalization evidence
- **WHEN** the user requests a message preview
- **THEN** the system blocks draft eligibility for that contact and requests a specific detail

#### Scenario: Generated claim lacks input evidence
- **GIVEN** rendered content contains a recipient or company claim absent from campaign inputs
- **WHEN** validation runs
- **THEN** the system flags the claim and does not approve the message

### Requirement: Concise message structure
The system SHALL render a descriptive subject, personalized greeting, concise introduction, no more than three selected experience proofs, one clear request, signature, and optional compliance footer. Default bodies SHALL target 80 to 140 words excluding signature and footer.

#### Scenario: Message exceeds default length
- **GIVEN** a rendered body exceeds 140 words
- **WHEN** validation runs
- **THEN** the system warns the user and requires explicit revision or approval

### Requirement: Placeholder and recipient validation
The system MUST detect unresolved placeholders, empty fields, malformed names, template instructions in final text, and mismatches between the supplied person and the resolved email local part.

#### Scenario: Unresolved placeholder
- **GIVEN** rendered content still contains a company placeholder
- **WHEN** approval is requested
- **THEN** the system rejects the message and identifies the placeholder

#### Scenario: Recipient mismatch warning
- **GIVEN** the resolved address does not plausibly match the normalized supplied name and confirmed pattern
- **WHEN** preview validation runs
- **THEN** the system blocks approval pending correction or re-resolution

### Requirement: Résumé attachment policy
The system SHALL attach the configured résumé by default for post-application recruiter drafts and SHALL default to no attachment for general employee networking drafts. The user MUST be able to preview and override the attachment decision.

#### Scenario: Recruiter follow-up includes résumé
- **GIVEN** a readable PDF résumé is configured for a post-application recruiter message
- **WHEN** the message is approved
- **THEN** the approved draft content includes that attachment metadata

#### Scenario: Résumé is invalid
- **GIVEN** the selected résumé is missing, unreadable, or not a PDF
- **WHEN** approval is requested
- **THEN** the system blocks the affected draft and reports the problem

### Requirement: Batch and per-recipient preview
The system SHALL show a batch summary and the exact recipient, subject, body, signature, footer, and attachment metadata for each eligible contact. The user SHALL be able to approve all unchanged eligible messages or review, edit, approve, or exclude contacts individually.

#### Scenario: Approve a clean batch
- **GIVEN** all eligible messages pass validation and the user has reviewed the batch summary
- **WHEN** the user explicitly approves the unchanged batch
- **THEN** each message receives an immutable approved revision for Gmail draft creation

#### Scenario: Edit one message in a batch
- **GIVEN** a batch preview exists
- **WHEN** the user edits one recipient's message and excludes another
- **THEN** the system revalidates the edit, preserves other approved messages, and creates no approved revision for the excluded contact
