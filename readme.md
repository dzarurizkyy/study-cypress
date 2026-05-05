# ⚡ Cypress – Quick Reference
A summary guide for End-to-End (E2E) testing with Cypress.

---

## Installation 🔧

**Prerequisites:** Node.js v18+, npm or yarn, modern browser (Chrome/Firefox/Edge/Electron)

```bash
# Initialize project
mkdir my-e2e-project && cd my-e2e-project
npm init -y

# Install Cypress
npm install cypress --save-dev

# Open for first time (auto-generates folder structure)
npx cypress open

# Install Mochawesome reporter (optional)
npm install mochawesome mochawesome-merge mochawesome-report-generator --save
```

**`package.json` scripts:**
```json
{
  "scripts": {
    "cy:run": "cypress run",
    "cy:open": "cypress open",
    "report:merge": "mochawesome-merge cypress/results/*.json -o cypress/report.json",
    "report:generate": "marge cypress/report.json --charts true"
  }
}
```

> ⚠️ Use `marge` (not `merge`) for the `report:generate` script.

---

## Project Structure 🗂️

```
my-e2e-project/
├── cypress/
│   ├── e2e/              # Test spec files (*.cy.js)
│   ├── fixtures/         # Static test data (JSON)
│   ├── support/
│   │   ├── commands.js   # Custom commands
│   │   └── e2e.js        # Global hooks
│   └── results/          # Mochawesome JSON reports
├── cypress.config.js
└── package.json
```

**`cypress.config.js`:**
```javascript
const { defineConfig } = require('cypress');

module.exports = defineConfig({
  viewportWidth: 1920,
  viewportHeight: 1080,
  screenshotOnRunFailure: false,
  video: false,
  e2e: {
    baseUrl: 'http://localhost:8000',
    specPattern: 'cypress/e2e/**/*.cy.{js,jsx,ts,tsx}',
    setupNodeEvents(on, config) {},
  },
});
```

---

## List of Material 📚

* 📘 **Writing Tests**

  Cypress uses **Mocha** syntax (`describe` / `it` blocks) with the **AAA pattern** (Arrange → Act → Assert).

  ```javascript
  describe('Suite Name', () => {
    beforeEach(() => {
      cy.exec("cd ./demo-app && php artisan migrate:refresh --seed");
      cy.visit('/');
    });

    it('user can login with valid credentials', () => {
      // Arrange
      cy.get('h4').should('have.text', 'Login');

      // Act
      cy.get('[data-cy="email"]').type('admin@gmail.com');
      cy.get('[data-cy="password"]').type('password');
      cy.get('[data-cy="submit"]').click();

      // Assert
      cy.get('.nav-link > .d-sm-none').should('have.text', 'Hi, Admin');
    });
  });
  ```

  Lifecycle hooks:

  | Hook | When It Runs |
  |------|--------------|
  | `before()` | Once before all tests |
  | `beforeEach()` | Before every test — ideal for login & DB reset |
  | `after()` | Once after all tests |
  | `afterEach()` | After every test — ideal for cleanup |

* 📗 **Selecting Elements**

  Always prefer `data-cy` attributes over fragile class/ID selectors.

  ```html
  <!-- HTML -->
  <button data-cy="submit">Login</button>
  ```

  ```javascript
  // Cypress
  cy.get('[data-cy="submit"]').click();
  ```

  Common selector commands:

  | Command | Description |
  |---------|-------------|
  | `cy.get('selector')` | Select by CSS selector |
  | `cy.contains('text')` | Select by visible text |
  | `.find('child')` | Find child within element |
  | `.parent()` | Get parent element |
  | `.next()` | Get next sibling |

  Chaining example — find a row by name and click its Delete button:
  ```javascript
  cy.get('.table td').contains('user').parent().contains('Delete').click();
  ```

* 📙 **Assertions**

  Cypress uses **Chai** assertions via `.should()`:

  | Assertion | Description |
  |-----------|-------------|
  | `.should('be.visible')` | Element is visible |
  | `.should('have.text', 'val')` | Exact text match |
  | `.should('contain', 'val')` | Partial text match |
  | `.should('have.class', 'cls')` | Has CSS class |
  | `.should('not.contain', 'val')` | Does not contain text |

  ```javascript
  // Success message
  cy.get('#app p').should('be.visible').should('have.text', 'Data Added Successfully');

  // Validation error
  cy.get('.invalid-feedback').should('be.visible').should('contain', 'The email must be a valid email address.');

  // Alert with class check
  cy.get('.alert').should('be.visible').and('have.class', 'alert-success').contains('User Deleted Successfully');

  // Record removed from table
  cy.get('.table').should('not.contain', 'deleted-user');
  ```

* 📕 **User Interactions**

  | Command | Description |
  |---------|-------------|
  | `.click()` | Click an element |
  | `.type('text')` | Type into an input |
  | `.clear()` | Clear an input |
  | `cy.visit('url')` | Navigate to URL |

  Use `.only` to isolate a single test during debugging:
  ```javascript
  it.only('user can login', () => { /* only this runs */ });
  ```

* 📓 **Running Tests**

  ```bash
  # Headless (CI/CD)
  npm run cy:run

  # Run a single spec
  npm run cy:run -- --spec cypress/e2e/user-can-login.cy.js

  # With Mochawesome reporter
  npm run cy:run -- --reporter mochawesome

  # Interactive (local development)
  npm run cy:open
  ```

* 📒 **Generating Reports**

  ```bash
  # Step 1 — Merge individual JSON reports
  npm run report:merge

  # Step 2 — Generate HTML report with charts
  npm run report:generate
  ```

  The final report is saved at `mochawesome-report/report.html`.

  To embed screenshots on failure, add to `cypress/support/e2e.js`:
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
---

## 📍 References
- [Udemy](https://www.udemy.com/course/quality-assurance-engineer-cypress-dari-awal-sampai-mahir)

## 👨‍💻 Contributors
- [Dzaru Rizky Fathan Fortuna](https://www.linkedin.com/in/dzarurizky)
