# 🧪 Cypress Hands-On Exercise — User Management App

A complete hands-on scenario that covers all core Cypress concepts: installation, configuration, writing tests, selectors, assertions, hooks, reporting, and best practices — all in one real-world user management use case.

---

## 📖 Scenario

You are a QA Engineer assigned to test a user management web application called **UserHub** — an admin dashboard where a super admin can manage users. You will test:

- **Authentication** — login and logout flows
- **CRUD Operations** — create, read, update, and delete users
- **Validation** — form validation for invalid inputs
- **Reporting** — generating a visual HTML test report

By the end of this exercise, you will have a full E2E test suite covering every key Cypress concept.

---

## 🗂️ Table of Contents

1. [Setup & Installation](#1-setup--installation)
2. [Configure Cypress](#2-configure-cypress)
3. [Writing Your First Test](#3-writing-your-first-test)
4. [Login & Logout Tests](#4-login--logout-tests)
5. [Create User Tests](#5-create-user-tests)
6. [Edit User Tests](#6-edit-user-tests)
7. [Delete User Tests](#7-delete-user-tests)
8. [Running Tests](#8-running-tests)
9. [Generating Reports](#9-generating-reports)
10. [Challenge Tasks](#-challenge-tasks)

---

## 1. Setup & Installation

### 1a. Prerequisites

Before starting, make sure all required tools are installed:

| Requirement | Version |
|-------------|---------|
| Node.js | v18+ |
| npm | — |
| Browser | Chrome, Firefox, or Edge |

### 1b. Initialize Project

Create a new project folder and initialize it:

```bash
mkdir userhub-e2e
cd userhub-e2e
npm init -y
```

### 1c. Install Cypress

Install Cypress as a dev dependency:

```bash
npm install cypress --save-dev
```

Open Cypress for the first time to auto-generate the folder structure:

```bash
npx cypress open
```

> ✅ Cypress will create the `cypress/` folder with all subfolders automatically.

### 1d. Install Mochawesome Reporter

Install the reporter packages for generating HTML test reports:

```bash
npm install mochawesome mochawesome-merge mochawesome-report-generator --save
```

### 1e. Configure package.json Scripts

Add the following scripts to your `package.json`:

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

> ⚠️ Use `marge` (not `merge`) in the `report:generate` script. `marge` is the CLI from `mochawesome-report-generator`.

---

## 2. Configure Cypress

### 2a. cypress.config.js

Update `cypress.config.js` with the settings below:

```javascript
const { defineConfig } = require('cypress');

module.exports = defineConfig({
  viewportWidth: 1920,
  viewportHeight: 1080,
  screenshotOnRunFailure: true,
  video: false,
  watchForFileChanges: false,
  e2e: {
    baseUrl: 'http://localhost:8000',
    specPattern: 'cypress/e2e/**/*.cy.{js,jsx,ts,tsx}',
    setupNodeEvents(on, config) {},
  },
});
```

> ❓ **Think about it**: What does `baseUrl` allow you to do in your tests? How does it shorten your `cy.visit()` calls?

### 2b. Project Structure

After setup, your folder structure should look like this:

```
userhub-e2e/
├── cypress/
│   ├── e2e/                   # Your test spec files
│   ├── fixtures/              # Static test data (JSON)
│   ├── support/
│   │   ├── commands.js        # Custom Cypress commands
│   │   └── e2e.js             # Global hooks & config
│   ├── results/               # Mochawesome JSON output
│   └── videos/                # Test recordings
├── cypress.config.js
└── package.json
```

---

## 3. Writing Your First Test

### 3a. The AAA Pattern

Every test case in Cypress must follow the **AAA pattern**:

| Stage | Purpose |
|-------|---------|
| **Arrange** | Visit the page, verify initial state |
| **Act** | Fill forms, click buttons, interact with UI |
| **Assert** | Confirm the expected result |

### 3b. Basic Test Structure

Create the file `cypress/e2e/sample.cy.js`:

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

### 3c. Selector Best Practice

Always use `data-cy` attributes instead of class or ID selectors:

```html
<!-- Add to your HTML element -->
<button data-cy="submit">Login</button>
```

```javascript
// Target it in Cypress
cy.get('[data-cy="submit"]').click();
```

> ❓ **Why is this better** than using `cy.get('.btn-primary')`? What happens when the UI is restyled?

---

## 4. Login & Logout Tests

### 4a. Login — Happy Path

Create `cypress/e2e/user-can-login.cy.js`:

```javascript
describe('User can login to system', () => {
  it('user can login with valid username and password', () => {
    // Arrange
    cy.visit('/');
    cy.get('h4').should('have.text', 'Login');

    // Act
    cy.get('[data-cy="email"]').type('superadmin@gmail.com');
    cy.get('[data-cy="password"]').type('password');
    cy.get('[data-cy="submit"]').click();

    // Assert
    cy.get('.nav-link > .d-sm-none').should('have.text', 'Hi, SuperAdmin');
  });
});
```

> ❓ **What is the difference** between `.should('have.text', 'Login')` and `.should('contain', 'Login')`?

### 4b. Logout Test

Create `cypress/e2e/user-can-logout.cy.js`:

```javascript
describe('User can logout from system', () => {
  beforeEach(() => {
    cy.visit('/');
    cy.get('[data-cy="email"]').type('superadmin@gmail.com');
    cy.get('[data-cy="password"]').type('password');
    cy.get('[data-cy="submit"]').click();
  });

  it('user can logout from system', () => {
    // Arrange
    cy.get('.nav-link > .d-sm-none').should('have.text', 'Hi, SuperAdmin');

    // Act
    cy.get('#app a.nav-link-user').click();
    cy.get("[data-cy='logout']").click();

    // Assert
    cy.get('h4').should('have.text', 'Login');
  });
});
```

> ❓ **What does `beforeEach()` do here?** Why is this better than repeating the login steps in every `it` block?

---

## 5. Create User Tests

### 5a. Setup with Database Reset

Create `cypress/e2e/user-can-create.cy.js`:

```javascript
describe('User can create new user', () => {
  beforeEach(() => {
    cy.exec('cd ./demo-app-cypress-automation && php artisan migrate:refresh --seed');
    cy.visit('/');
    cy.get('[data-cy="email"]').type('superadmin@gmail.com');
    cy.get('[data-cy="password"]').type('password');
    cy.get('[data-cy="submit"]').click();
  });

  // Test cases go here
});
```

> ❓ **Why do we reset the database in `beforeEach()`?** What could go wrong if we skipped this step?

### 5b. Create User — Happy Path

Add inside the `describe` block:

```javascript
it('user can create new user', () => {
  // Arrange
  cy.get('#app a.icon-left').click();

  // Act
  cy.get('[name="name"]').type('John Doe');
  cy.get('[name="email"]').type('johndoe@gmail.com');
  cy.get('[name="password"]').type('password123');
  cy.get('#app button.btn-primary').click();

  // Assert
  cy.get('#app p').should('be.visible');
  cy.get('#app p').should('have.text', 'Data Added Successfully');
});
```

### 5c. Create User — Invalid Email (Negative Test)

```javascript
it('user cannot create new user with invalid email format', () => {
  // Arrange
  cy.get('#app a.icon-left').click();

  // Act
  cy.get('[name="name"]').type('John Doe');
  cy.get('[name="email"]').type('johndoe');   // missing domain
  cy.get('[name="password"]').type('password123');
  cy.get('#app button.btn-primary').click();

  // Assert
  cy.get('.invalid-feedback').should('be.visible');
  cy.get('.invalid-feedback').should('contain', 'The email must be a valid email address.');
});
```

> ❓ **Check the test case rule**: Every feature needs at least 1 positive and more than 1 negative test. What other negative cases could you add for the Create User form?

---

## 6. Edit User Tests

Create `cypress/e2e/user-can-edit.cy.js` with 4 test cases.

### 6a. Edit User — Happy Path

```javascript
describe('User can edit existing user', () => {
  beforeEach(() => {
    cy.exec('cd ./demo-app-cypress-automation && php artisan migrate:refresh --seed');
    cy.visit('/');
    cy.get('[data-cy="email"]').type('superadmin@gmail.com');
    cy.get('[data-cy="password"]').type('password');
    cy.get('[data-cy="submit"]').click();
  });

  it('user can update name and email', () => {
    // Act
    cy.get('.table td').contains('user').parent().contains('Edit').click();
    cy.get('[name="name"]').type(' updated');
    cy.get('[name="email"]').clear().type('userupdated@gmail.com');
    cy.get('.btn-primary').contains('Submit').click();

    // Assert
    cy.get('.alert')
      .should('be.visible')
      .and('have.class', 'alert-success')
      .contains('User Updated Successfully');
    cy.get('.table td').contains('user').should('have.text', 'user updated');
  });
```

> ❓ **Explain the chaining** in `cy.get('.table td').contains('user').parent().contains('Edit').click()`. What does each part do?

### 6b. Edit User — Negative Tests

Add these 3 tests inside the same `describe` block:

```javascript
  it('user cannot update with empty name field', () => {
    cy.get('.table td').contains('user').parent().contains('Edit').click();
    cy.get('[name="name"]').clear();
    cy.get('.btn-primary').contains('Submit').click();

    cy.get('#app div.invalid-feedback')
      .contains('The name field is required.')
      .should('have.class', 'invalid-feedback')
      .should('be.visible');
  });

  it('user cannot update with empty email field', () => {
    cy.get('.table td').contains('user').parent().contains('Edit').click();
    cy.get('[name="email"]').clear();
    cy.get('.btn-primary').contains('Submit').click();

    cy.get('#app div.invalid-feedback')
      .contains('The email field is required.')
      .should('have.class', 'invalid-feedback')
      .should('be.visible');
  });

  it('user cannot update with both name and email fields empty', () => {
    cy.get('.table td').contains('user').parent().contains('Edit').click();
    cy.get('[name="name"]').clear();
    cy.get('[name="email"]').clear();
    cy.get('.btn-primary').contains('Submit').click();

    cy.get('#name').next().contains('The name field is required.').should('be.visible');
    cy.get('#email').next().contains('The email field is required.').should('be.visible');
  });
});
```

---

## 7. Delete User Tests

Create `cypress/e2e/user-can-delete.cy.js`:

```javascript
describe('User can delete user', () => {
  beforeEach(() => {
    cy.exec('cd ./demo-app-cypress-automation && php artisan migrate:refresh --seed');
    cy.visit('/');
    cy.get('[data-cy="email"]').type('superadmin@gmail.com');
    cy.get('[data-cy="password"]').type('password');
    cy.get('[data-cy="submit"]').click();
  });

  it('user can delete user', () => {
    // Act
    cy.get('.table td').contains('user').parent().contains('Delete').click();
    cy.get('.swal-button-container').find('button').contains('OK').click();

    // Assert
    cy.get('.alert')
      .should('be.visible')
      .and('have.class', 'alert-success')
      .contains('User Deleted Successfully');
    cy.get('.table').should('not.contain', 'user');
  });

  it('user can cancel delete action and data remains', () => {
    // Act
    cy.get('.table td').contains('user').parent().contains('Delete').click();
    cy.get('.swal-button-container').find('button').contains('Cancel').click();

    // Assert
    cy.get('.table').should('contain', 'user').should('be.visible');
  });
});
```

> ❓ **Why is the cancel test important?** What user behavior does it protect?

---

## 8. Running Tests

### 8a. Run All Tests (Headless Mode)

```bash
npm run cy:run
```

### 8b. Run a Single Spec File

```bash
npm run cy:run -- --spec cypress/e2e/user-can-login.cy.js
```

### 8c. Run with Reporter

```bash
npm run cy:run -- --reporter mochawesome
```

This generates individual JSON files per spec inside `cypress/results/`.

### 8d. Open Interactive Mode

```bash
npm run cy:open
```

> ❓ **When would you use headless mode vs interactive mode?** What is the difference in use case for each?

### 8e. Run Only One Test (During Debugging)

Use `.only` to isolate a single test without running the full suite:

```javascript
it.only('user can login with valid username and password', () => {
  // only this test will run
});
```

> ⚠️ Remember to remove `.only` before committing your code.

---

## 9. Generating Reports

After running all tests with the Mochawesome reporter, follow these two steps.

### Step 1 — Merge All Reports

```bash
npm run report:merge
```

This merges all individual JSON files from `cypress/results/` into one `cypress/report.json`.

### Step 2 — Generate HTML Report

```bash
npm run report:generate
```

This produces a visual HTML report with charts, saved at:

```
mochawesome-report/report.html
```

Open `report.html` in your browser to view the full test results with pass/fail status and durations.

> ⚠️ **Common mistake**: Using `merge` instead of `marge` for the generate step causes `sh: merge: command not found`. Always use `marge`.

---

## 📈 Expected Test Results Summary

After completing all spec files, your full run should produce:

| Spec File | Tests | Expected Result |
|-----------|-------|-----------------|
| `user-can-login.cy.js` | 1 | ✅ All Pass |
| `user-can-logout.cy.js` | 1 | ✅ All Pass |
| `user-can-create.cy.js` | 2 | ✅ All Pass |
| `user-can-edit.cy.js` | 4 | ✅ All Pass |
| `user-can-delete.cy.js` | 2 | ✅ All Pass |
| **Total** | **10** | **✅ 10 Pass, 0 Fail** |

---

## 🏆 Challenge Tasks

Once you've completed all the steps above, try these on your own:

1. **Add a negative login test** — attempt to log in with a wrong password and assert the error message that appears.

2. **Add a negative login test** — attempt to log in with an unregistered email and assert the appropriate error message.

3. **Write a test to verify the user list page** loads correctly after login, including asserting that the table heading columns (`Name`, `Email`, `Action`) are visible.

4. **Create a custom Cypress command** in `cypress/support/commands.js` called `cy.login()` that encapsulates the login steps, then refactor all `beforeEach()` blocks to use it.

5. **Enable Cypress Studio** by adding `experimentalStudio: true` to `cypress.config.js`, then use it to record a new test for a feature of your choice instead of writing it manually.

6. **Add screenshot and video attachment** to the Mochawesome report by configuring `cypress/support/e2e.js` with the `Cypress.on('test:after:run', ...)` hook, then regenerate the report and verify the attachments appear.

7. **Add a test for searching/filtering users** — type in a search box and assert that the table only shows matching results. Add both a positive (found) and negative (no results) test case.

---

## ✅ Concepts Covered

| Concept | Where Practiced |
|---|---|
| Installation & Setup | Step 1 |
| cypress.config.js | Step 2a |
| Project Structure | Step 2b |
| AAA Pattern | Step 3a |
| `data-cy` Selectors | Step 3c |
| `cy.visit()` | Step 4a |
| `cy.get()` | Steps 4–7 |
| `cy.type()`, `cy.clear()`, `cy.click()` | Steps 4–7 |
| `.should('have.text', ...)` | Step 4a |
| `.should('contain', ...)` | Step 5c |
| `.should('be.visible')` | Steps 5–7 |
| `.should('not.contain', ...)` | Step 7 |
| `.and('have.class', ...)` | Steps 6a, 7 |
| `beforeEach()` Hook | Steps 4b, 5–7 |
| `cy.exec()` — DB Reset | Steps 5–7 |
| DOM Traversal & Chaining | Step 6a |
| Positive Test Cases | Steps 4a, 5b, 6a, 7 |
| Negative Test Cases | Steps 5c, 6b |
| `it.only()` — Isolation | Step 8e |
| Headless Mode | Step 8a |
| Mochawesome Reporter | Step 8c |
| Report Merge & Generate | Step 9 |
