# AI Lifecycle Strategy Agent

An agentic lifecycle intelligence system that reviews CRM, engagement, company, and sales signals to identify which contacts and accounts need attention, determine the best next action, and execute approved follow-up in HubSpot.

## Overview

Traditional lifecycle systems rely on static rules:

* A contact visits a page
* A score increases
* A workflow updates a field
* A task or email is triggered

That works for simple automation, but it becomes unreliable when the right action depends on context.

A website visit may be meaningful for one contact and irrelevant for another. A senior title may indicate influence, but not an active need. A highly engaged contact may already be in an open sales process and should not receive duplicate outreach.

I built the AI Lifecycle Strategy Agent to evaluate these signals together.

Instead of applying the same fixed workflow to every record, the agent reviews the available evidence, decides whether more context is needed, selects the appropriate tools, and continues investigating until it can make a sufficiently confident recommendation.

## Business Goal

Improve pipeline quality by identifying which contacts and accounts require action, deciding the most appropriate next step, and triggering the right marketing or sales follow-up.

The system is designed to answer:

> Given everything we know about this contact and account, what should happen next?

## My Role

I designed:

* The business objective
* The agent architecture
* Contact eligibility rules
* ICP and persona logic
* Buying-role and Jobs-to-Be-Done classification
* Lifecycle decision rules
* Intent and confidence criteria
* CRM action permissions
* Human-review safeguards
* Token and investigation limits
* Test scenarios and edge cases

I used Claude Code to implement the application in TypeScript, connect the APIs, generate tests, validate the system, and document the setup.

The value of the project is not the model call itself. It is the commercial judgment encoded into the system: what evidence matters, when further investigation is justified, how conflicting signals should be handled, and which actions should or should not be taken.

## How It Works

The agent supports both scheduled and manual analysis.

### Scheduled lifecycle review

On a scheduled basis, the system:

1. Identifies recently engaged contacts.
2. Pulls relevant CRM, company, engagement, and sales context.
3. Creates a normalized contact and account summary.
4. Evaluates whether enough evidence exists to make a reliable decision.
5. Retrieves additional context only where it could materially change the outcome.
6. Repeats until confidence is sufficient or no useful tools remain.
7. Produces a structured lifecycle recommendation.
8. Creates approved CRM tasks or updates selected CRM properties.
9. Sends the complete analysis to a spreadsheet for review and audit.

### Manual contact analysis

A user can also enter one email address and run the full analysis immediately.

```bash
npm run analyze-contact -- --email person@company.com
```

The result can also be written to the review spreadsheet:

```bash
npm run analyze-contact -- --email person@company.com --write-sheet
```

This gives Marketing or Sales a fast way to investigate a specific contact without waiting for the scheduled run.

## Signals Reviewed

The system can evaluate:

* CRM contact data
* Associated company data
* Website activity
* Form submissions
* Marketing email engagement
* Webinar attendance
* CRM notes
* Deal history
* Existing sales activity
* Account-level engagement
* Recency and frequency of activity

Eligibility determines which contacts should be reviewed. It does not automatically make them qualified.

The agent still evaluates:

* Company fit
* Contact fit
* Buying role
* Business need
* Engagement strength
* Intent
* Existing sales activity
* Opportunity or customer status

## Agentic Investigation Loop

The system does not call every available tool for every contact.

It first reviews a compact baseline summary, then decides whether additional context could materially change the recommendation.

```text
Review current evidence
        ↓
Identify missing context
        ↓
Choose the most relevant tool
        ↓
Retrieve additional evidence
        ↓
Update the assessment
        ↓
Stop when confidence is sufficient
```

The investigation stops when:

* The configured confidence threshold is reached
* No remaining tool is likely to change the decision
* The maximum investigation-step limit is reached

Every tool call and the reason for using it are recorded.

## Classification Output

For every reviewed contact, the agent returns:

* Recommended lifecycle stage
* Current CRM lifecycle stage
* Lifecycle mismatch
* Buyer persona
* Functional area
* Buying role
* Primary Job to Be Done
* Intent score and intent level
* Primary pain point
* Next best action
* Recommended email angle
* Sales alert
* ICP fit
* Deal status
* Sales-sequence status
* Confidence score
* Facts
* Inferences
* Missing context
* Tools checked
* Reasoning summary

## Decision Logic

The system evaluates several dimensions separately.

### Company fit

Signals may include:

* Company size
* Industry
* Region
* International presence
* Distributed workforce
* Multilingual teams

### Contact fit

Signals may include:

* Function
* Seniority
* Buying influence
* Budget ownership
* Research activity on behalf of another stakeholder

### Use-case fit

Signals may include:

* Communication and collaboration challenges
* Leadership-development needs
* Customer-facing performance issues
* International growth
* Team integration
* Learning or development initiatives

The system does not rely on job title, company size, or engagement volume alone.

## Buyer-Persona Framework

The agent distinguishes between several B2B buying situations, including:

