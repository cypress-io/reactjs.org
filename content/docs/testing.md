---
id: testing
title: Testing Overview
permalink: docs/testing.html
redirect_from:
  - "community/testing.html"
next: testing-recipes.html
---

You can test React components similar to testing other JavaScript code.

There are a few ways to test React components. Broadly, they divide into two categories:

- **Rendering component trees** in a simplified test environment and asserting on their output.
- **Running a complete app** in a real web browser (also known as [end-to-end testing](/docs/end-to-end-testing.html)).

This documentation focuses on both [unit testing strategies](/docs/testing-recipes.html) (the first category) and [end-to-end testing](/docs/end-to-end-testing.html).

### Tradeoffs {#tradeoffs}

When choosing testing tools, it is worth considering a few tradeoffs:

- **Iteration speed vs Realistic environment:** Some tools offer a very quick feedback loop between making a change and seeing the result, but don't model the browser behavior precisely. Other tools might use a real browser environment, but reduce the iteration speed and are flakier on a continuous integration server.
- **How much to mock:** With components, the distinction between a "unit" and "integration" test can be blurry. If you're testing a form, should its test also test the buttons inside of it? Or should a button component have its own test suite? Should refactoring a button ever break the form test?

Different answers may work for different teams and products.

### Recommended Tools {#tools}

**[Jest](https://facebook.github.io/jest/)** is a JavaScript test runner that lets you access the DOM via [`jsdom`](/docs/testing-environments.html#mocking-a-rendering-surface). While jsdom is only an approximation of how the browser works, it is often good enough for testing React components. Jest provides a great iteration speed combined with powerful features like mocking [modules](/docs/testing-environments.html#mocking-modules) and [timers](/docs/testing-environments.html#mocking-timers) so you can have more control over how the code executes.

**[React Testing Library](https://testing-library.com/react)** is a set of helpers that let you test React components without relying on their implementation details. This approach makes refactoring a breeze and also nudges you towards best practices for accessibility. Although it doesn't provide a way to "shallowly" render a component without its children, a test runner like Jest lets you do this by [mocking](/docs/testing-recipes.html#mocking-modules).

**[Cypress](https://www.cypress.io/)** is a tool for writing, running, and debugging end-to-end tests in real web browsers. Read more about end-to-end testing React applications on the [End-to-end Testing](/docs/end-to-end-testing.html) guide.

### Learn More {#learn-more}

This section is divided into multiple guides:

- [Recipes](/docs/testing-recipes.html): Common patterns when writing tests for React components.
- [Environments](/docs/testing-environments.html): What to consider when setting up a testing environment for React components.
- [End-to-end Testing](/docs/end-to-end-testing.html): Learn about the benefits of end-to-end tests and see an example of end-to-end testing against the [Thinking In React](/docs/thinking-in-react.html) app.
