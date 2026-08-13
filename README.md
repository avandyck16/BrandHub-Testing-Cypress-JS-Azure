# Confidentiality Notice

> This repository contains an anonymized portfolio case study based on a real-world project.
>
> Company names, client information, URLs, credentials, proprietary business logic, and sensitive implementation details have been removed or modified to protect confidentiality.
>
> Any code snippets included are examples only and do not represent any protected production codebase.

---

# BrandHub — E2E Automation Framework

**AI-Powered Branding Platform | Cypress | JavaScript | Azure DevOps | CI/CD**

E2E automation framework designed to validate critical workflows within an AI-powered branding platform and integrate automated validation into the software delivery pipeline.

---

## Quick Scope

| Area                    | Details                                    |
| ----------------------- | ------------------------------------------ |
| **Role**                | QA Engineer                                |
| **Testing**             | E2E · Regression · Negative                |
| **Automation**          | Cypress · JavaScript                       |
| **Reporting**           | Mochawesome                                |
| **CI/CD**               | Azure DevOps Pipelines                     |
| **Cloud**               | Azure Static Web Apps · Azure Blob Storage |
| **Environments**        | Local · Development · Production           |
| **AI Integration**      | Generative AI workflows                    |
| **Automated Scenarios** | 36                                         |

### Key Results

* **~89% reduction** in validation time: ~45 min → ~5m 10s.
* **36 automated scenarios** covering critical product workflows.
* Execution across **3 environments**.
* Automated BVT integrated into Azure DevOps.
* Automated test reports published as pipeline artifacts.
* Resolved **7 technical challenges** involving backend availability, selectors, asynchronous rendering, AI response times, dynamic data, and reporting.
* Improved E2E stability and reduced manual validation effort before deployment.

---

## Case Study Index

