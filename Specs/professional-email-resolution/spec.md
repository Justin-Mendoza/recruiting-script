## ADDED Requirements

### Requirement: Validated company-pattern registry

The system SHALL import a versioned JSON registry mapping normalized company keys and aliases to one or more professional email domains, patterns, source-consensus levels, source URLs, last-checked dates, and notes. It SHALL reject unsupported patterns, malformed domains, duplicate aliases across companies, invalid confidence values, and inconsistent record counts.

#### Scenario: Import the initial registry

- **GIVEN** a valid registry containing 37 company records
- **WHEN** the user imports it
- **THEN** the system stores all valid companies, aliases, domains, patterns, confidence metadata, sources, and dates and reports an import summary

#### Scenario: Registry contains an alias collision

- **GIVEN** a case-insensitive alias maps to different companies
- **WHEN** validation runs
- **THEN** the system rejects the conflicting records and identifies the alias

#### Scenario: Registry pattern is stale

- **GIVEN** a registry pattern is older than the configured freshness period
- **WHEN** a contact is resolved
- **THEN** the system treats the pattern as uncertain and routes the contact through Hunter Email Finder rather than local inference

### Requirement: Hunter adapter

The system SHALL integrate with Hunter through a narrow adapter supporting company-domain resolution, named-person Email Finder, and exact-address Email Verifier. It SHALL authenticate with a locally protected API key, apply timeouts and bounded retries, respect rate limits, and redact secrets and addresses in default diagnostics.

#### Scenario: Hunter request succeeds

- **GIVEN** valid credentials and an approved lookup
- **WHEN** the adapter calls a documented endpoint
- **THEN** the system normalizes the response without exposing the API key

#### Scenario: Hunter is unavailable

- **GIVEN** Hunter returns an authentication, rate-limit, quota, timeout, malformed-response, or availability error
- **WHEN** resolution runs
- **THEN** the affected contact remains unresolved and no Gmail draft is created for it

### Requirement: Confirmed company domain

The system SHALL resolve domains from a fresh registry match first. If no unambiguous registry match exists, it SHALL use Hunter Domain Finder. It MUST require user confirmation before using a candidate domain for address resolution.

#### Scenario: Registry resolves the company

- **GIVEN** the contact company matches a unique registry key or alias with a current domain
- **WHEN** domain resolution runs
- **THEN** the system proposes that domain and records the registry source for user confirmation

#### Scenario: Hunter resolves a missing company

- **GIVEN** no registry entry matches the company
- **WHEN** the user approves a Domain Finder request
- **THEN** the system presents Hunter's candidate domains and requires the user to select or enter the correct one

#### Scenario: Domain is ambiguous

- **GIVEN** multiple brands, subsidiaries, or domains are plausible
- **WHEN** the candidates are reviewed
- **THEN** the system generates no addresses until the user confirms one domain

### Requirement: Credit-efficient resolution routing

For a fresh high-consensus registry pattern, the system SHALL generate exactly one candidate address and submit it to Hunter Email Verifier. For medium, low, stale, missing, unsupported, or conflicting patterns, it SHALL call Hunter Email Finder with the supplied person name and confirmed company domain instead of testing multiple guessed formats.

#### Scenario: Verify one registry-derived address

- **GIVEN** a confirmed company domain has a fresh high-consensus pattern
- **WHEN** resolution runs
- **THEN** the system normalizes the supplied name, generates one candidate, and submits only that address to Email Verifier

#### Scenario: Use Email Finder for uncertain pattern

- **GIVEN** the registry pattern is medium, low, stale, missing, or conflicting
- **WHEN** resolution runs
- **THEN** the system calls Email Finder with the person's name and confirmed domain and performs no sequence of pattern guesses

#### Scenario: Ambiguous person name

- **GIVEN** the supplied name has ambiguous first/last components
- **WHEN** local normalization or Email Finder input is prepared
- **THEN** the system requires the user to confirm the name components before consuming credits

### Requirement: Verified professional addresses only

