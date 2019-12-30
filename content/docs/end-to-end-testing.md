---
id: end-to-end-testing
title: End-to-End Testing
permalink: docs/end-to-end-testing.html
redirect_from:
  - "community/end-to-end-testing.html"
prev: testing-environments.html
---


// TODO: re-write intro

Modern web applications have evolved from stateless to state-full applications; obtaining data from APIs directly in addition to managing a client-side data store (Redux, MobX, etc) where parts of state maybe serialized locally (localStorage, IndexedDB, WebSQL, etc.). This can lead to a great deal of complexity.

End-to-end testing is a methodology in which the entire web application and infrastructure is tested through a _real_ browser against _real_ user actions.

They can make the applications you build more resilient to changes made throughout the application stack and compliment other forms of testing.

A single end-to-end test can offer assurance of layers of complexity for components, state and packaging on the frontend. The same test can provide assurance of backend layers, as well.

## Testing _Thinking In React_ App

To demonstrate end-to-end testing, we will test the simple, but well-known [Thinking In React](https://reactjs.org/docs/thinking-in-react.html) application with [Cypress](https://cypress.io), an open source _(MIT License)_ browser testing tool.

> Note:
>
> All the examples within this guide are available in the [Thinking In React with Cypress repository](https://github.com/cypress-io/thinking-in-react-with-cypress).
>
>If you would like to try the concepts within this guide against a new or different project, refer to the [Cypress installation docs](https://on.cypress.io/installing-cypress). 
>If you prefer to start projects with [Create React App (CRA)](https://create-react-app.dev/), you can use the [Cypress CRA template](https://github.com/cypress-io/cra-template-cypress), which is also [available in typescript](https://github.com/cypress-io/cra-template-cypress-typescript).

### Testing Approach

One approach to composing valuable E2E tests is to simulate the realistic journey of a user through relevant paths within an application to validate their full experience. 

As shown below, the Thinking in React UI provides the following functionality:
  - Presents the user with a categorized table of products with their associated price.
  - Flags out-of-stock products in red.
  - Allows the user to filter products via a search input box.
  - Allows the user to hide out-of-stock products via a checkbox toggle.

![Thinking in React App](../images/blog/thinking-in-react-mock.png)

To validate this functionality we will need to select DOM elements from the UI to assert their state or to interact with them. Let's first discuss element selection.


### Selecting Elements

A DOM element needs a specific identifier attribute to be selected for testing. It is common to rely on an element's `class` or `id` attribute for selection, however these attributes are prone to change as CSS style or implementation of the UI are refactored in a growing codebase. To mitigate this issue, one [recommended practice](https://docs.cypress.io/guides/references/best-practices.html#Selecting-Elements) is to add a dedicated `data-test` attribute to desired elements for the sole purpose of testing. 

For example, we can add a `data-test="product-table"` attribute to the `ProductTable` component for convenient selection of its DOM element for testing:

```javascript
class ProductTable extends Component {
  render() {
    const rows = [];
    // ...

    return (
      <table data-test="product-table">
        <thead>
          <tr>
            <th>Name</th>
            <th>Price</th>
          </tr>
        </thead>
        <tbody>{rows}</tbody>
      </table>
    );
  }
}
```

### Writing Tests

[Cypress](https://cypress.io), an open source (MIT License) browser testing tool, will be used for all the following examples which are available in the [Thinking In React with Cypress](https://github.com/cypress-io/thinking-in-react-with-cypress) repository.

Installation and setup of Cypress for your project is [described in detail in the Cypress ](https://on.cypress.io/installing-cypress).

Here we test that our complete list of products renders correctly:

```javascript
describe("Thinking In React", function() {
  it("renders complete list of products", function() {
    // Visit the application
    cy.visit("http://localhost:3000");
    // Select and assert that the product table is rendered
    cy.get("[data-test='product-row']").should("have.length", 6);
  });
});
```

This simple test relies on 3 Cypress commands:

- First we [visit](https://on.cypress.io/visit) the web application
- Next we [get](https://on.cypress.io/get) the product row using it's `data-test` selector
- Finally, we [assert (via should)](https://on.cypress.io/should) the length of products in the table matches our data

### Testing Interactivity

Adding a `data-test="search"` allows us to get to that element like the product row:

```javascript
class SearchBar extends Component {
  // ..
  render() {
    return (
      <form>
        <input
          data-test="search"
          //...
        />
    )
  }
}
```

When we perform a search, we can verify that the correct product rows are listed in the product table using the same assertion.

```javascript
it("searches for a product", function() {
  // Visit the application
  cy.visit("http://localhost:3000");
  // Select the search box and type the search term
  cy.get("[data-test='search']").type("Foot");
  // Select and assert that the product table is rendered with the proper search results
  cy.get("[data-test='product-row']").should("have.length", 1);
});
```

With the exception of adding the search term via [type](https://on.cypress.io/type), we performed the same steps from the previous example.

Using the [time travel feature](https://docs.cypress.io/guides/getting-started/writing-your-first-test.html#Time-travel), we are able to see and step through the state of our application before and after the search.

### Awaiting State

A core feature of Cypress that assists with testing dynamic web applications is [retry-ability](https://on.cypress.io/retry-ability).

Given the following test, we can be assured of two things:

```javascript
it("displays products in stock", function() {
  cy.get("form input[type=checkbox]").check();
  // Cypress automatically waits
  cy.get("[data-test='product-row']").should("have.length", 4);
});
```

- If the assertion that follows the `cy.get()` command passes, then the command finishes successfully.
- If the assertion that follows the `cy.get()` command fails, then the `cy.get()` command will re-query the application’s DOM again. Then Cypress will try the assertion against the elements yielded from `cy.get()`. If the assertion still fails, `cy.get()` will try re-querying the DOM again, and so on until the `cy.get()` [command timeout](https://on.cypress.io/retry-ability#Timeouts) is reached.

The retry-ability allows the tests to complete each command as soon as the assertion passes, without hard-coding waits.

### Style Testing

Consider the follow test to validate the CSS color value for an out of stock product row, `red`.

```javascript
it("renders an out of stock product", function() {
  cy.visit("http://localhost:3000");
  cy.contains("[data-test='product-row'] span", "Basketball")
    .should("be.visible")
    .and("have.css", "color", "rgb(255, 0, 0)");
});
```

If we update our out of stock product row CSS to render the color `green`, our test will fail, as the `rgb` value would not match our assertion.

While we've technically written a functional test asserting the CSS property `color`, this style of testing may quickly become cumbersome to write and maintain, especially when visual styles rely on a lot of CSS styles.

Your visual styles may also rely on more than CSS, perhaps you want to ensure an SVG or image has rendered correctly or shapes were correctly drawn to a canvas.

If your end-to-end tests become full of assertions checking visibility, color and other style properties, it might be time to start using visual diffing to verify the page appearance.


### Visual Testing
Out of the scope of this overview, but an important topic in the end-to-end testing world, is the concept of Visual Testing. Visual testing automates the detection of visual changes to the DOM layout, fonts, colors or other visual properties.

Luckily, Cypress provides a stable platform for [writing plugins](https://docs.cypress.io/plugins) that _can perform visual testing_.

Typically [such plugins](https://github.com/meinaart/cypress-plugin-snapshots) take an image snapshot of the entire application under test or a specific element, and then compare the image to a previously approved baseline image. If the images are the same (within a set pixel tolerance), it is determined that the web application looks the same to the user. If there are differences, then there has been some change to the DOM layout, fonts, colors or other visual properties that needs to be investigated.

### Complete application test suite and refactors

Our application test suite is starting to take shape.

There are two refactors we can make to take.

First, we'll refactor our `visit` commands from each test block into a `beforeEach`

```javascript
describe("Thinking In React", function() {
  // Execute visit command before each test block
  beforeEach(function() {
    cy.visit("http://localhost:3000");
  });

  it("renders complete list of products", function() {
    cy.get("[data-test='product-row']").should("have.length", 6);
  });

  it("searches for a product", function() {
    cy.get("[data-test='search']").type("Foot");
    cy.get("[data-test='product-row']").should("have.length", 1);
  });

  it("renders an out of stock product", function() {
    cy.contains("[data-test='product-row'] span", "Basketball")
      .should("be.visible")
      .and("have.css", "color", "rgb(255, 0, 0)");
  });
});
```

Finally, we'll add `fixture` for our data and update our assertion to check the length of the `this.PRODUCTS` variable:

```javascript
describe("Thinking In React", function() {
  beforeEach(function() {
    // Load cypress/fixtures/products.json into the PRODUCTS variable
    cy.fixture("products").as("PRODUCTS"); // this.PRODUCTS
    cy.visit("http://localhost:3000");
  });

  it("renders complete list of products", function() {
    // Should expectation value to PRODUCTS.length
    cy.get("[data-test='product-row']").should(
      "have.length",
      this.PRODUCTS.length
    );
  });
  // ...
});
```

## Testing Application State

Testing centralized application state from [Redux](https://redux.js.org), [MobX](https://mobx.js.org/) or [React Hooks useReducer](https://reactjs.org/docs/hooks-reference.html#usereducer) is a common scenario in Modern Web Applications.

In such scenarios we construct and end-to-end test to interact with our application directly and wish do one of the following:

- Preload internal state of the application
- Assert internal state of the application after interactions

**Note:** Reference the [useReducer branch in the Thinking In React with Cypress](https://github.com/cypress-io/thinking-in-react-with-cypress/tree/usereducer) repository for the examples in this section.

Let's modify the application to use a reducer to manage our state.

Moving our state into this structure is straightforward:

```js
import React, { useReducer } from "react";

const PRODUCTS = [ ... ];

// Create an initialState
const initialState = (window.Cypress && window.initialState) || {
  products: PRODUCTS
};

// Create a reducer method
const reducer = (state, action) => {
  switch (action.type) {
    default:
      return state;
  }
};

const App = () => {
  const [state, dispatch] = useReducer(reducer, initialState);
  if (window.Cypress) {
    // If we are under test in Cypress set our state and dispatch method on the window object
    window.__state__ = state;
    window.__dispatch__ = dispatch;
  }

  return (
    <div>
      {state.testing && <div>test worked</div>}
      <FilterableProductTable products={state.products} />;
    </div>
  );
};

export default App;
```

Exposing `state` and `dispatch` on the `window` object allows us to interact with them in our end-to-end tests.

We can write a simple test to assert that our application state is available under our test environment.

```js
it("has expected state on load", () => {
  cy.window()
    .its("__state__")
    .should("have.property", "products");
});
```

Should we want to pre-load our state prior to an end-to-end test, we can pass an [`onBeforeLoad` option to visit()](https://on.cypress.io/visit#Arguments) with `initialState` set on the `window` object:

```js
it("has custom initial state on load", () => {
  cy.visit("/", {
    onBeforeLoad: win => {
      win.initialState = {
        products: [
          {
            category: "Sporting Goods",
            price: "$49.99",
            stocked: true,
            name: "Football"
          },
          {
            category: "Sporting Goods",
            price: "$9.99",
            stocked: true,
            name: "Baseball"
          }
        ]
      };
    }
  });

  cy.get("[data-test='product-row']").should("have.length", 2);
});
```

Let's modify our reducer to include an action:

```js
const reducer = (state, action) => {
  switch (action.type) {
    case "TEST": {
      return {
        ...state,
        testing: true
      };
    }
    default:
      return state;
  }
};
```

And our return to include a visual message if the `testing` attribute is set:

```js
const App = () => {
  // ...
  return (
    <div>
      {testing && <div>test worked</div>}
      <FilterableProductTable products={products} />;
    </div>
  );
};
```

Let's dispatch an action and test the DOM and application state change once an action is dispatched:

```js
it("can dispatch actions", () => {
  cy.window()
    .invoke("__dispatch__", { type: "TEST" })
    .then(() => {
      // DOM Test
      cy.contains("test worked");

      // Application State
      cy.window()
        .its("__state__")
        .should("contain", { testing: true });
    });
});
```
