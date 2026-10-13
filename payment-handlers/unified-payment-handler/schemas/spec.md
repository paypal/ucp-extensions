# PayPal Unified Payment Handler Specification

- **Handler Name:** `com.paypal.payments`
- **Version:** `2026-10-08`

## 1. Introduction

The `com.paypal.payments` handler enables businesses that are PYPL merchants (PPCP or Braintree) to convey the payment instruments they can accept when a customer buys an offering from the merchant through a UCP-compatible platform.

This handler enables a headless integration model where businesses provide enough configuration in their payment handlers for platforms to render the correct user interface elements needed for customers to use one of the business's available payment instruments.

## 2. Business Integration

### 2.1 Requirements

Before publicly exposing their UCP profiles, businesses must complete the following steps:

1. Determine their available payment instruments through their Braintree or PPCP integration.

2a. If they are a Braintree merchant, they should retrieve their merchant identifier through the Braintree control panel. To do this log into the control panel, click on the gear icon at the top right, navigate to API, merchant ID will be shown in the Client-Side Encryption Keys section.

2b. If they are a PPCP merchant, they should retrieve their PPCP Merchant ID from their PPCP control panel. To find it, do the following:

Log in to your PayPal business account and open your Business Profile: https://www.paypal.com/unifiedsettings/unified-profile/business

Your Merchant ID is listed in the Business Profile as ID #.

Example: ID #L92QX7RBHF63K

