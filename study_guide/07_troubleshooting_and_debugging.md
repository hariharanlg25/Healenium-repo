# Module 07: Enterprise Troubleshooting & Debugging Matrix

[⬅ Previous: Configuration & Integration](06_configuration_and_integration.md) | [📚 Master Index](../README.md) | [Next: Best Practices & Limitations ➡](08_best_practices_and_limitations.md)

---

## 7.1 Troubleshooting Matrix Summary
When implementing Healenium in real-world environments, SDETs often encounter operational and environmental issues. This matrix provides structured diagnoses and resolutions.

```
+-------------------------------------------------------------------------------------+
|                             SDET TROUBLESHOOTING MATRIX                             |
+-----------------------------------+-------------------------------------------------+
| Symptom                           | Primary Root Cause                              |
+-----------------------------------+-------------------------------------------------+
| 1. Healing Not Triggered          | Baseline not established or heal-enabled=false  |
| 2. Backend Connection Refused     | Container down, wrong port, or network barrier  |
| 3. PostgreSQL Database Failure    | Volume permissions, container crash, bad creds  |
| 4. Missing Selector History       | Ephemeral Docker container without persistent DB|
| 5. Dependency Conflict / NoSuchMethod | Mismatch between Selenium and Healenium-Web |
| 6. Wrong Element Selected         | score-cap too low or ambiguous candidate markup |
| 7. Empty / 404 Report Dashboard   | Backend URL misconfigured or test class filtered|
| 8. Passes Locally, Fails in CI    | CI container cannot resolve localhost:7883      |
| 9. Large DOM Trees Timeout        | Backend compute exhaustion on complex SPA trees |
| 10. Frame/Shadow DOM Element Lost | Healenium looking in top DOM instead of frame   |
+-----------------------------------+-------------------------------------------------+
```

---

## 7.2 Detailed Problem Breakdown (10 Enterprise Scenarios)

### Problem 1: Healing is Not Triggered on NoSuchElementException
* **Symptoms:** Test fails immediately with standard Selenium `NoSuchElementException`. No healing events appear in the terminal logs.
* **Likely Causes:**
  1. `heal-enabled=false` is set in `healenium.properties`.
  2. The test is running on an element that was **never successfully located before** (no baseline exists in PostgreSQL).
  3. The driver instance was initialized directly (`new ChromeDriver()`) without being wrapped in `SelfHealingDriver.create()`.
* **Diagnostic Steps:**
  * Verify driver initialization in your BaseTest.
  * Query the database to check if the baseline selector exists:
    ```sql
    SELECT * FROM selector WHERE locator = 'your-locator';
    ```
* **Corrective Action:** Ensure baseline passes at least once with the original locator before asserting self-healing capabilities.

---

### Problem 2: Backend Connection Refused (`ConnectException`)
* **Symptoms:** Terminal logs display: `java.net.ConnectException: Connection refused: connect to http://localhost:7883/healenium`.
* **Likely Causes:**
  1. Docker containers for `healenium-backend` are stopped or crashed.
  2. Test execution environment is attempting to reach `localhost` from inside another container.
* **Diagnostic Steps:**
  * Run `docker ps` to verify running containers.
  * Run `curl -I http://localhost:7883/actuator/health` to confirm backend service health.
* **Corrective Action:** Start the containers via `docker-compose up -d`. If tests run inside Docker, change `back-end-url` to point to the Docker host or network service name (e.g., `http://healenium-backend:7883`).

---

### Problem 3: PostgreSQL Database Failure or Crash
* **Symptoms:** Backend service logs throw `PSQLException: Connection to localhost:5432 refused` or `FATAL: password authentication failed`.
* **Likely Causes:**
  * PostgreSQL container failed to start due to port conflicts or mismatched credentials in `docker-compose.yaml`.
* **Diagnostic Steps:**
  * Run `docker logs healenium-postgres` to inspect PostgreSQL startup logs.
* **Corrective Action:** Verify `POSTGRES_USER` and `POSTGRES_PASSWORD` match between `postgres-db` and `healenium-backend` service definitions.

---

### Problem 4: Missing Selector History After Container Restart
* **Symptoms:** After running `docker-compose down`, all baseline records vanish; tests treat every selector as new and fail to heal.
* **Likely Causes:**
  * Docker Compose was deployed without a named volume or persistent host directory mount.
