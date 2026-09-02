# Test Results

# Test Scope

The Delivery Orchestration Engine was tested using six controlled scenarios covering order validation, provider assignment, provider failure and human escalation.

Provider responses were simulated to verify that each workflow branch reached the expected operational outcome.

# Test Summary


### T01 Invalid Delivery Address

- **Expected:** Stop operational processing and request human review.
- **Actual:** The order was marked `Needs Attention` and escalated to Slack.
- **Status:** Pass

### T02  Unsupported Refrigerated Vehicle

- **Expected:** Stop provider assignment when no provider supports the required vehicle.
- **Actual:** No eligible provider was selected, and the order was escalated to Slack.
- **Status:** Pass

### T03  Primary Provider Accepts

- **Expected:** Assign the selected provider and continue processing.
- **Actual:** The provider was assigned, and the order continued through the successful path.
- **Status:** Pass

### T04  Primary Provider Cancels

- **Expected:** Record the cancellation and evaluate fallback availability.
- **Actual:** Cancellation after acceptance was recorded, and the exception path was triggered.
- **Status:** Pass

### T05  Primary Provider Rejects

- **Expected:** Record the rejection and evaluate the remaining providers.
- **Actual:** The rejection was recorded. No remaining eligible provider was available, so the order was escalated.
- **Status:** Pass

### T06  Primary Provider Times Out

- **Expected:** Record the timeout and evaluate the remaining providers.
- **Actual:** The 30-minute timeout was recorded. No remaining eligible provider was available, so the order was escalated.
- **Status:** Pass


# Detailed Results

##  T01  Invalid Delivery Address

Input: An order with a delivery address that could not be validated.

### Result:

* Validation failed before provider assignment.
* The order was marked Needs Attention.
* A Slack notification requested manual review.
* Invalid data did not continue through normal dispatch processing.

Evidence: [Invalid Order Validation](screenshots/02-invalid-order-validation.png)

## T02  Unsupported Refrigerated Vehicle

Input: An order requiring a refrigerated van when no eligible provider supported that vehicle type.

### Result:

* The order passed initial field validation.
* Provider eligibility checks rejected the available providers.
* No provider was assigned.
* The order was escalated to Slack for manual intervention.

Evidence: [No Eligible Provider](screenshots/07-no-eligible-provider.png)

## T03  Primary Provider Accepts

Input: A valid standard delivery with the simulated provider response set to accept.

### Result:

* An eligible provider was selected.
* The provider accepted the delivery request.
* The order continued through the successful processing path.
* The operational record and customer communication were updated.

Evidence:

* [Initial Provider Accepts](screenshots/03-initial-provider-accepts.png)
* [Customer Email Update](screenshots/08-customer-email-update.png)

## T04  Primary Provider Cancels

Input: A valid delivery with the simulated response set to cancel_after_acceptance.

### Result:

* The primary provider initially accepted the request.
* A cancellation after three minutes was recorded.
* The workflow entered the exception-recovery path.
* Remaining provider availability was evaluated.

Evidence: [Initial Provider Cancels](screenshots/05-initial-provider-cancel.png)

## T05  Primary Provider Rejects

Input: A valid delivery with the simulated provider response set to reject.

### Result:

* The primary provider declined the delivery request.
* The rejection reason was recorded.
* The workflow checked for another eligible provider.
* When no remaining eligible provider was available, the order was escalated.

Evidence: [Initial Provider Rejects](screenshots/04-initial-provider-rejects.png)

## T06  Primary Provider Times Out

Input: A valid delivery with the simulated provider response set to timeout.

### Result:

* The provider did not respond within the configured 30-minute window.
* The timeout was recorded as the primary failure reason.
* The workflow checked for another eligible provider.
* When no remaining eligible provider was available, the order was escalated to Slack.

Evidence:

* [Initial Provider Timeout](screenshots/06-initial-provider-timeout.png)
* [Slack No Provider Escalation](screenshots/11-slack-no-provider-escalation.png)

# Data Persistence Validation

Google Sheets append-and-update behaviour was also verified:

* A new external order ID created a new row.
* Reusing the same external order ID updated the existing row.
* A duplicate order row was not created.

## Notification Validation

The workflow successfully produced:

* Customer email updates for successfully processed orders.
* Slack escalation for invalid delivery information.
* Slack escalation when no eligible provider was available.
* Slack escalation when provider recovery could not be completed.

# Final Result

All six portfolio QA scenarios reached their expected workflow branches without an unhandled workflow failure.

The POC demonstrates:

* Input validation
* Strategy-based provider selection
* Provider eligibility checks
* Acceptance and exception handling
* Duplicate prevention
* Google Sheets operational updates
* Customer communication
* Human escalation through Slack

# POC Limitations

Provider responses are simulated for controlled testing. Google Sheets is used as the POC operational datastore. A production implementation would use live provider APIs, secure environment-specific credentials, retry controls and a production database or transport-management platform.