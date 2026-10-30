# Test Plan & Strategy: Sandbox 01 — SauceDemo

**Sandbox Target:** SauceDemo (Swag Labs)  
**Automation Framework:** Playwright (TypeScript)  
**CI/CD Pipeline Engine:** Jenkins  
**Defect Management System:** Jira  
**Document Version:** 1.0  
**Author / Lead:** QA Community Team  

---

## 1. Context & Purpose

This document specifies the sandbox-specific testing scope, architectural choices, quality goals, and execution pipeline for the **SauceDemo (`01-saucedemo`)** sandbox using **Playwright (TypeScript)**. 

It aligns with the master QA strategy while serving as a standalone guide for community members contributing to, maintaining, or evaluating this sandbox module.

---

## 2. In-Scope vs. Out-of-Scope

### 2.1 In-Scope Functional Areas
* **Authentication:** Valid user login, locked-out user scenarios, invalid credentials, logout flow.
* **Product Catalog & Inventory:** Item listing, sorting options (A-Z, Z-A, Price low-high, Price high-low), adding/removing items to/from cart.
* **Shopping Cart & State Management:** Cart badge counts, dynamic persistent cart state across navigation.
* **E2E Checkout Flow:** Two-step checkout inputs (First Name, Last Name, Zip), order summary verification, checkout completion message.
* **Cross-Browser & Visual Layout:** Responsive viewports across Chrome, Firefox, Safari (WebKit), and mobile emulators.

### 2.2 Out-of-Scope
* Payment gateway integrations (mocked/static backend).
* Real-time database persistence checks (front-end state only).
* Non-functional load or performance stress testing beyond basic page load performance checks.

---

## 3. Tooling & Technical Architecture

```
                          +-----------------------------------+
                          |     Playwright (TypeScript)       |
                          +-----------------------------------+
                                            |
                                            v
                 +-----------------------------------------------------+
                 |       Page Object Model Architecture (POM)          |
                 +-----------------------------------------------------+
                 |  - LoginPage.ts       - InventoryPage.ts            |
                 |  - CartPage.ts        - CheckoutPage.ts             |
                 +-----------------------------------------------------+
                                            |
                                            v
                 +-----------------------------------------------------+
                 |  Jenkins CI Pipeline Execution (`Jenkinsfile`)      |
                 +-----------------------------------------------------+
                                            |
                         +------------------+------------------+
                         |                                     |
                         v                                     v
         +-------------------------------+     +-------------------------------+
         |   Allure Test Report Artifact |     | Jira Bug & Traceability Sync  |
         +-------------------------------+     +-------------------------------+
```

| Domain | Tool / Framework | Strategy / Specification |
| :--- | :--- | :--- |
| **Language & Runtime** | TypeScript / Node.js | Strict type definitions (`tsconfig.json`), ES2022+ features |
| **Test Runner & Engine** | Playwright (`@playwright/test`) | Native parallelization, fixture-based isolation, auto-waiting |
| **Design Pattern** | Page Object Model (POM) | Clean separation of selectors, page actions, and test assertions |
| **Reporting Engine** | Allure Report / Playwright HTML | Detailed step logging, failure screenshots, video recordings, trace logs |
| **CI/CD Integration** | Jenkins Pipeline | Declarative `Jenkinsfile` executing headless runs on agents |
| **Defect Tracking** | Jira | Issues tagged with `@US-ID` and logged using standard bug templates |

---

## 4. Test Strategy & Execution Matrix

### 4.1 Target Browser Matrix

Tests run in parallel across the following browser projects configured in `playwright.config.ts`:

| Project ID | Engine / Browser | Viewport / Emulation | CI Agent OS |
| :--- | :--- | :--- | :--- |
| `chromium-desktop` | Chromium (Google Chrome) | $1920 \times 1080$ | Linux / Docker Agent |
| `firefox-desktop` | Firefox | $1920 \times 1080$ | Linux / Docker Agent |
| `webkit-desktop` | WebKit (Safari Engine) | $1440 \times 900$ | Linux / Docker Agent |
| `mobile-chrome` | Chromium | Pixel 7 Emulation | Linux / Docker Agent |
| `mobile-safari` | WebKit | iPhone 14 Emulation | Linux / Docker Agent |

