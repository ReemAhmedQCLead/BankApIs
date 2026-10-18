# Test-Design-Retail-Beneficiary-Transfer

## 1. Status and Scope

**Status:** Draft preview for human review. No Test Plan, suite, or Test Cases have been created in Azure DevOps.

**Source:** Approved `Req-Analysis-Retail-Beneficiary-Transfer` artifact.

**Maximum:** 12 test cases.

**Scope:** Functional, API, integration, UI, and security-oriented coverage for a retail transfer to a registered beneficiary. Cases validate the approved story and explicitly identify where provisional analysis assumptions need confirmation.

**Planning limitation:** Azure DevOps test-plan/work-item tools are not available in this session. The User Story ID, project, and current iteration/sprint were not provided, so existing plans cannot be checked and no Test Plan or suite can be selected. The workspace contains no project files or detectable test framework/API contract; executable test code is therefore not generated in this design-only preview.

## 2. Assumptions and Preconditions

- Tests run in an isolated non-production environment with synthetic customers and controlled test rails; no real funds are moved.
- Test services expose or simulate account balances, transfer status, fee configuration, OTP, ledger/reference data, and SMS outcomes.
- Bank-approved currency precision, daily-window definition, fee rules, OTP expiry/attempt policy, and transfer success state must be supplied before final execution.
- Where exact behavior is unresolved in the analysis, expected results below are framed as the risk-control assumption to confirm, not as a new business requirement.
- Each test is isolated or starts from a known account balance, daily usage, OTP state, and fee configuration.

## 3. Proposed Test Cases

### TC-01: Successful transfer with OTP and full confirmation

- **Priority:** P0
- **Requirement mapping:** FR-01, FR-02, FR-03, FR-04, FR-05, FR-07, FR-08; NFR-02, NFR-04
- **Test level:** UI, API, Integration
- **Tags:** Smoke, Regression, Automate
- **Preconditions:** Customer has an eligible funded source account, an active registered beneficiary, a configured fee, and a reachable OTP/SMS sandbox.

| Step | Action | Expected Result |
|---|---|---|
| 1 | Sign in as the customer and open the transfer flow. | Only the customer's eligible source accounts and registered beneficiaries are available for selection. |
| 2 | Select the source account and beneficiary; enter a valid in-limit amount and optional purpose note. | Entered details are accepted; fee and total debit are calculated from the active configuration and presented for review. |
| 3 | Submit and enter the valid OTP bound to the transfer details. | OTP is accepted once and the transfer is executed exactly once. |
| 4 | Inspect the account, transfer record, confirmation screen, and SMS sandbox. | Source is debited by the correct total; a unique reference exists; confirmation shows accurate details/status; confirmation SMS is submitted to the customer's configured destination. |

### TC-02: Reject unauthorized, ineligible, or stale account/beneficiary selection

- **Priority:** P0
- **Requirement mapping:** FR-01, FR-02, FR-09; NFR-01
- **Test level:** API, Integration, UI, Security
- **Tags:** Regression, Automate
- **Preconditions:** Prepare a second customer's account/beneficiary and test records for blocked/inactive account or beneficiary states.

| Step | Action | Expected Result |
|---|---|---|
| 1 | Attempt to retrieve or select another customer's source account or beneficiary through the UI and directly through the transfer API. | Unauthorized records are not exposed or accepted; the server rejects the request without disclosing protected details. |
| 2 | Select a valid account/beneficiary, then change its status to blocked, closed, removed, or otherwise ineligible before execution. | Execution-time eligibility checks reject the transfer and show a safe error. |
| 3 | Check the account balance, ledger, and transfer records. | No debit, fee posting, or completed transfer record is created for rejected attempts. |

### TC-03: Validate amount and optional purpose note

- **Priority:** P1
- **Requirement mapping:** FR-03, FR-09; NFR-01, NFR-05
- **Test level:** Unit, API, UI, Security
- **Tags:** Regression, Automate
- **Preconditions:** Approved currency precision and purpose-note length/character rules are configured for the test environment.

