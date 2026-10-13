# Test Cases — Move CDD Approvals to CR Level (SW Implementations)

**Document type:** Functional / System test cases
**Source requirement:** SW Implementations — "Move all CDD approvals to CR level" + Configuration Changes + CRC final message change
**Status:** Draft for review
**Owner (QA):** _to be assigned_

---

## 1. Purpose & Scope

This document defines the test cases required to verify the requirement to **move all CDD (Civil Defense Department) approvals from CP level to CR level**, adjust the associated process-code configuration, change the CRC-calling behaviour, and update the CRC final notification message.

In scope:
1. Process-code assignment/de-assignment at CR vs. CP level for **New Main CR**, **Add Activity to CR**, **Issue CP of Main-CR**, **Renew CR / Renew CP**, **Closure**, **Delete Activity**.
2. Branch handling: **Add Branch** (no CDD approval) and **Issue-CP of Branch-CR** (keep `PROC-MOI73-4`).
3. CRC calling behaviour (no CRC call except when an SR contains both Delete and Add activity).
4. EID payload content for **Renewal (CR)** and **Closure (CR)** — main EID + all CDD activities of the affected branches.
5. Notification template `application_follow_up_moi_crc` final message change (EN + AR).

Out of scope:
- Non-CDD business activities (other than the de-assignment regression check).
- Performance, security, and infrastructure testing.
- UI/UX cosmetic checks unrelated to the flows above.

---

## 2. Definitions

| Term | Meaning |
|---|---|
| **CDD** | Civil Defense Department (Ministry of Interior). |
| **CR** | Company Registration / Commercial Registration (main registration entity). |
| **CP** | Commercial Permit / Company Practice (branch or activity-level permit). |
| **EID** | Establishment ID (main EID identifies the company). |
| **SR** | Service Request (single-window request submitted by the user). |
| **CRC** | Civil Registration / MOI request channel invoked by the system (call to MOI). |
| **CDD Business Activity** | Any Business Activity whose code is in the CDD list (§3). |
| **Non-CDD Business Activity** | Any Business Activity whose code is NOT in the CDD list (§3). |

### 2.1 CDD Business Activity list (from requirement)

`432122`, `432911`, `465963`, `465980`, `475961`, `477388`

### 2.2 Process codes under test

| Process code | EN name | New usage |
|---|---|---|
| `PROC-MOI73-3` | Approval to Register Safety Systems Companies | Assign — New Main CR & Add Activity to CR |
| `PROC-MOI73-4` | Approval to license Safety Systems Companies | De-assign from Main-CR Issue-CP; **kept** for Issue-CP of Branch-CR |
| `PROC-MOI73-440` | Approval to Renew Registration of Safety Systems Companies | Assign — Renew CR |
| `PROC-MOI73-460` | Approval to Renew License of Safety Systems Companies | De-assign — Renew CP |
| `PROC-MOI73-600` | Approval to Delete Registration of Safety Systems Companies | Assign — Closure & Delete Activity |
| `PROC-MOI73-650` | Approval to Delete License of Safety Systems Companies | De-assign — Closure & Delete Activity |

---

## 3. Test Data & Preconditions

- **CDD activities:** `432122`, `432911`, `465963`, `465980`, `475961`, `477388`.
- **Non-CDD activity (control):** any activity code not in the list above (e.g. `432199`) — replace with a real non-CDD code from the environment.
- A company with a valid **main EID**, at least one CR, and at least two branches where a branch holds a CDD activity are available.
- Configuration is versioned so mapping can be asserted directly (Config/DB level) in addition to behaviour level.
- Notification/email capture (outbox, DB, or log) is available to assert the CRC message content.

### 3.1 Test case fields

Each case below uses: **ID | Title | Priority | Type | Preconditions | Steps | Expected Result**.

Priority legend: **P1** critical, **P2** high, **P3** medium.

---

## 4. Test Cases

### Module A — Configuration: Business Activity ↔ Process Code Mapping

