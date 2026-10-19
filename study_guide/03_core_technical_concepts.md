# Module 03: Core Technical Concepts & Locator Healing Engine

[⬅ Previous: Architecture & Workflow](02_healenium_architecture.md) | [📚 Master Index](../README.md) | [Next: Components & Technologies ➡](04_components_and_technologies.md)

---

## 3.1 Locators and Selector Strategies
In standard web automation, locators are query expressions instructing the browser engine how to identify DOM nodes:
* Direct attributes: `By.id("username")`, `By.name("email")`
* Stylistic classes: `By.className("form-input")`
* Link heuristics: `By.linkText("Sign Out")`
* Hierarchical queries: `By.xpath("//div[@class='auth']//input[@type='text']")`
* CSS expressions: `By.cssSelector("form.auth-form > input[type='text']")`

When front-end teams modify class names, wrap inputs in layout containers, or swap static IDs for framework-generated tokens, hardcoded selectors fail.

---

## 3.2 The DOM Tree Representation
The Document Object Model (DOM) is a hierarchical tree composed of nodes. Each node possesses:
* **Tag name** (`DIV`, `BUTTON`, `INPUT`)
* **Attribute map** (`id`, `class`, `placeholder`, `aria-label`, `style`)
* **Text contents** (inner text, whitespace trimmed text)
* **Topological coordinates** (depth in tree, sibling index, parent tag, child count)
* **Rendered layout metrics** (height, width, screen coordinates)

Healenium converts the DOM into an internal tree data structure. When an element is first located, its path from the root `<html>` tag down to the target leaf, along with its full attribute state, is persisted.

---

## 3.3 The Healing Algorithm & Similarity Scoring
Healenium calculates node similarity using a combination of **Tree Comparison**, **Longest Common Subsequence (LCS)**, and **Weighted Feature Matching**:

```
Total Similarity Score S = w1*S_tag + w2*S_id + w3*S_class + w4*S_text + w5*S_path + w6*S_attr
Where:
- w1..w6: Predefined weight coefficients representing attribute importance
- S_xxx:  Individual similarity metric between baseline and candidate (0.0 to 1.0)
```

1. **Tag Match ($S_{tag}$):** Evaluates if the candidate shares the same tag. If the baseline is `<button>` and the candidate is `<div>`, the score drops significantly unless specific styling/role equivalents apply.
2. **Path Distance & Tree Hierarchy ($S_{path}$):** Computes tree-edit distance or LCS between the ancestor path strings of the baseline and candidate nodes.
3. **Class & Attribute Overlap ($S_{class}, S_{attr}$):** Evaluates Jaccard similarity across class tokens (e.g., `["btn", "btn-primary", "large"]` vs `["btn", "btn-primary", "extra-large"]`).
4. **Text Content Similarity ($S_{text}$):** Uses string distance algorithms (such as Levenshtein or Cosine distance) on inner text values.
5. **Score Cap Threshold:** Healenium defines a cutoff threshold (default: `0.6` or 60%). Candidates scoring below this threshold are discarded.

---

## 3.4 Healing Outcomes: Success, Failure, and False Positives

```
                                  [Element Not Found]
                                           |
                                  Traverse Candidate Nodes
                                           |
                         +-----------------+-----------------+
                         |                                   |
              Max Score >= score-cap              Max Score < score-cap
                         |                                   |
                Select Candidate                     [Healing Failure]
                         |                         NoSuchElementException
             +-----------+-----------+                     Thrown
             |                       |
      Matches Real Target    Wrong Element Picked
             |                       |
      [Healing Success]       [False Positive]
     Test Continues Validly   Wrong Node Clicked!
```

### 1. Healing Success (True Positive)
The candidate chosen is indeed the element intended by the test engineer. The test performs its action, assertions execute, and the overall test verdict remains accurate.

### 2. Healing Failure (True Negative)
No candidate in the current DOM achieves a similarity score equal to or higher than `score-cap` (e.g., the element was completely removed from the page). Healenium respects this, rethrows `NoSuchElementException`, and alerts the engineer that the component is missing.

### 3. False-Positive Healing (The SDET's Primary Hazard)
The engine finds an element that achieves a score $\ge 0.6$, but it is **not** the intended element. For instance, in a table with ten identical "Delete" buttons in different rows, Healenium might heal a broken locator for Row 1 by binding to the "Delete" button in Row 2. The test clicks the wrong button, passing the step but corrupting application state.

---

## 3.5 Recovering an Element vs. Validating Application Behavior
A vital distinction for any Senior SDET:
* **Element Recovery** is purely syntactic. It asks: *"Can we find a DOM node that looks structurally similar to the one we used previously?"*
* **Behavioral Validation** is semantic. It asks: *"Is the application delivering the correct business outcome?"*

> **Crucial Rule:** Self-healing recovers *locators*, not *requirements*. If a developer changes a "Submit" button to a "Cancel" button in the same form container, Healenium might recover the button (high structural similarity), click it, and cause the form to clear instead of submit. **Hard assertions on page state, URLs, database records, and success banners must always validate behavior.**

---

[⬅ Previous: Architecture & Workflow](02_healenium_architecture.md) | [📚 Master Index](../README.md) | [Next: Components & Technologies ➡](04_components_and_technologies.md)
