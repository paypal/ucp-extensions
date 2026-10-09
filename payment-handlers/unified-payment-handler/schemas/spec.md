# PayPal Universal Payment Handler Specification

- **Handler Name:** `com.paypal.payments`
- **Version:** `2026-10-08`

## 1. Introduction

The `com.paypal.payments` handler enables businesses that are PYPL merchants (PPCP or Braintree), to convey the payment instruments they can accept when a customer buys an offering from the merchant through a UCP-compatible platform.

This handler enables a headless integration model where business provide enough configuration in their payment handlers for platforms to render the correct user interface elements needed for customers to use one of the business's available payment instruments.

## 2. Business Integration

### 2.1 Requirements

Before publicly exposing their UCP profiles, businesses must do the following steps:

1. Determine their available payment instruments through their Braintree or PPCP integration.

2a. If they are a Braintree merchant, they should retrieve their merchant identifier through the Braintree control panel.

2b. If they are a PPCP merchant, they should retrieve their PPCP Merchant ID from their PPCP control panel.

3. If they wish to convey that they want to receive GooglePay Payments, they will need to retrieve their Google Pay Merchant ID through the Google Pay Business Console.

4. If they wish to convey that they want to receive Apple Pay payments, they will need to retrieve their Apple Pay Merchant Identifier.

5. If they wish to convey that they want to receive PayPal payments, they will need to retieve their PayPal Merchant Identifier.

### 2.2 Handler Configuration

Business that want to use their PYPL PSP enabled payment instruments will need to include the handler in their payment handlers registry. They will need to include a [configuration object](#221-configuration-schema).

#### 2.2.1 Configuration Schema

The configuration object defines the environment and merchant identity.

##### 2.2.2.1 Example Braintree Handler Declaration

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

##### 2.2.2.2 Example PPCP Handler Declaration

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

Platforms will also need to onboard to <insert what ever we want here> platform if they plan on allowing PYPL merchants to receive PayPal payments. If interested please contact your PYPL integration support specialist. <insert documentation for PayPal approval token integration here when available>.

Platforms that are interested in facilitating ApplePay payments to PYPL merchants will need to be registered with ApplePay and be an official ApplePay platform integrator. <insert link to apply pay documentation here>.

Platforms that are interested in facilitating GooglePay payments to PYPL merchants will need to do the following:

1.  Be capable of loading the **Google Pay API for Web** (or Android
    equivalent).
2.  Adhere to the
    [Google Pay Brand Guidelines](https://developers.google.com/pay/api/web/guides/brand-guidelines)

### 3.2 Payment Flow

### 3.3 Tokenization Guide

#### Onboarding

1. Generate an ECDSA P-256 (ecdsa-p256-sha256) key pair.

2. Send PayPal your public key and a `key_id` you've assigned to it. **Never send the private key** — PayPal only ever needs the public half to verify your signatures.

3. PayPal registers the key and returns a profile URL — you'll send this back on every request in the `ucp-agent` header.

#### Calling the tokenization service

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

** Example Tokenize Request Object**

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

##### Compose the HTTP message signature

Each request must be signed over these components, in this order:

1. `@method`
2. `@authority`
3. `@path`
4. `ucp-agent`
5. `content-digest`
6. `content-type`

The content-digest value is `sha-256=:<base64 SHA-256 digest of the raw request body>:`. The signature parameters (the covered component list, plus `created`, `keyid`, and `alg`) are themselves included as the final line of the signed base, per [RFC 9421](https://www.rfc-editor.org/info/rfc9421/).