| ID | Title | Priority | Type | Preconditions | Steps | Expected Result |
|---|---|---|---|---|---|---|
| CFG-001 | All six CDD activities exist and are flagged as CDD | P1 | Positive | Config admin access | 1. Query the business-activity configuration. 2. Filter for the six listed codes. | All six codes (`432122`, `432911`, `465963`, `465980`, `475961`, `477388`) exist and are flagged as CDD Business Activities. |
| CFG-002 | CDD process codes de-assigned from all non-CDD activities | P1 | Negative / Regression | Config access | 1. For each of `PROC-MOI73-3`, `-4`, `-440`, `-460`, `-600`, `-650`, query every business activity NOT in the CDD list. 2. Check the mapping. | None of the six CDD process codes is assigned to any non-CDD business activity. |
| CFG-003 | `PROC-MOI73-3` assigned to all CDD activities for **New Main CR** | P1 | Positive | Config access | 1. For each CDD activity, check the New Main CR mapping. | `PROC-MOI73-3` is assigned to all six CDD activities for New Main CR. |
| CFG-004 | `PROC-MOI73-3` assigned to all CDD activities for **Add Activity to CR** | P1 | Positive | Config access | 1. For each CDD activity, check the Add Activity to CR mapping. | `PROC-MOI73-3` is assigned to all six CDD activities for Add Activity to CR. |
| CFG-005 | `PROC-MOI73-4` de-assigned from all CDD activities for **Issue CP of Main-CR** | P1 | Negative | Config access | 1. For each CDD activity, check the Issue CP of Main-CR mapping. | `PROC-MOI73-4` is NOT assigned to any CDD activity for Issue CP of Main-CR. |
| CFG-006 | `PROC-MOI73-440` assigned to all CDD activities for **Renew CR** | P1 | Positive | Config access | 1. For each CDD activity, check the Renew CR mapping. | `PROC-MOI73-440` is assigned to all six CDD activities for Renew CR. |
| CFG-007 | `PROC-MOI73-460` de-assigned from all CDD activities for **Renew CP** | P1 | Negative | Config access | 1. For each CDD activity, check the Renew CP mapping. | `PROC-MOI73-460` is NOT assigned to any CDD activity for Renew CP. |
| CFG-008 | `PROC-MOI73-600` assigned to all CDD activities for **Closure** | P1 | Positive | Config access | 1. For each CDD activity, check the Closure mapping. | `PROC-MOI73-600` is assigned to all six CDD activities for Closure. |
| CFG-009 | `PROC-MOI73-600` assigned to all CDD activities for **Delete Activity** | P1 | Positive | Config access | 1. For each CDD activity, check the Delete Activity mapping. | `PROC-MOI73-600` is assigned to all six CDD activities for Delete Activity. |
| CFG-010 | `PROC-MOI73-650` de-assigned from all CDD activities for **Closure** | P1 | Negative | Config access | 1. For each CDD activity, check the Closure mapping. | `PROC-MOI73-650` is NOT assigned to any CDD activity for Closure. |
| CFG-011 | `PROC-MOI73-650` de-assigned from all CDD activities for **Delete Activity** | P1 | Negative | Config access | 1. For each CDD activity, check the Delete Activity mapping. | `PROC-MOI73-650` is NOT assigned to any CDD activity for Delete Activity. |
| CFG-012 | `PROC-MOI73-4` **kept** for Issue-CP of **Branch-CR** | P1 | Positive | Config access | 1. For each CDD activity, check the Issue-CP of Branch-CR mapping. | `PROC-MOI73-4` remains assigned to all six CDD activities for Issue-CP of Branch-CR (unchanged). |
| CFG-013 | No CDD process code assigned for **Add Branch** | P1 | Negative | Config access | 1. Check the Add Branch mapping for all CDD activities. | No CDD process code (including `-3`/`-4`/`-440`/`-460`/`-600`/`-650`) is assigned for Add Branch. |
| CFG-014 | No orphaned/duplicate process-code assignments after change | P2 | Regression | Config snapshots before & after | 1. Diff the mapping before and after the configuration change. | Only the intended rows changed (add `-3`,`-440`,`-600`; remove `-4` (Main-CR only),`-460`,`-650`; non-CDD rows cleared); no unintended rows. |
| CFG-015 | Configuration applies consistently across all CDD activities | P2 | Positive | Config access | 1. For each process code and CR/CP event, confirm the same result applies to all six activities. | Behaviour is identical for every one of the six CDD activities; no activity is missed. |

### Module B — New Main CR (CDD approval moved to CR)