| Step | Action | Expected Result |
|---|---|---|
| 1 | Submit amounts that are blank, zero, negative, malformed, and above permitted decimal precision. | Invalid values are rejected consistently by client and server with actionable validation; no OTP execution or financial posting occurs. |
| 2 | Submit a valid amount with the purpose note omitted. | Transfer details are accepted without requiring a note. |
| 3 | Submit notes at the allowed length, beyond the allowed length, and with characters requiring validation/sanitization. | Boundary behavior follows the approved note policy; unsafe content is not executed or rendered unsafely, and stored/displayed text is handled consistently. |

### TC-04: Apply configured fee and show the correct transfer total

- **Priority:** P0
- **Requirement mapping:** FR-05, FR-07, FR-08; NFR-02
- **Test level:** Unit, API, Integration, UI
- **Tags:** Regression, Automate
- **Preconditions:** Fee configuration is controllable and includes at least two fee rules with known expected outputs.

| Step | Action | Expected Result |
|---|---|---|
| 1 | Initiate otherwise identical transfers under two different active fee configurations. | Each fee is calculated using the applicable configured rule and currency rounding. |
| 2 | Review the transfer details before OTP. | Principal, fee, and total debit are displayed distinctly and consistently with the fee configuration. |
| 3 | Confirm one transfer and inspect the balance/ledger/confirmation. | Debit and recorded fee match the reviewed values; principal and total are not confused in the transaction record or confirmation. |
| 4 | Change the fee configuration during a pending confirmation, then attempt confirmation. | Behavior follows the bank-approved quote/effective-date policy; the executed amount is not silently different from the amount the customer authorized. |

### TC-05: Enforce the 10,000 per-transaction limit

- **Priority:** P0
- **Requirement mapping:** FR-06, FR-09
- **Test level:** Unit, API, Integration, UI
- **Tags:** Smoke, Regression, Automate
- **Preconditions:** Daily usage is low enough not to affect the per-transaction boundary; approved currency precision is known.

| Step | Action | Expected Result |
|---|---|---|
| 1 | Initiate a transfer for exactly 10,000. | The per-transaction limit does not reject an amount equal to the limit, subject to the approved inclusive/exclusive rule. |
| 2 | Initiate a transfer for the smallest representable amount above 10,000. | The server rejects it with a clear limit error before execution. |
| 3 | Inspect balance and transaction records after the rejected attempt. | No debit or completed transaction is recorded for the over-limit request. |

### TC-06: Enforce the 20,000 daily limit, including concurrent requests

- **Priority:** P0
- **Requirement mapping:** FR-06, FR-09; NFR-02, NFR-03
- **Test level:** API, Integration, Performance
- **Tags:** Regression, Automate
- **Preconditions:** Configure the customer's daily transfer usage and reset window; verify that requests can be submitted concurrently in the test environment.

| Step | Action | Expected Result |
|---|---|---|
| 1 | Submit transfers that bring eligible daily usage to exactly 20,000. | Acceptance/rejection at the boundary follows the approved inclusive rule, with ledger and limit usage consistent. |
| 2 | Submit a further transfer that would exceed 20,000. | The request is rejected before debit, using a safe and understandable limit message. |
| 3 | Reset test data to a remaining allowance that is smaller than each of two concurrent requests; submit both concurrently. | Limit reservation/serialization prevents the combined accepted amount from exceeding the daily limit; rejected or pending outcomes are reported accurately. |
| 4 | Repeat around the configured daily reset boundary and time zone. | Only transactions in the correct business-day window count toward the limit. |

### TC-07: Reject invalid, expired, reused, or mismatched OTP

- **Priority:** P0
- **Requirement mapping:** FR-04, FR-07, FR-09; NFR-01
- **Test level:** Unit, API, Integration, UI, Security
- **Tags:** Regression, Automate
- **Preconditions:** OTP sandbox can produce invalid, expired, valid, and already-used codes; bank OTP policies are configured.

| Step | Action | Expected Result |
|---|---|---|
| 1 | Attempt confirmation with an invalid and then an expired OTP. | Each attempt is rejected according to the retry policy; no transfer is executed or debited. |
| 2 | Change the amount, source account, or beneficiary after OTP issuance and submit the previous OTP. | OTP is rejected if it is not bound to the exact current transfer details; updated details require the approved reauthorization flow. |
| 3 | Successfully confirm once, then replay the same OTP/confirmation request. | OTP/request replay does not execute a second transfer; attempt limits and lockout behavior follow policy. |
| 4 | Inspect balance, ledger, and audit data. | Only a valid, authorized confirmation can cause one debit; rejected attempts are recorded without storing or exposing the OTP value. |

