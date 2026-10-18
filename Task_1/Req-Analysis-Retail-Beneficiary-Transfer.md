# Req-Analysis-Retail-Beneficiary-Transfer

## 1. Feature Summary

A retail customer transfers money from an account they own to a beneficiary they have already registered. The customer chooses the source account and beneficiary, enters an amount, may add a purpose note, confirms the transfer using an OTP, and receives an outcome. A successful transfer debits the source account, receives a transaction reference, displays confirmation details, and triggers an SMS confirmation.

## 2. Requirements Inventory

### 2.1 Explicit functional requirements

| ID | Requirement | Source |
|---|---|---|
| FR-01 | The customer can select a source account from their accounts. | AC1 |
| FR-02 | The customer can select a beneficiary from their registered beneficiaries. | AC1 |
| FR-03 | The customer can enter a transfer amount and may enter an optional purpose note. | AC1 |
| FR-04 | The transfer requires OTP confirmation before execution. | AC2 |
| FR-05 | A transfer fee is applied according to the bank's fee configuration. | AC3 |
| FR-06 | Transfers are subject to a daily limit of 20,000 and a per-transaction limit of 10,000. | AC4 |
| FR-07 | On success, the source account is debited and a transaction reference is generated. | AC5 |
| FR-08 | On success, a confirmation screen shows transfer details and an SMS confirmation is sent to the customer. | AC5 |
| FR-09 | If the transfer cannot be completed, the customer is shown an error message. | AC6 |

### 2.2 Non-functional and control requirements

No measurable non-functional requirements are explicitly specified. The following are quality and control needs to clarify and verify; they are not additional approved acceptance criteria:

- **Security:** OTP handling, authentication/session checks, protection of account and beneficiary data, and prevention of replay or unauthorized execution.
- **Integrity:** debit, fee, transaction record, reference number, and transfer status must remain consistent, including when downstream services fail.
- **Reliability:** avoid duplicate transfers on retries or repeated submissions; define recovery behavior for timeouts and uncertain outcomes.
- **Auditability:** record enough transfer and authorization details for investigation and reconciliation, subject to the bank's data-retention policy.
- **Usability/accessibility:** errors and confirmation details should be understandable and usable across supported channels and assistive technologies.
- **Performance/availability:** response-time and service-availability targets are not supplied and must be agreed for the channel and transfer services.

## 3. Ambiguities, Missing Criteria, and Edge Cases

1. **Transfer eligibility and account ownership:** Which account types, currencies, statuses, and customer/account combinations are eligible? Are joint, blocked, dormant, or overdraft accounts supported?
2. **Beneficiary validity:** Must the beneficiary still be active and eligible at submission time? What happens if the beneficiary is removed, changed, or restricted after selection?
3. **Amount validation:** What are the minimum amount, decimal precision, currency rules, and permitted characters? Are zero, negative, malformed, or excessive-precision amounts rejected, and with what message?
4. **Fee calculation:** The fee schedule, effective-date behavior, fee currency, rounding, tax, fee cap/minimum, and treatment of a fee change during the flow are unspecified. Is the fee shown before OTP? Does it come from the source balance in addition to the principal?
5. **Limit semantics:** It is unclear whether the limits are inclusive; whether they apply to principal or principal plus fee; how “daily” is defined (time zone and reset time); which transfers count; and how concurrent requests are prevented from exceeding the daily limit. The limit currency is also unstated.
6. **Balance and holds:** No behavior is defined for insufficient available funds, pending transactions, holds, overdrafts, or a balance that changes between review and execution.
7. **OTP lifecycle:** Delivery channel, OTP expiry, retry/resend limits, attempt lockout, invalid/expired OTP behavior, transaction binding, and whether a new OTP is required after transfer details change are undefined.
8. **Execution timing and status:** It is unclear whether transfers execute immediately or can remain pending, and how the UI represents accepted, pending, rejected, timed-out, or unknown outcomes.
9. **Duplicate submission and idempotency:** There is no criterion for repeated taps, client retries, network timeouts, or replayed OTP submissions. A retry must not unintentionally debit the customer twice.
10. **Transaction atomicity and downstream failures:** The relationship between the debit, fee debit, transfer ledger entry, reference generation, and beneficiary credit is unspecified. Define rollback, compensation, and reconciliation behavior if any stage fails.
11. **Purpose note:** Maximum length, accepted characters, sanitization, visibility, storage, and whether it is included in SMS or beneficiary-facing records are not specified.
12. **Confirmation details:** Required fields, amount/fee presentation, masked account/beneficiary identifiers, timestamp/time zone, transfer status, and reference format/uniqueness are unspecified.
13. **SMS behavior:** Destination source, SMS content, localization, timing, retry policy, provider failure behavior, and whether SMS failure affects transfer success are unspecified. “Sent” should be distinguished from provider acceptance or delivery.
14. **Failure messaging:** AC6 does not define error categories, user-safe wording, localization, recovery guidance, or whether errors must avoid revealing sensitive account or beneficiary information.
15. **Regulatory and operational controls:** Required sanctions/fraud screening, transfer cut-off windows, audit events, retention, customer notifications, and support/reconciliation procedures are not described.
16. **Supported channels and service targets:** The channel(s), supported devices/browsers, accessibility standard, performance targets, and availability expectations are absent.