### 4.2 Test Tagging & Grouping Guidelines

Playwright tests use annotations to map directly to Jira requirements:

```typescript
// Example Tagging Syntax
test('User can complete standard checkout @US-101 @smoke', async ({ page }) => {
  // Test implementation
});
```

* `@smoke`: Critical core functional path runs (triggered on PRs and quick Jenkins smoke jobs).
* `@regression`: Full E2E suite execution (triggered nightly or on release branch merges).
* `@US-XXX`: Direct mapping to the corresponding Jira User Story ID.

---

## 5. Jenkins CI/CD Pipeline Workflow

The execution of this sandbox's Playwright suite is driven by a dedicated declarative Jenkins pipeline stage (`Jenkinsfile`).

### 5.1 Pipeline Execution Stages

```
 +-----------------+      +-----------------+      +-----------------+      +-----------------+
 | 1. Checkout     | ---> | 2. Setup &      | ---> | 3. Execute      | ---> | 4. Generate     |
 |    Repository   |      |    Dependencies |      |    Playwright   |      |    Allure &     |
 |                 |      |    (`npm ci`)   |      |    Tests (Matrix)|     |    Publish Logs |
 +-----------------+      +-----------------+      +-----------------+      +-----------------+
```

### 5.2 Jenkins Run Commands
```bash
# Clean install dependencies
npm ci

# Install Playwright browser binaries
npx playwright install --with-deps

# Execute Smoke Test Suite in Headless Mode
npm run test:smoke

# Execute Full Regression Suite with Allure Reporting
npm run test:regression -- --reporter=line,allure-playwright
```

---

## 6. Jira Defect & Traceability Management

### 6.1 Defect Logging Rules
When an automated test fails in the Jenkins CI pipeline:
1. Validate whether the failure is a genuine defect or a framework/flakiness issue.
2. If confirmed as a bug, log a defect in **Jira** matching the bug template:
   * **Project:** `SAQA` (QA Community Hub)
   * **Issue Type:** `Bug`
   * **Component:** `01-saucedemo`
   * **Summary:** `[SauceDemo][Playwright] Short concise failure description`
   * **Traceability:** Link to the specific Jira User Story (e.g., `Blocks US-101`).
   * **Attachments:** Attach Jenkins Allure failure screenshots, trace archives (`.zip`), and console log snippets.

### 6.2 Severity & Priority Definitions

| Jira Severity | Definition | Action Required |
| :--- | :--- | :--- |
| **Highest / Blocker** | Core checkout or login complete failure; user cannot complete purchase. | Immediate fix required before PR merge. |
| **High** | Inventory sorting broken, or cart badge count mismatch. | Resolve within current sprint. |
| **Medium** | Minor UI alignment defect, non-critical visual overlap. | Backlog prioritizing. |
| **Low** | Typos in product descriptions or minor footer alignment. | Low priority polish ticket. |

---

## 7. Entry & Exit Criteria

### 7.1 Entry Criteria
* Target URL (`https://www.saucedemo.com`) is reachable and stable.
* Node.js ($\ge$ v18) and Playwright dependencies installed on the Jenkins worker node.
* Environment variables (`BASE_URL`, `STANDARD_USER`, `PASSWORD`) correctly injected via Jenkins Credentials Plugin.

### 7.2 Exit Criteria
* **Pass Rate:** $\ge 98\%$ test pass rate across desktop and mobile execution matrices in Jenkins.
* **Zero Critical Bugs:** Zero open Highest/High severity Jira defects logged against the sandbox.
* **Artifact Availability:** Allure report successfully published to Jenkins workspace or artifact storage with video/trace recordings attached for failures.