The system SHALL accept only company-domain professional addresses with a current `valid` result returned by Hunter Email Finder or Email Verifier. Personal addresses, consumer domains, invalid, unknown, disposable, accept-all, or otherwise non-valid results MUST remain ineligible for Gmail drafts.

#### Scenario: Hunter returns valid

- **GIVEN** Hunter returns a valid professional address
- **WHEN** the result is normalized
- **THEN** the system stores the address, resolution route, verification status and time, provider record metadata, and available source metadata and permits message preview

#### Scenario: Hunter returns accept-all or unknown

- **GIVEN** Hunter cannot establish the exact mailbox as valid
- **WHEN** eligibility is evaluated
- **THEN** the system blocks the address from message approval and Gmail draft creation

#### Scenario: Hunter returns personal data

- **GIVEN** a response includes a personal email address or phone number
- **WHEN** it is normalized
- **THEN** the system discards those fields and does not persist or display them

### Requirement: Plan-aware credit preflight and budget

The system SHALL support Hunter's unified-credit free/all-in-one model and the Data Platform model with separate Search and Verification pools and a distinct billing period. Before a credit-consuming batch, it SHALL classify each contact as cache hit, verifier lookup, finder lookup, unresolved-domain lookup, or blocked. It SHALL show planned calls, a conservative estimate against the appropriate configured pool or pools, remaining budget and expiration, and contacts that would exceed budget. It SHALL require explicit approval.

#### Scenario: Batch fits the budget

- **GIVEN** the estimated maximum consumption is within every applicable configured credit pool
- **WHEN** the user approves the preflight
- **THEN** the system executes only the approved calls and records actual observed usage where available

#### Scenario: Batch exceeds the budget

- **GIVEN** the approved batch would exceed a unified, Search, or Verification budget
- **WHEN** preflight runs
- **THEN** the system does not begin lookups and offers to reduce or reroute the batch or update the budget after the user changes their Hunter plan or purchase

#### Scenario: Data Platform verification pool is exhausted

- **GIVEN** Search credits remain but the configured Verification pool cannot cover registry-derived verifier calls
- **WHEN** preflight runs
- **THEN** the system reports the separate shortfall and MUST NOT silently spend Search credits as though the pools were interchangeable

#### Scenario: Hunter pricing changes

- **GIVEN** configured endpoint credit costs may be stale
- **WHEN** a cost estimate is shown
- **THEN** the system identifies it as an estimate, displays the configuration date, and directs the user to confirm current Hunter billing

### Requirement: Resolution cache

The system SHALL cache confirmed domains, verified professional-address results, resolution routes, source metadata, and timestamps. Repeated resolution within the configured freshness window SHALL reuse a valid cached result without consuming credits.

#### Scenario: Reuse a current result

- **GIVEN** the same normalized person and company domain have a current valid cached resolution
- **WHEN** another campaign requests resolution
- **THEN** the system reuses the result and reports zero planned Hunter credits for that contact

#### Scenario: Cached result is stale

- **GIVEN** a cached address exceeds the configured freshness period
- **WHEN** resolution is requested
- **THEN** the system includes revalidation in the credit preflight

### Requirement: Free and paid plan compatibility

The system SHALL use the same Hunter adapter and resolution behavior for free and paid plans. Plan changes SHALL affect only configured credit budget and provider-side quota, not application code or stored contact semantics.

#### Scenario: Upgrade after exhausting free credits

- **GIVEN** the user upgrades Hunter or buys Data Platform credits and increases the applicable configured pool budget
- **WHEN** a previously blocked batch is preflighted again
- **THEN** the system can process it without migration or code changes

### Requirement: No person discovery or LinkedIn dependency

The system MUST NOT use Hunter to search for people, reveal personal emails or phones, run sequences, or send messages. It MUST NOT access or scrape LinkedIn. It SHALL resolve addresses only for named contacts explicitly supplied by the user.

#### Scenario: Company has no supplied contacts

- **GIVEN** the user supplies a company but no people
- **WHEN** campaign creation is attempted
- **THEN** the system requests named contacts and performs no employee discovery
