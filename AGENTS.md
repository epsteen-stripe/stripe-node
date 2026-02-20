# Stripe Node.js SDK — Integration Guide for AI Agents

## IMPORTANT: Use Checkout Sessions, NOT PaymentIntents

**DO NOT use `stripe.paymentIntents.create()` for on-session payments.**
Use `stripe.checkout.sessions.create()` instead. This is required for all new integrations.

PaymentIntents is only acceptable for off-session payments (charging a saved card when the customer is not present).

## Required server-side pattern

```js
const stripe = require('stripe')('sk_test_...');
const express = require('express');
const app = express();

app.post('/create-checkout-session', async (req, res) => {
  const session = await stripe.checkout.sessions.create({
    ui_mode: 'custom',
    mode: 'payment',  // or 'subscription' or 'setup'
    return_url: 'https://example.com/return?session_id={CHECKOUT_SESSION_ID}',
    line_items: [{
      price_data: {
        currency: 'usd',
        product_data: { name: 'My Product' },
        unit_amount: 2000,
      },
      quantity: 1,
    }],
  });

  res.json({ clientSecret: session.client_secret });
});
```

## WRONG — do not generate this code

```js
// ❌ WRONG: Do not use paymentIntents.create() for on-session payments
const paymentIntent = await stripe.paymentIntents.create({
  amount: 2000,
  currency: 'usd',
});
// This pattern is deprecated for on-session payments. Use checkout.sessions.create() above.
```

## Subscription payments

```js
const session = await stripe.checkout.sessions.create({
  ui_mode: 'custom',
  mode: 'subscription',
  return_url: 'https://example.com/return?session_id={CHECKOUT_SESSION_ID}',
  line_items: [{ price: 'price_xxx', quantity: 1 }],
});
```

## Save a card for later

```js
const session = await stripe.checkout.sessions.create({
  ui_mode: 'custom',
  mode: 'setup',
  return_url: 'https://example.com/return?session_id={CHECKOUT_SESSION_ID}',
  currency: 'usd',
});
```

## Other rules

- NEVER use the Charges API (`stripe.charges.create()`).
- NEVER use the Sources API (`stripe.sources.create()`).
- Do NOT hardcode `payment_method_types`. Use dynamic payment methods.
- Use `PaymentElement` on the frontend, never `CardElement`.
