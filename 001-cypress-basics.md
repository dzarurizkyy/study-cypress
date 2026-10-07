# ⚡ Cypress – Complete Guide
A comprehensive guide for learning Cypress, a modern End-to-End (E2E) testing framework for web applications.

---

## 📋 Table of Contents

- [What is Cypress](#-what-is-cypress)
- [Why E2E Testing](#-why-e2e-testing)
- [Installation](#-installation)
- [Installing the Demo App](#-installing-the-demo-app)
- [Project Structure](#-project-structure)
- [Writing Test Files](#-writing-test-files)
- [Cypress Commands](#-cypress-commands)
- [Assertions](#-assertions)
- [Running Tests](#-running-tests)
- [Generating Reports](#-generating-reports)
- [Test Cases List](#-test-cases-list)
- [Test Results Summary](#-test-results-summary)
- [Best Practices](#-best-practices)

---

## 🔥 What is Cypress

Cypress is a modern, JavaScript-based End-to-End testing framework built for the web. Unlike traditional tools like Selenium, Cypress runs **inside** the browser alongside the application, giving it native access to the DOM, network requests, and application state.

- Official website: [https://www.cypress.io/](https://www.cypress.io/)
- GitHub: [https://github.com/cypress-io/cypress](https://github.com/cypress-io/cypress)

- #### Platform Support

  Cypress is a **web-only** automation tool — it does not support mobile or desktop native applications. It is reliable for testing front-end frameworks like React through its Component Testing feature.

- #### Editions

  Cypress is available in two editions depending on your needs:

  | Edition | Description |
  |---------|-------------|
  | **Local** | Free, runs on your own machine |
  | **Cloud** | Paid, for enterprise needs and integrated CI/CD |

---

## 🧪 Why E2E Testing

End-to-End testing sits at the **top of the testing pyramid** because it is slower and more expensive than unit or integration tests. However, it is the most valuable type of testing because it:

- Simulates real user behavior across multiple scenarios
- Validates that the **entire system works correctly** before releasing to end users
- Catches issues that unit tests cannot — such as broken UI flows, incorrect routing, or backend integration failures

> E2E testing is the final safety net before your application reaches real users.

- #### Test Case Rule

  Every feature under test must always have a minimum number of test cases for both valid and invalid scenarios:

  | Type | Minimum Count |
  |------|---------------|
  | Positive test case | 1 |
  | Negative test case | More than 1 |

---

## 📦 Installation

- #### Prerequisites

  Before getting started, make sure the following tools are installed on your machine:

  | Requirement | Version |
  |-------------|---------|
  | Node.js | v18+ (tested with v22.11.0) |
  | npm or yarn | — |
  | Browser | Chrome, Firefox, Edge, or Electron |

- #### Operating System

  The recommended package manager differs by operating system:

  | OS | Package Manager |
  |----|----------------|
  | Windows | Chocolatey |
  | macOS | Homebrew |
  | Linux | apt |

- #### Initialize Project

  Create a new folder and initialize a Node.js project:

  ```bash
  mkdir my-e2e-project
  cd my-e2e-project
  npm init -y
  ```

- #### Install Cypress

  Install Cypress as a dev dependency:

  ```bash
  npm install cypress --save-dev
  ```

  Then run it for the first time to automatically generate the folder structure and config file:

  ```bash
  npx cypress open
  ```

- #### Install Mochawesome Reporter (Optional)

  Mochawesome is used to generate test results in a visual HTML format. Install all three packages:

  ```bash
  npm install mochawesome mochawesome-merge mochawesome-report-generator --save
  ```

- #### package.json Scripts

  Add these scripts to `package.json` to simplify running tests and generating reports:

  ```json
  {
    "scripts": {
      "cy:run": "cypress run",
      "cy:open": "cypress open",
      "report:merge": "mochawesome-merge cypress/results/*.json -o cypress/report.json",
      "report:generate": "marge cypress/report.json --charts true"
    },
    "devDependencies": {
      "cypress": "^15.14.1"
    },
    "dependencies": {
      "mochawesome": "^7.1.4",
      "mochawesome-merge": "^5.1.1",
      "mochawesome-report-generator": "^6.3.2"
    }
  }
  ```

  > ⚠️ Use `marge` (not `merge`) for the `report:generate` script. `marge` is the CLI provided by `mochawesome-report-generator`.

- #### Clone from GitHub

  If the Cypress project already exists on GitHub, simply clone and install dependencies — no separate Cypress install needed:

  ```bash
  git clone https://github.com/your-username/your-repo-name.git
  cd your-repo-name
  npm install
  ```

  > ✅ `npm install` is all you need as long as `cypress` is listed in `devDependencies`.

---

## 🛠️ Installing the Demo App

The demo app used as the test target is a Laravel application. Follow these steps to set it up locally before running any Cypress tests.

- #### Clone the Repository

  ```bash
  git clone https://github.com/siubie/demo-app-cypress-automation.git
  cd demo-app-cypress-automation
  ```

- #### Copy the Environment File

  Laravel requires a `.env` file to run. Copy the example file provided in the repository:

  ```bash
  cp .env.example .env
  ```

- #### Install PHP Dependencies

  Install all required PHP packages using Composer:

  ```bash
  composer install
  ```

- #### Configure the Database

  Open the `.env` file and update the MySQL connection settings to match your local database:

  ```env
  DB_CONNECTION=mysql
  DB_HOST=127.0.0.1
  DB_PORT=3306
  DB_DATABASE=study-cypress
  DB_USERNAME=root
  DB_PASSWORD=your_password
  ```

- #### Migrate and Seed the Database

  Run the migration to create all tables and seed them with initial data:

  ```bash
  php artisan migrate:fresh --seed
  ```

  > ✅ Make sure MySQL is running and the `study-cypress` database exists before running this command.

- #### Update Dependencies (Optional)

  If you encounter any package version conflicts, run:

  ```bash
  composer update
  ```

- #### Start the Server

  Start the Laravel development server:

  ```bash
  php artisan serve
  ```

  The application will be available at `http://localhost:8000`.

- #### Reset Database During Testing

  Since tests modify the database, it needs to be reset to a clean state before each run. This can be done directly from within a test using `cy.exec()`:

  ```javascript
  cy.exec("cd ./demo-app-cypress-automation && php artisan migrate:refresh --seed");
  ```

---

## 🗂️ Project Structure

After running `npx cypress open` for the first time, the following folder structure is automatically generated:

```
my-e2e-project/
├── cypress/
│   ├── e2e/                   # Test spec files go here
│   │   ├── user-can-login.cy.js
│   │   └── user-can-create.cy.js
│   ├── fixtures/              # Static test data (JSON files)
│   ├── support/
│   │   ├── commands.js        # Custom Cypress commands
│   │   └── e2e.js             # Global configuration / hooks
│   ├── results/               # Mochawesome JSON reports per spec
│   └── videos/                # Recorded videos of each test run
├── cypress.config.js          # Cypress configuration file
└── package.json
```

- #### cypress.config.js

  The main configuration file for Cypress. Below is a full example with the most commonly used options:

  ```javascript
  const { defineConfig } = require('cypress');

  module.exports = defineConfig({
    allowCypressEnv: false,
    viewportWidth: 1920,
    viewportHeight: 1080,
    screenshotOnRunFailure: false,
    video: false,
    watchForFileChanges: false,
    screenshotsFolder: "cypress/screenshots",
    videosFolder: "cypress/videos",
    e2e: {
      baseUrl: 'http://localhost:8000',
      specPattern: 'cypress/e2e/**/*.cy.{js,jsx,ts,tsx}',
      setupNodeEvents(on, config) {},
    },
  });
  ```

  Each option controls a specific behavior of the test runner:

  | Option | Description |
  |--------|-------------|
  | `allowCypressEnv` | Allow or disallow Cypress environment variables |
  | `viewportWidth` / `viewportHeight` | Browser viewport size during testing |
  | `screenshotOnRunFailure` | Automatically capture a screenshot on test failure |
  | `video` | Record video during the test run |
  | `watchForFileChanges` | Auto re-run when files change |
  | `baseUrl` | Base URL so `cy.visit('/')` goes directly to the app |

- #### Experimental Studio

  Cypress Studio is a visual code recorder that generates test code automatically from browser interactions. To enable it, add this option to `cypress.config.js`:

  ```javascript
  expirementalStudio: true
  ```

  Once enabled, hovering over a test title in the Spec menu reveals an **Open Studio** button. Click it to start recording interactions in the browser, then click **Stop Recording** — Cypress writes the corresponding test code automatically and saves it to the spec file.

---

## ✏️ Writing Test Files

Cypress uses **Mocha** syntax with `describe` and `it` blocks.

- #### Basic Structure

  A test file consists of a `describe` block (the suite) wrapping one or more `it` blocks (individual test cases). The `beforeEach` hook runs setup code before every test:

  ```javascript
  describe('Suite Name', () => {
    beforeEach(() => {
      cy.visit('/');
    });

    it('test case description', () => {
      // Arrange → Act → Assert
    });
  });
  ```

- #### AAA Pattern (Arrange, Act, Assert)

  Every test case should follow the **AAA pattern** — a consistent structure that makes tests easy to read and debug:

  | Stage | Purpose |
  |-------|---------|
  | **Arrange** | Setup — visit the page, verify the initial state |
  | **Act** | Action — fill forms, click buttons, interact with the UI |
  | **Assert** | Verify — confirm the result matches expectations |

  Here is a complete login test following the AAA pattern:

  ```javascript
  it('user can login with valid credentials', () => {
    // Arrange
    cy.visit('http://localhost:8000/');
    cy.get('h4').should('have.text', 'Login');

    // Act
    cy.get('[data-cy="email"]').type('superadmin@gmail.com');
    cy.get('[data-cy="password"]').type('password');
    cy.get('[data-cy="submit"]').click();

    // Assert
    cy.get('.nav-link > .d-sm-none').should('have.text', 'Hi, SuperAdmin');
  });
  ```

- #### Hooks (Before & After)

  Cypress provides four lifecycle hooks to run code at specific points around your tests:

  | Hook | When It Runs |
  |------|--------------|
  | `before()` | Once before all tests in the `describe` block |
  | `beforeEach()` | Before **every** test — ideal for login and DB reset |
  | `after()` | Once after all tests complete |
  | `afterEach()` | After **every** test — ideal for cleanup |

  The most common pattern is using `beforeEach` to reset the database and log in before every test, so each test always starts from a clean, authenticated state:

  ```javascript
  describe('user can create new user', () => {
    beforeEach(() => {
      cy.visit('http://localhost:8000/');
      cy.exec("cd ./demo-app-cypress-automation && php artisan migrate:refresh --seed");
      cy.get('[data-cy="email"]').type('superadmin@gmail.com');
      cy.get('[data-cy="password"]').type('password');
      cy.get('[data-cy="submit"]').click();
    });

    it('user can create new user', () => {
      // test steps...
    });
  });
  ```

---

## 🔧 Cypress Commands

- #### Selecting Elements

  Cypress provides several commands for traversing and selecting elements in the DOM:

  | Command | Description |
  |---------|-------------|
  | `cy.get('selector')` | Select element(s) by CSS selector |
  | `cy.contains('text')` | Select element containing matching text |
  | `cy.get('selector').find('child')` | Find a child within the selected element |
  | `cy.get('selector').parent()` | Get the parent of the selected element |
  | `cy.get('selector').next()` | Get the next sibling element |
  | `cy.get('selector').nextAll()` | Get all siblings after the element |
  | `cy.get('selector').prev()` | Get the previous sibling element |

- #### Selector Best Practice

  Avoid **highly brittle** selectors — class or ID-based selectors break whenever the UI is refactored. The recommended approach is to add `data-cy` attributes to your HTML elements and target those instead:

  ```html
  <!-- Add data-cy to the element in HTML -->
  <button data-cy="submit">Login</button>
  ```

  ```javascript
  // Target it in Cypress using the attribute selector
  cy.get('[data-cy="submit"]').click();
  ```

  When Cypress Studio generates selectors automatically, it follows this priority order:

  1. `data-cy`
  2. `data-test`
  3. `data-testid`
  4. `id`
  5. `class`

- #### User Interactions

  These are the most commonly used commands for simulating user actions:

  | Command | Description |
  |---------|-------------|
  | `cy.get('...').click()` | Click on an element |
  | `cy.get('...').type('text')` | Type text into an input field |
  | `cy.get('...').clear()` | Clear an input field |
  | `cy.visit('url')` | Navigate to a URL |

- #### Chaining

  Cypress commands are chainable, which makes it easy to traverse the DOM and interact with related elements in a single expression. For example, to find a row by its content and click the Delete button in that same row:

  ```javascript
  cy.get('.table td')
    .contains('user')
    .parent()
    .contains('Delete')
    .click();
  ```

- #### Run a Single Test

  During debugging, use `.only` to isolate and run just one test without executing the rest of the suite:

  ```javascript
  it.only('user can login with valid username and password', () => {
    // only this test will run
  });
  ```

---

## ✅ Assertions

Cypress uses **Chai** assertions via the `.should()` command.

- #### Common Assertions

  These are the most frequently used assertions when verifying UI state:

  | Assertion | Description |
  |-----------|-------------|
  | `.should('be.visible')` | Element is visible on the page |
  | `.should('have.text', 'value')` | Element has the **exact** text |
  | `.should('contain', 'value')` | Element **contains** the text |
  | `.should('have.class', 'class-name')` | Element has the specified CSS class |
  | `.should('not.contain', 'value')` | Element does **not** contain the text |

  > `have.text` checks for an exact match. `contain` only requires the word to appear somewhere in the element.

- #### Examples

  Asserting a success message after a form submission:

  ```javascript
  cy.get('#app p').should('be.visible');
  cy.get('#app p').should('have.text', 'Data Added Successfully');
  ```

  Asserting a validation error when invalid data is submitted:

  ```javascript
  cy.get('.invalid-feedback')
    .should('be.visible')
    .should('contain', 'The email must be a valid email address.');
  ```

  Asserting a success alert that also has a specific CSS class:

  ```javascript
  cy.get('.alert')
    .should('be.visible')
    .and('have.class', 'alert-success')
    .contains('User Deleted Successfully');
  ```

  Asserting that a deleted record no longer appears in the table:

  ```javascript
  cy.get('.table').should('not.contain', 'deleted-user');
  ```

---

## ▶️ Running Tests

- #### Headless Mode (CI/CD)

  The most common way to run tests in a pipeline. All specs are executed without opening a browser UI:

  ```bash
  npm run cy:run
  ```

- #### Run a Specific Spec

  To run only one spec file instead of the entire suite, pass the file path using the `--spec` flag:

  ```bash
  npm run cy:run -- --spec cypress/e2e/user-can-open-login-page.cy.js
  ```

- #### With Mochawesome Reporter

  To generate a JSON report file for each spec, pass the `--reporter` flag:

  ```bash
  npm run cy:run -- --reporter mochawesome
  ```

  Individual report files are saved to `cypress/results/`:

  ```
  cypress/results/
  ├── user-can-create-new-user.json
  ├── user-can-create-new-user.html
  ├── user-can-delete-data.json
  └── ...
  ```

- #### Interactive Mode

  Opens the Cypress Test Runner with a full UI — useful for writing and debugging tests locally:

  ```bash
  npm run cy:open
  ```

  The Test Runner has four main menus:

  | Menu | Description |
  |------|-------------|
  | **Specs** | Run tests locally |
  | **Runs** | Run tests using Cypress Cloud |
  | **Debug** | Debug tests running on cloud |
  | **Settings** | Project configuration |

---

## 📊 Generating Reports

After running all tests with the Mochawesome reporter, follow these two steps to produce a single combined HTML report.

- #### Step 1 — Merge Individual Reports

  Each spec file produces its own JSON report. Merge them all into one file:

  ```bash
  npm run report:merge
  ```

  This runs the following command under the hood:

  ```bash
  mochawesome-merge cypress/results/*.json -o cypress/report.json
  ```

- #### Step 2 — Generate HTML Report

  Convert the merged JSON into a visual HTML report with charts:

  ```bash
  npm run report:generate
  ```

  This runs the following command under the hood:

  ```bash
  marge cypress/report.json --charts true
  ```

  The final report will be saved at:

  ```
  ✓ Reports saved:
  /Users/dzarurizky/Documents/test/mochawesome-report/report.html
  ```

- #### Attach Screenshots and Videos

  To automatically embed screenshots and videos into the report, add the following to `cypress/support/e2e.js`. When a test fails, it attaches the screenshot; for every test it attaches the recorded video:

  ```javascript
  import addContext from "mochawesome/addContext";

  const titleToFileName = (title) => title.replace(/[:\/]/g, "");

  Cypress.on("test:after:run", (test, runnable) => {
    if (test.state === "failed") {
      let parent = runnable.parent;
      let filename = "";
      while (parent && parent.title) {
        filename = `${titleToFileName(parent.title)} -- ${filename}`;
        parent = parent.parent;
      }
      filename += `${titleToFileName(test.title)} (failed).png`;
      addContext({ test }, `../screenshots/${Cypress.spec.name}/${filename}`);
    }
    addContext({ test }, `../videos/${Cypress.spec.name}.mp4`);
  });
  ```

  > ⚠️ **Common mistake**: The script must use `marge` (the CLI from `mochawesome-report-generator`), not `merge`. Using `merge` causes `sh: merge: command not found`.

---

## 🗒️ Test Cases List

A detailed breakdown of every spec file and the test cases it covers.

---

### 1. `user-can-open-login-page.cy.js`

**Suite:** User can login to system

| # | Test Case | Status |
|---|-----------|--------|
| 1 | user can login with valid username and password | ✅ Pass |

The test visits the login page, fills in valid credentials, and asserts that the user is redirected and greeted by name:

```javascript
it('user can login with valid username and password', () => {
  cy.visit('http://localhost:8000/');
  cy.get('h4').should('have.text', 'Login');

  cy.get('[data-cy="email"]').type('superadmin@gmail.com');
  cy.get('[data-cy="password"]').type('password');
  cy.get('[data-cy="submit"]').click();

  cy.get('.nav-link > .d-sm-none').should('have.text', 'Hi, SuperAdmin');
});
```

---

### 2. `user-can-create-new-user.cy.js`

**Suite:** user can create new user

| # | Test Case | Status |
|---|-----------|--------|
| 1 | user can create new user | ✅ Pass |
| 2 | user cannot create new user because invalid email | ✅ Pass |

The first test submits the form with valid data and asserts the success message. The second test submits with a malformed email (no domain) and asserts the validation error:

```javascript
// Test 1 — Happy path
it('user can create new user', () => {
  cy.get('#app a.icon-left').click();
  cy.get('[name="name"]').type('User Baru');
  cy.get('[name="email"]').type('userbaru@gmail.com');
  cy.get('[name="password"]').type('user1234');
  cy.get('#app button.btn-primary').click();

  cy.get('#app p').should('be.visible');
  cy.get('#app p').should('have.text', 'Data Added Successfully');
});

// Test 2 — Negative path: invalid email format
it('user cannot create new user because invalid email', () => {
  cy.get('#app a.icon-left').click();
  cy.get('[name="name"]').type('User Baru');
  cy.get('[name="email"]').type('userbaru'); // missing domain
  cy.get('[name="password"]').type('user1234');
  cy.get('#app button.btn-primary').click();

  cy.get('.invalid-feedback').should('be.visible');
  cy.get('.invalid-feedback').should('contain', 'The email must be a valid email address.');
});
```

---

### 3. `user-can-edit-existing-data.cy.js`

**Suite:** User Can Edit Existing Data

| # | Test Case | Status |
|---|-----------|--------|
| 1 | user can update name and email | ✅ Pass |
| 2 | user cannot update name with empty name field | ✅ Pass |
| 3 | user cannot update email with empty email field | ✅ Pass |
| 4 | user cannot update with empty email and name field | ✅ Pass |

The first test updates both fields and asserts the success alert and the updated value in the table. Tests 2–4 clear required fields one by one and assert that the correct validation messages appear:

```javascript
// Test 1 — Happy path
it('user can update name and email', () => {
  cy.get('.table td').contains('user').parent().contains('Edit').click();
  cy.get('[name="name"]').type(' baru');
  cy.get('[name="email"]').clear().type('userbaru@gmail.com');
  cy.get('.btn-primary').contains('Submit').click();

  cy.get('.alert').should('be.visible').and('have.class', 'alert-success').contains('User Updated Successfully');
  cy.get('.table td').contains('user').should('have.text', 'user baru');
});

// Test 2 — Negative: empty name field
it('user cannot update name with empty name field', () => {
  cy.get('.table td').contains('user').parent().contains('Edit').click();
  cy.get('[name="name"]').clear();
  cy.get('.btn-primary').contains('Submit').click();

  cy.get('#app div.invalid-feedback')
    .contains('The name field is required.')
    .should('have.class', 'invalid-feedback')
    .should('be.visible');
});

// Test 3 — Negative: empty email field
it('user cannot update email with empty email field', () => {
  cy.get('.table td').contains('user').parent().contains('Edit').click();
  cy.get('[name="email"]').clear();
  cy.get('.btn-primary').contains('Submit').click();

  cy.get('#app div.invalid-feedback')
    .contains('The email field is required.')
    .should('have.class', 'invalid-feedback')
    .should('be.visible');
});

// Test 4 — Negative: both fields empty
it('user cannot update with empty email and name field', () => {
  cy.get('.table td').contains('user').parent().contains('Edit').click();
  cy.get('[name="name"]').clear();
  cy.get('[name="email"]').clear();
  cy.get('.btn-primary').contains('Submit').click();

  cy.get('#name').next().contains('The name field is required.').should('be.visible');
  cy.get('#email').next().contains('The email field is required.').should('be.visible');
});
```

---

### 4. `user-can-delete-data.cy.js`

**Suite:** user can delete user

| # | Test Case | Status |
|---|-----------|--------|
| 1 | user can delete user | ✅ Pass |
| 2 | user can cancel delete action | ✅ Pass |

The first test confirms the deletion dialog and asserts the success alert and that the record is removed from the table. The second test cancels the dialog and asserts the record is still present:

```javascript
// Test 1 — Happy path: confirm deletion
it('user can delete user', () => {
  cy.get('.table td').contains('user').parent().contains('Delete').click();
  cy.get('.swal-button-container').find('button').contains('OK').click();

  cy.get('.alert').should('be.visible').and('have.class', 'alert-success').contains('User Deleted Successfully');
  cy.get('.table').should('not.contain', 'user');
});

// Test 2 — Cancel: data should remain
it('user can cancel data', () => {
  cy.get('.table td').contains('user').parent().contains('Delete').click();
  cy.get('.swal-button-container').find('button').contains('Cancel').click();

  cy.get('.table').should('contain', 'user').should('be.visible');
});
```

---

### 5. `user-can-logout-system.cy.js`

**Suite:** User can logout from system

| # | Test Case | Status |
|---|-----------|--------|
| 1 | user can logout from system | ✅ Pass |

The test logs in, clicks the profile menu, triggers logout, and asserts that the user is redirected back to the login page:

```javascript
it('user can logout from system', () => {
  cy.visit('http://localhost:8000/');
  cy.get('h4').should('have.text', 'Login');

  cy.get('[data-cy="email"]').type('superadmin@gmail.com');
  cy.get('[data-cy="password"]').type('password');
  cy.get('[data-cy="submit"]').click();
  cy.get('#app a.nav-link-user').click();
  cy.get("[data-cy='logout']").click();

  cy.get('h4').should('have.text', 'Login');
});
```

---

## 📈 Test Results Summary

Results from a complete test run (10 tests across 5 spec files):

| Spec File | Duration | Tests | Passing | Failing |
|-----------|----------|-------|---------|---------|
| `user-can-open-login-page.cy.js` | 00:02 | 1 | 1 | 0 |
| `user-can-create-new-user.cy.js` | 00:19 | 2 | 2 | 0 |
| `user-can-edit-existing-data.cy.js` | 01:18 | 4 | 4 | 0 |
| `user-can-delete-data.cy.js` | 00:16 | 2 | 2 | 0 |
| `user-can-logout-system.cy.js` | 00:02 | 1 | 1 | 0 |
| **Total** | **01:59** | **10** | **10** | **0** |

> ✔️ All 10 specs passed with 0 failures.

---

## 💡 Best Practices

- #### Selectors
  - Always prefer `data-cy` attributes over CSS classes or IDs that may change
  - Avoid XPath — keep selectors simple and readable
  - Use `.contains()` to find elements by their visible text for readability

- #### Test Structure
  - Follow the **AAA pattern** (Arrange, Act, Assert) for every test case
  - Use `beforeEach()` for shared setup like visiting a page or logging in
  - Keep each test **independent** — avoid relying on state from a previous test

- #### Naming
  - Name spec files descriptively: `user-can-create-new-user.cy.js`
  - Write test descriptions as user stories: `"user can login with valid credentials"`

- #### Reporting
  - Always run tests with `--reporter mochawesome` in CI to generate reports
  - Merge and generate the HTML report after each full test run for documentation
  - Store the `mochawesome-report/report.html` as a CI artifact for team visibility

- #### Performance
  - Use `cy.intercept()` to stub API calls when testing UI-only behavior — it's faster and more reliable
  - Avoid hard-coded `cy.wait(ms)` — use assertions or `cy.intercept()` aliases instead
  - Run tests in headless mode (`cypress run`) for significantly faster execution in CI/CD pipelines