## 4. Traceability Matrix and Risk Ranking

Risk reflects customer, financial, security, and operational impact if the requirement fails.

| Requirement ID | Recommended test levels | Risk | Rationale |
|---|---|---|---|
| FR-01 | Unit, API, Integration, UI, Security | High | Incorrect account selection or authorization can expose data or debit the wrong account. |
| FR-02 | Unit, API, Integration, UI, Security | High | A stale, unauthorized, or incorrect beneficiary can cause irreversible misdirection of funds. |
| FR-03 | Unit, API, Integration, UI | High | Amount validation errors can cause financial loss; purpose-note handling also has data and injection risks. |
| FR-04 | Unit, API, Integration, UI, Security | High | Weak or bypassable OTP controls enable unauthorized transfers. |
| FR-05 | Unit, API, Integration, UI, Performance | High | Incorrect or stale fees can cause financial, contractual, and regulatory disputes. |
| FR-06 | Unit, API, Integration, Performance, Security | High | A limit bypass or race condition can create losses and breach policy or regulation. |
| FR-07 | Unit, API, Integration, Performance, Security | High | Debit/reference inconsistency, duplicate execution, or an incorrect ledger result directly affects funds and reconciliation. |
| FR-08 | Integration, UI, Security | Medium | Missing or inaccurate confirmation/SMS harms trust and can cause support disputes; SMS failure should not obscure the actual transfer state. |
| FR-09 | Unit, API, Integration, UI, Security | High | Misleading or overly revealing errors can cause unsafe retries, data exposure, or customer confusion about funds. |
| NFR-01 Security and OTP controls | API, Integration, UI, Security | High | Unauthorized execution or disclosure has direct financial and privacy impact. |
| NFR-02 Integrity and idempotency | Unit, API, Integration, Performance | High | Partial or duplicate financial processing can produce customer loss and costly reconciliation. |
| NFR-03 Reliability and recovery | Integration, Performance | High | Timeouts and downstream outages can leave funds or transaction state uncertain. |
| NFR-04 Auditability | API, Integration, Security | Medium | Incomplete records impede dispute resolution, fraud investigation, and compliance evidence. |
| NFR-05 Usability and accessibility | UI | Medium | Confusing amounts, fees, or errors can lead to customer mistakes and exclude users. |
| NFR-06 Performance and availability | API, Integration, Performance | Medium | Targets are not defined; poor responsiveness or availability can trigger retries and uncertain outcomes. |

## 5. Questions and Assumptions

Questions are ordered by the decisions most likely to change implementation, test design, or financial risk. Assumptions below are provisional only and require business approval.

| Priority | Question for Product/Business/Engineering | Why it matters | Provisional assumption if unanswered |
|---|---|---|---|
| P0 | What exactly counts toward the 10,000 transaction and 20,000 daily limits: principal only or principal plus fee? Are limits inclusive, what currency applies, what is the daily reset time zone, and which transfer statuses count? | Determines boundary behavior and prevents limit bypass, especially across concurrent requests. | Apply limits to principal in the account currency, allow an amount equal to a limit, and use the bank-configured business-day reset; do not finalize tests until confirmed. |
| P0 | What is the authoritative fee schedule and how are currency conversion, rounding, taxes, effective dates, and insufficient funds for the fee handled? Must the customer see and accept the fee before OTP? | Affects amount displayed, debit total, consent, and reconciliation. | Calculate using the bank's active configuration at execution and show principal, fee, and total before OTP; require sufficient available funds for the total. |
| P0 | How is OTP delivered and bound to a transfer? What are expiry, retry/resend, lockout, and detail-change rules? | Establishes the actual authorization boundary and security expectations. | OTP is single-use, expires, is bound to the customer and exact transfer details, and must be reissued if those details change. Exact values need bank confirmation. |
| P0 | What is the required behavior for timeout or service failure after debit or submission, and what idempotency key/status lookup guarantees prevent duplicate execution? | Without a clear outcome model, retrying may double-debit or hide a completed transfer. | The server enforces idempotency; an uncertain outcome is queried by stable request/reference identifier before retrying, never blindly resubmitted. |
| P0 | Is a transfer synchronous, and what constitutes “success”: accepted by the bank, posted to the ledger, or credited to the beneficiary? Can it be pending or later reversed? | Defines customer messaging, confirmation timing, and financial state transitions. | Show success only when the authoritative transfer service confirms the defined successful state; represent pending separately if supported. |
| P1 | Which accounts and beneficiaries qualify, and which conditions (blocked, inactive, restricted, currency mismatch, closed) prevent a transfer? | Determines eligibility and authorization checks at selection and execution. | Validate current source-account and beneficiary eligibility on the server at execution, even if they appeared selectable earlier. |
| P1 | What are minimum amount, decimal precision, currency rules, purpose-note length/character policy, and required localization? | Prevents inconsistent client/server validation and malformed ledger inputs. | Enforce currency-specific precision and reject non-positive amounts; apply a business-approved note length once supplied. |
| P1 | What exact fields must appear on the confirmation screen and in the SMS, and what identifiers must be masked? | Prevents misleading confirmations and accidental disclosure. | Include status, principal, fee/total, beneficiary, masked source account, timestamp, and reference on-screen; SMS uses approved minimal details. |
| P1 | Does SMS provider failure change transfer status? What delivery/retry evidence is available? | Separates the financial outcome from the notification outcome. | A successful transfer remains successful if SMS delivery fails; record and retry notification independently according to bank policy. |
| P1 | Which error messages, recovery actions, and localization are approved for validation, OTP, limits, funds, and service failures? | Helps customers understand whether it is safe to retry without disclosing sensitive information. | Provide a generic safe message for unexpected service failures and specific actionable messages for known validation failures. |
| P2 | What audit, retention, fraud/sanctions screening, cut-off, accessibility, performance, and availability standards apply? | These controls may be mandatory even though the story does not state them. | Follow existing bank-wide standards; obtain their identifiers and measurable targets before release sign-off. |

