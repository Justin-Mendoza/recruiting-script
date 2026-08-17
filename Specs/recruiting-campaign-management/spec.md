## ADDED Requirements

### Requirement: Candidate profile configuration
The system SHALL maintain one local candidate profile containing the candidate's name, school, degree, graduation date, target role types, experience highlights, professional links, signature, résumé path, and preferred Canada/United States search geography.

#### Scenario: Configure a valid candidate profile
- **GIVEN** the user provides all required identity, experience, signature, and résumé fields
- **WHEN** the user saves the profile
- **THEN** the system stores the profile locally and reports it as ready for campaign use

#### Scenario: Missing candidate identity
- **GIVEN** the candidate profile is missing the candidate's name or school
- **WHEN** the user starts a campaign
- **THEN** the system rejects campaign creation and identifies the missing fields

### Requirement: Explicit contact input
The system SHALL accept contacts through an interactive command, CSV, or JSON. Each contact MUST contain a full name and company; title, audience category, geography, and personalization evidence SHALL be supported. The system MUST NOT discover or append people who were not in the user's input.

#### Scenario: Import valid contacts
- **GIVEN** a UTF-8 CSV or JSON file contains valid named contacts and companies
- **WHEN** the user imports the file
- **THEN** the system validates and stages exactly those contacts without resolving or drafting yet

#### Scenario: Reject an incomplete row
- **GIVEN** an imported row lacks a full name or company
- **WHEN** import validation runs
- **THEN** the system rejects that row with its location and continues validating the remaining rows

#### Scenario: Duplicate input rows
- **GIVEN** the same normalized name and company appear more than once in an import
- **WHEN** validation runs
- **THEN** the system presents one staged contact and reports the duplicate source rows

### Requirement: Networking campaign creation
The system SHALL create a networking campaign from staged contacts categorized as recruiters, hiring managers, or relevant employees.

#### Scenario: Create a networking campaign
- **GIVEN** a valid candidate profile and staged contacts
- **WHEN** the user creates a networking campaign
- **THEN** the system records the campaign type, target role families, geography, and selected input contacts

### Requirement: Post-application campaign creation
The system SHALL require role title, job URL or identifier, application date, and application geography for a post-application campaign, and SHALL restrict eligible contacts to recruiting-related roles.

#### Scenario: Create a post-application campaign
- **GIVEN** the user supplies the required application context and recruiter contacts
- **WHEN** the user creates the campaign
- **THEN** the system records the application and stages only recruiting-related contacts for resolution

#### Scenario: Non-recruiter in post-application input
- **GIVEN** a post-application contact is categorized as an employee or hiring manager
- **WHEN** campaign validation runs
- **THEN** the system marks the contact ineligible and explains the recruiter-only rule

### Requirement: Partial batch progress
The system SHALL process each imported contact independently and SHALL represent validation, domain, resolution route, verification, message, and draft status per contact.

#### Scenario: One contact fails resolution
- **GIVEN** a batch contains multiple contacts and one cannot be resolved
- **WHEN** the batch is processed
- **THEN** the failed contact is reported without preventing eligible contacts from reaching preview

### Requirement: Human-controlled campaign state
The system SHALL track contacts through imported, invalid, resolving-domain, resolving-email, unresolved, verified, ineligible, previewed, approved, draft-created, manually-marked-sent, replied, declined, bounced, and suppressed states. It MUST NOT infer that a Gmail draft was sent.

#### Scenario: User records a sent draft
- **GIVEN** a draft exists and the user sent it from Gmail
- **WHEN** the user marks the outreach as sent
- **THEN** the system records the user-supplied send time

#### Scenario: Draft remains unsent
- **GIVEN** a Gmail draft exists
- **WHEN** no manual sent action is recorded
- **THEN** the system continues to represent the outreach as draft-created

### Requirement: Local campaign history
The system SHALL show imported source, registry match, confirmed domain, Hunter resolution route and verification result, estimated and observed credit usage, draft identifier, user-recorded outcome, and safety decisions without displaying secrets.

#### Scenario: Review company history
- **GIVEN** campaigns exist for a company
- **WHEN** the user requests company history
- **THEN** the system displays prior supplied contacts, inferred addresses in explicitly requested detail mode, relevant dates, outcomes, and suppressions
