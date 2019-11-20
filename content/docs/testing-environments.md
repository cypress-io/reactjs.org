---
id: testing-environments
title: Testing Environments
permalink: docs/testing-environments.html
prev: testing-recipes.html
next: end-to-end-testing.html
---

<!-- This document is intended for folks who are comfortable with JavaScript, and have probably written tests with it. It acts as a reference for the differences in testing environments for React components, and how those differences affect the tests that they write. This document also assumes a slant towards web-based react-dom components, but has notes for other renderers. -->

This document goes through the factors that can affect your environment and recommendations for some scenarios.

### Test runners {#test-runners}

Test runners like [Jest](https://jestjs.io/), [mocha](https://mochajs.org/), [ava](https://github.com/avajs/ava), [Cypress](https://www.cypress.io/) and [puppeteer](https://github.com/GoogleChrome/puppeteer) let you write test suites as regular JavaScript, and run them as part of your development process. Additionally, test suites are run as part of continuous integration.

Test runners may be broken down into two categories: unit and end-to-end.

#### Unit Test Runners

- [Jest](https://jestjs.io/)
- [mocha](https://mochajs.org/)
- [ava](https://github.com/avajs/ava)

#### End-to-end Test Runners

- [Cypress](https://www.cypress.io/)
- [puppeteer](https://github.com/GoogleChrome/puppeteer)

### Which Test Runner Should I Use?

- Jest is widely compatible with React projects, supporting features like mocked [modules](#mocking-modules) and [timers](#mocking-timers), and [`jsdom`](#mocking-a-rendering-surface) support. **If you use Create React App, [Jest is already included out of the box](https://facebook.github.io/create-react-app/docs/running-tests) with useful defaults.**
- Libraries like [mocha](https://mochajs.org/#running-mocha-in-the-browser), maybe used with [karma](https://karma-runner.github.io/) to run unit tests "on real browsers and real devices such as phones, tablets or on a headless PhantomJS instance", which may be incompatible against `jsdom`.
- [End-to-end testing](#end-to-end-tests-aka-e2e-tests) is a methodology in which all of the layers of an application are tested within a _real_ web browser via _real_ user actions. They can be implemented using test runners like [Cypress](https://www.cypress.io/) or a library like [puppeteer](https://github.com/GoogleChrome/puppeteer). Read more about end-to-end testing on the [End-to-end Testing](/docs/end-to-end-testing.html) page.

### Mocking a rendering surface {#mocking-a-rendering-surface}

Tests often run in an environment without access to a real rendering surface like a browser. For these environments, we recommend simulating a browser with [`jsdom`](https://github.com/jsdom/jsdom), a lightweight browser implementation that runs inside Node.js.

In most cases, jsdom behaves like a regular browser would, but doesn't have features like [layout and navigation](https://github.com/jsdom/jsdom#unimplemented-parts-of-the-web-platform). This is still useful for most web-based component tests, since it runs quicker than having to start up a browser for each test. It also runs in the same process as your tests, so you can write code to examine and assert on the rendered DOM.

Just like in a real browser, jsdom lets us model user interactions; tests can dispatch events on DOM nodes, and then observe and assert on the side effects of these actions [<small>(example)</small>](/docs/testing-recipes.html#events).

A large portion of UI tests can be written with the above setup: using Jest as a test runner, rendered to jsdom, with user interactions specified as sequences of browser events, powered by the `act()` helper [<small>(example)</small>](/docs/testing-recipes.html). For example, a lot of React's own tests are written with this combination.

If you're writing a library that tests mostly browser-specific behavior, and requires native browser behavior like layout or real inputs, you could use a framework like [mocha.](https://mochajs.org/)

In an environment where you _can't_ simulate a DOM (e.g. testing React Native components on Node.js), you could use [event simulation helpers](https://reactjs.org/docs/test-utils.html#simulate) to simulate interactions with elements. Alternately, you could use the `fireEvent` helper from [`@testing-library/react-native`](https://testing-library.com/docs/native-testing-library).

Frameworks like [Cypress](https://www.cypress.io/), [puppeteer](https://github.com/GoogleChrome/puppeteer) and [webdriver](https://www.seleniumhq.org/projects/webdriver/) are useful for running [end-to-end tests](#end-to-end-tests-aka-e2e-tests).

### Mocking functions {#mocking-functions}

When writing tests, we'd like to mock out the parts of our code that don't have equivalents inside our testing environment (e.g. checking `navigator.onLine` status inside Node.js). Tests could also spy on some functions, and observe how other parts of the test interact with them. It is then useful to be able to selectively mock these functions with test-friendly versions.

This is especially useful for data fetching. It is usually preferable to use "fake" data for tests to avoid the slowness and flakiness due to fetching from real API endpoints [<small>(example)</small>](/docs/testing-recipes.html#data-fetching). This helps make the tests predictable. Libraries like [Jest](https://jestjs.io/) and [sinon](https://sinonjs.org/), among others, support mocked functions. For end-to-end tests, mocking network can be more difficult, but you might also want to test the real API endpoints in them anyway.

### Mocking modules {#mocking-modules}

Some components have dependencies for modules that may not work well in test environments, or aren't essential to our tests. It can be useful to selectively mock these modules out with suitable replacements [<small>(example)</small>](/docs/testing-recipes.html#mocking-modules).

On Node.js, runners like Jest [support mocking modules](https://jestjs.io/docs/en/manual-mocks). You could also use libraries like [`mock-require`](https://www.npmjs.com/package/mock-require).

### Mocking timers {#mocking-timers}

Components might be using time-based functions like `setTimeout`, `setInterval`, or `Date.now`. In testing environments, it can be helpful to mock these functions out with replacements that let you manually "advance" time. This is great for making sure your tests run fast! Tests that are dependent on timers would still resolve in order, but quicker [<small>(example)</small>](/docs/testing-recipes.html#timers). Most frameworks, including [Jest](https://jestjs.io/docs/en/timer-mocks), [sinon](https://sinonjs.org/releases/v7.3.2/fake-timers/) and [lolex](https://github.com/sinonjs/lolex), let you mock timers in your tests.

Sometimes, you may not want to mock timers. For example, maybe you're testing an animation, or interacting with an endpoint that's sensitive to timing (like an API rate limiter). Libraries with timer mocks let you enable and disable them on a per test/suite basis, so you can explicitly choose how these tests would run.

### End-to-end tests {#end-to-end-tests-aka-e2e-tests}

End-to-end tests can make the applications you build more resilient to changes made throughout the application stack and compliment other forms of testing.

They can test longer workflows including critical business paths, such as payments or signups.

End-to-end tests allow you to assert:

- Real browser rendering of the entire app
- Data fetching against real API endpoints
- The use of sessions and cookies
- Navigation between multiple routes
- Side effects within the web browser or backend

In this scenario, you would use a test runner like [Cypress](https://www.cypress.io/) or a library like [puppeteer](https://github.com/GoogleChrome/puppeteer).

Read more about end-to-end tests on the [End-to-end Testing](/docs/end-to-end-testing.html) page.