| ID | Title | Priority | Type | Preconditions | Steps | Expected Result |
|---|---|---|---|---|---|---|
| CRN-001 | New Main CR with CDD activity triggers `PROC-MOI73-3` **at CR level** | P1 | Positive | Company ready; CDD activity selected | 1. Submit New Main CR with a CDD activity. 2. Observe the approval task generated. | Approval task `PROC-MOI73-3` (Approval to Register Safety Systems Companies) is generated **at CR level**, routed to CDD. |
| CRN-002 | New Main CR with non-CDD activity does not trigger CDD approval | P1 | Negative | Company ready | 1. Submit New Main CR with a non-CDD activity. 2. Observe approvals. | No CDD approval process code is generated; flow proceeds without CDD. |
| CRN-003 | CR approval cannot be completed before CDD approves | P2 | Positive | CR under CDD review | 1. Attempt to progress/complete the CR while CDD approval is pending. | System blocks progression until CDD approval on `PROC-MOI73-3` is granted. |
| CRN-004 | Issue CP of Main-CR does **not** require `PROC-MOI73-4` | P1 | Negative | New Main CR approved at CR | 1. Issue CP of the Main-CR with a CDD activity. 2. Observe approvals. | No CDD approval (`PROC-MOI73-4`) is generated at CP level; CP can be issued without CDD. |
| CRN-005 | CDD rejection at CR level rejects/blocks the CR correctly | P1 | Negative | CR under CDD review | 1. CDD rejects `PROC-MOI73-3`. 2. Observe CR status. | CR is rejected/blocked per business rules; appropriate rejection reason/notification is recorded. |
| CRN-006 | Audit trail shows the CDD decision at CR level | P2 | Positive | CR completed | 1. Open the CR history. | History records the CDD `PROC-MOI73-3` approval event at CR level (actor, timestamp, decision). |

### Module C — Add Activity to CR

| ID | Title | Priority | Type | Preconditions | Steps | Expected Result |
|---|---|---|---|---|---|---|
| ADD-001 | Add CDD activity to CR triggers `PROC-MOI73-3` at CR level | P1 | Positive | Existing CR | 1. Add a CDD activity to the CR. 2. Observe approvals. | `PROC-MOI73-3` is generated at CR level for the added activity. |
| ADD-002 | Add non-CDD activity does not trigger CDD approval | P1 | Negative | Existing CR | 1. Add a non-CDD activity. | No CDD approval is generated. |
| ADD-003 | CRC of the Add activity is sent when SR contains **both** Delete and Add activity | P1 | Positive | SR with Delete + Add activity | 1. Submit SR containing both a Delete activity and an Add activity. 2. Observe CRC call. | CRC is called and the **CRC of the Add activities** is sent (see Module H). |
| ADD-004 | Multiple CDD activities added in one CR all covered | P2 | Positive | Existing CR | 1. Add two or more CDD activities. | All added CDD activities are covered by the CR-level approval requirement. |

### Module D — Branch flows (unchanged behaviour)

| ID | Title | Priority | Type | Preconditions | Steps | Expected Result |
|---|---|---|---|---|---|---|
| BR-001 | Add Branch still requires no CDD approval | P1 | Negative | Company with CR | 1. Add a branch. 2. Observe approvals. | No CDD approval process code is generated for Add Branch. |
| BR-002 | Issue-CP of Branch-CR still uses `PROC-MOI73-4` | P1 | Positive | Branch added | 1. Issue CP of the branch with a CDD activity. 2. Observe approvals. | `PROC-MOI73-4` (Approval to license Safety Systems Companies) is generated at CP level for the branch. |
| BR-003 | Branch-CR with non-CDD activity does not trigger CDD approval | P2 | Negative | Branch added | 1. Issue CP of branch with a non-CDD activity. | No CDD approval generated. |
| BR-004 | Main-CR and Branch-CR behave independently in same company | P2 | Positive | Company with main CR + branch | 1. Issue CP of Main-CR (CDD activity). 2. Issue CP of Branch-CR (CDD activity). | Main-CR Issue-CP has no CDD approval; Branch-CR Issue-CP generates `PROC-MOI73-4`. |

### Module E — Renewal (CDD approval moved from CP to CR)

