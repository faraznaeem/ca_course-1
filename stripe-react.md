Create
`$ touch cypress/integration/userCanMakePayment.feature.js`

Add

```js
describe("User can add a product to his/her order", () => {
  beforeEach(() => {
    cy.server();
    cy.route({
      method: "GET",
      url: "http://localhost:3000/api/products",
      response: "fixture:product_data.json"
    });

    cy.route({
      method: "POST",
      url: "http://localhost:3000/api/orders",
      response: "fixture:post_response.json"
    });

    cy.route({
      method: "PUT",
      url: "http://localhost:3000/api/orders/1",
      response: "fixture:put_response.json"
    });

    cy.visit("http://localhost:3001");
    cy.get("#product-2").within(() => {
      cy.get("button")
        .contains("Add to order")
        .click();
    });
    cy.get("#product-3").within(() => {
      cy.get("button")
        .contains("Add to order")
        .click();
    });
    cy.get("button")
      .contains("View order")
      .click();
  });

  it("user can pay for his order", () => {
    cy.get("button")
      .contains("Confirm!")
      .click();
    cy.get("#payment-form").should("exist");
  });
});
```

Add a new state

To the component

```js
state = {
  productData: [],
  message: {},
  orderDetails: {},
  showOrder: false,
  orderTotal: "",
  showPaymentform: false
};
```

refactor the button

and import the <PaymentForm/> on the top of the file.

```js
<button onClick={() => this.setState({ showPaymentform: true })}>
  Confirm!
</button>;
{
  this.state.showPaymentform && (
    <div id="payment-form">
      <PaymentForm />
    </div>
  );
}
```

```bash
$ touch src/components/PaymentForm.jsx
```

```js
import React, { Component } from "react";

class PaymentForm extends Component {
  render() {
    return (
      <div>
        <h1>Here we will show a payment form</h1>
      </div>
    );
  }
}

export default PaymentForm;
```

add test for iframe

```js
cy.wait(1000);
cy.get('iframe[name^="__privateStripeFrame5"]').then($iframe => {
  const $body = $iframe.contents().find("body");
  cy.wrap($body)
    .find('input[name="cardnumber"]')
    .type("4242424242424242", { delay: 50 });
});
```

Add the package

```bash
$ yarn add react-stripe-elements
```

Add a script to the head in index.html

```html
<script src="https://js.stripe.com/v3/"></script>
```

import to `index.js`

```js
import { StripeProvider } from "react-stripe-elements";
```

wrap <App> component

```js
ReactDOM.render(
  <StripeProvider>
    <App />
  </StripeProvider>,
  document.getElementById("root")
);
```

Get your API key from stripe dashboard

Add API key to stripe provider.

```js
ReactDOM.render(
  <StripeProvider apiKey="put_key_here">
    <App />
  </StripeProvider>,
  document.getElementById("root")
);
```

Import to Payment form

```js
import { injectStripe, CardNumberElement } from "react-stripe-elements";
```

```js
{
  this.state.showPaymentform && (
    <div id="payment-form">
      <Elements>
        <PaymentForm />
      </Elements>
    </div>
  );
}
```
update your payment form

```js
import React, { Component } from "react";
import { injectStripe, CardNumberElement } from "react-stripe-elements";

class PaymentForm extends Component {
  render() {
    return (
      <>
        <label>Card Number</label>
        <CardNumberElement />
      </>
    );
  }
}

export default injectStripe(PaymentForm);
```
