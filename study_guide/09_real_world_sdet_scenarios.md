# Module 09: Real-World SDET Case Studies

[⬅ Previous: Best Practices & Governance](08_best_practices_and_limitations.md) | [📚 Master Index](../README.md) | [Next: Interview Preparation ➡](10_interview_preparation.md)

---

## 9.1 Overview
In enterprise test engineering, self-healing tools must be evaluated within the context of complex business applications, continuous delivery pipelines, and legacy modernization initiatives. The following case studies provide architectural blueprints for real-world scenarios.

---

## Case Study 1: Modernizing a Legacy Banking Portal (AngularJS to React)

### Context & Challenge
A Tier-1 retail banking organization maintained a Selenium-Java test suite of over 1,800 automated regression tests. Front-end engineering initiated a 6-month migration from a legacy AngularJS frontend to a modern React architecture. 
* Angular-specific attributes (`ng-click="processTransfer()"`, `ng-model="accountNumber"`) and legacy static IDs were replaced with nested React components, dynamic CSS modules (`.TransferCard__btn--2xK9`), and randomized auto-generated IDs.
* Prior to Healenium, every sprint release broke hundreds of tests, halting the nightly CI pipeline and forcing two SDETs to spend 3 full days repairing broken Page Object locators.

### Implementation Strategy
1. The SDET lead introduced `healenium-web` into the core framework's `DriverFactory`, wrapping the existing `RemoteWebDriver` instances.
2. `healenium-backend` and PostgreSQL were deployed as persistent Docker containers on an internal QA server reachable by all Jenkins CI worker agents.
3. The baseline run was performed against the pre-migration staging environment to populate PostgreSQL with DOM tree snapshots for all 1,800 test scenarios.
4. `score-cap` was set to `0.68` to prevent low-confidence matching.

### Results & Metrics
* During the first major React sprint release, **142 tests encountered broken locators**.
* Healenium successfully healed **131 selectors (92.2%)** without human intervention. The engine matched elements primarily through text matching (`"Transfer Funds"`), button hierarchy, and input field positioning.
* The nightly CI regression pipeline completed in 4 hours instead of aborting after 30 minutes.
* **Hours Saved:** The QA team saved approximately 35 engineering hours in a single sprint.
* **The Unhealed 11 Selectors:** Analyzed during morning triage. These 11 failures represented features where the UI was completely redesigned from direct button triggers into a multi-step modal wizard. The SDETs recognized that healing rightly failed, as the business flow itself had changed.

---

## Case Study 2: Dynamic Data Grids in an E-Commerce Product Catalog

### Context & Challenge
An e-commerce automation suite validates dynamic search results, product listings, and cart operations.
* A test scenario searched for a `"Wireless Mouse"` and intended to click the `"Add to Cart"` button for that specific product row.
* Front-end developers introduced a UI change that altered row wrapper IDs from `id="product-row-101"` to dynamically generated UUIDs (`id="sku-row-a89c-44b2"`).

### The Incident (False Positive)
1. The original locator `By.xpath("//div[@id='product-row-101']//button[text()='Add to Cart']")` failed native lookup.
2. Healenium intercepted the failure and searched for candidate buttons in the current DOM.
3. It identified another `"Add to Cart"` button belonging to a sponsored product (`"Mechanical Keyboard"`) at the top of the grid. Because both buttons shared identical tags (`BUTTON`), classes (`btn-cart-primary`), and inner text (`"Add to Cart"`), the calculated similarity score reached `0.76`!
4. Healenium clicked the sponsored product's button.
5. The test proceeded to the cart page. However, because downstream assertions only checked `cart.getItemCount() == 1`, the test passed!
6. Later, an automated checkout test failed because the expected product price was $25.00, but the cart contained a $120.00 keyboard.

### Corrective Engineering Actions Taken by the SDET Team
1. **Root Cause Identification:** The locator relied on a fragile parent ID that lacked semantic anchoring to the unique product title.
2. **Refactored Locator:** The SDET refactored the Page Object to anchor specifically to the product's unique title text:
   ```java
   // Resilient semantic anchor
   By.xpath("//div[contains(@class, 'product-card')][.//h3[text()='Wireless Mouse']]//button[contains(., 'Add to Cart')]");
   ```
3. **Enhanced Downstream Assertions:** The SDET added an explicit validation on the cart page verifying both product name and item price:
   ```java
   Assert.assertEquals(cartPage.getProductName(), "Wireless Mouse", "Wrong product was added to cart!");
   ```
4. **Configuration Adjustment:** Increased `score-cap` from `0.60` to `0.72` to reduce ambiguity in repeated grid components.

---

[⬅ Previous: Best Practices & Governance](08_best_practices_and_limitations.md) | [📚 Master Index](../README.md) | [Next: Interview Preparation ➡](10_interview_preparation.md)
