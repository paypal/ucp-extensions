# PayPal Wallet Payment Handler Specification

- **Handler Name:** `com.paypal.wallet`
- **Version:** `2026-10-08`

## 1. Introduction

The `com.paypal.wallet` handler enables businesses that accept PayPal as a payment method to convey they can accept PayPal when a customer buys an offering from the merchant through a UCP-compatible platform.

This handler enables a headless integration model where businesses provide enough configuration in their PayPal wallet payment handlers for platforms to render the PayPal button on behalf of the merchant.

## 2. Business Integration

### 2.1 Requirements

Before publicly exposing their UCP profiles, businesses must complete the following steps:

1. Retrieve Your PayPal Merchant ID

Your PayPal Merchant ID is the payee identifier used throughout this guide — it is the value you advertise as config.paypal_merchant_identifier, and it is the merchant the approval token is scoped to and settles to.

To find it:

Log in to your PayPal business account and open your Business Profile: https://www.paypal.com/unifiedsettings/unified-profile/business

Your Merchant ID is listed in the Business Profile as ID #.

Example: ID #L92QX7RBHF63K

Use this value (without the ID # prefix) wherever this guide references your PayPal Merchant ID.

### 2.2 Integration Overview

When a buyer pays with PayPal through an agent, the agent forwards a PayPal Approval Token into your checkout session on `/complete`. Your responsibility is to:

Advertise the `com.paypal.wallet` handler so agents know PayPal is accepted, and share your PayPal Merchant ID (payee).

Accept the approval token on `/complete` and hand it to your PSP.

Your PSP redeems the token via PayPal Orders v2 (`payment_source.token`, type `APPROVAL`) to capture.

### 2.3 Handler Configuration

Advertise the handler in your UCP business config under `ucp.payment_handlers`, keyed by `com.paypal.wallet`. The critical config value is your PayPal Merchant ID (payee) — it scopes the approval token to you.

#### 2.3.1. Example Handler Configuration

```json
{
  "ucp": {
    "version": "2026-08-25",
    "payment_handlers": {
      "com.paypal.wallet": [
        {
          "id": "paypal",
          "version": "2026-09-02",
          "spec": "<url>/<this file>",
          "schema": "https://developer.paypal.com/ucp/paypal-wallet/schema.json",
          "config": {
            "environment": "production",
            "paypal_merchant_identifier": "Q7C9N6S4YRQXG"
          }
        }
      ]
    }
  }
}
```

#### 2.3.2 Handler Configuration Fields

| Field                      | Required | Description               |
| -------------------------- | -------- | ------------------------- |
| environment                | yes      | `production` or `sandbox` |
| paypal_merchant_identifier | Yes      | Your PayPal Merchant ID   |

Today: PayPal does not yet publish a systemic handler-discovery mechanism, so in early integrations you may share your merchant ID with the agent offline (or via the standard PayPal partner mechanism — merchant-id SDK param / PayPal-Auth-Assertion). The payment_handlers declaration above is the target state.

### 3. Accepting and processing the approval token on complete

#### 3.1 Receive the token on /complete

The agent sends the approval token as a UCP payment instrument on POST `/checkout-sessions/{id}/complete`:

```JSON
{
  "payment": {
    "instruments": [
      {
        "id": "pi_paypal_1",
        "handler_id": "paypal",
        "type": "paypal_wallet",
        "selected": true,
        "credential": {
          "type": "approval_token",
          "token": "1AB23456CD789012E"
        }
      }
    ]
  }
}
```

##### Processing steps

Validate — confirm `handler_id` ("paypal") matches your configured `com.paypal.wallet` handler and `credential.type` is `approval_token`.

Extract — read the approval token from `credential.token`.

Redeem — create and capture or authorize an order using the v2 Orders API.

Return — respond with the finalized checkout `state` and `order_id`.

#### 3.2 Redeem the token via Orders v2

You create an order with PayPal Orders v2, passing the approval token as `payment_source.token` with type `APPROVAL`.

`POST /v2/checkout/orders`

##### Headers

```
Authorization: Bearer <access-token>
Content-Type: application/json
Prefer: return=representation
```

Orders v2 defaults to return=minimal (order id + status only). Send Prefer: return=representation to get the capture id back inline; otherwise do a follow-up `GET /v2/checkout/orders/{id}`.

##### Request — intent: CAPTURE (authorize + capture in one call; use intent: AUTHORIZE to capture later)

```JSON
{
  "intent": "CAPTURE",
  "purchase_units": [
    { "amount": { "currency_code": "USD", "value": "89.99" } }
  ],
  "payment_source": {
    "token": {
      "id": "1AB23456CD789012E",
      "type": "APPROVAL"
    }
  }
}
```

##### Response — 201 Created. Returns the order id (top-level id) and the capture id (nested):

```JSON
{
  "id": "5O190127TN364715T",
  "status": "COMPLETED",
  "purchase_units": [
    {
      "payments": {
        "captures": [
          { "id": "3C679366HH908993F", "status": "COMPLETED" }
        ]
      }
    }
  ]
}
```

- Capture id: `purchase_units[0].payments.captures[0].id`.
- With intent: `AUTHORIZE`, you instead get an authorization id at `purchase_units[0].payments.authorizations[0].id`.

Once redeemed, the approval token moves to `COMPLETED` (consumed). The order carries `payment_source.paypal` with the payer's `account_id`, `name`, and `email_address`.

### 3.3 Redemption guardrails

Because an approval token may be redeemed with no buyer present, Orders v2 validates it before consuming it and fails closed — on any failure the token is left unconsumed rather than partially succeeding. Handle these cases:

| Condition                                                    | Error                                                     | What it means                                                                                         |
| ------------------------------------------------------------ | --------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| Transacting merchant ≠ the payee on the token                | `MERCHANT_ID_MISMATCH`                                    | The order's merchant does not match the `payee.merchant_id` the token was scoped to.                  |
| Order total exceeds the approved amount, or currency differs | `APPROVED_AMOUNT_LIMIT_EXCEEDED`                          | The token carries a fixed amount + currency ceiling; the order must be at or under it, same currency. |
| Token already redeemed                                       | `APPROVAL_TOKEN_ALREADY_COMPLETED`<br/>`PAYMENT_DECLINED` | the token is single-use and already COMPLETED.                                                        |
| Buyer-authentication contingency raised (3DS, SCA, CVV)      | Transaction fails                                         | There is no buyer present to satisfy the challenge, so the order fails outright.                      |

Design your capture path to treat these as terminal for the token: do not retry the same token. If the amount changed, the agent must create a new token for the new amount.

## Platform Integration Guide

### Overview

The PayPal Wallet payment handler lets a UCP agent platform offer the branded PayPal wallet as a payment method during agentic checkout. The buyer approves the payment through PayPal; PayPal issues a single-use, scoped Approval Token that captures the buyer's consent. The agent platform forwards that token into the merchant's checkout session on /complete. The merchant's PSP redeems the token via PayPal Orders v2 — the agent platform is never in the capture/settlement path.

This handler is designed for branded merchants that use PayPal as a payment instrument (not necessarily as their payment processor).

**Roles**

| Role                      | Responsibility                                                                                                                           |
| ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| Agent / platform          | Runs the agentic checkout session; renders the PayPal button; forwards the approval token to the merchant on /complete.                  |
| Agent's PSP               | Renders the PayPal button; makes one call to PayPal to create + approve the approval token; receives the token. Not in the capture path. |
| PayPal                    | Issues and approves the approval token; redeems it on the merchant's PSP's Orders v2 capture.                                            |
| Merchant / merchant's PSP | Accepts the approval token and captures via Orders v2. Covered in the Merchant Integration Guide.                                        |

Note: "Agent platform" and "agent's PSP" may be the same company or different companies. If your platform renders the PayPal button and calls the approval-token API directly, you play both roles.

### 1. Advertising the PayPal Wallet payment handler

A UCP platform advertises the handlers it supports in its profile (and discovers merchant support via the merchant's ucp.payment_handlers). The PayPal Wallet handler is keyed by the reverse-DNS name com.paypal.wallet and conforms to the UCP payment-handler object.

```JSON
{
  "ucp": {
    "version": "2026-08-25",
    "payment_handlers": {
      "com.paypal.wallet": [
        {
          "id": "paypal",
          "version": "2026-09-02",
          "spec": "https://developer.paypal.com/ucp/paypal-wallet/spec",
          "schema": "https://developer.paypal.com/ucp/paypal-wallet/schema.json"
        }
      ]
    }
  }
}

```

### 2. Discovering merchant support

Look for com.paypal.wallet in the merchant's `ucp.payment_handlers` map (returned in the checkout-session response or the merchant profile). The `config.paypal_merchant_identifier` is the payee you use when creating the approval token.

### 3. Creating an approval token

There are two flows:

- One-time checkout (§3.1) — the buyer approves in the moment via the PayPal button. Generally available.
- Vaulted PayPal wallet (§3.2) — no button, no buyer action at purchase; the token is created from a saved vault_id

Both end the same way: you forward one approval token into the merchant's /complete call, and the merchant's PSP redeems it

#### 3.1 Requirements (both flows)

- OAuth 2.0 client credentials → a Bearer access token for the approval-token API.
- Merchant ID (payee) — from the handler config.paypal_merchant_identifier.
- Headers — Authorization: Bearer <token>, Content-Type: application/json.
  - PayPal-Client-Metadata-Id carries the risk tracking_id. Optional on the one-time button flow (the button collects risk data); required on the vaulted flow (§3.2).
- Hosts — Production https://api-m.paypal.com, Sandbox https://api-m.sandbox.paypal.com.

#### 3.2 Endpoints

## Appendix

- Human-present agentic checkout — the buyer approves in the moment (the common case).
- Human-not-present execution — the buyer approves once and the agent completes the order later, within the token's lifetime (default ~3 hours). This is a single one-time token, not vaulting.
- Consent before the order — the approval token captures consent up front, before the merchant order exists; the order is created/executed later by applying the token. (Trade-off: the amount is fixed on the token, so a later order for a different amount fails.)
- PSP decoupling — the agent and the merchant do not need the same PSP. PayPal issues portable consent (the approval token) that the merchant's PSP redeems directly.
