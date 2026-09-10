# PayPal UCP and UCP Extension Specifications

This repository contains the [JSON Schema](https://json-schema.org/) definitions merchants and merchant aggregators use to configure their **Universal Commerce Protocol (UCP)** profile for integrating PayPal Payment Service Providers (PSPs) and PayPal payment methods.

These schemas describe the structure for **payment handler** and **shopping extension**, so that a merchant's UCP profile can be authored, validated, and consumed consistently across API platforms.

## What is UCP?

[UCP](https://ucp.dev/) is a protocol that platforms (AI platforms) can implement to facilitate e-commerce on their platforms.This repository contains example JSON schema files that merchants can use so that they can participate in UCP.

## Repository Structure

```
payment-handlers/
├── unified-payment-handler/
│   └── schemas/                  # JSON Schema files defining the Unified Payment Handler
├── paypal-wallet-payment-handler/
│   └── schemas/                  # JSON Schema files defining the PayPal Wallet Payment Handler                # JSON Schema files defining the Venmo Wallet Payment Handler
shopping/
└── custom-shopping-extension/
    └── schemas/                  # JSON Schema files defining the Custom Shopping Extension
```

### Payment Handlers

| Handler                           | Description                                                                                                    |
| --------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| **Unified Payment Handler**       | Configuration schema that can be used by Braintree and PPCP merchants.                                         |
| **PayPal Wallet Payment Handler** | Configuration schema for merchants that only want to present the PayPal wallet as an available payment method. |

### Shopping Extensions

| Extension                     | Description                                                                  |
| ----------------------------- | ---------------------------------------------------------------------------- |
| **Custom Shopping Extension** | Schema definitions for extending shopping capabilities for PayPal merchants. |

## Who This Is For

- **Merchants** configuring their own UCP profile to accept PayPal PSPs and payment methods directly.
- **Merchant aggregators / platforms** building tooling that generates or validates UCP profiles on behalf of multiple merchants.
- **AI Platforms** that wish to integrate with the PYPL ecosystem.

## Getting Started

1. Identify which payment handler(s) and/or shopping extension(s) your integration needs.
2. Reference the relevant schema(s) from the corresponding `schemas/` directory in your UCP profile.
3. Validate your configuration against the schema using any standard JSON Schema validator (e.g., [ajv](https://ajv.js.org/), [jsonschema](https://pypi.org/project/jsonschema/)).

```bash
# Example: validating a merchant config against the PayPal Wallet handler schema
ajv validate -s payment-handlers/paypal-wallet-payment-handler/schemas/<schema-file>.json -d my-merchant-config.json
```

## Versioning

TBD

## Contributing

TBD

## Support

TBD, most PSPs include a discord channel here.

## License

TBD
