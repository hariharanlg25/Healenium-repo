# Module 12: 14-Day Structured SDET Study & Mastery Roadmap

[⬅ Previous: Revision Notes & Glossary](11_revision_notes_and_glossary.md) | [📚 Master Index](../README.md)

---

## 12.1 Curriculum Overview
This 14-day study plan is tailored for Senior SDETs, Automation Leads, and QA Architects. It emphasizes conceptual understanding, scenario analysis, trade-off evaluation, and interview readiness without requiring full software development tasks.

```
+------------------------------------------------------------------------------------+
|                           14-DAY HEALENIUM MASTERY ROADMAP                         |
+---------------------+--------------------------------------------------------------+
| Phase               | Daily Focus                                                  |
+---------------------+--------------------------------------------------------------+
| Phase 1: Core       | Day 1: Locator Flakiness | Day 2: Architecture               |
| Fundamentals        | Day 3: Lifecycle of a Healed Locator                         |
|                     | Day 4: Tree Comparison & Algorithm Deep Dive                 |
+---------------------+--------------------------------------------------------------+
| Phase 2: Components | Day 5: Backend & PostgreSQL Schemas                          |
| & Infrastructure    | Day 6: Healenium-Web vs Healenium-Proxy                      |
|                     | Day 7: Configuration Tuning (score-cap, recovery-tries)      |
+---------------------+--------------------------------------------------------------+
| Phase 3: Integration| Day 8: Page Object Model & Framework Design                  |
| & Engineering       | Day 9: False Positives & Risk Management                     |
|                     | Day 10: Troubleshooting Matrix & Root Cause Analysis         |
+---------------------+--------------------------------------------------------------+
| Phase 4: Enterprise | Day 11: CI/CD Pipeline Orchestration                         |
| Scale & Interviews  | Day 12: Framework Comparison (Playwright, Selenium, Healenium)|
|                     | Day 13: Enterprise Governance & Triage Cadence               |
|                     | Day 14: Mock Interview & Architectural Defense               |
+---------------------+--------------------------------------------------------------+
```

---

## 12.2 Daily Breakdown & Conceptual Exercises

### Day 1: The Problem of Locator Fragility
* **Learning Objective:** Articulate why traditional Selenium locators break, quantify maintenance overhead, and contrast heuristic locator recovery with explicit waits.
* **Topics to Revise:** DOM volatility, dynamic framework IDs (React/Angular), `NoSuchElementException`, automation maintenance costs.
* **Conceptual Exercise:** Take a standard login form HTML snippet. Rewrite it as if a frontend team migrated it to Tailwind CSS with dynamic IDs. List which traditional Selenium locator strategies would break and which would survive.

### Day 2: Healenium Architecture & Microservices
* **Learning Objective:** Diagram and explain the interactions between test runner, `SelfHealingDriver`, `healenium-backend`, and PostgreSQL.
* **Topics to Revise:** Microservice boundaries, role of Docker, REST communication between client and backend.
* **Conceptual Exercise:** Draw a sequence diagram from memory showing all network calls that occur when a locator fails during test execution.

### Day 3: The Lifecycle of a Healed Locator
* **Learning Objective:** Walk through the complete lifecycle of a locator from initial baseline capture to healing, reporting, and source code synchronization.
* **Topics to Revise:** Baseline creation, exception interception, DOM tree capture, reporting UI, IntelliJ IDEA plugin.
* **Conceptual Exercise:** Write a 1-page technical memo explaining to a junior QA engineer why an element cannot heal if its test has never passed before.

### Day 4: Deep Dive into the Tree Comparison Algorithm
* **Learning Objective:** Understand how node similarity scores are calculated using tags, paths, attributes, and text.
* **Topics to Revise:** Longest Common Subsequence (LCS), Tree Edit Distance, Jaccard similarity for classes, weighted attribute scoring.
* **Conceptual Exercise:** Manually calculate a hypothetical similarity score between a baseline `<button id="submit" class="btn primary">Submit</button>` and three candidate nodes. Determine which candidate wins under a `0.6` threshold.

### Day 5: Backend Services & Database Schema
* **Learning Objective:** Understand the relational schema supporting Healenium and know how to query it for troubleshooting.
* **Topics to Revise:** PostgreSQL schema design, `selector` table, `healing` log table, Docker volume persistence.
* **Conceptual Exercise:** Write hypothetical SQL queries to: (a) find the top 5 most frequently healed locators in a test suite, and (b) identify any healing events with a similarity score between 0.60 and 0.65.

