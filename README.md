# Banking QA and API Test Artifacts

This repository brings together quality-assurance documentation and test-planning materials for banking workflows. It includes requirements analysis, proposed test designs, defect reports, an FX-rates API collection, and a release-readiness strategy memo.

The repository is an artifact library, not a banking application or a complete automated test framework. Test designs and defect notes describe intended or reported work; their presence here does not mean the scenarios have been executed or the defects independently reproduced.

## Repository Guide

| Location | Contents |
| --- | --- |
| [`requirements_analysis/`](requirements_analysis/) | Retail beneficiary-transfer requirements analysis. |
| [`test_designs/`](test_designs/) | Draft 12-case and expanded 32-case beneficiary-transfer test designs. |
| [`Task_1/`](Task_1/) | Task-specific requirements analysis and 12-case test design. |
| [`Task_2/`](Task_2/) | ParaBank defect reports and supporting screenshots. |
| [`Task_3/`](Task_3/) | FX-rates Postman collection, sample CSV data, and API test-analysis notes. |
| [`Task_5/`](Task_5/) | International-transfer release-readiness strategy memo. |
| [`.agents/`](.agents/) | QA workflow templates for requirements analysis, test design, and bug reporting. |

## Work Products

### Retail Beneficiary Transfer

The requirements analysis describes a retail customer transferring funds to a registered beneficiary. It covers authorization and OTP confirmation, configured fees, transaction and daily limits, transfer references, customer notifications, and failure handling. The analysis also records assumptions and unresolved business rules that need product or business-owner decisions before implementation or final acceptance criteria can be considered complete.

- [Requirements analysis](requirements_analysis/Req-Analysis-Retail-Beneficiary-Transfer.md)
- [Task 1 requirements copy](Task_1/Req-Analysis-Retail-Beneficiary-Transfer.md)
- [12-case test design](Task_1/Test-Design-Retail-Beneficiary-Transfer-12-Cases.md)
- [12-case design in the test-design directory](test_designs/Test-Design-Retail-Beneficiary-Transfer-12-Cases.md)
- [Expanded 32-case test design](test_designs/Test-Design-Retail-Beneficiary-Transfer.md)

The proposed coverage includes the successful transfer path, authorization, input validation, fee and limit boundaries, OTP behavior, insufficient funds, duplicate requests, service failures, notifications, and data protection. The test-design documents identify themselves as drafts/previews; they do not establish that cases were created in Azure DevOps or executed.

### Defect Reports

Task 2 contains two reports about observed ParaBank behavior:

- A newly created savings account is listed, but opening its details reportedly returns “Could not find account # 5374.” See the [account report](Task_2/Bug_NewcreatedAccounts%20cannont%20checktheirdetails) and [supporting screenshot](Task_2/image.png).
- A registered user can access the Admin Page. See the [access-control report](Task_2/Bug_registredUserabletoAccessAdminPage) and [supporting screenshot](Task_2/image-1.png).

These are issue reports and attached evidence, not confirmation that the behavior has been reproduced against a current system or that a fix has been made.

### FX Rates API Materials

Task 3 contains a Postman collection for requests to the ExchangeRate-API open endpoint, a CSV with five base/target currency pairs and transfer amounts (USD/EUR, EUR/GBP, GBP/USD, EGP/AED, and AED/EGP), and a set of API/security analysis notes.

- [Postman collection](Task_3/BankApIs_FX_Rates.postman_collection.json)
- [Sample CSV data](Task_3/fx-rates-data.csv)
- [Additional API test-analysis notes](Task_3/Task3_part2)

The additional notes discuss cross-customer account access, idempotency, safe retry behavior, transfer lifecycle checks, and ledger reconciliation. They are analysis prompts rather than executable test cases.

**Review before running the collection:** the repository does not include a documented Postman environment or a configured CSV data-runner workflow. The CSV fields `targetCurrency` and `transferAmount` are not used by the request as currently defined. The positive status assertion appears to check `20` rather than the expected HTTP success status `200`, and the invalid-currency variable/configuration differs between the request description and the collection. Treat the collection as a starting artifact and inspect its variables and assertions before relying on its results.

### Release-Readiness Strategy

The [Task 5 strategy memo](Task_5/TestStrategy), dated 18 October 2026, recommends a no-go decision for an international-transfer release and outlines a conditional 72-hour recovery plan. It reports regression completion at 68%, approximately 30 uninvestigated flaky failures, two open critical shared-service defects, UAT at roughly half completion, and limited FX testing.

Those figures and recommendations are claims made in the dated memo. They are included here to describe the artifact and should not be interpreted as independently validated or current release status.

## How to Use This Repository

1. Start with the requirements analysis to understand the beneficiary-transfer scope, assumptions, and open decisions.
2. Review the test-design drafts to understand proposed functional, negative, boundary, and resilience coverage.
3. Use the Task 2 reports and screenshots as issue-reporting examples; verify any reported behavior in the target environment before treating it as a confirmed defect.
4. Inspect the Task 3 collection and its variables/assertions before running it. The included CSV is sample input, not a documented or pre-wired collection runner configuration.
5. Read the Task 5 memo as a dated strategy artifact, not a live project dashboard.

## Scope and Status

- This repository contains QA documentation and test-planning artifacts; no application source code is included in the documented work products.
- Test cases are proposed designs. No execution reports or results are provided for the beneficiary-transfer suites.
- The FX collection has not been represented here as passing or production-ready.
- Requirements, test strategy, and reported defects may need confirmation with the relevant product, QA, or engineering owners before operational use.



## Time for tasks

1. Task 1  - 1 hour as i preparet the agents promots so can be used in another tasks.
2. task 2  - 30 min 
3. task 3 - 40 min 
4, task 5 - 30 min  