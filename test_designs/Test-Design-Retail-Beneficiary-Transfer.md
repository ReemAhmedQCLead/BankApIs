# Test-Design-Retail-Beneficiary-Transfer

## 1. Status and Scope

**Status:** Draft preview for human review. No Test Plan, suite, or Test Cases have been created in Azure DevOps.

**Source:** Approved `Req-Analysis-Retail-Beneficiary-Transfer` artifact.

**Version:** Full suite, 32 proposed test cases. The original 12-case version is preserved separately.

**Scope:** Functional, API, integration, UI, security, reliability, accessibility, and performance-oriented coverage for the retail transfer story. Cases validate the approved requirements while marking provisional policies that require confirmation.

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

### TC-13: Validate amount precision and currency handling

- **Priority:** P1 | **Requirement mapping:** FR-03, FR-09 | **Test level:** Unit, API, UI | **Tags:** Regression, Automate
- **Preconditions:** Supported currencies and precision rules are configured.

| Step | Action | Expected Result |
|---|---|---|
| 1 | Submit valid precision, excess decimal places, localized separators, and unsupported currency values. | Values are parsed consistently or rejected according to currency rules; no silent amount change occurs. |
| 2 | Compare UI, API, and recorded amounts. | Amount and currency agree exactly under approved precision rules. |

### TC-14: Validate purpose-note boundaries and safe rendering

- **Priority:** P1 | **Requirement mapping:** FR-03, FR-08, FR-09; NFR-01, NFR-05 | **Test level:** Unit, API, UI, Security | **Tags:** Regression, Automate
- **Preconditions:** Note length, accepted characters, and display policy are agreed.

| Step | Action | Expected Result |
|---|---|---|
| 1 | Submit omitted, maximum-length, overlength, Unicode, and special-character notes. | Optional and boundary behavior follows policy; supported text is preserved. |
| 2 | Inspect stored note and confirmation rendering. | Unsafe markup is escaped; note appears only in approved places and is not added to SMS unless approved. |

### TC-15: Apply fee rounding, tax, and currency rules

- **Priority:** P0 | **Requirement mapping:** FR-05, FR-07, FR-08; NFR-02 | **Test level:** Unit, API, Integration | **Tags:** Regression, Automate
- **Preconditions:** Applicable rounding, tax, fee-currency, and conversion rules are configured.

| Step | Action | Expected Result |
|---|---|---|
| 1 | Calculate fees at fractional and rounding-boundary values for supported currencies. | Fee, tax, and conversion follow the configured rule without precision drift. |
| 2 | Execute a transfer and reconcile display with ledger. | Fee and total debit match reviewed values and approved currency precision. |

### TC-16: Fail safely when fee configuration is unavailable or invalid

- **Priority:** P0 | **Requirement mapping:** FR-05, FR-09; NFR-02, NFR-03 | **Test level:** API, Integration, UI | **Tags:** Regression, Automate
- **Preconditions:** Fee configuration can be unavailable, malformed, or unsupported.

| Step | Action | Expected Result |
|---|---|---|
| 1 | Initiate transfer while fee configuration cannot be retrieved or validated. | No undocumented/default fee is used; transfer is not executed and a safe error is shown. |
| 2 | Restore configuration and retry through the approved path. | A valid fee is quoted and retry cannot duplicate an accepted transfer. |

### TC-17: Reconfirm transfer if fee changes after review

- **Priority:** P0 | **Requirement mapping:** FR-04, FR-05, FR-07, FR-08 | **Test level:** API, Integration, UI | **Tags:** Regression, Automate
- **Preconditions:** Fee can change during authorization and quote-validity policy is known.

| Step | Action | Expected Result |
|---|---|---|
| 1 | Review fee and total, change configuration, then submit OTP. | Quote/effective-time policy is applied consistently. |
| 2 | Compare customer-authorized and posted amounts. | Customer is not silently charged a different total; renewed review/authorization or rejection follows policy. |

### TC-18: Recheck available balance, holds, and changed funds

- **Priority:** P0 | **Requirement mapping:** FR-05, FR-07, FR-09; NFR-02 | **Test level:** Unit, API, Integration, UI | **Tags:** Regression, Automate
- **Preconditions:** Holds, pending debits, and account balance are controllable.

| Step | Action | Expected Result |
|---|---|---|
| 1 | Submit when available funds are below principal plus fee, including a case where principal alone is affordable. | Transfer is rejected before posting with an understandable error. |
| 2 | Add a hold or reduce funds after review but before confirmation. | Execution rechecks authoritative available funds and rejects if insufficient. |
| 3 | Inspect balance and ledger. | No partial principal debit, fee-only debit, or completed transfer remains. |