## 6. Required Test Data and Environments

### Test data

- Retail customer profiles with valid, invalid, expired, locked, and retry-exhausted OTP states; include multiple configured notification destinations where permitted.
- Source accounts covering eligible/ineligible types, multiple currencies, normal and insufficient available balances, holds, overdraft conditions, and account status changes.
- Registered beneficiaries covering active, inactive/removed/restricted states, differing currencies, and valid/invalid destination details, using masked or synthetic data.
- Amounts around zero, minimum and maximum precision, per-transaction limit, daily remaining limit, and limit boundaries; include amounts where principal fits but fee-inclusive total does not.
- Fee configurations covering zero, fixed, percentage, minimum/maximum, rounding, tax, currency, effective-date transition, and unavailable/malformed configuration, as supported by bank policy.
- Daily activity histories immediately below, at, and above the daily threshold, including concurrent submissions and transactions in different statuses.
- Purpose notes with empty, maximum-length, overlength, Unicode/localized, and characters requiring validation or sanitization.
- Stable request identifiers and prior transfer states to exercise retries, duplicate submissions, timeout recovery, and reconciliation without generating real transfers.

### Environments and dependencies

- Isolated non-production environment connected to a transfer/ledger simulator or controlled test rails; no uncontrolled real-money movement.
- OTP delivery sandbox with controllable valid, invalid, expired, delayed, and unavailable responses.
- Fee and limit configuration representative of production, with approved test-only overrides and documented reset behavior.
- SMS provider sandbox or stub capable of accepted, delayed, failed, and duplicate notification responses.
- Ability to simulate network timeouts, retries, partial downstream failures, concurrency, and account/beneficiary state changes.
- Test access to authoritative transaction status, ledger/audit events, idempotency records, and notification outcomes for reconciliation.
- Agreed supported client channels/devices, test identities, access roles, locale/time-zone settings, and performance environment before execution.

## 7. Risk Assessment

1. **Incorrect, duplicate, or partial financial execution (High):** Failures between debit, fee posting, ledger recording, and beneficiary credit can leave balances inconsistent or execute twice. The customer may lose access to funds; the bank may incur direct loss, disputes, and reconciliation workload; the delivery partner faces a severe production incident and trust impact. The design needs an authoritative state model, idempotency, atomicity or explicit compensation, and observable reconciliation.
2. **OTP bypass or weak transaction binding (High):** Reusable, unbound, or insufficiently rate-limited OTPs can authorize a transfer the customer did not approve. This can cause fraud and regulatory exposure for the bank, direct customer harm, and security incident liability for the delivery partner. OTP must authorize the exact transfer and follow bank security policy.
3. **Fee and limit miscalculation (High):** Unclear calculation order, concurrency, reset boundary, or fee inclusion can charge the wrong total or permit excess daily movement. Customers may be overcharged, and the bank may breach published terms or controls; delivery teams may face urgent remediation and audit findings.
4. **Stale or misidentified beneficiary/source (High):** Client-side-only eligibility checks or ambiguous selection/confirmation can route funds incorrectly or disclose financial information. Transfers may be difficult or impossible to recover, with direct customer and bank impact.
5. **Misleading outcome and retry guidance (High):** If a timeout is displayed as failure after a transfer actually completed, the customer may retry and duplicate it; if success is shown before posting, the customer may rely on funds that did not arrive. This creates financial disputes and support burden for the bank and a high-severity reliability issue for the delivery partner.
6. **SMS or confirmation disclosure/omission (Medium):** Incorrect destination or excessive detail can disclose private information; missing notification can cause uncertainty. The financial transfer should remain independently traceable and its status must not be inferred from SMS delivery alone.

## 8. Readiness Assessment

The story is sufficient to identify the main customer journey but is not yet test-ready for financial execution. Before approval for detailed test design, the P0 decisions on limit semantics, fee calculation and presentation, OTP lifecycle, idempotency/recovery, and the definition of transfer success should be resolved. Account/beneficiary eligibility, amount and note validation, confirmation/SMS content, failure messaging, and applicable bank-wide non-functional controls should also be confirmed or explicitly assigned approved assumptions.
