# Module 08: Best Practices, Risks, Pitfalls & Governance

[⬅ Previous: Troubleshooting Matrix](07_troubleshooting_and_debugging.md) | [📚 Master Index](../README.md) | [Next: Real-World Case Studies ➡](09_real_world_sdet_scenarios.md)

---

## 8.1 The Core Governance Philosophy
Deploying self-healing tools without strict governance can lead to unintended consequences. An SDET must establish architectural boundaries:

```
+------------------------------------------------------------------------------------+
|                         SDET HEALENIUM GOVERNANCE RULES                            |
|                                                                                    |
|  [RULE 1]  Healenium is a safety net for CI, NOT an excuse for sloppy locators!   |
|  [RULE 2]  High-priority locators must use unique test attributes (data-testid).   |
|  [RULE 3]  Hard business assertions must remain active on every single test step.  |
|  [RULE 4]  Mandatory Weekly Review: Healed locators must be accepted or rejected.  |
|  [RULE 5]  Never lower score-cap below 0.6 in enterprise environments.             |
+------------------------------------------------------------------------------------+
```

---

## 8.2 When to Use Healenium vs. Stable Test IDs

| Scenario | Recommended Strategy | Rationale |
| :--- | :--- | :--- |
| **Internal Enterprise App with Dev Collaboration** | Dedicated `data-testid` attributes | Deterministic, immutable, zero performance overhead. |
| **Legacy Apps / Third-Party Vendor Portals** | **Healenium Self-Healing** | QA lacks access to modify source code; DOM changes frequently. |
| **Rapid Prototyping / Early MVP Sprints** | **Healenium Self-Healing** | UI shifts daily; healing keeps smoke test pipelines green. |
| **Financial Calculations & Mission-Critical Data** | Strict Locators + Strict Assertions | Risk of false-positive interaction is unacceptable. |
| **Nightly Regression Pipelines (1000+ Tests)** | **Healenium Self-Healing** | Prevents overnight suite failure due to minor CSS updates. |

---

## 8.3 The Healing Governance & Triage Workflow
Self-healing must not become a "black box" that engineers ignore. Establish a structured weekly cadence:

```
[Nightly CI Run] 
       │ (Healing events occur; tests stay green)
       ▼
[Healenium Dashboard Updated]
       │
       ▼
[Weekly Triage Meeting (15 mins)]
       │
       ├─► Candidate Valid? ────► Click "Accept" in Dashboard ──► Commit updated locator to Codebase
       │
       └─► Candidate Dubious? ──► Click "Reject" ──► Refactor Page Object to unique semantic anchor
```

1. **Never Let Healed Locators Linger Indefinitely:** Use the Healenium IntelliJ Plugin or Dashboard to merge stable healed locators back into your Git Page Object files. Self-healing is an operational safety net, not permanent storage for your selector definitions.
2. **Strict Score Thresholds:** Maintain `score-cap` at `0.65` or higher in production-grade pipelines to eliminate low-confidence false positives.
3. **Keep Assertions Rigid:** If you click a healed button, assert that the resulting URL, header, or modal is explicitly displayed. If the click was a false positive, your assertion will catch it immediately.

---

## 8.4 How Healing Can Mask Genuine Defects
A major danger for SDET teams is "healing masking a bug":
* **Scenario:** A frontend bug accidentally deletes the "Submit Application" button and renders an extra "Save Draft" button in the same container.
* **The Failure Mode:** Healenium evaluates the "Save Draft" button, observes identical tag (`BUTTON`), identical parent container (`DIV.form-actions`), and matching CSS classes (`btn-action`). The score exceeds `0.60`. Healenium clicks "Save Draft". The form saves instead of submitting.
* **The Mitigation:** If the test script does not strictly assert that the confirmation modal states `"Application Successfully Submitted"`, the test passes, and a severe release blocker slips into production!

---

[⬅ Previous: Troubleshooting Matrix](07_troubleshooting_and_debugging.md) | [📚 Master Index](../README.md) | [Next: Real-World Case Studies ➡](09_real_world_sdet_scenarios.md)