### TC-19: Enforce OTP resend, retry, and lockout policy

- **Priority:** P0 | **Requirement mapping:** FR-04, FR-09; NFR-01 | **Test level:** Unit, API, Integration, UI, Security | **Tags:** Regression, Automate
- **Preconditions:** Approved resend interval, retry count, and lockout policy are configured.

| Step | Action | Expected Result |
|---|---|---|
| 1 | Request OTP repeatedly within and outside the resend interval. | Resends are throttled and code invalidation follows policy. |
| 2 | Exceed allowed invalid attempts. | Further attempts are blocked/throttled and recovery guidance follows policy. |
| 3 | Inspect transfer/security records. | No transfer executes and OTP secrets are not recorded. |

### TC-20: Reject expired OTP

- **Priority:** P0 | **Requirement mapping:** FR-04, FR-09; NFR-01 | **Test level:** Unit, API, Integration, UI, Security | **Tags:** Regression, Automate
- **Preconditions:** OTP expiry is controllable in the sandbox.

| Step | Action | Expected Result |
|---|---|---|
| 1 | Obtain an OTP, advance beyond expiry, and submit it. | OTP is rejected with safe guidance. |
| 2 | Inspect balance and transaction records. | No debit or transfer execution occurs. |

### TC-21: Bind OTP to transfer details and prevent replay

- **Priority:** P0 | **Requirement mapping:** FR-04, FR-07, FR-09; NFR-01, NFR-02 | **Test level:** API, Integration, Security | **Tags:** Regression, Automate
- **Preconditions:** Ability to change details after OTP issuance and replay requests.

| Step | Action | Expected Result |
|---|---|---|
| 1 | Change amount, source, or beneficiary after OTP issuance and submit the old OTP. | Changed details are rejected or require renewed authorization per policy. |
| 2 | Complete a transfer and replay its OTP/request. | Replay resolves to the original outcome or is rejected; no second debit occurs. |
| 3 | Reconcile the ledger. | At most one transfer exists for the authorization. |

### TC-22: Test per-transaction limit boundaries

- **Priority:** P0 | **Requirement mapping:** FR-06, FR-09 | **Test level:** Unit, API, Integration, UI | **Tags:** Smoke, Regression, Automate
- **Preconditions:** Daily usage is below limit and inclusivity/precision policy is confirmed.

| Step | Action | Expected Result |
|---|---|---|
| 1 | Submit immediately below 10,000, exactly 10,000, and the smallest representable amount above it. | Boundary handling follows the confirmed rule; above-limit amount is rejected before execution. |
| 2 | Submit above-limit amount directly through API. | Server enforces the limit; client-side checks cannot be bypassed. |

### TC-23: Apply daily time window and status-counting rules

- **Priority:** P0 | **Requirement mapping:** FR-06, FR-07, FR-09 | **Test level:** Unit, API, Integration | **Tags:** Regression, Automate
- **Preconditions:** Daily reset/time zone and qualifying statuses are agreed; clock and histories are controllable.

| Step | Action | Expected Result |
|---|---|---|
| 1 | Seed pending, posted, rejected, and reversed transfers, then calculate remaining allowance. | Each status counts or is excluded according to policy without double counting. |
| 2 | Submit around daily reset and applicable daylight-saving transitions. | Transfers are assigned to the correct bank-defined day. |
| 3 | Reconcile limit counters with qualifying transfers. | Counter matches the approved currency and fee basis. |

### TC-24: Prevent concurrent requests exceeding daily limit

- **Priority:** P0 | **Requirement mapping:** FR-06, FR-07; NFR-02, NFR-03, NFR-06 | **Test level:** API, Integration, Performance | **Tags:** Regression, Automate
- **Preconditions:** Concurrent submissions and limit reservations are observable.

| Step | Action | Expected Result |
|---|---|---|
| 1 | Set remaining allowance below the combined amount of two concurrent requests and submit both. | Accepted/reserved requests together do not exceed 20,000. |
| 2 | Repeat identical idempotent retries and distinct requests. | Retries resolve to one operation; distinct requests are evaluated against current usage. |
| 3 | Reconcile outcomes, balance, and counters. | Responses match actual postings and limit usage. |

### TC-25: Apply approved limit basis for principal, fee, and currency

- **Priority:** P0 | **Requirement mapping:** FR-05, FR-06, FR-09 | **Test level:** Unit, API, Integration | **Tags:** Regression, Automate
- **Preconditions:** Business confirms principal-only versus fee-inclusive and currency-conversion rules.

