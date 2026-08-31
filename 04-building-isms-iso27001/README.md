# Building an ISO/IEC 27001:2022 ISMS: Abionic Inc.

**What this shows:** an ISMS documentation set built from scope and leadership through risk assessment, Annex A control policies and performance evaluation, governed by a full 93-control Statement of Applicability, together with a self-assessed clause evidence map that states plainly which requirements are evidenced and which are not.

Built as the practical component of ISO/IEC 27001:2022 Lead Auditor certification work, run as though the deliverables were going to a real audit. **Abionic Inc. is a fictitious organisation**, a small AWS and SaaS based software company used as the subject.

**This is not a certifiable ISMS.** Of the 27 main-clause requirements, 6 can only be evidenced by an organisation that actually operates over time, and are marked as such rather than counted as failures. Of the 21 a documentation-stage build can evidence, 20 are complete and 1 is in progress. Nothing is left unstarted.

---

## By the numbers

**Statement of Applicability and control coverage**

| | |
|---|---|
| Annex A controls assessed in the SoA | **93** |
| Applicable / excluded | **63 / 30**, every exclusion with a written justification |
| Of the 63 applicable: implemented | **47** |
| Partially implemented | **2** (A.8.7 malware protection, A.8.27 secure architecture principles) |
| Planned, with a named gap | **14** |
| Annex A control policies written | **11** |
| Information assets registered / risks scored | **5 / 5** |

**Main-clause conformity, self-assessed (clauses 4 to 10, 27 requirements)**

| Clause | Complete | Of those achievable | Position |
|---|---|---|---|
| 4. Context | 4 | 4 / 4 | Scope, interested parties and ISMS boundary defined |
| 5. Leadership | 3 | 3 / 3 | Policy, commitment, roles and responsibilities in place |
| 6. Planning | 5 | 5 / 5 | Risks and opportunities register, risk assessment, completed treatment plan, objectives, change management |
| 7. Support | 7 | 7 / 7 | Resources, competence, awareness programme, communication plan, and all three parts of documented-information control |
| 8. Operation | 1 | 1 / 2 | Risk assessment performed and retained (8.2); operational planning (8.1) defined but with no execution records yet; treatment implementation (8.3) needs an operating organisation |
| 9. Performance evaluation | 0 | not achievable | All three procedures written and scheduled; execution needs an operating organisation |
| 10. Improvement | 0 | not achievable | Nonconformity log created and correctly empty; nothing yet to improve from |
| **Total** | **20** | **20 / 21** | 20 complete, 1 in progress, 0 not started, 6 not achievable at this stage |

---

## How the ISMS is organised

Documents are grouped by ISO 27001 clause, so the folder structure is the clause structure.

| Folder | Clauses | Contents |
|---|---|---|
| `01-Context-and-Leadership/` | 4, 5 | ISMS scope, information security policy, roles and responsibilities |
| `02-Planning/` | 6 | Risks and opportunities register, ISMS objectives, risk assessment process, risk treatment process and plan, change management procedure, and the **Statement of Applicability** (asset register, completed risk treatment, full 93-control SoA) |
| `03-Support/` | 7 | ISMS resources statement, competence records, awareness programme, ISMS communication plan, documented information control procedure |
| `04-Annex-A-Policies/` | Annex A | 11 control policies: acceptable use, access control, asset management and data classification, business continuity and disaster recovery, cryptography, data retention and disposal, incident management, monitoring and logging, secure development lifecycle, supplier and vendor management, training and awareness |
| `05-Performance-Evaluation/` | 9 | Monitoring and measurement procedure, internal audit programme, management review procedure and meeting record template |
| `06-Improvement/` | 10 | Nonconformity and corrective action log |
| `07-Operation/` | 8 | Operational planning and control procedure |

Two key documents:

- **`ISO27001_Clause_Evidence_Map_Abionic_Inc.docx`** traces every clause requirement to the document that evidences it, with a status. This is the file to open first.
- **`Abionic_Control_Evidence_Tracker.xlsx`** traces every applicable Annex A control to the policy that implements it, with a status and, where the control is not met, the specific gap.