### TC-08: Reject transfer when available balance cannot cover principal and fee

- **Priority:** P0
- **Requirement mapping:** FR-05, FR-07, FR-09; NFR-02
- **Test level:** Unit, API, Integration, UI
- **Tags:** Regression, Automate
- **Preconditions:** Configure an account with available balance below principal plus fee, including cases where principal alone is affordable and a hold reduces available funds.

| Step | Action | Expected Result |
|---|---|---|
| 1 | Submit a valid transfer whose principal is within limits but whose principal plus fee exceeds available funds. | Transfer is rejected with an insufficient-funds message before posting. |
| 2 | Repeat with a balance affected by a hold or a balance reduction immediately before confirmation. | Execution uses the authoritative available balance at execution time and rejects if funds are insufficient. |
| 3 | Inspect account balance and financial records. | No partial debit, fee-only debit, or completed transfer is left behind. |

### TC-09: Prevent duplicate execution on repeated submission or retry

- **Priority:** P0
- **Requirement mapping:** FR-04, FR-07, FR-09; NFR-02, NFR-03, NFR-04
- **Test level:** API, Integration, Performance
- **Tags:** Regression, Automate
- **Preconditions:** Transfer service supports controlled response loss/timeouts and exposes authoritative transfer status and idempotency records.

| Step | Action | Expected Result |
|---|---|---|
| 1 | Submit a valid OTP-confirmed transfer while simulating loss of the response after the server processes it. | The client receives an uncertain/timeout outcome rather than a false definitive failure if execution status is unknown. |
| 2 | Retry using the same stable request/idempotency identifier or query status before retrying, as supported by the contract. | The original outcome is returned or discoverable; no second transfer is created. |
| 3 | Repeat with rapid duplicate taps or parallel identical submissions. | At most one financial execution occurs for the same authorized request. |
| 4 | Reconcile account balance, ledger, references, and audit trail. | Exactly one debit/fee/transfer record and one authoritative reference exist; records explain the retry outcome. |

### TC-10: Handle transfer-service failure and uncertain execution safely

- **Priority:** P0
- **Requirement mapping:** FR-07, FR-09; NFR-02, NFR-03, NFR-04
- **Test level:** API, Integration, UI, Performance
- **Tags:** Regression, Automate
- **Preconditions:** Test doubles can fail before processing, after debit, and after transfer acceptance; recovery/reconciliation status is observable.

| Step | Action | Expected Result |
|---|---|---|
| 1 | Cause the transfer service to fail before accepting the request. | Customer sees a safe error; no debit, fee, or completed transfer record exists. |
| 2 | Cause a timeout or downstream failure after acceptance/debit but before the client receives a definitive result. | UI does not claim a confirmed failure or success without authoritative status; customer can safely check status or follow the approved recovery path. |
| 3 | Allow recovery/reconciliation to complete and query the transfer status. | Final status, debit, fee, reference, and ledger record agree; compensation or reversal occurs only according to the approved design. |

### TC-11: Keep transfer success distinct from SMS delivery failure

- **Priority:** P1
- **Requirement mapping:** FR-08, FR-09; NFR-03, NFR-04
- **Test level:** Integration, UI
- **Tags:** Regression, Automate
- **Preconditions:** SMS sandbox can accept, delay, reject, or time out; a valid transfer can be executed.

| Step | Action | Expected Result |
|---|---|---|
| 1 | Complete a valid transfer while the SMS provider rejects or times out. | Financial transfer status and confirmation screen reflect the authoritative transfer result; SMS failure does not incorrectly reverse or relabel a successful transfer. |
| 2 | Inspect notification status and retry behavior. | Notification failure is recorded separately and retried/handled according to policy without causing another transfer. |
| 3 | Check SMS destination and content for a successful provider response in a separate run. | Message goes to the customer's configured destination and contains only approved, accurate, appropriately masked details. |