| Step | Action | Expected Result |
|---|---|---|
| 1 | Use a case where principal fits but principal plus fee exceeds the limit. | Acceptance/rejection follows the approved basis. |
| 2 | Run supported foreign-currency cases. | Limit is evaluated in the approved currency and conversion basis. |
| 3 | Compare limit usage and ledger. | Counters follow policy and cannot be bypassed through fees or currency changes. |

### TC-26: Verify confirmation fields, masking, and transfer state

- **Priority:** P1 | **Requirement mapping:** FR-07, FR-08; NFR-01, NFR-05 | **Test level:** UI, Integration, Security | **Tags:** Smoke, Regression, Automate
- **Preconditions:** Confirmation fields, masking, timestamp zone, and status labels are agreed.

| Step | Action | Expected Result |
|---|---|---|
| 1 | Complete a transfer and inspect confirmation at supported viewports/locales. | Approved principal, fee/total, beneficiary, masked source, reference, timestamp, status, and permitted note are shown accurately. |
| 2 | Compare displayed details with authoritative record. | Values match; pending/unknown outcomes are not presented as success. |
| 3 | Inspect sensitive data. | OTP and restricted account details are not exposed. |

### TC-27: Verify SMS destination, content, and delivery status

- **Priority:** P1 | **Requirement mapping:** FR-08; NFR-01, NFR-03 | **Test level:** Integration, Security | **Tags:** Regression, Automate
- **Preconditions:** SMS sandbox captures destination/content and provider acceptance/delivery statuses.

| Step | Action | Expected Result |
|---|---|---|
| 1 | Complete a successful transfer. | SMS is addressed to the customer's configured destination, not a beneficiary or another customer's number. |
| 2 | Inspect message fields, locale, masking, and note treatment. | Content is accurate, appropriately masked, and limited to approved details. |

### TC-28: Keep SMS failures and duplicate callbacks independent of transfer status

- **Priority:** P1 | **Requirement mapping:** FR-08, FR-09; NFR-03, NFR-04 | **Test level:** Integration, UI | **Tags:** Regression, Automate
- **Preconditions:** SMS sandbox supports delay, rejection, timeout, retry, and duplicate callback.

| Step | Action | Expected Result |
|---|---|---|
| 1 | Complete a successful transfer while the SMS provider fails or times out. | Transfer retains its authoritative status; notification failure does not reverse it. |
| 2 | Retry notification and deliver a duplicate callback. | No second transfer is created and financial status does not change. |
| 3 | Inspect notification records. | Notification outcome is separately recorded and retry follows policy. |

### TC-29: Handle transfer-service failure before acceptance

- **Priority:** P0 | **Requirement mapping:** FR-07, FR-09; NFR-02, NFR-03 | **Test level:** API, Integration, UI | **Tags:** Regression, Automate
- **Preconditions:** Transfer service can fail before accepting a request.

| Step | Action | Expected Result |
|---|---|---|
| 1 | Force service rejection or unavailability before acceptance. | Customer receives a safe failure message. |
| 2 | Inspect account, fee, ledger, and reference records. | No debit, fee, completed transaction, or success reference is created. |

### TC-30: Recover after acceptance/debit and represent pending or unknown states

- **Priority:** P0 | **Requirement mapping:** FR-07, FR-08, FR-09; NFR-02, NFR-03, NFR-04 | **Test level:** API, Integration, UI, Performance | **Tags:** Regression, Automate
- **Preconditions:** Failures can be injected after acceptance/debit and status/reconciliation is observable.

| Step | Action | Expected Result |
|---|---|---|
| 1 | Cause timeout after acceptance/debit and before a definitive client response. | UI does not claim confirmed success/failure while unknown; status lookup or safe recovery is available. |
| 2 | Allow recovery/reconciliation to complete. | Final status, balance, fee, reference, and ledger converge to one valid outcome; compensation follows policy. |
| 3 | Exercise pending, rejected, and reversed states if supported. | UI and notifications reflect authoritative state without contradictions. |

### TC-31: Prevent duplicate execution on retries, repeated taps, and parallel requests

- **Priority:** P0 | **Requirement mapping:** FR-04, FR-07, FR-09; NFR-02, NFR-03, NFR-04 | **Test level:** API, Integration, Performance | **Tags:** Regression, Automate
- **Preconditions:** Idempotency identifiers and response-loss simulation are available.

