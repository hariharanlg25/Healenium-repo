# Module 04: Healenium Components & Technologies

[⬅ Previous: Core Technical Concepts](03_core_technical_concepts.md) | [📚 Master Index](../README.md) | [Next: Conceptual Examples ➡](05_conceptual_examples.md)

---

## 4.1 Enterprise Technology Stack Overview
To deploy and maintain Healenium in an enterprise environment, an SDET must understand its underlying technology stack and the distinct responsibilities of each layer.

```
+----------------------------------------------------------------------------------+
|                            HEALENIUM TECHNOLOGY STACK                            |
|                                                                                  |
|   +-----------------------+   +-----------------------+   +------------------+   |
|   |   Java / Test Engine  |   |   Healenium Backend   |   |    Datastore     |   |
|   |  - JDK 8, 11, 17+     |   |  - Spring Boot        |   |  - PostgreSQL    |   |
|   |  - Selenium 3.x / 4.x |   |  - RESTful APIs       |   |  - Port 5432     |   |
|   |  - TestNG / JUnit     |   |  - Tree Compare Engine|   |  - Schemas:      |   |
|   |  - Maven / Gradle     |   |  - Port 7883          |   |    selectors,    |   |
|   +-----------------------+   +-----------------------+   |    healing_log   |   |
|                                                           +------------------+   |
|   +-----------------------+   +-----------------------+                          |
|   |   Containerization    |   |     Client Proxy      |                          |
|   |  - Docker Compose     |   |  - Healenium-Proxy    |                          |
|   |  - Docker Engine      |   |  - Netty / Reverse    |                          |
|   |  - Microservice mesh  |   |    HTTP Proxy         |                          |
|   +-----------------------+   +-----------------------+                          |
+----------------------------------------------------------------------------------+
```

---

## 4.2 Detailed Component Breakdown

### 1. Selenium WebDriver
* **Role:** The industry-standard browser automation protocol and API.
* **Relationship:** Healenium is built directly on top of Selenium WebDriver. It does not replace `WebDriver`; it decorates it. Any call to standard methods like `findElement`, `findElements`, `click`, or `sendKeys` delegates to native Selenium first.

### 2. Java & Build Systems (Maven / Gradle)
* **Role:** The primary programming language and build management ecosystem for `healenium-web`.
* **Relationship:** Brought in via standard dependency coordinates (`com.epam.healenium:healenium-web`). Dependencies include ByteBuddy and reflection utilities used to intercept driver invocations.

### 3. Docker & Docker Compose
* **Role:** Container runtime and multi-container orchestration.
* **Relationship:** Provides a reproducible, zero-configuration setup for local workstations and CI/CD agent nodes. A single `docker-compose.yaml` file launches both `healenium-backend` and `postgres-db`, configuring their networking and storage volumes automatically.

### 4. PostgreSQL Relational Database
* **Role:** The persistence layer.
* **Relationship:** Houses historical selector baselines, DOM snapshots, and healing telemetry. Ensures that locator baselines persist across different test runs and CI/CD pipelines.

### 5. Healenium Backend (Spring Boot Microservice)
* **Role:** Computes similarity metrics and serves the HTML reporting dashboard.
* **Relationship:** Exposes ports (typically `7883`) to accept DOM snapshots from running tests, run tree-diff algorithms, and serve the visual review dashboard.

---

## 4.3 Deep Comparison: Healenium-Web vs. Healenium-Proxy

One of the most frequent misconceptions is assuming `healenium-web` and `healenium-proxy` are identical or must always be deployed together. They are **two distinct architectural patterns** for integrating self-healing into automation suites.

```
Approach 1: In-Process Library (Healenium-Web)
+-------------------------------------------------------------+
| JVM Process (Test Runner)                                    |
| [Test Code] --> [SelfHealingDriver Decorator]               |
|                         |                                   |
|                         +--> [Selenium WebDriver] --> Browser
+-------------------------|-----------------------------------+
                          v (HTTP on failure)
               [Healenium-Backend + PostgreSQL]

Approach 2: Out-of-Process Proxy (Healenium-Proxy)
+----------------------------+
| Test Runner (Any Language) |
| Python / JS / C# / Java    |
+--------------+-------------+
               | (W3C WebDriver Wire Protocol HTTP)
               v
+-------------------------------------------------------------+
| Docker Container (Healenium-Proxy :8085)                    |
| - Intercepts /session/{id}/element                          |
| - Coordinates with Backend on Error                         |
| - Rewrites command with Healed Selector                     |
+--------------+----------------------------------------------+
               |
               v
+----------------------------+
| Selenium Grid / Chromedriver|
+----------------------------+
```

### Architectural Comparison Matrix

| Dimension | Healenium-Web | Healenium-Proxy |
| :--- | :--- | :--- |
| **Architectural Model** | In-process Client Library (Decorator) | Out-of-process Reverse Network Proxy |
| **Language Support** | Java only | Language-Agnostic (Python, C#, JS, TS, Robot Framework, Java) |
| **Test Code Changes** | Wrap driver: `SelfHealingDriver.create(delegate)` | Zero code changes: Point RemoteWebDriver URL to proxy URL |
| **Network Latency** | Low (direct method invocation; network call only during healing) | Moderate (all WebDriver wire protocol commands pass through proxy) |
| **Infrastructure Needed** | Backend + PostgreSQL | Backend + PostgreSQL + Proxy Container |
| **Ideal For** | Pure Java/Maven/TestNG enterprise automation frameworks | Polyglot engineering teams, Selenium Grid / Selenoid clusters |

---

[⬅ Previous: Core Technical Concepts](03_core_technical_concepts.md) | [📚 Master Index](../README.md) | [Next: Conceptual Examples ➡](05_conceptual_examples.md)
