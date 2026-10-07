# Study Cypress ⚡

This repository contains a comprehensive reference guide for Cypress — covering installation, project structure, writing tests, commands, assertions, running tests, and reporting, worked through hands-on against a Laravel demo app.

## Installation 🔧

1. **Install Node.js** (v18+):

   ```bash
   node -v
   npm -v
   ```

   > Download the **LTS** version from [nodejs.org](https://nodejs.org/)

2. **Create the Project and Install Cypress**:

   ```bash
   mkdir my-e2e-project && cd my-e2e-project
   npm init -y
   npm install cypress --save-dev
   ```

3. **Verify the Installation** (auto-generates the folder structure on first run):

   ```bash
   npx cypress open
   ```

4. **Install the Mochawesome Reporter** (optional):

   ```bash
   npm install mochawesome mochawesome-merge mochawesome-report-generator --save
   ```

   > Reference: [docs.cypress.io/app/get-started/install-cypress](https://docs.cypress.io/app/get-started/install-cypress)

## List of Material 📚

- ⚡ **[Cypress E2E Testing](001-cypress-basics.md)**

  Project setup and demo app, project structure, writing tests with the AAA pattern, hooks, commands and selectors, assertions, running tests, and generating reports:

  ```javascript
  describe("User can login", () => {
    beforeEach(() => {
      cy.exec("cd ./demo-app && php artisan migrate:refresh --seed");
      cy.visit("/");
    });

    it("user can login with valid credentials", () => {
      // Arrange
      cy.get("h4").should("have.text", "Login");

      // Act
      cy.get('[data-cy="email"]').type("admin@gmail.com");
      cy.get('[data-cy="password"]').type("password");
      cy.get('[data-cy="submit"]').click();

      // Assert
      cy.get(".nav-link > .d-sm-none").should("have.text", "Hi, Admin");
    });
  });
  ```

  Run the test:

  ```bash
  npm run cy:run -- --reporter mochawesome
  ```

## 📍 References

- [Udemy](https://www.udemy.com/course/quality-assurance-engineer-cypress-dari-awal-sampai-mahir)

## 👨‍💻 Contributors

- [Dzaru Rizky Fathan Fortuna](https://www.linkedin.com/in/dzarurizky)