| ID | Title | Priority | Type | Preconditions | Steps | Expected Result |
|---|---|---|---|---|---|---|
| REN-001 | Renew CR with CDD activity triggers `PROC-MOI73-440` at CR level | P1 | Positive | CR due for renewal | 1. Start Renew CR with a CDD activity. 2. Observe approvals. | `PROC-MOI73-440` (Approval to Renew Registration of Safety Systems Companies) is generated at CR level. |
| REN-002 | Renew CP no longer uses `PROC-MOI73-460` | P1 | Negative | CR renewed | 1. Perform Renew CP with a CDD activity. 2. Observe approvals. | `PROC-MOI73-460` (Renew License) is NOT generated at CP level. |
| REN-003 | Renew CR of non-CDD activity does not trigger CDD approval | P1 | Negative | CR due for renewal | 1. Start Renew CR with a non-CDD activity. | No CDD approval generated. |
| REN-004 | Renew CR EID payload includes main EID + all CDD activities of to-be-renewed branches | P1 | Positive | Company with branches holding CDD activities | 1. Start Renew CR covering branches. 2. Inspect the payload sent to CDD. | Payload contains the **main EID** and **all civil-defense activities of all branches to be renewed** (aggregated). |
| REN-005 | Renew CR with multiple branches aggregates activities correctly | P2 | Positive | ≥2 branches with CDD activities | 1. Renew CR for the company. 2. Inspect payload. | All CDD activities from every branch to be renewed are included; none missing, none duplicated. |
| REN-006 | Renew CR with a branch holding no CDD activity | P2 | Edge | Branch without CDD activity | 1. Renew CR including a branch with no CDD activity. 2. Inspect payload. | Payload includes main EID and CDD activities only; no empty/invalid entries for the non-CDD branch. |
| REN-007 | Renewal blocks progression until CR-level CDD approval | P2 | Positive | Renewal under CDD review | 1. Attempt to complete renewal while pending. | Progression blocked until `PROC-MOI73-440` is approved. |
| REN-008 | Renew CP of branch (Branch-CR) behaviour unaffected | P2 | Edge | Branch due for renewal | 1. Perform renewal for a branch. | Branch renewal behaviour is unaffected by the main-CR change. |

### Module F — Closure (CDD approval moved from CP to CR)

| ID | Title | Priority | Type | Preconditions | Steps | Expected Result |
|---|---|---|---|---|---|---|
| CLO-001 | Closure with CDD activity triggers `PROC-MOI73-600` at CR level | P1 | Positive | Company to be closed | 1. Start Closure with a CDD activity. 2. Observe approvals. | `PROC-MOI73-600` (Approval to Delete Registration of Safety Systems Companies) is generated at CR level. |
| CLO-002 | Closure no longer uses `PROC-MOI73-650` | P1 | Negative | Company to be closed | 1. Perform Closure. 2. Observe approvals. | `PROC-MOI73-650` (Delete License) is NOT generated at CP level. |
| CLO-003 | Closure of non-CDD activity does not trigger CDD approval | P1 | Negative | Company to be closed | 1. Start Closure with a non-CDD activity. | No CDD approval generated. |
| CLO-004 | Closure CR EID payload includes main EID + all CDD activities of to-be-closed branches | P1 | Positive | Company with branches holding CDD activities | 1. Start Closure CR covering branches. 2. Inspect payload sent to CDD. | Payload contains the **main EID** and **all civil-defense activities of all branches to be closed** (aggregated). |
| CLO-005 | No CRC call for Closure | P1 | Negative | Company to be closed, no Add activity in SR | 1. Perform Closure. 2. Observe whether CRC is called. | CRC is **not** called for Closure (per requirement). |
| CLO-006 | Closure with multiple branches aggregates activities correctly | P2 | Positive | ≥2 branches with CDD activities | 1. Close CR for the company. 2. Inspect payload. | All CDD activities from every branch to be closed are included. |
| CLO-007 | Closure blocks progression until CR-level CDD approval | P2 | Positive | Closure under CDD review | 1. Attempt to complete closure while pending. | Progression blocked until `PROC-MOI73-600` is approved. |

### Module G — Delete Activity (CDD approval moved from CP to CR)

