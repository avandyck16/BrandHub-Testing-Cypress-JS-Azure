# Confidentiality Notice

> This repository contains an anonymized portfolio case study based on a real-world project.
>
> Company names, client information, URLs, credentials, proprietary business logic, and sensitive implementation details have been removed or modified to protect confidentiality.
>
> Any code snippets included are examples only and do not represent any protected production codebase.

---
# BrandHub — E2E Automation Framework

**AI-Powered Branding Platform | Cypress | JavaScript | Azure DevOps | CI/CD**

E2E automation framework for validating critical workflows in an AI-powered branding platform and integrating automated testing into Azure DevOps.

- **~89% less validation time:** from ~45 minutes to ~5 minutes 10 seconds.
- **36 automated scenarios** covering critical functional and negative workflows.
- **3 environments:** Local, Development, and Production.
- **Azure DevOps quality gate** for pre-deployment validation.
- **Automated reports** published as pipeline artifacts.

## Quick Scope

| Category | Details |
|---|---|
| Role | QA Engineer |
| Testing | E2E, Regression, Negative |
| Automation | Cypress, JavaScript |
| Reporting | Mochawesome |
| CI/CD | Azure DevOps Pipelines, YAML |
| Environments | Local, Development, Production |
| Automated Scenarios | 36 |

## 1. Project Overview

**Objective:** Automate high-value user journeys and integrate E2E validation into the delivery workflow.

As the QA Engineer responsible for the framework, I developed and maintained Cypress tests, improved selector stability, handled asynchronous workflows, generated dynamic test data, and integrated automated execution and reporting into Azure DevOps.

**Technology Stack:** Cypress · JavaScript · Azure DevOps · YAML · Mochawesome · Azure Static Web Apps · GitHub

## 2. QA Strategy and Automation Scope

The suite prioritizes repetitive, business-critical workflows suitable for repeatable regression and pre-deployment validation.

| Workflow | Coverage |
|---|---|
| Login | Functional and negative scenarios |
| Signup | Registration and negative scenarios |
| Questionnaire | Main continuous E2E workflow |
| Color Palette Delivery | Generated artifact delivery |
| Brand Kit Delivery | Generated artifact delivery |
| Assets Delivery | Generated asset delivery |

The suite provides representative coverage of critical workflows, not complete application coverage.

## 3. Test Architecture

Tests are organized by functional workflow, with negative cases separated where appropriate.

```text
automated-tests/
└── cypress/
    ├── e2e/
    │   └── Regression-Tests/
    │       ├── 1.login.cy.js
    │       ├── 2.login.negative.cy.js
    │       ├── 3.signup.cy.js
    │       ├── 4.signup.negative.cy.js
    │       └── 5.questionnaire.cy.js
    └── fixtures/
        └── bh-credentials.json
```

Fixtures centralize reusable test data, including credentials used by negative scenarios.

## 4. Execution and Environment Strategy

NPM scripts support local execution, environment-specific runs, and interactive debugging. URLs and credentials are supplied through environment configuration rather than committed to the repository.

| Script | Purpose |
|---|---|
| `test:local` | Run against the local application |
| `test:dev` | Run against Development |
| `test:prod` | Run against Production |
| `debug:local` | Open Cypress interactively |

## 5. Technical Implementation & Helpers

### Backend Availability — `pingAPI()`

**Challenge:** The backend could become inactive after extended periods without traffic, causing initial test failures.

**Implementation:** Added a reusable API request before the suite to help wake up the backend.

```javascript
Cypress.Commands.add("pingAPI", () => {
  const API_URL = Cypress.env("API_URL");

  cy.request({
    method: "POST",
    url: API_URL + "/api/connect/login",
    failOnStatusCode: false,
    body: {
      username: "invalid@yopmail.com",
      password: "invalidpass",
      secured: false,
    },
  }).then((response) => {
    expect(response.status).to.be.oneOf([200, 400, 401, 405]);
  });
});
```

The command runs from the suite-level setup hook:

```javascript
before(() => {
  cy.pingAPI();
});
```

**Result:** Helps reduce false failures associated with backend cold starts. It is a wake-up request, not a comprehensive health check.

### Automation-Oriented Selectors

**Challenge:** Some UI elements lacked stable selectors, increasing test fragility.

**Implementation:** Worked with development to introduce dedicated `data-cy` attributes for automation.

**Result:** Improved selector stability and reduced dependence on changes to the UI structure.

