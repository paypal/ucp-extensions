# PayPal Wallet Payment Handler Specification

- **Handler Name:** `com.paypal.wallet`
- **Version:** `2026-10-08`
- **Status:** Draft

> ⚠️ **DRAFT — NOT THE FINAL VERSION.** This specification is a work in progress and is subject to change without notice. Field names, endpoints, flows, and behaviors described here may be modified, renamed, or removed before the final release, and sections marked `TODO` or `TBD` are incomplete. Do not rely on this document for production integrations.

## 1. Introduction

The `com.paypal.wallet` handler enables businesses that are PayPal merchants to accept the PayPal-branded wallet when a customer buys an offering from the merchant through a UCP-compatible platform.

The platform coordinates the creation of a single-use PayPal approval token, scoped to the merchant, amount, and currency the buyer approved, and forwards it to the merchant on `/complete`. The merchant's PSP redeems the token via PayPal Orders v2.

## 2. Business Integration

### 2.1 Requirements

Before publicly exposing their UCP profiles, businesses must retrieve their PayPal Merchant ID:

1. Log in to your PayPal business account and open your Business Profile: <https://www.paypal.com/unifiedsettings/unified-profile/business>

2. Your Merchant ID is listed in the Business Profile as ID #.

   Example: ID #XXXXXXXXXXXXX

   Use this value (without the ID # prefix) wherever this guide references `<paypal_merchant_id>`.

### 2.2 Handler Configuration

Businesses that want to accept the PayPal wallet will need to include the handler in their payment handlers registry. They will need to include a [configuration object](#221-configuration-schema).

#### 2.2.1 Configuration Schema

The configuration object defines the environment and merchant identity. It carries no secret credentials.

| Field                        | Type   | Required | Description                                                                                                                  |
| ---------------------------- | ------ | -------- | ---------------------------------------------------------------------------------------------------------------------------- |
| `environment`                | string | Yes      | `sandbox` or `production`                                                                                                    |
| `paypal_merchant_identifier` | string | Yes      | The merchant's PayPal Merchant ID (payee). PayPal uses this as the payee on the approval token, scoping it to this merchant. |

##### 2.2.1.1 Example Handler Declaration

```json
{
  "ucp": {
    "version": "2026-10-08",
    "services": {},
    "payment_handlers": {
      "com.paypal.wallet": [
        {
          "id": "paypal-wallet-handler",
          "version": "2026-10-08",
          "spec": "<url to this file>",
          "schema": "<some paypal developer url>/schema.json",
          "config": {
            "environment": "production",
            "paypal_merchant_identifier": "<paypal_merchant_id>"
          }
        }
      ]
    }
  }
}
```

### 2.3 UCP Capability Discovery

Merchants that wish to participate in UCP facilitated payments need to have their UCP profile available over HTTPS at `/.well-known/ucp`. If you are a store-sync merchant, PayPal may be hosting this on your behalf. **[TODO: add documentation when it becomes available.]**

## 3. Platform Integration

### 3.1 Requirements

Platforms will need to onboard to the PayPal business platform if they plan on allowing PayPal merchants to receive PayPal wallet payments through the PayPal approval token. If interested, please contact your PayPal integration support specialist. Once you have been onboarded, see how to create PayPal approval tokens [here](#321-creating-a-paypal-approval-token).

### 3.2 Payment Integration

| Instrument      | How the Agent Collects It                                                           | Credential Forwarded to Merchant                      |
| --------------- | ----------------------------------------------------------------------------------- | ----------------------------------------------------- |
| `paypal_wallet` | Agent renders PayPal button and coordinates the creation of a PayPal approval token | Opaque string and `credential.type: "approval_token"` |

First read the merchant's handler configuration:

```json
{
  "config": {
    "environment": "production",
    "paypal_merchant_identifier": "<paypal_merchant_id>"
  }
}
```

Use `config.paypal_merchant_identifier` as the `payee.merchant_id` when creating the approval token, and `config.environment` to select the [host](#hosts). Then forward the token id as the credential (`credential.type: "approval_token"`).

#### 3.2.1 Creating a PayPal Approval Token

There are two flows:

- One-time checkout — the buyer approves in the moment via the PayPal button. Generally available.
- Vaulted PayPal wallet — no button, no buyer action at purchase; the token is created from a saved `vault_id`.

Both end the same way: you forward one approval token into the merchant's `/complete` call, and the merchant's PSP redeems it.

**Requirements**

- OAuth 2.0 client credentials → a Bearer access token for the approval-token API.
- Merchant ID (payee) — from the handler's `config.paypal_merchant_identifier`.
- Headers — `Authorization: Bearer <token>`, `Content-Type: application/json`.
  - `PayPal-Client-Metadata-Id` carries the risk `tracking_id`.
    - **Optional** on the one-time button flow (the button collects risk data).
    - **Required** on the vaulted flow.

<a id="hosts"></a>**Hosts**

| Production                 | Sandbox                            |
| -------------------------- | ---------------------------------- |
| <https://api-m.paypal.com> | <https://api-m.sandbox.paypal.com> |

**Endpoints**

| Method | Path                                       | Purpose                                                                     |
| ------ | ------------------------------------------ | --------------------------------------------------------------------------- |
| `POST` | `/v1/commerce/approval-tokens`             | Create a scoped approval token (one-time button flow or vaulted/BA-backed). |
| `GET`  | `/v1/commerce/approval-tokens/{id}`        | Check token status; reconciles with the underlying order state.             |
| `POST` | `/v1/commerce/approval-tokens/{id}/update` | Update the shipping address only, while the token is not yet `COMPLETED`.   |

**Lifecycle**

`CREATED` → `PAYER_ACTION_REQUIRED` → `APPROVED` → `COMPLETED`

| State                   | Meaning                                                                                                               |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `CREATED`               | Token created; underlying order not yet approved.                                                                     |
| `PAYER_ACTION_REQUIRED` | A buyer approval step is required (one-time button flow). The response includes a payer-action redirect link.         |
| `APPROVED`              | Buyer approval is complete. A token created from a vaulted instrument skips straight to this state — no buyer action. |
| `COMPLETED`             | The token has been redeemed against a merchant order via Orders v2 and is now consumed. Terminal.                     |

You forward the token once it is `APPROVED` (the button flow reaches `APPROVED` after the buyer completes the payer-action step; the vaulted flow returns `APPROVED` immediately). Poll GET `/v1/commerce/approval-tokens/{id}` if you need to confirm the state before forwarding. An unredeemed token simply expires at its TTL (~3 hours).

Amount and payee are bound at creation. The token carries a fixed amount, currency, and payee (merchant ID). Redemption fails closed if the merchant order's total exceeds the approved amount, the currency differs, or the transacting merchant does not match the payee — so set `total_amount` and `payee.merchant_id` to exactly what the buyer is approving. The merchant-side error codes are listed in the Merchant Integration Guide.

**One-Time Checkout**

`POST /v1/commerce/approval-tokens`

_Request_

```json
{
  "total_amount": { "value": "50.00", "currency_code": "USD" },
  "payee": { "merchant_id": "<paypal_merchant_id>" },
  "payment_method": {
    "paypal": {
      "experience_context": {
        "return_url": "https://example.com/return",
        "cancel_url": "https://example.com/cancel"
      }
    }
  }
}
```

_Response_

`201 - Created`

```json
{
  "id": "1AB23456CD789012E",
  "status": "PAYER_ACTION_REQUIRED",
  "links": [
    {
      "href": "https://www.paypal.com/checkoutnow?token=1AB23456CD789012E",
      "rel": "payer-action",
      "method": "GET"
    }
  ]
}
```

The `id` is the approval token you forward on `/complete`.

Amount note: the amount is fixed on the token at creation. A later order for a different amount will fail. Set `total_amount` to the amount the buyer is approving.

_SDK — render the PayPal button for buyer approval_

Include the SDK on your page:

```html
<paypal-button id="paypal-button">Pay</paypal-button>
<script
  async
  src="https://www.paypal.com/web-sdk/v6/core"
  onload="onLoad()"
></script>
```

Wire up the button:

```js
async function onLoad() {
  const sdkInstance = await window.paypal.createInstance({
    clientId: "YOUR_CLIENT_ID",
    components: ["paypal-payments"],
  });
  const session = await sdkInstance.createPayPalOneTimePaymentSession({
    async onApprove(data) {
      // data.orderId is the approval token id (...)
      // Forward it into the merchant's checkout session on /complete.
      await completeCheckout(data.orderId);
    },
  });
  document
    .getElementById("paypal-button")
    .addEventListener("click", async () => {
      // Not awaited before start() — preserves transient user activation.
      const createOrderPromise = createOrder();
      await session.start({ presentationMode: "auto" }, createOrderPromise);
    });
}
// Create the approval token; return its id as the order id the SDK expects.
async function createOrder() {
  const res = await fetch("/v1/commerce/approval-tokens", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(/* approval-token request from One-Time Checkout above */),
  });
  const { id } = await res.json();
  return { orderId: id };
}
```

`completeCheckout()` is a wrapper around the UCP compliant `/complete` call.

**Vaulted PayPal Wallet**

With a vaulted PayPal wallet there is no button and no buyer action at purchase. The approval token is created from a saved `vault_id` and returns `APPROVED` immediately.

Two phases:

- Enrollment (once, outside agentic checkout): the buyer saves their PayPal wallet as a channel-initiated billing agreement (CIB), producing a `vault_id`.
- Purchase (agentic checkout): the agent's PSP submits risk signals, then creates an approval token from the `vault_id` + merchant ID. No button.

Where the `vault_id` comes from. The vaulted flow assumes a billing agreement is already on file. If one is not, there are two ways to create it:

- Standalone vaulting, ahead of time — the buyer saves their PayPal wallet in a dedicated save-payment step (or a subscription signup) outside the approval-token flow.
- Billing-with-purchase, in the same flow — the approval-token create call carries a vaulting flag, so PayPal embeds a billing-agreement token alongside the one-time order token; the buyer approves both in a single PaySheet interaction. After the token is approved and applied to the merchant order, a separate confirm step mints the reusable `vault_id` for future purchases.

Once a `vault_id` exists, every subsequent purchase uses the buyer-absent path below, no matter which way it was created.

_Enrollment — save the PayPal wallet (SDK)_

```html
<paypal-button id="paypal-button">Save PayPal</paypal-button>
<script
  async
  src="https://www.paypal.com/web-sdk/v6/core"
  onload="onLoad()"
></script>
```

```js
async function onLoad() {
  const sdkInstance = await window.paypal.createInstance({
    clientToken, // minted server-side
    components: ["paypal-payments"],
  });
  const session = sdkInstance.createPayPalSavePaymentSession({
    async onApprove(data) {
      // data.vaultSetupToken → server exchanges it for the vault_id
      await fetch("/paypal-api/vault/payment-token/create", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ vaultSetupToken: data.vaultSetupToken }),
      });
    },
  });
  document
    .getElementById("paypal-button")
    .addEventListener("click", async () => {
      const setupTokenPromise = createVaultSetupToken();
      await session.start({ presentationMode: "auto" }, setupTokenPromise);
    });
}
async function createVaultSetupToken() {
  const res = await fetch("/paypal-api/vault/setup-token/create", {
    method: "POST",
  });
  const { id } = await res.json();
  return { setupToken: id };
}
```

Server routes (PayPal Server SDK):

```js
import {
  PaypalPaymentTokenUsageType,
  VaultInstructionAction,
  VaultTokenRequestType,
} from "@paypal/paypal-server-sdk";
// Mint the CIB setup token → { id } back to the browser.
app.post("/paypal-api/vault/setup-token/create", async (req, res) => {
  const { result } = await vaultController.createSetupToken({
    paypalRequestId: Date.now().toString(),
    body: {
      paymentSource: {
        paypal: {
          usageType: PaypalPaymentTokenUsageType.Platform, // CIB, not Merchant
          experienceContext: {
            returnUrl: "https://mychannel.com/vault/return",
            cancelUrl: "https://mychannel.com/vault/cancel",
            vaultInstruction: VaultInstructionAction.OnPayerApproval,
          },
        },
      },
    },
  });
  res.json({ id: result.id });
});
// Exchange the approved setup token for the vault_id (store it).
app.post("/paypal-api/vault/payment-token/create", async (req, res) => {
  const { result } = await vaultController.createPaymentToken({
    paypalRequestId: Date.now().toString(),
    body: {
      paymentSource: {
        token: {
          id: req.body.vaultSetupToken,
          type: VaultTokenRequestType.SetupToken,
        },
      },
    },
  });
  res.json({ id: result.id });
});
```

_Purchase — submit risk signals_

Because there is no button, submit device/session risk metadata first and correlate it with a `tracking_id`.

`POST /v3/risk/partner-data` → returns `{ tracking_id, create_time, update_time }`. Pass that `tracking_id` in the `PayPal-Client-Metadata-Id` header on the approval-token create.

_Purchase — create the approval token from the vault_

`POST /v1/commerce/approval-tokens`

_Headers_

```
Authorization: Bearer <access-token>
Content-Type: application/json
PayPal-Client-Metadata-Id: <tracking_id>
```

_Request_

```json
{
  "total_amount": { "value": "99.99", "currency_code": "USD" },
  "payee": { "merchant_id": "<paypal_merchant_id>" },
  "payment_method": {
    "paypal": { "vault_id": "71430604w5310851h" }
  }
}
```

_Response_

`201 - Created`

```json
{
  "id": "5TC14653UR845035G",
  "status": "APPROVED",
  "total_amount": { "currency_code": "USD", "value": "99.99" },
  "payee": { "merchant_id": "<paypal_merchant_id>" },
  "payment_method": {
    "paypal": {
      "email_address": "buyer@example.com",
      "account_id": "JU9Q8UHAPLB9J",
      "name": { "given_name": "James", "surname": "Larry" }
    }
  }
}
```

#### 3.2.2 Risk Signals and Dispute Fields

Risk data is submitted via `POST /v3/risk/partner-data`. The response returns the `tracking_id` you pass on to the approval-token create via `PayPal-Client-Metadata-Id` header. Risk signals and dispute fields are part of the PayPal agentic trust framework that enables PayPal Buyer Protection and Seller protection. Please contact your PayPal rep for the risk signal and dispute field set that is best suited for your use case.

#### 3.2.3 Sending the Approval Token on Complete

You forward the approval token into the merchant's checkout session using the standard UCP `/complete` call. The token is carried as a UCP payment instrument — this handler's instrument follows the UCP `payment.instruments[]` structure.

_Request_

`POST /checkout-sessions/{id}/complete`

_Headers_

```
UCP-Agent: profile="https://platform.example/profile"
Content-Type: application/json
```

_Body_

```json
{
  "payment": {
    "instruments": [
      {
        "id": "pi_paypal_1",
        "handler_id": "paypal-wallet-handler",
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

_Field Mapping_

| Field                   | Value                     | Notes                                                           |
| ----------------------- | ------------------------- | --------------------------------------------------------------- |
| `payment.instruments[]` | array                     | UCP base structure. Send the single selected PayPal instrument. |
| `instrument.id`         | `"pi_paypal_1"`           | Unique id for this instrument instance, assigned by the agent.  |
| `instrument.handler_id` | `"paypal-wallet-handler"` | Matches the id in the `com.paypal.wallet` handler declaration.  |
| `instrument.type`       | `"paypal_wallet"`         | The instrument type this handler produces.                      |
| `instrument.selected`   | `true`                    | The buyer selected PayPal.                                      |
| `credential.type`       | `"approval_token"`        | Discriminator for the PayPal approval-token credential.         |
| `credential.token`      | `1AB23456CD789012E`       | The approval token.                                             |

On success, the merchant returns the completed checkout state with an `order_id`. The merchant's PSP redeems the approval token via PayPal Orders v2 as `payment_source.token` with type `APPROVAL`.
