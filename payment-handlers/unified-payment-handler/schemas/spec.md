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

2a. If they are a Braintree merchant, they should retrieve their merchant identifier through the Braintree control panel.

2b. If they are a PPCP merchant, they should retrieve their PPCP Merchant ID from their PPCP control panel.

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

## 3. Platform Integration

### 3.1 Requirements

Platforms will need to onboard to PYPL's universal tokenizer service if they plan on allowing PYPL merchants to receive credit card payments. If interested please contact your PYPL integration support specialist. <insert documentation for PYPL universal tokenizer onboarding here>

Platforms will also need to onboard to <insert whatever we want here> platform if they plan on allowing PYPL merchants to receive PayPal wallet payments through the PayPal approval token. If interested please contact your PYPL integration support specialist. <insert documentation for PayPal approval token integration here when available>.

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

The `paypal` instrument is the PayPal-branded wallet — the same approval-token flow as the standalone `com.paypal.wallet` handler. Rather than repeat it, follow the Wallet Agent guide for token creation (button or vaulted) and forward the approval token to the merchant on the `/complete` action.

Read from the merchant's instrument entry:

```json
{
  "type": "paypal",
  "config": {
    "merchant_id": "<paypal_merchant_id>"
  }
}
```

Use `config.merchant_id` as the `payee.merchant_id` when creating the approval token, then forward the token id as the credential (`credential.type: "approval_token"`). See the Agent / UCP Platform Integration Guide — PayPal Wallet Payment Handler for the button (`createPayPalOneTimePaymentSession`) and vaulted flows, the approval-token API, and the lifecycle.

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
