# Module 05: Conceptual Scenarios & Educational Code Snippets

[⬅ Previous: Components & Technologies](04_components_and_technologies.md) | [📚 Master Index](../README.md) | [Next: Configuration & Integration ➡](06_configuration_and_integration.md)

---

## 5.1 Overview
The following educational scenarios illustrate the exact internal mechanics of locator baseline creation, failure detection, tree scoring, successful recovery, failed recovery, and false-positive matching. 

> **Important Note:** These are concise educational snippets to understand concepts; they are not runnable projects.

---

## Scenario 5.1: The Baseline State (Passing Lookup)

### Initial Application HTML
```html
<!-- Production UI: Sprint 1 -->
<div class="header-auth-panel">
    <button id="btn-login-submit" class="ui-btn primary-action" type="submit">
        Log In
    </button>
</div>
```

### Automation Test Code (Page Object Model)
```java
// Standard locator written by automation engineer
private By loginButton = By.id("btn-login-submit");

public void clickLogin() {
    // Wrapped SelfHealingDriver executes this call
    driver.findElement(loginButton).click();
}
```

### What Happens Behind the Scenes:
1. `SelfHealingDriver` calls Selenium's native `findElement(By.id("btn-login-submit"))`.
2. Native Selenium finds the element immediately.
3. Healenium captures the node signature:
   * **Tag:** `BUTTON`
   * **ID:** `btn-login-submit`
   * **Classes:** `ui-btn`, `primary-action`
   * **Type:** `submit`
   * **Text:** `Log In`
   * **Parent:** `DIV.header-auth-panel`
4. Healenium saves this baseline signature to PostgreSQL under the key `loginButton`.

---

## Scenario 5.2: UI Refactoring Breaks the Locator

### Updated Application HTML (Sprint 2)
```html
<!-- Production UI: Sprint 2 Refactoring -->
<!-- Developer replaced the ID with dynamic GUID and changed wrapper classes -->
<div class="auth-card-v2">
    <button id="auth-guid-98214" class="ui-btn primary-action active" type="submit">
        Log In
    </button>
</div>
```

### What Happens During Execution:
```java
// Test code remains UNCHANGED
driver.findElement(By.id("btn-login-submit")).click();
```

1. Native Selenium searches for `id="btn-login-submit"` in the new DOM.
2. **Outcome:** `NoSuchElementException` is raised.
3. `SelfHealingDriver` intercepts the exception. It suspends test failure and grabs current DOM tree.

---

## Scenario 5.3: Successful Recovery (True Positive)

### Candidate Analysis by Healenium Backend Engine:
The backend algorithm compares the baseline signature against candidates in the new DOM:

```
Candidate Node: <button id="auth-guid-98214" class="ui-btn primary-action active" type="submit">Log In</button>

Feature Match Scoring:
+ Tag Match:       BUTTON == BUTTON                 --> Score: 1.0 (Weight 0.20) = 0.20
+ ID Match:        btn-login-submit != auth-guid-.. --> Score: 0.0 (Weight 0.25) = 0.00
+ Class Match:     2 of 3 classes match             --> Score: 0.8 (Weight 0.20) = 0.16
+ Text Match:      "Log In" == "Log In"             --> Score: 1.0 (Weight 0.20) = 0.20
+ Attribute Match: type="submit" == type="submit"   --> Score: 1.0 (Weight 0.15) = 0.15
--------------------------------------------------------------------------------------
Total Composite Score: 0.71 (71% Similarity)
Configured Threshold:  0.60 (60% Minimum)
Verdict: Threshold Exceeded -> ACCEPT AS HEALED CANDIDATE
```

### Result:
* Healenium constructs a new selector: `//button[contains(@class, 'ui-btn') and text()='Log In']`.
* It clicks the button.
* Test completes successfully.
* Event logged to Healenium Report: **"Healed By.id('btn-login-submit') with score 0.71"**.

---

## Scenario 5.4: Failed Recovery (True Negative)

### Scenario Context
The application design team completely removed the button and replaced it with a social single-sign-on (SSO) redirect panel:

```html
<!-- Production UI: Sprint 3 -->
<div class="sso-container">
    <a href="/oauth/google" class="sso-provider-link">Continue with Google</a>
    <a href="/oauth/github" class="sso-provider-link">Continue with GitHub</a>
</div>
```

### Candidate Analysis:
* No node in the DOM contains tag `BUTTON` or text `"Log In"`.
* Candidates (the `<a>` links) score ~`0.15` (well below `0.60`).
* **Verdict:** Healing fails.
* `SelfHealingDriver` re-throws the original `NoSuchElementException`.
* **SDET Implication:** Test fails as it should, alerting the team that the login flow was fundamentally altered.

---

## Scenario 5.5: False Positive (Incorrect Healing)

### Scenario Context
A checkout screen contains multiple identical icon buttons for removing items:

```html
<!-- Cart Page with 2 Items -->
<tr id="item-row-1">
    <td>Wireless Mouse</td>
    <td><button class="btn-delete" title="Remove Item"><i class="icon-trash"></i></button></td>
</tr>
<tr id="item-row-2">
    <td>Mechanical Keyboard</td>
    <td><button class="btn-delete" title="Remove Item"><i class="icon-trash"></i></button></td>
</tr>
```

### What Goes Wrong:
1. The test intends to delete **Item 1** using an absolute path: `By.xpath("//tr[@id='item-row-1']//button")`.
2. Developer renames `item-row-1` to `product-row-mouse`.
3. Native Selenium fails to locate the selector.
4. Healenium backend evaluates candidate trash buttons:
   * It analyzes `//tr[@id='item-row-2']//button`.
   * Tag matches (`BUTTON`), class matches (`btn-delete`), child matches (`icon-trash`).
   * The calculated score is `0.78`!
5. Healenium clicks the trash button for **Item 2** (the keyboard) instead of Item 1.
6. **Result:** The test mistakenly passes its click action, but the wrong item is deleted! If subsequent assertions only check `"cart count decreased by 1"`, the test reports a false pass, masking a serious bug.

---

[⬅ Previous: Components & Technologies](04_components_and_technologies.md) | [📚 Master Index](../README.md) | [Next: Configuration & Integration ➡](06_configuration_and_integration.md)
