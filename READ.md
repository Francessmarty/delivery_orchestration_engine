### GreenCart Delivery Orchestration Engine

An n8n proof of concept that validates delivery orders, selects eligible logistics providers, handles provider responses and routes operational exceptions to human reviewers.

The workflow demonstrates how supply-chain dispatch decisions can be converted into explainable automation while retaining human intervention for orders that cannot be safely completed automatically.

## Project Overview

Delivery operations often require teams to manually:

* Validate customer and delivery information
* Determine the correct dispatch strategy
* Compare available logistics providers
* Confirm vehicle compatibility
* Evaluate provider reliability and estimated arrival time
* Handle rejection, cancellation and timeout events
* Update operational records
* Notify customers and internal teams

This project brings those activities into one orchestration workflow.

A submitted order is validated, classified and matched against eligible providers. The selected provider response is evaluated, and the workflow either completes the order, attempts fallback recovery or escalates the exception to Slack.

Workflow Overview

flowchart TD
    A["Customer order intake"] --> B["Validate and standardise"]
    B -->|Invalid| C["Human review in Slack"]
    B -->|Valid| D["Determine dispatch strategy"]
    D --> E["Filter and rank providers"]
    E -->|No eligible provider| C
    E --> F["Select primary provider"]
    F --> G{"Provider response"}
    G -->|Accept| H["Update order and customer"]
    G -->|Reject, timeout or cancel| I["Evaluate fallback provider"]
    I -->|Fallback available| F
    I -->|No fallback available| C

## Core Capabilities

# Order validation

The workflow checks required order information before operational processing begins.

Validation covers:

* External order ID
* Customer name
* Customer email
* Pickup address
* Delivery address
* Priority
* Required vehicle
* Package size
* Delivery time window

Invalid orders stop before provider assignment and are routed for human review.

# Dispatch strategy selection

The workflow selects a dispatch strategy based on operational requirements.

The conditions are evaluated in this order:

1. Urgent priority
2. Express priority
3. Fragile package
4. Large package or high-capacity vehicle requirement
5. Standard delivery

Order condition	Dispatch strategy	Minimum reliability	Maximum ETA
Urgent	Fastest Eligible	85	45 minutes
Express	Balanced Speed and Reliability	88	60 minutes
Fragile	Highest Reliability	92	90 minutes
Large, Van or Truck	Vehicle Capacity First	85	90 minutes
Standard	Lowest Cost	80	120 minutes

This precedence ensures that an urgent fragile order is treated as urgent first, while a standard order can prioritise cost efficiency.

# Provider eligibility

A provider must satisfy the delivery requirements before it can be considered for assignment.

The workflow evaluates:

* Provider availability
* Supported vehicle type
* Package and delivery requirements
* Reliability threshold
* Estimated arrival-time threshold
* Delivery cost
* Previous provider failure

Providers that do not qualify are excluded before ranking and selection.

# Provider-response handling

The proof of concept simulates four provider responses:

* accept
* reject
* timeout
* cancel_after_acceptance

An accepted order continues through customer notification and operational storage.

A rejection, timeout or cancellation triggers the exception-recovery path. The failed provider is excluded, and the workflow checks whether another eligible provider can take the order.

If no suitable fallback provider remains, the order is escalated for human intervention.

Data persistence and duplicate prevention

Google Sheets acts as the operational datastore for the proof of concept.

The workflow uses the external order ID to determine whether to:

* Append a new order
* Update an existing order
* Prevent duplicate operational rows

Testing confirmed that submitting an existing external order ID updates its record instead of creating another row.

# Human escalation

Slack notifications are created when automation cannot safely complete an order.

Escalation scenarios include:

* Invalid delivery information
* Unsupported vehicle requirements
* No eligible provider
* Provider rejection without a suitable fallback
* Provider timeout without a suitable fallback
* Cancellation without successful recovery

The escalation message gives the operations team the order context, failure reason and recommended action.

# Exception Paths

Invalid delivery address

An order with an address that cannot be validated is stopped before dispatch and marked for manual review.

Primary provider rejection

The rejection is recorded, the failed provider is excluded and the workflow evaluates the remaining provider pool.

Cancellation after acceptance

The workflow records that the provider accepted the delivery and later cancelled it before checking for recovery options.

Provider timeout

If the provider does not respond within the configured 30-minute window, the workflow records the timeout and evaluates the fallback path.

No eligible provider

When the required vehicle or delivery conditions cannot be supported, the workflow prevents an unsuitable assignment and escalates the order.

## Quality Assurance

Six controlled scenarios were retained as the final public QA dataset.

Test	Scenario	Result
T01	Invalid delivery address	Passed
T02	Unsupported refrigerated vehicle	Passed
T03	Primary provider accepts	Passed
T04	Primary provider cancels	Passed
T05	Primary provider rejects	Passed
T06	Primary provider times out	Passed

All six scenarios reached their expected workflow branches without an unhandled workflow failure.

Detailed evidence is available in test_results.md⁠￼.