* Strategic capability leaders
* Functional performance buyers
* Regional talent partners
* Frontline managers
* Other business stakeholders
* Unknown or insufficient-data cases

Persona, functional area, buying role, and Job to Be Done are stored separately.

This allows the system to distinguish, for example, between a coordinator conducting research and the senior stakeholder who owns the underlying business problem.

## CRM Actions

The agent can take approved actions in HubSpot, including:

* Creating sales follow-up tasks
* Updating selected CRM properties
* Recording lifecycle recommendations
* Updating intent or classification fields
* Flagging high-priority contacts
* Preventing duplicate actions when Sales is already engaged

Write actions are controlled through configuration and business rules.

The system does not blindly execute every recommendation. Actions are allowed only when:

* Confidence is sufficient
* The contact meets the required conditions
* No conflicting deal, sequence, or customer state exists
* The action is explicitly enabled
* Required safeguards pass

## Deterministic Safeguards

Important rules are implemented in code rather than left entirely to the model.

Examples include:

* An open deal takes precedence over marketing engagement
* Current customers should not receive acquisition outreach
* An active sales sequence should not create a duplicate task
* Weak fit should lead to nurture rather than direct outreach
* Internal, test, or irrelevant contacts should be excluded
* Missing information should not automatically be treated as negative
* Seniority alone should not create a high-fit classification
* Low confidence should trigger further investigation
* CRM updates are restricted to an approved property allowlist

## Implementation

The application was built with:

* TypeScript
* Node.js
* Anthropic API
* HubSpot API
* Google Sheets API
* Claude Code
* Zod
* Vitest
* ESLint
* Prettier
* GitHub Actions

### Authentication and secrets

The system uses environment-based credentials rather than storing secrets in source code.

Local configuration is managed through a `.env` file containing values such as:

```text
ANTHROPIC_API_KEY
HUBSPOT_ACCESS_TOKEN
GOOGLE_SHEET_ID
GOOGLE_SERVICE_ACCOUNT_KEY_FILE
```

Implementation details:

* Anthropic API key for model reasoning and tool use
* HubSpot service key or private-app token for CRM reads and approved writes
* Google Cloud service account for Google Sheets access
* Service-account JSON stored outside source control
* `.env` and credential folders excluded through `.gitignore`
* GitHub Actions secrets used for scheduled cloud execution
* No API keys committed to the repository

### HubSpot integration

The HubSpot integration includes:

* Contact lookup
* Company associations
* Deal associations
* CRM notes
* Engagement data
* Selected CRM property updates
* Sales-task creation
* Pagination
* Rate-limit handling
* Retry logic
* Per-contact error isolation

Write permissions are limited to the exact objects and properties required by the system.

### Google Sheets integration

Google Sheets is used as an audit and review layer.

The system:

* Creates headers automatically
* Preserves column order
* Appends one row per reviewed contact
* Avoids duplicate rows for the same review period
* Records classification, evidence, confidence, actions, and errors

Access is provided through a Google Cloud service account with permission only to the selected spreadsheet.

## Token and Cost Controls

The agent is designed to use model calls selectively.

Controls include:

* Compact normalized summaries
* CRM field allowlists
* Event deduplication
* Limits on notes and activity history
* Maximum events per category
* Configurable input and output limits
* Maximum investigation steps
* Confidence-based stopping
* Reuse of previously gathered evidence
* No repeated tool calls without justification
* Deterministic handling of hard business rules
* Model escalation only for ambiguous or high-value cases
* Per-contact and per-run token tracking where available

The system stores concise reasoning summaries rather than private model reasoning.

## Reliability

The application includes:

* API pagination
* Rate-limit handling
* Exponential-backoff retries
* Configurable batch concurrency
* Per-contact error isolation
* Idempotency
* Structured logging
* Failed-record logging
* Dry-run mode
* Mock mode
* Runtime schema validation
* Malformed-response handling
* Controlled write permissions
* Environment-based secret management

One failed record does not stop the full run.

## Validation

The system has been validated through:

* Multi-contact mock runs
* Real HubSpot contact retrieval
* Real Anthropic analysis
* Real Google Sheets output
* Manual analysis by email
* CRM task-creation tests
* CRM property-update tests
* Schema validation
* ICP and persona-classification tests
* Lifecycle and business-rule tests
* Idempotency tests
* Agent stopping-condition tests
* Spreadsheet header and row-alignment tests
* Type-checking and linting

## Current Status

Completed:

* Full agent architecture
* HubSpot integration
* Anthropic tool-use loop
* Google Sheets audit layer
* Scheduled multi-contact mode
* Manual analysis by email
* ICP and persona framework
* Buying-role classification
* Jobs-to-Be-Done classification
* Lifecycle decision logic
* Controlled CRM task creation
* Selected CRM property updates
* Token controls
* Reliability safeguards
* Automated tests

The result is a lifecycle system that does more than score contacts.

It gathers evidence, investigates uncertainty, applies commercial rules, recommends the next best action, and executes approved CRM follow-up with safeguards.