* [Project Overview](#project-overview)
* [QA Strategy](#qa-strategy)
* [Automation Scope & Decisions](#automation-scope--decisions)
* [Technical Implementation](#technical-implementation)
* [Test Architecture](#test-architecture)
* [CI/CD & Reporting](#cicd--reporting)
* [Results](#results)
* [Limitations & Known Issues](#limitations--known-issues)

---

# Project Overview

## Context

BrandHub is an AI-powered branding platform that uses generative AI integrations to assist users in creating branding materials and digital assets.

The platform includes workflows such as authentication, registration, questionnaires, brand definition, and delivery of generated branding artifacts.

## Objective

Design and implement E2E automation focused on critical and repetitive workflows, with the additional goal of integrating automated validation into CI/CD as a Build Validation Testing (BVT) quality gate.

The framework was designed to support execution against Local, Development, and Production environments.

## My Role

* Designed E2E test coverage for critical business workflows.
* Developed and maintained Cypress/JavaScript specifications.
* Added automation-oriented selectors to the frontend.
* Implemented dynamic test data generation.
* Maintained and optimized test suites.
* Investigated flaky tests and execution failures.
* Implemented the automation framework and integrated it with Azure DevOps pipelines.
* Configured multi-environment execution.
* Implemented automated reporting and pipeline artifacts.

## Technology Stack

* Cypress
* JavaScript
* Azure DevOps
* YAML Pipelines
* Mochawesome Reporter
* Azure Static Web Apps
* Azure Blob Storage
* Azure CLI
* dotenv-cli
* GitHub

---

# QA Strategy

The automation strategy prioritized workflows that were:

* Business-critical.
* Repetitive.
* Representative of the core product experience.
* Suitable for reliable E2E validation.
* Valuable for pre-deployment regression coverage.

The suite combines functional and negative validations across authentication, registration, questionnaire processing, and generated branding artifacts.

---

# Automation Scope & Decisions

| Flow                    | Decision   | Coverage                     |
| ----------------------- | ---------- | ---------------------------- |
| Login                   | ✅ Automate | Functional + Negative        |
| Signup                  | ✅ Automate | Functional + Negative        |
| Questionnaire           | ✅ Automate | Main E2E workflow            |
| Color Palettes Delivery | ✅ Automate | Artifact delivery validation |
| Brand Kit Delivery      | ✅ Automate | Artifact delivery validation |
| Assets Delivery         | ✅ Automate | Artifact delivery validation |

The selected workflows represent the highest-value automated coverage currently implemented for the platform.

---

# Technical Implementation

## Backend Availability — Cold Starts

The backend could enter a suspended state after extended periods without traffic, causing initial test failures unrelated to application functionality.

**Implementation:** Added a reusable API wake-up command executed before the test suite to initialize the backend and prevent false negatives.

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
    expect(response.status).to.be.oneOf([200, 400, 401]);
  });
});
```

Executed globally through:

```javascript
before(() => {
  cy.pingAPI();
});
```

**Result:** Reduced false failures caused by backend cold starts and improved suite reliability.

---

## Automation-Oriented Selectors

Several UI elements lacked stable selectors, increasing test fragility.

**Implementation:** Collaborated with development to introduce dedicated `data-cy` selectors for automation.

**Result:** Improved selector stability and reduced failures caused by changes in UI structure.

![Automation selector implementation](https://github.com/user-attachments/assets/cba89f29-43cf-490f-b6b8-61d2b8b2dd6a)

---

## Asynchronous Rendering

Some elements were rendered asynchronously or outside the expected DOM structure, resulting in intermittent test failures.

**Implementation:** Investigated the rendering behavior and worked with development to correct the underlying implementation. Dedicated automation selectors were then incorporated.

**Result:** Stabilized affected E2E scenarios and improved UI rendering consistency.

---

## Continuous Questionnaire Workflow

The questionnaire depended on persistent session state, dynamically generated URLs, and sequential user context.

**Implementation:** Designed the questionnaire as a single continuous E2E scenario, maintaining the user session while validating each question, input, and expected result sequentially.

**Result:** Preserved workflow state and prevented failures caused by splitting dependent steps across independent tests.

---

## Generative AI Response Times

A generative AI model update increased the response time for name generation beyond Cypress's default waiting behavior.

**Implementation:** Added a Cypress network intercept to wait for the specific API response and extended the timeout for the AI operation.

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

**Result:** Reliable validation of AI-generated content without premature test failures.

---

## Dynamic Test Data

Registration workflows required unique users on every execution to prevent account duplication.

**Implementation:** Generated unique credentials dynamically during test execution.

```javascript
const email = `cypress_user${Date.now()}@yopmail.com`;
```

**Result:** Enabled repeatable registration testing without conflicts caused by previously created accounts.

---

## Automated Reporting

Test execution results needed to be accessible from Azure DevOps for analysis after pipeline execution.

**Implementation:** Integrated Mochawesome reporting and configured the pipeline to publish generated reports as artifacts.

**Result:** Centralized execution evidence and improved failure analysis after automated runs.

---

# Test Architecture

The framework separates E2E regression scenarios within the Cypress structure:

```text
automated-tests
└── cypress
    ├── e2e
    │   └── Regression-Tests
    │       ├── 1.login.cy.js
    │       ├── 2.login.negative.cy.js
    │       ├── 3.signup.cy.js
    │       ├── 4.signup.negative.cy.js
    │       └── 5.questionnaire.cy.js
    │
    └── fixtures
        └── bh-credentials.json
```

Fixtures centralize reusable test data, particularly information required for negative scenarios.

---

# Execution & Environment Strategy

The framework supports independent execution against Local, Development, and Production environments.

| Script        | Purpose                                  |
| ------------- | ---------------------------------------- |
| `test:local`  | Execute suites against localhost         |
| `test:dev`    | Execute suites against Development       |
| `test:prod`   | Execute suites against Production        |
| `debug:local` | Open Cypress interactively for debugging |

Environment-specific URLs and credentials are managed through environment variables and are not stored in the repository.

---

# CI/CD & Reporting

## Pipeline Strategy

The Cypress suite is integrated into Azure DevOps as a validation gate before the final deployment.

The target branch determines which build is generated and validated.

### Development Flow

```text
Merge → dev
   ↓
Build DEV
   ↓
Deploy to Test Environment
   ↓
Execute Cypress Tests
   ↓
Tests Passed?
   ├── No  → Pipeline Stops
   └── Yes → Deploy to DEV
```

### Production Flow

```text
Merge → main
   ↓
Build PROD
   ↓
Deploy to Test Environment
   ↓
Execute Cypress Tests
   ↓
Tests Passed?
   ├── No  → Pipeline Stops
   └── Yes → Deploy to PROD
```

This approach ensures that the build intended for the target environment is validated before the final deployment.

A failed Cypress execution prevents the corresponding deployment from continuing.

## Test Reports

Mochawesome reports are published as Azure pipeline artifacts, providing:

* Execution results.
* Failure details.
* Screenshots on failure.
* Post-execution evidence.
* Historical analysis of test runs.

---

# Results

| Metric                        |   Result |
| ----------------------------- | -------: |
| Automated scenarios           |   **36** |
| Manual validation time        |  ~45 min |
| Automated execution           |  ~5m 10s |
| Time reduction                | **~89%** |
| Supported environments        |    **3** |
| Technical challenges resolved |    **7** |

### Functional Coverage

* Login and authentication.
* Signup and account creation.
* Negative authentication and registration scenarios.
* Questionnaire workflow.
* Color Palette delivery.
* Brand Kit delivery.
* Generated Assets delivery.
* Generative AI response validation.

### Additional Improvements

* Reduced manual pre-deployment validation effort.
* Improved E2E test stability.
* Automated test reporting.
* CI/CD integration.
* Multi-environment execution.
* Dynamic test data generation.
* Backend cold-start mitigation.
* Automation-oriented frontend selectors.

---

# Limitations & Known Issues

* Some scenarios depend on external generative AI response times.
* Backend availability may require an initial wake-up request after extended inactivity.
* Some test data depends on environment configuration.
* Production execution requires environment-specific credentials and configuration.
* AI-generated responses may vary in content while remaining functionally valid.
* The current automation scope focuses on the highest-value workflows rather than complete application coverage.

---

<div style="text-align:center; color:#888; font-size:10px;">
    QA Documentation | Axel Van Dyck | QA Engineer 
</div>