Use this value (without the ID # prefix) wherever this guide references <paypal_merchant_id>.

3. If they wish to convey that they want to receive Google Pay payments, they will need to retrieve their Google Pay Merchant ID through the Google Pay Business Console.

4. If they wish to convey that they want to receive Apple Pay payments, they will need to retrieve their Apple Pay Merchant Identifier.

5. If they wish to convey that they want to receive PayPal payments, they will need to retrieve their PayPal Merchant Identifier.

### 2.2 Handler Configuration

Businesses that want to use their PYPL PSP-enabled payment instruments will need to include the handler in their payment handlers registry. They will need to include a [configuration object](#221-configuration-schema).

#### 2.2.1 Configuration Schema

The configuration object defines the environment and merchant identity.

##### 2.2.1.1 Example Braintree Handler Declaration

```json
{
  "version": "2026-10-08",
  "payment_handlers": {
    "com.paypal.payments": [
      {
        "id": "unified-paypal-handler",
        "version": "2026-10-08",
        "spec": "<url to this file>",
        "schema": "<url>/schema.json",
        "available_instruments": [
          {
            "type": "card",
            "constraints": {
              "brands": [
                "visa",
                "mastercard",
                "amex",
                "discover",
                "jcb",
                "diners",
                "elo",
                "hiper",
                "hipercard",
                "maestro",
                "switch",
                "solo",
                "unionpay"
              ],
              "tokenization": "required"
            }
          },
          {
            "type": "paypal",
            "config": {
              "merchant_id": "L92QX7RBHF63K"
            }
          },
          {
            "type": "google_pay",
            "config": {
              "api_version": 2,
              "api_version_minor": 0,
              "environment": "PRODUCTION",
              "merchant_info": {
                "merchant_name": "Example Merchant",
                "merchant_id": "1234567890123456789",
                "merchant_origin": "checkout.merchant.com"
              },
              "allowed_payment_methods": [
                {
                  "type": "CARD",
                  "parameters": {
                    "allowed_auth_methods": ["PAN_ONLY"],
                    "allowed_card_networks": [
                      "AMEX",
                      "DISCOVER",
                      "JCB",
                      "MASTERCARD",
                      "VISA"
                    ]
                  },
                  "tokenization_specification": {
                    "type": "PAYMENT_GATEWAY",
                    "parameters": {
                      "gateway": "braintree",
                      "gatewayMerchantId": "bn7r4qtsvjs746dm"
                    }
                  }
                }
              ]
            }
          },
          {
            "type": "apple_pay",
            "config": {
              "merchant_identifier": "merchant.com.example.store",
              "supported_networks": ["visa", "mastercard", "amex", "discover"]
            }
          }
        ],
        "config": {
          "environment": "production",
          "merchant_id": "bn7r4qtsvjs746dm"
        }
      }
    ]
  }
}
```

##### 2.2.1.2 Example PPCP Handler Declaration

```json
{
  "version": "2026-10-08",
  "payment_handlers": {
    "com.paypal.payments": [
      {
        "id": "unified-paypal-handler",
        "version": "2026-10-08",
        "spec": "<url to this file>",
        "schema": "<url>/schema.json",
        "available_instruments": [
          {
            "type": "card",
            "constraints": {
              "brands": [
                "visa",
                "mastercard",
                "amex",
                "discover",
                "jcb",
                "diners",
                "elo",
                "hiper",
                "hipercard",
                "maestro",
                "switch",
                "solo",
                "unionpay"
              ],
              "tokenization": "required"
            }
          },
          {
            "type": "paypal",
            "config": {
              "merchant_id": "L92QX7RBHF63K"
            }
          },
          {
            "type": "google_pay",
            "config": {
              "api_version": 2,
              "api_version_minor": 0,
              "environment": "PRODUCTION",
              "merchant_info": {
                "merchant_name": "Example Merchant",
                "merchant_id": "1234567890123456789",
                "merchant_origin": "checkout.merchant.com"
              },
              "allowed_payment_methods": [
                {
                  "type": "CARD",
                  "parameters": {
                    "allowed_auth_methods": ["PAN_ONLY"],
                    "allowed_card_networks": [
                      "AMEX",
                      "DISCOVER",
                      "JCB",
                      "MASTERCARD",
                      "VISA"
                    ]
                  },
                  "tokenization_specification": {
                    "type": "PAYMENT_GATEWAY",
                    "parameters": {
                      "gateway": "paypalppcp",
                      "gatewayMerchantId": "L92QX7RBHF63K"
                    }
                  }
                }
              ]
            }
          },
          {
            "type": "apple_pay",
            "config": {
              "merchant_identifier": "merchant.com.example.store",
              "supported_networks": ["visa", "mastercard", "amex", "discover"]
            }
          }
        ],
        "config": {
          "environment": "production",
          "merchant_id": "L92QX7RBHF63K"
        }
      }
    ]
  }
}
```

### 2.3 UCP Capability Discovery

Merchants that wish to participate in UCP facilitated payments need to have their UCP profile available over HTTPS at `./well-known/ucp`. If you are a story-sync merchant, PayPal maybe hosting this on your behalf <insert documentation here when it becomes available>.

## 3. Platform Integration

### 3.1 Requirements

Platforms will need to onboard to PYPL's universal tokenizer service if they plan on allowing PYPL merchants to receive credit card payments. If interested please contact your PYPL integration support specialist. <insert documentation for PYPL universal tokenizer onboarding here>

Platforms will also need to onboard to the PayPal business platform if they plan on allowing PYPL merchants to receive PayPal wallet payments through the PayPal approval token. If interested please contact your PYPL integration support specialist. Once you have been onboarded, see how to create PayPal approval tokens [here]().

Platforms that are interested in facilitating Apple Pay payments to PYPL merchants will need to be registered with Apple Pay and be an official Apple Pay platform integrator. <insert link to Apple Pay documentation here>.

Platforms that are interested in facilitating Google Pay payments to PYPL merchants will need to do the following:

1. Be capable of loading the **Google Pay API for Web** (or Android equivalent).
2. Adhere to the [Google Pay Brand Guidelines](https://developers.google.com/pay/api/web/guides/brand-guidelines).

### 3.2 Payment Integration

| Instrument | How the Agent Collects It                                                                | Credential Forwarded to Merchant                                                                                                     |
| ---------- | ---------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| card       | Collects card info through PCI compliant mechanism, tokenizes with merchant's PSP        | Opaque string and `credential.type: "nonce"`                                                                                         |
| PayPal     | Agent renders PayPal button and coordinates the creation of a PayPal approval token      | Opaque string and `credential.type: "approval_token"`                                                                                |
| Google Pay | Agent renders Google Pay button and configures it with merchant information              | Braintree: nonce and `credential.type: "nonce"`<br>PPCP: Google Pay ECv2 token, passed through `credential.type: "google_pay_token"` |
| Apple Pay  | Agent renders Apple Pay button and configures it with the merchant's merchant_identifier | Encrypted Apple Pay payment token, passed through `credential.type: "apple_pay_token"`                                               |

#### 3.2.1 Card

The card instrument requires tokenization: the agent must never forward a raw PAN on `/complete`. The merchant's card instrument declares this in its business profile:

```json
{
  "type": "card",
  "constraints": {
    "brands": [
      "visa",
      "mastercard",
      "amex",
      "discover",
      "jcb",
      "diners",
      "elo",
      "hiper",
      "hipercard",
      "maestro",
      "switch",
      "solo",
      "unionpay"
    ],
    "tokenization": "required"
  }
}
```

- Restrict the cards you collect to `constraints.brands`. The advertised set is the 13 card brands common to both PayPal Orders v2 and Braintree, so an advertised brand clears regardless of which processor backs the merchant's card instrument.
- `tokenization: "required"` means you must call `/tokenize` and forward the resulting token — never the card details. See the [tokenization guide](#33-tokenization-guide) for more information.

#### 3.2.2 PayPal

The `paypal` instrument is the PayPal-branded wallet.

First read from the merchant's instrument entry:

```json
{
  "type": "paypal",
  "config": {
    "merchant_id": "<paypal_merchant_id>"
  }
}
```

Use `config.merchant_id` as the `payee.merchant_id` when creating the approval token, then forward the token id as the credential (`credential.type: "approval_token"`).

##### 3.2.2.1 Creating a PayPal Approval Token

There are two flows:

- One-time checkout — the buyer approves in the moment via the PayPal button. Generally available.
- Vaulted PayPal wallet — no button, no buyer action at purchase; the token is created from a saved vault_id

Both end the same way: you forward one approval token into the merchant's /complete call, and the merchant's PSP redeems it

**Requirements**

- OAuth 2.0 client credentials → a Bearer access token for the approval-token API.
- Merchant ID (payee) — from the handler `config.paypal_merchant_identifier`.
- Headers — `Authorization: Bearer <token>`, `Content-Type: application/json`.
  - `PayPal-Client-Metadata-Id` carries the risk `tracking_id`.
    - **Optional** on the one-time button flow (the button collects risk data)
    - **Required** on the vaulted flow (§3.2).

**Hosts**

| Production               | Sandbox                           |
| ------------------------ | --------------------------------- |
| https://api-m.paypal.com | https://api-m.sandbox.paypal.com. |

**Endpoints**

| Method | Path                                      | Purpose                                                                     |
| ------ | ----------------------------------------- | --------------------------------------------------------------------------- |
| `POST` | `/v1/commerce/approval-tokens`            | Create a scoped approval token (one-time button flow or vaulted/BA-backed). |
| `GET`  | `/v1/commerce/approval-tokens/{id}`       | Check token status; reconciles with the underlying order state.             |
| `POST` | `v1/commerce/approval-tokens/{id}/update` | Update the shipping address only, while the token is not yet `COMPLETED`.   |

**LifeCycle**

CREATED → PAYER_ACTION_REQUIRED → APPROVED → COMPLETED

| State                   | Meaning                                                                                                               |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `CREATED`               | Token created; underlying order not yet approved.                                                                     |
| `PAYER_ACTION_REQUIRED` | A buyer approval step is required (one-time button flow). The response includes a payer-action redirect link.         |
| `APPROVED`              | Buyer approval is complete. A token created from a vaulted instrument skips straight to this state — no buyer action. |
| `COMPLETED`             | The token has been redeemed against a merchant order via Orders v2 and is now consumed. Terminal.                     |

You forward the token once it is `APPROVED` (the button flow reaches `APPROVED` after the buyer completes the payer-action step; the vaulted flow returns `APPROVED` immediately). Poll GET `/v1/commerce/approval-tokens/{id}` if you need to confirm the state before forwarding. An unredeemed token simply expires at its TTL (~3 hours).

Amount and payee are bound at creation. The token carries a fixed amount, currency, and payee (merchant id). Redemption fails closed if the merchant order's total exceeds the approved amount, the currency differs, or the transacting merchant does not match the payee — so set total_amount and payee.merchant_id to exactly what the buyer is approving. The merchant-side error codes are listed in the Merchant Integration Guide.

**One Time Checkout**

`POST /v1/commerce/approval-tokens`

_Request_

```JSON
{
  "total_amount": { "value": "50.00", "currency_code": "USD" },
  "payee": { "merchant_id": "Q7C9N6S4YRQXG" },
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

```JSON
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

The id is the approval token you forward on /complete.

Amount note: the amount is fixed on the token at creation. A later order for a different amount will fail. Set total_amount to the amount the buyer is approving.

_SDK — Render the PayPal button for buyer approval_

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
sync function onLoad() {
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
  document.getElementById("paypal-button").addEventListener("click", async () => {
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
    body: JSON.stringify(/* approval-token request from 3.1.1 */),
  });
  const { id } = await res.json();
  return { orderId: id };
}
```

`completeCheckout()` is wrapper around the UCP compliant `/complete` call.

**Vaulted PayPal Wallet**

With a vaulted PayPal wallet there is no button and no buyer action at purchase. The approval token is created from a saved `vault_id` and returns `APPROVED` immediately.

Two phases:

- Enrollment (once, outside agentic checkout): the buyer saves their PayPal wallet as a channel-initiated billing agreement (CIB), producing a `vault_id`.
- Purchase (agentic checkout): the agent's PSP submits risk signals, then creates an approval token from the vault_id + merchant ID. No button.

Where the vault_id comes from. The vaulted flow assumes a billing agreement is already on file. If one is not, there are two ways to create it:

- Standalone vaulting, ahead of time — the buyer saves their PayPal wallet in a dedicated save-payment step (or a subscription signup) outside the approval-token flow.
- Billing-with-purchase, in the same flow — the approval-token create call carries a vaulting flag, so PayPal embeds a billing-agreement token alongside the one-time order token; the buyer approves both in a single PaySheet interaction. After the token is approved and applied to the merchant order, a separate confirm step mints the reusable Vault Id for future purchases.

Once a vault_id exists, every subsequent purchase uses the buyer-absent path below, no matter which way it was created.

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

`POST /v3/risk/partner-data `→ returns `{ tracking_id, create_time, update_time }`. Pass that `tracking_id` in the PayPal-Client-Metadata-Id header on the approval-token create. (See the Risk Signals appendix for the metadata field set.)

**Purchase — create the approval token from the vault**

`POST /v1/commerce/approval-tokens`

_Headers_

```
Authorization: Bearer <access-token>
Content-Type: application/json
PayPal-Client-Metadata-Id: <tracking_id>

```

_Request_

```JSON
{
  "total_amount": { "value": "99.99", "currency_code": "USD" },
  "payee": { "merchant_id": "Q7C9N6S4YRQXG" },
  "payment_method": {
    "paypal": { "vault_id": "71430604w5310851h" }
  }
}
```

_Response_

`201 - created`

```JSON
{
  "id": "5TC14653UR845035G",
  "status": "APPROVED",
  "total_amount": { "currency_code": "USD", "value": "99.99" },
  "payee": { "merchant_id": "Q7C9N6S4YRQXG" },
  "payment_method": {
    "paypal": {
      "email_address": "buyer@example.com",
      "account_id": "JU9Q8UHAPLB9J",
      "name": { "given_name": "James", "surname": "Larry" }
    }
  }
}
```

**Sending the Approval token on complete**

You forward the approval token into the merchant's checkout session using the standard UCP `/complete` call. The token is carried as a UCP payment instrument — this handler's instrument follows the UCP `payment.instruments[]` structure.

_Request_

`POST /checkout-sessions/{id}/complete`

_Headers_

```
UCP-Agent: profile="https://platform.example/profile"
Content-Type: application/json
```

_Body_

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

_Field Mapping_

| Field                   | Value               | Notes                                                           |
| ----------------------- | ------------------- | --------------------------------------------------------------- |
| `payment.instruments[]` | array               | UCP base structure. Send the single selected PayPal instrument. |
| `instrument.handler_id` | `"paypal"`          | Matches the id in the com.paypal.payments handler declaration   |
| `instrument.type`       | `"paypal_wallet"`   | The instrument type this handler produces.                      |
| `instrument.selected`   | `true`              | The buyer selected PayPal.                                      |
| `credential.type`       | `"approval_token"`  | Discriminator for the PayPal approval-token credential.         |
| `credential.token`      | `1AB23456CD789012E` | The approval token                                              |

On success, the merchant returns the completed checkout state with an `order_id`.

#### 3.2.3 Google Pay

When the buyer selects Google Pay, the agent renders the Google Pay sheet, receives Google's encrypted token, and forwards it on `/complete`.

**1. Read the merchant's `google_pay` config from discovery**

The merchant's `com.paypal.payments` handler advertises a `google_pay` instrument whose configuration carries the Google Pay parameters the agent needs to build the sheet:

```json
{
  "type": "google_pay",
  "config": {
    "api_version": 2,
    "api_version_minor": 0,
    "environment": "PRODUCTION",
    "merchant_info": {
      "merchant_name": "Example Merchant",
      "merchant_id": "<google_pay_merchant_id>",
      "merchant_origin": "checkout.merchant.com"
    },
    "allowed_payment_methods": [
      {
        "type": "CARD",
        "parameters": {
          "allowed_auth_methods": ["PAN_ONLY"],
          "allowed_card_networks": [
            "AMEX",
            "DISCOVER",
            "JCB",
            "MASTERCARD",
            "VISA"
          ]
        },
        "tokenization_specification": {
          "type": "PAYMENT_GATEWAY",
          "parameters": {
            "gateway": "paypalppcp",
            "gatewayMerchantId": "<paypal_merchant_id>"
          }
        }
      }
    ]
  }
}
```

The agent uses these values as-is to construct the Google Pay request — it does not author them. What the agent reads:

| Field the Agent Reads                                  | Used For                                                                                                                                                                   |
| ------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `merchant_info`                                        | Merchant name, ID, and origin shown in the Google Pay sheet                                                                                                                |
| `allowed_payment_methods[].parameters`                 | The `allowedAuthMethods` (`PAN_ONLY` only — Google's schema allows no other value for this handler) and `allowedCardNetworks` to pass to Google Pay                        |
| `allowed_payment_methods[].tokenization_specification` | Passed straight into the sheet's `tokenizationSpecification`. `gateway: "paypalppcp"` means the resulting token is one PayPal can capture — the agent does not change this |

**2. Render the Google Pay sheet and get the token**

Build the Google Pay `PaymentDataRequest` from the config above and load the sheet.

On buyer approval, Google Pay returns a `PaymentData` object. Read the encrypted token from `paymentMethodData.tokenizationData.token` (a JSON string — Google's ECv2 payload).

Do not transform or re-tokenize it — forward it to the merchant unchanged.

**3. Forward the token on `/complete`**

```json
{
  "payment": {
    "instruments": [
      {
        "id": "instr_1",
        "handler_id": "paypal",
        "type": "google_pay",
        "selected": true,
        "credential": {
          "type": "google_pay_token",
          "token": "{\"signature\":\"...\",\"protocolVersion\":\"ECv2\",\"signedMessage\":\"...\"}"
        }
      }
    ]
  }
}
```

| Field the Agent Sets | Value                                                                                 |
| -------------------- | ------------------------------------------------------------------------------------- |
| `handler_id`         | `paypal` — the id from the handler declaration                                        |
| `type`               | `google_pay`                                                                          |
| `credential.type`    | `google_pay_token`                                                                    |
| `credential.token`   | The ECv2 payload from `paymentMethodData.tokenizationData.token`, forwarded unchanged |

#### 3.2.4 Apple Pay

When the buyer selects Apple Pay, the agent presents the Apple Pay sheet, receives Apple's encrypted payment token, and forwards it on `/complete`. There is no `/tokenize` call — Apple Pay returns the token directly.

> ⚠️ **Config shape TBD.** The exact shape of the merchant's `apple_pay` instrument config is not finalized — we are waiting on input from platforms before locking it down. The example below is illustrative; treat the field names as provisional.

**1. Read the merchant's `apple_pay` config from discovery**

The merchant's `com.paypal.payments` handler advertises an `apple_pay` instrument whose config carries the Apple Pay parameters the agent needs to build the sheet:

```json
{
  "type": "apple_pay",
  "config": {
    "merchant_identifier": "merchant.com.example.store",
    "supported_networks": ["visa", "mastercard", "amex", "discover"]
  }
}
```

The agent uses these values as-is to construct the Apple Pay request — it does not author them (the merchant sets them; see the Merchant guide). What the agent reads:

| Field the Agent Reads | Used For                                                                         |
| --------------------- | -------------------------------------------------------------------------------- |
| `merchant_identifier` | The Apple merchant identifier for the `PKPaymentRequest` and merchant validation |
| `supported_networks`  | The card networks to offer in the Apple Pay sheet                                |

**2. Present the Apple Pay sheet and get the token**

Create the `PKPaymentRequest` / `ApplePayPaymentRequest` with `merchantIdentifier` and `supportedNetworks` from the config above.

Perform merchant validation against `merchant_identifier`.

On buyer approval, Apple returns a `PKPaymentToken`. Read the encrypted payment token from `paymentData`.

Do not transform or re-tokenize it — forward it to the merchant unchanged.

**3. Forward the token on `/complete`**

```json
{
  "payment": {
    "instruments": [
      {
        "id": "instr_1",
        "handler_id": "paypal",
        "type": "apple_pay",
        "selected": true,
        "credential": {
          "type": "apple_pay_token",
          "token": "<apple pay encrypted payment token>"
        }
      }
    ]
  }
}
```

| Field the Agent Sets | Value                                                                              |
| -------------------- | ---------------------------------------------------------------------------------- |
| `handler_id`         | `paypal` — the id from the handler declaration (§2)                                |
| `type`               | `apple_pay`                                                                        |
| `credential.type`    | `apple_pay_token`                                                                  |
| `credential.token`   | The encrypted payment token from `PKPaymentToken.paymentData`, forwarded unchanged |

The agent's job ends here.

#### 3.2.5 Sending the Credential on Complete

You forward the selected instrument's credential into the merchant's checkout session using the standard UCP `/complete` call, as a UCP payment instrument. Send only the single selected instrument.

URL: `POST /checkout-sessions/{id}/complete`

Headers:

```
UCP-Agent: profile="https://platform.example/profile"
Content-Type: application/json
```

Request — the credential shape depends on the instrument. Example for card:

```json
{
  "payment": {
    "instruments": [
      {
        "id": "pi_card_1",
        "handler_id": "paypal",
        "type": "card",
        "selected": true,
        "credential": {
          "type": "nonce",
          "token": "<nonce>"
        }
      }
    ]
  }
}
```

**Credential by Instrument Type**

| `instrument.type` | `credential.type`  | `credential.token`                                                                    |
| ----------------- | ------------------ | ------------------------------------------------------------------------------------- |
| `card`            | `nonce`            | Nonce from the PayPal tokenizer                                                       |
| `paypal`          | `approval_token`   | Approval token id                                                                     |
| `google_pay`      | `google_pay_token` | Google Pay ECv2 token from `paymentMethodData.tokenizationData.token`, passed through |
| `apple_pay`       | `apple_pay_token`  | Encrypted Apple Pay payment token from `PKPaymentToken.paymentData`                   |

**Common Fields**

| Field                   | Value                                                 | Notes                                                           |
| ----------------------- | ----------------------------------------------------- | --------------------------------------------------------------- |
| `instrument.handler_id` | `"paypal"`                                            | Matches the id in the `com.paypal.payments` handler declaration |
| `instrument.type`       | One of `card`, `paypal`, `google_pay`, or `apple_pay` | The instrument the buyer selected                               |
| `instrument.selected`   | `true`                                                | The buyer selected this instrument                              |

On success, the merchant returns the completed checkout state with an `order_id`. The merchant's PSP redeems the credential.

### 3.3 Tokenization Guide

#### Onboarding

1. Generate an ECDSA P-256 (ecdsa-p256-sha256) key pair.

2. Send PayPal your public key and a `key_id` you've assigned to it. **Never send the private key** — PayPal only needs the public half to verify your signatures.

3. PayPal registers the key and returns a profile URL — you'll send this back on every request in the `UCP-Agent` header.

#### Calling the Tokenization Service

The PYPL universal tokenizer service authenticates requests using [RFC 9421 HTTP Message Signatures](https://www.rfc-editor.org/info/rfc9421/). Please reference the following documentation to successfully tokenize a credit card.

##### Compose the HTTP Request

**Tokenize Request Fields**

| Field                         | Description                                                                                                         |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| `credential`                  | The raw credential to tokenize                                                                                      |
| `credential.type`             | `card`                                                                                                              |
| `credential.card_number_type` | Format of number — `fpan` (raw PAN), `network_token`, or `dpan`. Network tokens are preferred with fallback to FPAN |
| `credential.number`           | Card number                                                                                                         |
| `credential.expiry_year`      | Card expiration year                                                                                                |
| `credential.expiry_month`     | Card expiration month                                                                                               |
| `credential.cvc`              | Card security code                                                                                                  |
| `credential.name`             | Card holder name                                                                                                    |
| `credential.cryptogram`       | value to be provided when `credential.card_number_type` = `network_token`                                           |
| `credential.eci_value`        | value to be provided when `credential.card_number_type` = `network_token`                                           |
| `binding.type`                | `dev.ucp.shopping.checkout`                                                                                         |
| `binding.id`                  | The checkout session this token is scoped to. Binds the token so it can't be replayed on another checkout           |

**Example Tokenize Request Object**

```json
{
  "credential": {
    "type": "card",
    "card_number_type": "fpan",
    "number": "4111111111111111",
    "expiry_month": 4,
    "expiry_year": 2028,
    "cvc": "123",
    "name": "Jane Doe"
  },
  "binding": {
    "type": "dev.ucp.shopping.checkout",
    "id": "<checkout_id>"
  }
}
```

##### Compose the HTTP Message Signature

Each request must be signed over these components, in this order:

1. `@method`
2. `@authority`
3. `@path`
4. `ucp-agent`
5. `content-digest`
6. `content-type`

The `content-digest` value is `sha-256=:<base64 SHA-256 digest of the raw request body>:`. The signature parameters (the covered component list, plus `created`, `keyid`, and `alg`) are themselves included as the final line of the signed base, per [RFC 9421](https://www.rfc-editor.org/info/rfc9421/).

## Appendix

**Risk signals (vaulted path)**

Risk data is submitted via `POST /v3/risk/partner-data` as `partner_channel_id`, `tracking_id`, and a `risk_metadata objec`t (industry code, sender/receiver, partner, device). The response returns the `tracking_id` you pass on the approval-token create via PayPal-Client-Metadata-Id. See the PayPal risk API reference for the full field set. Required only on the vaulted/no-button path — the one-time button flow collects this data via the SDK.