| Step | Action | Expected Result |
|---|---|---|
| 1 | Drop response after processing, then retry the same identifier and replay rapid/parallel submissions. | Retries resolve to original outcome; at most one debit, fee, reference, and transfer record exist. |
| 2 | Submit a distinct request identifier. | It is evaluated as a new operation against current balance and limits. |
| 3 | Reconcile ledger and audit data. | Every attempt is traceable and no duplicate posting exists. |

### TC-32: Verify accessibility, performance, audit, and bank-wide controls

- **Priority:** P1 (P2 for performance thresholds)
- **Requirement mapping:** FR-01 to FR-09; NFR-01 to NFR-06
- **Test level:** UI, Accessibility, API, Integration, Security, Performance
- **Tags:** Regression; automate where tooling and measurable thresholds permit
- **Preconditions:** Accessibility standard, performance targets, audit schema, and applicable screening/cut-off policies are supplied.

| Step | Action | Expected Result |
|---|---|---|
| 1 | Complete transfer and failure flows with keyboard/assistive technology across supported viewport/locales. | Controls, focus, validation, and messages meet the approved accessibility standard; critical values do not overlap or disappear. |
| 2 | Run representative peak workload and dependency degradation. | Approved latency, throughput, and availability targets are met without unsafe retry or integrity failures. |
| 3 | Exercise success, rejection, timeout, retry, reversal, and applicable screening/cut-off decisions; inspect audit data. | Lifecycle events reconcile with ledger and notifications, are protected, and support investigation per bank policy. |

## 4. Coverage Mapping

| Requirement | Covered by |
|---|---|
| FR-01: Select source account | TC-01, TC-02, TC-03, TC-12 |
| FR-02: Select registered beneficiary | TC-01, TC-02, TC-12 |
| FR-03: Enter amount and optional purpose note | TC-01, TC-03, TC-13, TC-14 |
| FR-04: OTP before execution | TC-01, TC-07, TC-09, TC-10, TC-12, TC-19, TC-20, TC-21, TC-31 |
| FR-05: Apply configured fee | TC-01, TC-04, TC-08, TC-15, TC-16, TC-17, TC-18, TC-25 |
| FR-06: Daily and per-transaction limits | TC-05, TC-06, TC-22, TC-23, TC-24, TC-25 |
| FR-07: Debit and generate reference | TC-01, TC-04, TC-07, TC-08, TC-09, TC-10, TC-15, TC-17, TC-18, TC-21, TC-24, TC-25, TC-29, TC-30, TC-31, TC-32 |
| FR-08: Confirmation screen and SMS | TC-01, TC-11, TC-17, TC-26, TC-27, TC-28, TC-30, TC-32 |
| FR-09: Show error when transfer cannot complete | TC-02, TC-03, TC-05, TC-06, TC-07, TC-08, TC-10, TC-11, TC-12, TC-13, TC-16, TC-18, TC-19, TC-20, TC-22, TC-23, TC-29, TC-30, TC-32 |
| NFR-01: Security and OTP controls | TC-02, TC-03, TC-07, TC-12, TC-14, TC-19, TC-20, TC-21, TC-26, TC-27, TC-32 |
| NFR-02: Integrity and idempotency | TC-01, TC-04, TC-06, TC-08, TC-09, TC-10, TC-15, TC-16, TC-17, TC-18, TC-21, TC-24, TC-25, TC-29, TC-30, TC-31, TC-32 |
| NFR-03: Reliability and recovery | TC-06, TC-09, TC-10, TC-11, TC-16, TC-24, TC-28, TC-29, TC-30, TC-31, TC-32 |
| NFR-04: Auditability | TC-01, TC-07, TC-09, TC-10, TC-11, TC-12, TC-21, TC-26, TC-28, TC-30, TC-31, TC-32 |
| NFR-05: Usability and accessibility | TC-03, TC-13, TC-14, TC-26, TC-32; standard and supported channels must be confirmed |
| NFR-06: Performance and availability | TC-06, TC-09, TC-10, TC-24, TC-30, TC-31, TC-32; pass thresholds must be supplied |

## 5. Suite Classification

- **Smoke:** TC-01, TC-05, TC-10, TC-11, TC-22, TC-26, TC-27
- **Regression:** TC-01 through TC-32, subject to environment availability and confirmation of unresolved business rules
- **Recommended automation:** TC-01 through TC-31 where stable contracts and controllable test doubles exist; automate TC-32's checks as specialist tooling and thresholds become available
- **Manual or specialist support likely:** Fee effective-time and quote behavior in TC-15/TC-17, daily status/reset and concurrency in TC-23/TC-24, failure recovery in TC-29/TC-31, and accessibility/performance/security policy checks in TC-32 need controlled environment or specialist support.

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