| ID | Title | Priority | Type | Preconditions | Steps | Expected Result |
|---|---|---|---|---|---|---|
| DEL-001 | Delete CDD activity triggers `PROC-MOI73-600` at CR level | P1 | Positive | CR with a CDD activity | 1. Delete a CDD activity from the CR. 2. Observe approvals. | `PROC-MOI73-600` is generated at CR level. |
| DEL-002 | Delete activity no longer uses `PROC-MOI73-650` | P1 | Negative | CR with a CDD activity | 1. Delete a CDD activity. 2. Observe approvals. | `PROC-MOI73-650` is NOT generated. |
| DEL-003 | Delete non-CDD activity does not trigger CDD approval | P1 | Negative | CR with a non-CDD activity | 1. Delete the activity. | No CDD approval generated. |
| DEL-004 | No CRC call for Delete Activity | P1 | Negative | Delete-only SR | 1. Delete a CDD activity. 2. Observe whether CRC is called. | CRC is **not** called for Delete Activity. |
| DEL-005 | Delete Activity blocks progression until CR-level CDD approval | P2 | Positive | Delete under CDD review | 1. Attempt to complete delete while pending. | Progression blocked until `PROC-MOI73-600` is approved. |

### Module H — CRC calling behaviour

| ID | Title | Priority | Type | Preconditions | Steps | Expected Result |
|---|---|---|---|---|---|---|
| CRC-001 | No CRC call for standalone Delete Activity | P1 | Negative | Delete-only SR | 1. Execute delete-only SR. | CRC is not invoked. |
| CRC-002 | No CRC call for standalone Closure | P1 | Negative | Closure-only SR | 1. Execute closure-only SR. | CRC is not invoked. |
| CRC-003 | No CRC call for standalone Add Activity | P1 | Negative | Add-only SR | 1. Execute add-only SR. | CRC is not invoked. |
| CRC-004 | CRC **is** called when SR has both Delete and Add activity | P1 | Positive | SR with Delete + Add | 1. Execute SR with both Delete and Add activity. 2. Observe CRC. | CRC is invoked. |
| CRC-005 | CRC payload contains the Add activities only | P1 | Positive | SR with Delete + Add | 1. Execute SR with both. 2. Inspect CRC payload. | CRC payload contains the **CRC of the Add activities** (not the deleted ones). |
| CRC-006 | CRC final message reflects success (see Module I) | P2 | Positive | CRC invoked | 1. Trigger CRC flow. 2. Capture notification. | Final CRC notification uses the updated success-message template. |

### Module I — Notification template `application_follow_up_moi_crc` (final message)

| ID | Title | Priority | Type | Preconditions | Steps | Expected Result |
|---|---|---|---|---|---|---|
| MSG-001 | EN final message content updated | P1 | Positive | CRC success event | 1. Trigger the template. 2. Read the EN context. | EN text is exactly: "We would like to inform you that ${company_name_en} has been registered in the Ministry of Interior - Civil Defense Department." + single-window service/request line + MOI reference line. |
| MSG-002 | AR final message content updated | P1 | Positive | CRC success event | 1. Trigger the template. 2. Read the AR context. | AR text matches the required Arabic success message using `${company_name_ar}`, `${service_type_ar}`, `${request_number}`, `${crc_ref_number}`. |
| MSG-003 | EN variable substitution | P1 | Positive | CRC success event | 1. Trigger; inspect rendered EN message. | `company_name_en`, `service_type_en`, `request_number`, `crc_ref_number` are all replaced with correct values. |
| MSG-004 | AR variable substitution | P1 | Positive | CRC success event | 1. Trigger; inspect rendered AR message. | `company_name_ar`, `service_type_ar`, `request_number`, `crc_ref_number` are all replaced with correct values. |
| MSG-005 | Previous (non-success) final text removed | P2 | Regression | CRC success event | 1. Compare against the old message. | Old final message text is fully replaced; no residual old wording. |
| MSG-006 | Long / special-character values render correctly | P3 | Edge | Long company name; request number with separators | 1. Trigger with long/special-char data. | Message renders correctly without truncation or broken formatting, in both EN and AR. |
| MSG-007 | Notification delivered on success only, exactly once | P2 | Positive | CRC success event | 1. Trigger CRC; capture delivery. | Notification is delivered on success with correct subject/recipient; sent exactly once. |
| MSG-008 | Unresolved placeholder guard | P3 | Negative | Missing variable value | 1. Trigger with a missing value. | No raw `${...}` placeholder is left visible; fallback/logging behaves per standard. |

### Module J — EID payload (Renewal CR & Closure CR) — cross-cutting