* **Diagnostic Steps:**
  * Check `docker volume ls` and inspect `docker-compose.yaml` for `volumes:`.
* **Corrective Action:** Mount a named volume (`healenium_data:/var/lib/postgresql/data`) in the PostgreSQL service definition to guarantee data persistence across restarts.

---

### Problem 5: Dependency Conflicts & Binary Incompatibilities (`NoSuchMethodError`)
* **Symptoms:** JVM crash during driver creation: `java.lang.NoSuchMethodError: org.openqa.selenium.WebDriver.manage()`.
* **Likely Causes:**
  * Incompatibility between the version of `selenium-java` in your project and the version shaded or transitively imported by `healenium-web`.
* **Diagnostic Steps:**
  * Generate a Maven dependency tree: `mvn dependency:tree -Dincludes=*selenium*`.
* **Corrective Action:** Align dependencies. For Selenium 4.x suites, ensure you are using `healenium-web` 3.x+ designed specifically for Selenium 4. Exclude transitive Selenium artifacts if necessary.

---

### Problem 6: Healed Locator Selects Wrong Element (False Positive)
* **Symptoms:** Test passes the click step, but application displays an error, navigates to the wrong view, or mutates unrelated records.
* **Likely Causes:**
  * `score-cap` is set too low (e.g., `0.5`).
  * Target element is part of a repeated list or table with identical markup for sibling nodes.
* **Diagnostic Steps:**
  * Review the Healenium report for that execution ID at `http://localhost:7883/healenium/report/`.
  * Inspect the calculated score and the screenshot diff.
* **Corrective Action:**
  1. Increase `score-cap=0.75` in `healenium.properties`.
  2. In your Page Object, rewrite the original locator to anchor to unique semantic landmarks (e.g., table row containing unique customer email).

---

### Problem 7: Empty or 404 Reporting Dashboard
* **Symptoms:** Navigating to `http://localhost:7883/healenium/report/` returns 404 or an empty table despite healing events occurring.
* **Likely Causes:**
  * Trailing slash omission (`/report` instead of `/report/`).
  * Backend failed to write report artifacts due to container filesystem permission restrictions.
* **Diagnostic Steps:** Check backend logs via `docker logs healenium-backend`.
* **Corrective Action:** Access the canonical URL `http://localhost:7883/healenium/report/` and ensure container file permissions permit report generation.

---

### Problem 8: Tests Pass Locally but Fail in CI/CD Pipeline
* **Symptoms:** Local developer runs heal smoothly; GitLab CI/GitHub Actions/Jenkins jobs throw connection errors or fail to heal.
* **Likely Causes:**
  * The CI pipeline executes tests inside an ephemeral Docker container. `http://localhost:7883` refers to the isolated container's loopback interface, not the host machine where Healenium Backend is running.
* **Diagnostic Steps:** Check network configuration in CI pipeline configuration (`docker network inspect`).
* **Corrective Action:** Use Docker Compose in CI with a shared bridge network, or deploy Healenium backend and PostgreSQL as shared persistent services on a dedicated internal QA server.

---

### Problem 9: Large DOM Trees Cause Execution Timeouts
* **Symptoms:** Tests experience 10+ second delays during element lookups on complex Single Page Applications (SPAs).
* **Likely Causes:**
  * Extremely deep DOM trees (e.g., virtualized tables with 10,000+ nodes) cause backend CPU exhaustion during tree comparison.
* **Diagnostic Steps:** Monitor backend container CPU/memory usage via `docker stats`.
* **Corrective Action:** Allocate higher CPU/memory limits to `healenium-backend` in Docker, or tune locators to search within specific container sub-trees using scoped `element.findElement()` rather than full page queries.

---

### Problem 10: Lost Healing Inside Frames or Shadow DOM
* **Symptoms:** Elements inside an embedded iFrame or Shadow Root fail with `NoSuchElementException` and cannot be healed.
* **Likely Causes:**
  * Selenium context was not switched into the frame prior to element lookup. Healenium captures the root document DOM which does not contain the frame's children.
* **Diagnostic Steps:** Verify if target element is enclosed within `<iframe>` or `#shadow-root`.
* **Corrective Action:** Execute `driver.switchTo().frame(...)` before locating elements within the frame.

---

[⬅ Previous: Configuration & Integration](06_configuration_and_integration.md) | [📚 Master Index](../README.md) | [Next: Best Practices & Limitations ➡](08_best_practices_and_limitations.md)