### TC-12: Enforce authenticated access and protect transfer information

- **Priority:** P0
- **Requirement mapping:** FR-01, FR-02, FR-04, FR-09; NFR-01, NFR-04
- **Test level:** API, Integration, UI, Security
- **Tags:** Regression, Automate
- **Preconditions:** Have two synthetic customers and the ability to expire/revoke a session and inspect application/audit logs.

| Step | Action | Expected Result |
|---|---|---|
| 1 | Attempt to list accounts/beneficiaries and submit a transfer without authentication or with an expired/revoked session. | Requests are rejected; no account/beneficiary details are disclosed and no transfer is executed. |
| 2 | Change customer/account/beneficiary identifiers in a valid customer's transfer request to another customer's identifiers. | Server-side authorization rejects the tampered request without revealing whether the other customer's records exist. |
| 3 | Inspect UI messages, API responses, and logs. | Sensitive account, beneficiary, OTP, and session data are not exposed in messages or logs; security-relevant outcomes are auditable under bank policy. |

## 4. Coverage Mapping

| Requirement | Covered by |
|---|---|
| FR-01: Select source account | TC-01, TC-02, TC-12 |
| FR-02: Select registered beneficiary | TC-01, TC-02, TC-12 |
| FR-03: Enter amount and optional purpose note | TC-01, TC-03 |
| FR-04: OTP before execution | TC-01, TC-07, TC-09, TC-12 |
| FR-05: Apply configured fee | TC-01, TC-04, TC-08 |
| FR-06: Daily and per-transaction limits | TC-05, TC-06 |
| FR-07: Debit and generate reference | TC-01, TC-04, TC-07, TC-08, TC-09, TC-10 |
| FR-08: Confirmation screen and SMS | TC-01, TC-11 |
| FR-09: Show error when transfer cannot complete | TC-02, TC-03, TC-05, TC-06, TC-07, TC-08, TC-10, TC-11, TC-12 |
| NFR-01: Security and OTP controls | TC-02, TC-03, TC-07, TC-12 |
| NFR-02: Integrity and idempotency | TC-01, TC-04, TC-06, TC-08, TC-09, TC-10 |
| NFR-03: Reliability and recovery | TC-06, TC-09, TC-10, TC-11 |
| NFR-04: Auditability | TC-01, TC-07, TC-09, TC-10, TC-11, TC-12 |
| NFR-05: Usability and accessibility | TC-03; add accessibility-specific checks when supported channel and standard are confirmed |
| NFR-06: Performance and availability | TC-06, TC-09, TC-10; measurable pass thresholds remain to be supplied |

## 5. Suite Classification

- **Smoke:** TC-01, TC-05
- **Regression:** TC-01 through TC-12, with TC-05 classified as smoke as well
- **Recommended automation:** TC-01 through TC-12 where the corresponding service/UI contracts and controllable test doubles are available
- **Manual or specialist support likely:** Fee effective-date behavior in TC-04, concurrency and reset-window behavior in TC-06, failure injection/reconciliation in TC-09/TC-10, and security review in TC-12 may require controlled environment support.

## 6. Open Items Before Execution

- Provide the Azure DevOps project, User Story/Work Item ID, and current iteration path so the existing Test Plan can be checked and the requirement-based suite located or created.
- Confirm the P0 business rules from the analysis: limit inclusivity, currency and daily reset/counting rules; fee computation and fee quote behavior; OTP binding/lifecycle; transfer success and pending-state definitions; idempotency and recovery behavior.
- Provide the existing test framework, API/UI contract, test configuration, and approved service simulators before generating executable automation code.
- Supply expected performance/availability thresholds and approved accessibility standard for the supported channel.

## 7. Handoff Status

- **Status:** Draft test-case preview; awaiting review. No Azure DevOps objects created.
- **Test Plan ID:** Not available; Azure DevOps tools and sprint metadata were not available in this session.
- **Suite ID:** Not available; not created.
- **Created Test Case IDs:** None.
- **Next recommended agent:** `qc-test-designer` after approval to create the Test Cases in Azure DevOps and provision of ADO project/iteration details.