### Day 6: Integration Models: Healenium-Web vs. Healenium-Proxy
* **Learning Objective:** Evaluate when to implement `healenium-web` vs. `healenium-proxy`.
* **Topics to Revise:** Client wrapper vs. W3C network reverse proxy, multi-language support (Python, C#, JS), performance overhead.
* **Conceptual Exercise:** Draft an architectural recommendation for an enterprise organization with 3 automation frameworks (Java TestNG, Python Pytest, and Cypress). Defend your choice of `healenium-web` vs `healenium-proxy`.

### Day 7: Configuration Tuning & Risk Engineering
* **Learning Objective:** Configure `healenium.properties` for different test environments and balance healing flexibility against false-positive risks.
* **Topics to Revise:** `score-cap`, `recovery-tries`, `heal-enabled`, `back-end-url`.
* **Conceptual Exercise:** Analyze the trade-offs of setting `score-cap=0.85` vs `score-cap=0.50` in a fast-paced agile sprint. Formulate a risk mitigation strategy for each.

### Day 8: Framework Integration (POM & Driver Factory)
* **Learning Objective:** Design a clean, maintainable Driver Factory incorporating `SelfHealingDriver` without polluting Page Object classes.
* **Topics to Revise:** Decorator Pattern, `WebDriver` interface wrapping, Page Object Model cleanliness, ThreadLocal driver patterns for parallel runs.
* **Conceptual Exercise:** Draft a thread-safe `DriverFactory` class in Java that inspects a system property (`-Dhealenium=true/false`) to conditionally return either a native or a self-healing driver.

### Day 9: The Danger of False Positives
* **Learning Objective:** Analyze failure modes where self-healing selects the wrong element, and design defensive assertions to catch them.
* **Topics to Revise:** True Positives vs. False Positives, data table row ambiguities, semantic validation vs. element recovery.
* **Conceptual Exercise:** Review a provided scenario where clicking a healed button executes a "Delete All Records" operation instead of "Delete Selected". Write the strict post-action assertions needed to catch this issue.

### Day 10: Troubleshooting Matrix & Root Cause Analysis
* **Learning Objective:** Rapidly diagnose the 10 most common Healenium operational and infrastructure errors.
* **Topics to Revise:** `ConnectException`, `NoSuchMethodError`, Docker networking issues, empty reports, broken baselines.
* **Conceptual Exercise:** Review three anonymized stack traces and identify whether the root cause is: (a) a network connectivity failure, (b) a binary library incompatibility, or (c) an unestablished baseline.

### Day 11: CI/CD Orchestration & Scale
* **Learning Objective:** Design a robust CI/CD pipeline running parallel automated tests backed by persistent Healenium services.
* **Topics to Revise:** Ephemeral vs. persistent backend deployments, Docker-in-Docker networking, pipeline failure thresholds.
* **Conceptual Exercise:** Draw a CI/CD architecture diagram showing how 4 parallel Jenkins agent nodes execute tests against a centralized Healenium cluster.

### Day 12: Tool Comparison: Healenium vs. Alternative Solutions
* **Learning Objective:** Articulate the exact strengths and weaknesses of Healenium compared to Playwright, Testim, Mabl, and Applitools.
* **Topics to Revise:** Commercial AI tools vs. Open-Source, auto-waiting vs. self-healing, DOM healing vs. Visual AI healing.
* **Conceptual Exercise:** Create a comprehensive comparative matrix evaluating Healenium against Playwright and commercial SaaS solutions across Cost, Ease of Setup, Resilience, and Vendor Lock-in.

### Day 13: Team Governance, Triage & Code Synchronization
* **Learning Objective:** Establish an enterprise process for reviewing, accepting, and merging healed locators into version control.
* **Topics to Revise:** Triage workflows, IntelliJ IDEA plugin usage, technical debt prevention, collaboration between QA and Frontend developers.
* **Conceptual Exercise:** Draft a standard operating procedure (SOP) document outlining the weekly QA triage routine for reviewing Healenium dashboard reports.

### Day 14: Final Interview Simulation & Architectural Defense
* **Learning Objective:** Confidently answer complex, scenario-based interview questions and defend Healenium's architectural trade-offs.
* **Topics to Revise:** All 40+ interview questions from Chapter 10, key revision points, architecture diagrams.
* **Conceptual Exercise:** Conduct a self-mock interview: Select 10 random questions from Chapter 10, answer each aloud within 2 minutes, and critique your answers for clarity, depth, and precision.

---

[⬅ Previous: Revision Notes & Glossary](11_revision_notes_and_glossary.md) | [📚 Master Index](../README.md)
