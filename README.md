# Healenium: SDET Comprehensive Study Material & Curriculum Guide

> **Role & Perspective:** Senior SDET & Technical Trainer Specializing in Selenium Automation & Healenium  
> **Target Audience:** SDETs, QA Automation Architects, Test Leads, and Automation Engineers  
> **Focus:** Theoretical Foundation, Architecture, Troubleshooting, Governance, and Interview Mastery  
> **Official References:** [Healenium.io](https://healenium.io) | [Healenium GitHub Organization](https://github.com/healenium)

---

## 📚 Modular Curriculum & Study Modules

This study material has been divided into 12 dedicated, topic-wise study modules located in the [`study_guide/`](study_guide/) directory. Each module includes conceptual explanations, architectural diagrams, structured matrices, and practical guidance.

```
Healenium Study Guide Architecture
├── README.md (Master Index & Curriculum Overview)
└── study_guide/
    ├── 01_introduction_to_healenium.md
    ├── 02_healenium_architecture.md
    ├── 03_core_technical_concepts.md
    ├── 04_components_and_technologies.md
    ├── 05_conceptual_examples.md
    ├── 06_configuration_and_integration.md
    ├── 07_troubleshooting_and_debugging.md
    ├── 08_best_practices_and_limitations.md
    ├── 09_real_world_sdet_scenarios.md
    ├── 10_interview_preparation.md
    ├── 11_revision_notes_and_glossary.md
    └── 12_learning_roadmap_14_days.md
```

---

## 📖 Module Directory & Direct Links

### [Module 01: Introduction to Healenium & Self-Healing Locators](study_guide/01_introduction_to_healenium.md)
* Definition, core mission, and the problem of brittle locators in modern web applications.
* What runtime self-healing actually means versus common misconceptions.
* Why locators break (framework IDs, design refactoring, DOM churn).
* Comprehensive trade-off analysis: Advantages, disadvantages, and technical limitations.
* Head-to-head comparison: Standard Selenium WebDriver vs. Healenium-enhanced automation.

### [Module 02: Healenium Architecture & Workflow Deep Dive](study_guide/02_healenium_architecture.md)
* Detailed roles of the four core pillars: `healenium-web`, `healenium-proxy`, `healenium-backend`, and PostgreSQL.
* Component relationships and network transport boundaries.
* Step-by-step locator healing workflow: Baseline capture, failure detection, healing computation, dynamic substitution, and report logging.
* **Mermaid sequence diagrams** capturing runtime interactions.
* Native Client Library (`healenium-web`) vs. Network Proxy (`healenium-proxy`) architectural comparison.

### [Module 03: Core Technical Concepts & Locator Healing Engine](study_guide/03_core_technical_concepts.md)
* Locators, selector strategies, and DOM tree structures.
* DOM changes and structural mutation analysis.
* The Healing Algorithm: Tree-comparing engine, Longest Common Subsequence (LCS), and the multi-factor weighted scoring formula.
* Cutoff score threshold (`score-cap`).
* Healing outcomes: Success (True Positive), Failure (True Negative), and False Positives.
* **Crucial SDET Distinction:** Recovering an element vs. validating application behavior.

### [Module 04: Healenium Components & Technologies](study_guide/04_components_and_technologies.md)
* Detailed breakdown of underlying technologies: Selenium WebDriver, Java, Maven/Gradle, Docker & Docker Compose, PostgreSQL, and Spring Boot.
* In-depth comparison: `healenium-web` vs. `healenium-proxy` topologies.
* Why they are not identical and how to select the right pattern for your team.

### [Module 05: Conceptual Scenarios & Educational Code Snippets](study_guide/05_conceptual_examples.md)
* Educational code walkthroughs (not runnable projects, but conceptual snippets):
  1. Baseline state on a passing lookup.
  2. Front-end DOM refactoring that breaks a standard selector.
  3. Step-by-step mathematical candidate scoring by the backend.
  4. Successful element recovery scenario.
  5. Legitimate healing failure when a feature is deleted.
  6. False-positive hazard where an unintended button is clicked.

### [Module 06: Configuration, Setup & Framework Integration](study_guide/06_configuration_and_integration.md)
* Technical prerequisites (JDK, Docker, network connectivity).
* Docker Compose deployment manifest (`docker-compose.yaml`) for backend and database.
* Maven dependencies (`pom.xml`) for Selenium and Healenium.
* Property configuration (`healenium.properties`) with detailed parameter explanations.
* Integrating Healenium into existing Page Object Model (POM) frameworks using the Decorator Pattern (`SelfHealingDriver.create()`).

### [Module 07: Enterprise Troubleshooting & Debugging Matrix](study_guide/07_troubleshooting_and_debugging.md)
* Diagnostic matrix covering the 10 most common enterprise issues:
  1. Healing not being triggered on `NoSuchElementException`.
  2. Backend connection refused (`ConnectException`).
  3. PostgreSQL database failure or container crash.
  4. Missing selector history after container restart (volume misconfiguration).
  5. Dependency conflicts & binary incompatibilities (`NoSuchMethodError`).
  6. Healed locator selects the wrong element (false positive).
  7. Empty or 404 reporting dashboard.
  8. Tests pass locally on workstation but fail in CI/CD pipeline.
  9. Large DOM trees causing execution timeouts.
  10. Lost healing inside embedded iFrames or Shadow DOM roots.
* Each problem includes symptoms, root causes, diagnostic commands, and corrective actions.

### [Module 08: Best Practices, Risks, Pitfalls & Governance](study_guide/08_best_practices_and_limitations.md)
* The SDET Governance Rules: Safety net vs. sloppy locator prevention.
* When to use Healenium vs. when to demand immutable `data-testid` attributes.
* Weekly triage cadence and locator synchronization.
* How self-healing can inadvertently mask real application regressions.
* Why hard assertions on business behavior must always be retained.

### [Module 09: Real-World SDET Case Studies](study_guide/09_real_world_sdet_scenarios.md)
* **Case Study 1:** Modernizing a legacy banking portal from AngularJS to React (142 broken locators, 92.2% automated healing rate, triage of unhealed flows).
* **Case Study 2:** Dynamic data grids in an e-commerce catalog (false-positive incident, wrong product added to cart, root-cause analysis, and resilient semantic locator refactoring).

### [Module 10: Comprehensive Interview Preparation (40+ Questions & Answers)](study_guide/10_interview_preparation.md)
* 40 exhaustive, technically rigorous interview questions and answers organized across 8 categories:
  1. Fundamentals & Conceptual Understanding (Q1–Q5)
  2. Architecture & Component Relationships (Q6–Q10)
  3. Selenium Integration & Implementation (Q11–Q15)
  4. Locator Healing Mechanics & Algorithms (Q16–Q20)
  5. Troubleshooting & Debugging (Q21–Q25)
  6. Reporting & Visual Triage (Q26–Q30)
  7. CI/CD & Pipeline Orchestration (Q31–Q35)
  8. Advanced Scenarios & Architectural Trade-offs (Q36–Q40)

### [Module 11: Quick Revision Notes, Component Comparison & Glossary](study_guide/11_revision_notes_and_glossary.md)
* High-yield revision sheet summarizing core facts.
* Component comparison table (`healenium-web` vs `healenium-proxy` vs `healenium-backend` vs `postgres`).
* Common misconceptions debunked.
* Essential SDET terminology glossary (Baseline, Candidate, Composite Score, Score-Cap, False Positive, Tree Edit Distance).

### [Module 12: 14-Day Structured SDET Study & Mastery Roadmap](study_guide/12_learning_roadmap_14_days.md)
* Structured 14-day study plan covering 4 distinct phases:
  * Phase 1: Core Fundamentals (Days 1–4)
  * Phase 2: Components & Infrastructure (Days 5–7)
  * Phase 3: Integration & Engineering (Days 8–10)
  * Phase 4: Enterprise Scale & Interviews (Days 11–14)
* Each day contains specific learning objectives, revision topics, and conceptual scenario exercises.

---

## 📌 Single-File Edition Available
For users who also prefer reading the entire study material in a single continuous document, an omnibus edition is preserved at:  
👉 **[HEALENIUM_SDET_COMPREHENSIVE_GUIDE.md](HEALENIUM_SDET_COMPREHENSIVE_GUIDE.md)**
