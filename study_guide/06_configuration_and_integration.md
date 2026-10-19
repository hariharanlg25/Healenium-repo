# Module 06: Configuration, Setup & Framework Integration

[⬅ Previous: Conceptual Examples](05_conceptual_examples.md) | [📚 Master Index](../README.md) | [Next: Troubleshooting Matrix ➡](07_troubleshooting_and_debugging.md)

---

## 6.1 Prerequisites for Using Healenium
To run Healenium, an enterprise test environment requires:
* **Java Development Kit (JDK):** Version 8, 11, or 17+.
* **Selenium WebDriver:** Versions 3.141.59 or 4.x (ensure library version alignment).
* **Docker & Docker Compose:** Installed on the local test machine or CI agent node to host backend services.
* **Network Connectivity:** Port `7883` (Backend/Dashboard) and `5432` (PostgreSQL) must be accessible from the test runner.

---

## 6.2 Backend Infrastructure Deployment (`docker-compose.yaml`)
The backend is typically managed using a Docker Compose manifest:

```yaml
version: '3.8'

services:
  # Relational Database for Storing Baselines and Healing History
  postgres-db:
    image: postgres:13-alpine
    container_name: healenium-postgres
    environment:
      POSTGRES_DB: healenium
      POSTGRES_USER: healenium_user
      POSTGRES_PASSWORD: healenium_password
    ports:
      - "5432:5432"
    volumes:
      - healenium_data:/var/lib/postgresql/data

  # Analytical Engine and Web Reporting Dashboard
  healenium-backend:
    image: healenium/hlm-backend:latest
    container_name: healenium-backend
    ports:
      - "7883:7883"
    environment:
      POSTGRES_HOST: postgres-db
      POSTGRES_PORT: 5432
      POSTGRES_DB: healenium
      POSTGRES_USER: healenium_user
      POSTGRES_PASSWORD: healenium_password
    depends_on:
      - postgres-db

volumes:
  healenium_data:
```

---

## 6.3 Maven Dependency Configuration (`pom.xml`)
For native Java integration, add the `healenium-web` dependency alongside Selenium:

```xml
<dependencies>
    <!-- Standard Selenium Dependencies -->
    <dependency>
        <groupId>org.seleniumhq.selenium</groupId>
        <artifactId>selenium-java</artifactId>
        <version>4.16.1</version>
    </dependency>

    <!-- Healenium Web Java Client Wrapper -->
    <dependency>
        <groupId>com.epam.healenium</groupId>
        <artifactId>healenium-web</artifactId>
        <version>3.4.7</version>
    </dependency>
</dependencies>
```

---

## 6.4 Client Configuration Properties (`healenium.properties`)
Placed in `src/test/resources/healenium.properties`, this configuration controls the engine's behavior:

```properties
# URL of the running Healenium Backend service
recovery-tries=1
score-cap=0.6
heal-enabled=true
back-end-url=http://localhost:7883

# Server URL when running in remote environments or grids
# hl.proxy.url=http://localhost:8085
```

### Property Explanations:
* `heal-enabled`: Global boolean flag. When `false`, Healenium acts as an invisible pass-through driver without healing.
* `recovery-tries`: Number of healing candidates to attempt sequentially before giving up.
* `score-cap`: Minimum float value (between `0.0` and `1.0`) required for an element candidate to be accepted as a valid match. Setting this to `0.8` demands higher fidelity; setting it to `0.5` is more lenient but increases the risk of false positives.
* `back-end-url`: The HTTP endpoint where `healenium-backend` is reachable.

---

## 6.5 Integrating Healenium into a Page Object Model (POM) Framework

In a standard POM framework, drivers are initialized in a centralized factory or BaseTest class. The transition to Healenium requires **wrapping** the native driver instance:

```java
package com.sdet.framework.driver;

import com.epam.healenium.SelfHealingDriver;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;

public class DriverFactory {

    public static WebDriver initializeDriver() {
        // 1. Instantiate the native browser driver
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--start-maximized");
        WebDriver nativeDelegate = new ChromeDriver(options);

        // 2. Wrap the delegate inside Healenium's SelfHealingDriver
        // SelfHealingDriver implements the standard org.openqa.selenium.WebDriver interface
        WebDriver driver = SelfHealingDriver.create(nativeDelegate);

        return driver;
    }
}
```

### Page Object Class Example (Remains 100% Native!):
```java
package com.sdet.framework.pages;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;

public class LoginPage {
    private WebDriver driver;

    // Standard Selenium By locators - No proprietary annotations required!
    private By usernameInput = By.id("user_login");
    private By passwordInput = By.name("user_password");
    private By loginSubmitBtn = By.cssSelector(".auth-container button[type='submit']");

    public LoginPage(WebDriver driver) {
        this.driver = driver;
    }

    public void login(String user, String pass) {
        driver.findElement(usernameInput).sendKeys(user);
        driver.findElement(passwordInput).sendKeys(pass);
        driver.findElement(loginSubmitBtn).click();
    }
}
```

> **Design Pattern Note:** Healenium utilizes the **Decorator Pattern**. Because `SelfHealingDriver` implements `WebDriver`, the rest of the automation framework (Page Objects, Test Listeners, Fluent Wait helpers) requires zero refactoring.

---

[⬅ Previous: Conceptual Examples](05_conceptual_examples.md) | [📚 Master Index](../README.md) | [Next: Troubleshooting Matrix ➡](07_troubleshooting_and_debugging.md)
