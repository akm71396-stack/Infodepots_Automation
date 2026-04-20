# 🤖 Hybrid Automation Framework

> **Selenium WebDriver + REST Assured + Cucumber BDD + TestNG**  
> Enterprise-grade hybrid test automation supporting UI, API, and cloud execution.

---

## 📋 Table of Contents

1. [Framework Overview](#framework-overview)
2. [Project Structure](#project-structure)
3. [Architecture Diagram](#architecture-diagram)
4. [Prerequisites](#prerequisites)
5. [Setup & Installation](#setup--installation)
6. [Configuration](#configuration)
7. [Running Tests](#running-tests)
8. [Cloud Execution](#cloud-execution)
9. [Reports](#reports)
10. [Writing New Tests](#writing-new-tests)
11. [CI/CD Integration](#cicd-integration)
12. [Best Practices](#best-practices)
13. [Troubleshooting](#troubleshooting)

---

## Framework Overview

This is a **hybrid BDD test automation framework** that combines:

| Layer | Technology | Purpose |
|-------|-----------|---------|
| BDD | Cucumber 7 (Gherkin) | Human-readable test scenarios |
| UI Automation | Selenium WebDriver 4 | Browser interaction & page automation |
| API Testing | REST Assured 5 | HTTP request/response validation |
| Test Runner | TestNG 7 | Parallel execution, test grouping |
| Build Tool | Maven | Dependency management, execution |
| Reporting | ExtentReports + Allure | Rich HTML dashboards with screenshots |
| Logging | Log4j 2 | Structured, thread-safe test logs |
| Cloud | BrowserStack / Sauce Labs | Cross-browser cloud execution |

### Key Design Patterns

- **Page Object Model (POM)**: Every screen = one class. Locators and actions are encapsulated.
- **BDD (Gherkin)**: Feature files describe behaviour in plain English.
- **Service Layer**: API calls are isolated in `UserApiService` — step defs never call REST Assured directly.
- **ThreadLocal Driver**: `DriverManager` uses `ThreadLocal<WebDriver>` for safe parallel execution.
- **Singleton Config**: `ConfigManager` reads `config.properties` once, with JVM property override support.

---

## Project Structure

```
hybrid-automation-framework/
│
├── pom.xml                              ← Maven dependencies & build config
├── testng.xml                           ← TestNG suite: parallel config & tag filters
├── README.md
│
├── src/
│   ├── main/java/com/framework/
│   │   ├── config/
│   │   │   └── ConfigManager.java       ← Reads config.properties; JVM property override
│   │   ├── core/
│   │   │   ├── BasePage.java            ← Parent class for all Page Objects
│   │   │   └── DriverManager.java       ← WebDriver lifecycle; local + cloud support
│   │   └── utils/
│   │       ├── ReportManager.java       ← ExtentReports HTML report manager
│   │       ├── TestDataLoader.java      ← Loads JSON/Excel external test data
│   │       └── WaitUtils.java           ← Advanced Selenium wait helpers
│   │
│   └── test/
│       ├── java/com/framework/
│       │   ├── api/
│       │   │   ├── BaseApiClient.java   ← REST Assured base config; shared request spec
│       │   │   └── UserApiService.java  ← User CRUD API methods (GET/POST/PUT/DELETE)
│       │   ├── pages/
│       │   │   ├── LoginPage.java       ← Page Object for the Login screen
│       │   │   └── DashboardPage.java   ← Page Object for post-login dashboard
│       │   ├── stepdefs/
│       │   │   ├── LoginStepDefs.java   ← UI step definitions for login.feature
│       │   │   └── ApiStepDefs.java     ← API step definitions for user_api.feature
│       │   ├── hooks/
│       │   │   └── Hooks.java           ← @Before/@After: driver init, screenshots, teardown
│       │   └── runners/
│       │       └── TestRunner.java      ← Cucumber-TestNG runner; parallel DataProvider
│       │
│       └── resources/
│           ├── features/
│           │   ├── login.feature        ← BDD scenarios for Login UI
│           │   └── user_api.feature     ← BDD scenarios for User API CRUD
│           ├── schemas/
│           │   └── users-schema.json    ← JSON Schema for API response validation
│           ├── testdata/
│           │   └── login-data.json      ← External test data for data-driven tests
│           ├── config.properties        ← Central configuration file
│           └── log4j2.xml              ← Log4j 2 logging configuration
│
└── .github/
    └── workflows/
        └── test.yml                     ← GitHub Actions CI/CD pipeline
```

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    FEATURE FILES (.feature)                  │
│              Gherkin: Given / When / Then / And              │
└─────────────────────────┬───────────────────────────────────┘
                          │ mapped by
┌─────────────────────────▼───────────────────────────────────┐
│                   STEP DEFINITIONS                           │
│           LoginStepDefs | ApiStepDefs                        │
└──────────┬──────────────────────────────────┬───────────────┘
           │ calls                             │ calls
┌──────────▼────────────┐        ┌────────────▼──────────────┐
│    PAGE OBJECTS        │        │     API SERVICE LAYER      │
│  LoginPage             │        │   UserApiService           │
│  DashboardPage         │        │   BaseApiClient            │
│  (extends BasePage)    │        │   (REST Assured specs)     │
└──────────┬────────────┘        └────────────┬──────────────┘
           │ uses                              │ uses
┌──────────▼────────────┐        ┌────────────▼──────────────┐
│     DRIVER MANAGER     │        │    CONFIG MANAGER          │
│  ThreadLocal<WebDriver>│        │  config.properties         │
│  Local | Cloud         │        │  JVM property overrides    │
└──────────┬────────────┘        └───────────────────────────┘
           │
┌──────────▼────────────────────────────────────────────────┐
│                      BROWSER / CLOUD                        │
│     Chrome/Firefox/Edge (local) | BrowserStack/SauceLabs   │
└───────────────────────────────────────────────────────────┘
```

---

## Prerequisites

| Requirement | Version | Notes |
|-------------|---------|-------|
| Java JDK | 11+ | JAVA_HOME must be set |
| Maven | 3.8+ | mvn must be on PATH |
| Chrome | Latest | Auto-driver via WebDriverManager |
| Git | Any | For cloning the repo |

---

## Setup & Installation

### 1. Clone the Repository

```bash
git clone https://github.com/your-org/hybrid-automation-framework.git
cd hybrid-automation-framework
```

### 2. Install Dependencies

```bash
mvn clean install -DskipTests
```

> This downloads all Maven dependencies to `~/.m2/repository` without running tests.

### 3. Verify Setup

```bash
mvn validate
```

---

## Configuration

All configuration is centralized in `src/test/resources/config.properties`.

### Key Configuration Options

```properties
# Execution mode: 'local' (default) or 'cloud'
execution.mode=local

# Browser: chrome (default), firefox, edge
browser=chrome

# Run without visible browser window (required for CI)
headless=false

# Application URL for UI tests
app.url=https://www.saucedemo.com

# REST API base URL
api.base.url=https://reqres.in/api
```

### Override with JVM Arguments

Any property can be overridden at runtime without changing the file:

```bash
mvn clean test -Dbrowser=firefox -Dheadless=true -Dapp.url=https://staging.example.com
```

### Environment Variables (for CI/CD Secrets)

| Environment Variable | Config Property | Purpose |
|---------------------|-----------------|---------|
| `BS_USERNAME` | `browserstack.username` | BrowserStack account |
| `BS_ACCESS_KEY` | `browserstack.accesskey` | BrowserStack secret |
| `SAUCE_USERNAME` | `saucelabs.username` | Sauce Labs account |
| `SAUCE_ACCESS_KEY` | `saucelabs.accesskey` | Sauce Labs secret |
| `API_AUTH_TOKEN` | `api.auth.token` | Bearer token for secured APIs |

---

## Running Tests

### Run All Tests

```bash
mvn clean test
```

### Run by Tag

```bash
# Smoke tests only (fast — ~2 min)
mvn clean test -Dcucumber.filter.tags="@smoke"

# All regression tests
mvn clean test -Dcucumber.filter.tags="@regression"

# API tests only (no browser needed)
mvn clean test -Dcucumber.filter.tags="@api"

# UI tests only
mvn clean test -Dcucumber.filter.tags="@ui"

# Smoke AND regression
mvn clean test -Dcucumber.filter.tags="@smoke or @regression"
```

### Run Specific Feature File

```bash
mvn clean test -Dcucumber.features="src/test/resources/features/login.feature"
```

### Run in Headless Mode (CI)

```bash
mvn clean test -Dheadless=true
```

### Run on Different Browser

```bash
mvn clean test -Dbrowser=firefox
mvn clean test -Dbrowser=edge
```

### Generate Allure Report

```bash
# Run tests first (generates raw Allure data)
mvn clean test

# Generate and open Allure HTML report
mvn allure:report
mvn allure:serve    # opens in browser automatically
```

---

## Cloud Execution

### BrowserStack

```bash
export BS_USERNAME="your_username"
export BS_ACCESS_KEY="your_access_key"

mvn clean test \
  -Dexecution.mode=cloud \
  -Dcloud.provider=browserstack \
  -Dcloud.browser=Chrome \
  -Dcloud.os=Windows \
  -Dcloud.os.version=11
```

### Sauce Labs

```bash
export SAUCE_USERNAME="your_username"
export SAUCE_ACCESS_KEY="your_access_key"

mvn clean test \
  -Dexecution.mode=cloud \
  -Dcloud.provider=saucelabs \
  -Dcloud.browser=Chrome \
  -Dcloud.browser.version=latest
```

---

## Reports

After running tests, reports are generated at:

| Report | Location | Description |
|--------|----------|-------------|
| Cucumber HTML | `target/cucumber-reports/cucumber-report.html` | Standard Cucumber HTML report |
| Cucumber JSON | `target/cucumber-reports/cucumber-report.json` | JSON data for CI/CD integration |
| ExtentReports | `target/extent-reports/ExtentReport.html` | Rich dashboard with screenshots |
| Allure Raw | `target/allure-results/` | Raw data for Allure report |
| Allure HTML | `target/site/allure-maven-plugin/` | After `mvn allure:report` |
| Logs | `target/logs/framework.log` | Full test execution logs |
| API Logs | `target/logs/api-calls.log` | Isolated API request/response logs |

---

## Writing New Tests

### Step 1: Write a Feature File

```gherkin
# src/test/resources/features/checkout.feature
@regression @ui
Feature: Checkout Process
  Scenario: User can add item to cart
    Given I am on the login page
    When I login with username "standard_user" and password "secret_sauce"
    And I add the first product to cart
    Then the cart should contain 1 item
```

### Step 2: Create a Page Object

```java
// src/test/java/com/framework/pages/CheckoutPage.java
package com.framework.pages;
import com.framework.core.BasePage;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;

public class CheckoutPage extends BasePage {
    private static final By ADD_TO_CART = By.cssSelector(".btn_add_to_cart_container button");

    public CheckoutPage(WebDriver driver) {
        super(driver); // calls BasePage — sets up wait, actions, PageFactory
    }

    public void addFirstProductToCart() {
        click(ADD_TO_CART); // inherited from BasePage
    }
}
```

### Step 3: Implement Step Definitions

```java
// src/test/java/com/framework/stepdefs/CheckoutStepDefs.java
package com.framework.stepdefs;
import io.cucumber.java.en.*;
import org.testng.Assert;
import com.framework.pages.CheckoutPage;
import com.framework.core.DriverManager;

public class CheckoutStepDefs {
    private CheckoutPage checkoutPage;

    @And("I add the first product to cart")
    public void iAddFirstProductToCart() {
        checkoutPage = new CheckoutPage(DriverManager.getDriver());
        checkoutPage.addFirstProductToCart();
    }

    @Then("the cart should contain {int} item")
    public void cartShouldContainItems(int count) {
        // assertion here
    }
}
```

### Step 4: Tag and Run

```bash
mvn clean test -Dcucumber.filter.tags="@regression"
```

---

## CI/CD Integration

### GitHub Actions

The pipeline (`.github/workflows/test.yml`) runs:

| Trigger | Job | Tags |
|---------|-----|------|
| Every push/PR | Smoke Tests | `@smoke` |
| Push to `main` | Full Regression | `@smoke or @regression` |
| Nightly (2AM UTC) | Full Regression | `@smoke or @regression` |
| Manual trigger | Configurable | User-specified |

### Jenkins (Freestyle or Pipeline)

```groovy
// Jenkinsfile
pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                sh 'mvn clean test -Dheadless=true -Dcucumber.filter.tags="@smoke"'
            }
            post {
                always {
                    // Publish Cucumber HTML reports in Jenkins
                    cucumber buildStatus: 'UNSTABLE',
                             fileIncludePattern: '**/cucumber-report.json',
                             jsonReportDirectory: 'target/cucumber-reports'
                }
            }
        }
    }
}
```

---

## Best Practices

### ✅ DO

- **Tag every scenario** — use `@smoke`, `@regression`, `@ui`, `@api` consistently
- **One assertion per Then step** — keeps failures specific and readable
- **Use Page Objects for ALL UI interactions** — never raw Selenium in step defs
- **Use service classes for ALL API calls** — never raw REST Assured in step defs
- **Store secrets in environment variables** — never hardcode credentials in feature files
- **Use `@FindBy` for stable locators** — prefer `id` > `data-testid` > `css` > `xpath`
- **Reuse `Background` steps** — for preconditions shared across all scenarios in a feature

### ❌ DON'T

- Don't use `Thread.sleep()` — always use explicit waits (`WebDriverWait`)
- Don't share state between scenarios — each scenario must be independent
- Don't duplicate step definitions — one step definition per unique step
- Don't hardcode URLs — always read from `config.properties`
- Don't skip `@After` — driver must always quit to prevent browser leaks

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| `WebDriver not initialized` | Ensure `@Before` hook runs; check `@ui`/`@api` tag is set |
| `SessionNotCreatedException` | Update Chrome or let WebDriverManager download fresh driver: `-Dwdm.forceDownload=true` |
| `Cannot load config.properties` | Verify file exists at `src/test/resources/config.properties` |
| Tests pass locally but fail in CI | Add `-Dheadless=true`; ensure `--no-sandbox` Chrome flag is set |
| `StepDefinition not found` | Check `glue` in `TestRunner.java` includes your step def package |
| Allure report is empty | Ensure `mvn allure:report` runs after `mvn test`; check `target/allure-results/` exists |
| Cloud tests not starting | Verify `BS_USERNAME` / `BS_ACCESS_KEY` env vars are set and valid |
| `NoSuchElementException` | Increase `explicit.wait` in config or add `waitForPageToLoad()` call |