## KPI Dashboard

The Google Sheets dashboard summarises order validation, processing outcomes and provider-routing activity across the final QA dataset.

It tracks:

* Total orders
* Validated orders
* Orders needing attention
* Processed orders
* Express and urgent orders
* Fragile packages
* Provider assignments
* Fallback-provider activity
* Reassignment requirements
* Automation rate

## Technology Stack

* n8n: Workflow orchestration and conditional routing
* Python: Validation, transformation and provider-selection logic inside n8n Code nodes
* Google Sheets: POC operational storage, QA evidence and KPI reporting
* Gmail: Customer order updates
* Slack: Human escalation and operational alerts
* JSON: Workflow export, sample test data and target schema

## Repository Structure

delivery_orchestration_engine/
├── sample_data/
│   ├── invalid_order.json
│   ├── no_eligible_provider.json
│   ├── primary_provider_accepts.json
│   ├── primary_provider_cancel.json
│   ├── primary_provider_rejects.json
│   └── primary_provider_timeout.json
├── schemas/
│   └── target_schema.json
├── screenshots/
│   ├── 01-workflow-overview.png
│   ├── 02-invalid-order-validation.png
│   ├── 03-initial-provider-accepts.png
│   ├── 04-initial-provider-rejects.png
│   ├── 05-initial-provider-cancel.png
│   ├── 06-initial-provider-timeout.png
│   ├── 07-no-eligible-provider.png
│   ├── 08-customer-email-update.png
│   ├── 09-slack-escalation-invalid-delivery-address.png
│   ├── 10-slack-escalation-no-eligible-provider.png
│   ├── 11-slack-no-provider-escalation.png
│   ├── 12-qa-test-results.png
│   └── 13-routing-kpi-dashboard.png
├── workflow/
│   └── delivery-orchestration-workflow.json
├── .gitignore
├── README.md
└── test_results.md

## Running the Proof of Concept

1. Import the workflow

In n8n:

1. Create a new workflow.
2. Select Import from File.
3. Import:

workflow/delivery-orchestration-workflow.json

2. Configure integrations

The public workflow export does not contain working credentials or private resource identifiers.

Configure your own:

* Google Sheets OAuth credential
* Gmail OAuth credential
* Slack OAuth credential
* Google Sheets document
* Slack escalation channels

Replace the placeholder resource IDs inside the imported nodes.

3. Prepare the datastore

Create a Google Sheet containing the columns expected by the workflow, or update the Google Sheets node mappings to match your own operational schema.

4. Run a test scenario

Use one of the JSON files inside sample_data/ as a reference when submitting the order form.

Each sample represents one controlled workflow path.

5. Confirm the outcome

Check:

* n8n execution history
* The updated Google Sheets order record
* Customer email output
* Slack escalation output
* The final order status and exception reason

## Security

The public workflow export has been sanitised for GitHub.

The repository does not contain:

* API keys
* OAuth access tokens
* OAuth refresh tokens
* Credential identifiers
* Google Sheets document identifiers
* Slack channel identifiers
* n8n instance identifiers
* Live customer information

Placeholder identifiers must be replaced after importing the workflow into another n8n environment.

## Design Decisions

Deterministic operational rules

Provider routing is based on explicit thresholds and conditions rather than an unexplained AI score score. This makes the dispatch decision easier to inspect, test and audit.

Separate validation and operational failure

Invalid customer input is handled separately from provider rejection, cancellation or timeout. This prevents data-quality problems from being confused with delivery-capacity problems.

Safe failure

The workflow does not assign a provider that fails the required vehicle, reliability or ETA conditions. When automation cannot make a safe assignment, it sends the order to a human operator.

Human-in-the-loop recovery

Slack escalation preserves human oversight for exceptions that could affect customers, delivery commitments or operational cost.

## POC Limitations

* Provider responses are simulated for controlled testing.
* Provider availability is represented by a predefined test pool.
* Google Sheets is used instead of a production database or transport-management system.
* The timeout event is simulated rather than waiting 30 minutes during QA.
* Live carrier quotation, tracking and booking APIs are outside the current scope.

## Production Enhancements

A production version could include:

* Live carrier availability and quotation APIs
* Geocoding and address-verification services
* Distance and route optimisation
* Real-time vehicle capacity
* Configurable retry and timeout policies
* Provider performance history
* Delivery cost optimisation
* Automated proof-of-delivery updates
* A production database
* Role-based operational controls
* Monitoring, alerting and execution logs
* Automated regression testing

## Business Value

This proof of concept shows how delivery orchestration can reduce repetitive dispatch work while improving operational consistency.

The system provides:

* Faster order triage
* Consistent provider eligibility checks
* Reduced risk of unsuitable vehicle assignment
* Clear exception reasons
* Duplicate-order prevention
* Faster operational escalation
* Traceable dispatch decisions
* A structured foundation for cost, reliability and delivery-time optimisation


## Author
Frances Ehinor
francesbuilds.com