| ID | Title | Priority | Type | Preconditions | Steps | Expected Result |
|---|---|---|---|---|---|---|
| EID-001 | Both Renewal CR and Closure CR send the **main** EID | P1 | Positive | Company with branches | 1. Trigger Renewal CR; trigger Closure CR. 2. Inspect payloads. | Both flows send the main EID of the company (not a branch EID). |
| EID-002 | Renewal payload = main EID + all CDD activities of to-be-renewed branches | P1 | Positive | ≥1 branch with CDD activity | 1. Renew CR; inspect payload. | Payload matches the requirement exactly. |
| EID-003 | Closure payload = main EID + all CDD activities of to-be-closed branches | P1 | Positive | ≥1 branch with CDD activity | 1. Close CR; inspect payload. | Payload matches the requirement exactly. |
| EID-004 | Non-CDD activities excluded from EID payload | P2 | Negative | Mixed CDD/non-CDD branches | 1. Renew/close CR; inspect payload. | Only CDD activities are included in the CD payload. |
| EID-005 | Payload activity list is de-duplicated and complete | P2 | Edge | Same CDD activity on multiple branches | 1. Renew/close CR; inspect payload. | Each distinct CDD activity appears as required (no unintended duplicates); all branches' CDD activities included. |

---

## 5. Requirement → Test Case Traceability Matrix

| Requirement statement | Covered by |
|---|---|
| New Main CR & Add Activity to CR → start using `PROC-MOI73-3` | CFG-003, CFG-004, CRN-001, CRN-005, ADD-001 |
| Main-CR Issue CP → stop using `PROC-MOI73-4` | CFG-005, CRN-004 |
| Add Branch → still no CDD approval | CFG-013, BR-001 |
| Issue-CP of Branch-CR → keep `PROC-MOI73-4` | CFG-012, BR-002 |
| Renewal → use `PROC-MOI73-440` at CR | CFG-006, REN-001, REN-007 |
| Renewal → stop `PROC-MOI73-460` at CP | CFG-007, REN-002 |
| Renewal CR → send main EID + all CDD activities of to-be-renewed branches | REN-004, REN-005, REN-006, EID-001, EID-002, EID-004, EID-005 |
| Closure → use `PROC-MOI73-600` at CR / stop `PROC-MOI73-650` | CFG-008, CFG-010, CLO-001, CLO-002, CLO-007 |
| Delete Activity → use `PROC-MOI73-600` at CR / stop `PROC-MOI73-650` | CFG-009, CFG-011, DEL-001, DEL-002, DEL-005 |
| No CRC call unless SR = Delete + Add activity (send CRC of Add) | CLO-005, DEL-004, CRC-001…CRC-006, ADD-003 |
| Closure CR → send main EID + all CDD activities of to-be-closed branches | CLO-004, CLO-006, EID-001, EID-003, EID-004, EID-005 |
| De-assign CDD process codes from non-CDD business activities | CFG-002, CFG-014 |
| Update `application_follow_up_moi_crc` final message (EN & AR) | MSG-001…MSG-008, CRC-006 |

---

## 6. Summary & Exit Criteria

| Module | Cases | P1 | P2 | P3 |
|---|---|---|---|---|
| A — Configuration mapping | 15 | 13 | 2 | 0 |
| B — New Main CR | 6 | 4 | 2 | 0 |
| C — Add Activity to CR | 4 | 3 | 1 | 0 |
| D — Branch flows | 4 | 2 | 2 | 0 |
| E — Renewal | 8 | 4 | 4 | 0 |
| F — Closure | 7 | 5 | 2 | 0 |
| G — Delete Activity | 5 | 4 | 1 | 0 |
| H — CRC calling | 6 | 5 | 1 | 0 |
| I — Notification template | 8 | 4 | 2 | 2 |
| J — EID payload | 5 | 3 | 2 | 0 |
| **Total** | **68** | **47** | **19** | **2** |

**Exit criteria**
- 100% of P1 test cases pass.
- No open P1/P2 defects.
- All configuration assertions (Module A) pass on the deployed configuration.
- EN + AR CRC message verified against the exact required wording.
- EID payloads for Renewal CR and Closure CR verified against the requirement.

**Open points / assumptions to confirm with BA**
1. Exact definition of "CRC" channel and how its invocation is observed/logged in the target environment.
2. Whether "Add Activity to CR" and "New Main CR" share the same approval task or are separate tasks (affects CRN/ADD granularity).
3. Behaviour for a branch that has no CDD activity during Renewal/Closure CR (assumed: excluded from payload — see REN-006).
4. Exact wording/format requirements for the service-type and request-number placeholders (concatenation vs. separate lines).