### Asynchronous Rendering

**Challenge:** Some elements rendered asynchronously or outside the expected DOM structure.

**Implementation:** Investigated the rendering behavior and collaborated with development to address the underlying implementation. Dedicated automation selectors were incorporated where needed.

**Result:** Stabilized affected E2E scenarios and improved rendering consistency.

### Continuous Questionnaire Workflow

**Challenge:** The questionnaire depended on persistent session state, dynamically generated URLs, and sequential user context.

**Implementation:** Kept dependent steps within one continuous E2E scenario, validating questions, inputs, and expected results sequentially.

**Result:** Preserved workflow state and reduced failures caused by separating dependent steps into independent tests.

### Generative AI Response Times

**Challenge:** An AI model update increased name-generation response times beyond Cypress's default waiting behavior.

**Implementation:** Used `cy.intercept()` to synchronize with the API response and extended the timeout for the AI operation.

```javascript
cy.intercept("POST", "[Described for privacy]").as("generateContent");

cy.get('[data-testid="submit"]').click();

cy.wait("@generateContent", { timeout: 55000 })
  .then((interception) => {
    expect(interception.response?.statusCode).to.eq(200);
  });

cy.contains(
  "Here are three name suggestions based on your brand description:",
  { timeout: 10000 }
).should("be.visible");
```

**Result:** Improved synchronization with the AI workflow and reduced premature failures.

*The endpoint shown above is redacted in the public example.*

### Dynamic Test Data

**Challenge:** Reusing email addresses caused registration conflicts when accounts already existed.

**Implementation:** Generated unique email values using a timestamp.

```javascript
const email = `cypress_user${Date.now()}@yopmail.com`;
```

**Result:** Reduced duplicate-account conflicts between test executions.

### Automated Reporting

**Challenge:** Execution results needed to remain accessible after pipeline runs.

**Implementation:** Integrated Mochawesome and configured Azure DevOps to publish generated reports as pipeline artifacts.

**Result:** Centralized execution evidence, screenshots on failure, and report access for post-run analysis.

## 6. CI/CD and Reporting

The Cypress suite is integrated into Azure DevOps as a pre-deployment quality gate. A push or merge to `dev` or `main` triggers the corresponding pipeline flow.

### Pipeline Flow

```text
Push or merge to dev / main
             |
             v
     Build application
      (branch-specific)
             |
             v
     Publish build artifact
             |
             v
     Download build artifact
             |
             v
 Serve artifact on localhost
             |
             v
       Run Cypress
             |
        Tests pass?
        /         \
      No           Yes
      |             |
      v             v
 Deployment      Deploy the same
 blocked         tested artifact
                 dev → DEV
                 main → PROD
```

### How It Works

1. **Branch-aware build:** The target branch determines the application build.
2. **Build artifact:** The generated application is published as an artifact.
3. **Pre-deployment testing:** The test stage downloads the artifact and serves it locally on the pipeline agent.
4. **Cypress execution:** Tests run against the built application using the corresponding environment configuration.
5. **Deployment gate:** A failed test stage blocks deployment; a successful run allows deployment to continue.
6. **Environment-specific deployment:** The same tested artifact is deployed to DEV for `dev` or PROD for `main`.

The application is tested from the build artifact served locally on the pipeline agent. It is not first deployed to a separate test website before Cypress runs.

### Reporting

Mochawesome reports are published as pipeline artifacts, providing:

- HTML and JSON execution reports.
- Test results and failure details.
- Cypress screenshots when tests fail.
- Evidence for post-execution investigation.

## 7. Results and Limitations

| Metric | Result |
|---|---:|
| Automated scenarios | **36** |
| Manual validation | ~45 minutes |
| Automated execution | ~5 min 10 sec |
| Time reduction | **~89%** |
| Supported environments | **3** |
| Technical challenges documented | **7** |

**Additional improvements:** More repeatable regression validation, improved E2E stability, dynamic registration data, backend cold-start mitigation, and automated reporting integrated into CI/CD.

**Known limitations:**

- AI-generated responses can vary in content and timing.
- Some workflows depend on session continuity.
- Backend availability may require an initial wake-up request.
- Environment-specific credentials and configuration are required.
- The current suite prioritizes critical workflows rather than complete application coverage.

---

<div style="text-align:center; color:#888; font-size:10px;">
    QA Documentation | Axel Van Dyck | QA Engineer 
</div>
