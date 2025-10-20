# Wallet Service

The Wallet Service provides API interfaces that allow users to calculate available funds, and determine buying power based on on-chain assets and future brokerage integration.

## Key Terminology

| Term                    | Description                                                                                               | Source                        |
| ----------------------- | --------------------------------------------------------------------------------------------------------- | ----------------------------- |
| On-chain funds          | On-chain cash (e.g., wrapped Fiat-WUSD) in the user's wallet.                                             | Blockchain (on-chain)         |
| Brokerage Funds         | Off-chain cash held in the user's brokerage account (off-chain).                                          | Brokerage (future)            |
| Buying Power            | Total purchasing capacity, combining on-chain assets (with margin) and brokerage funds (with multiplier). | Hybrid (on-chain + off-chain) |
| Asset Margin Percentage | The collateral value percentage of an asset (e.g., 50% → 10,000 BTC → 5,000 buying power contribution)    | Asset                         |
| User Multiplier         | User profile-based multiplier for brokerage funds (based on risk profile, trading history)                | User.TradeProfile             |

## API Endpoints Overview

**Authenticated:**

- GET `/api/user/funds?address=...`: Returns the available funds (stablecoins) for a specific wallet address
- GET `/api/user/buying-power?address=...`: Returns the buying power calculation for a specific wallet address

> **Note:** All authenticated endpoints require a valid Firebase token in Authorization header. The token must be:
>
> - Obtained from the Firebase Authentication service
> - Prepended with `Bearer: ` e.g.`"Authorization: Bearer: eyJhb....`
> - Include appropriate organization ID and network headers

## GET /api/user/funds?address=...

Returns the available funds for a specific wallet address. This endpoint focuses on immediately available stablecoins that can be used for trading.

### Example call

```bash
curl -X GET \
  https://comsolotex-admin.sologenic.org/api/user/funds?address=testcore15ms2zan4z3y9d29rxsyan5v5vecc04ycts2c85 \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ..." \
  -H "OrganizationID: 72c4c072-2fe4-4f72-ae9d-d9d52a05fd71"
  -H "Network: testnet"
```

### Example response

```json
[
  {
    "Denom": {
      "Currency": {
        "Symbol": "wusdc",
        "Version": "1"
      },
      "Subunit": "suwusdc_1",
      "Issuer": "testcore13s2mmgg4uu4fn8mue6s3lgn74jwdupndjtqah8uxufugtajkeq2qgznc28",
      "Precision": 2
    },
    "RawAmount": {
      "Value": 500000,
      "Exp": -2
    },
    "AssetMarginPercentage": 1
  }
]
```

## GET /api/user/buying-power?address=...

Returns the total buying power calculation for a specific wallet address, including all contributing factors.

### Example call

```bash
curl -X GET \
  https://comsolotex-admin.sologenic.org/api/user/buying-power?address=testcore15ms2zan4z3y9d29rxsyan5v5vecc04ycts2c85 \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ..." \
  -H "OrganizationID: 72c4c072-2fe4-4f72-ae9d-d9d52a05fd71"
  -H "Network: testnet"
```

### Example response

```json
{
  "AvailableOnchainFund": {
    "Value": 5000
  },
  "Collateral": {
    "Value": 10719422298797452,
    "Exp": -13
  },
  "BrokerageFund": {},
  "Total": {
    "Value": 60719422298797452,
    "Exp": -13
  },
  "Currency": "wusdc"
}
```

## Application start parameters

The application uses the following environment parameters:

- `HTTP_CONFIG` - the configuration for the http server (included from `github.com/sologenic/com-be-http-lib/`)
- `AUTH_FIREBASE_SERVICE` - the firebase authentication service endpoint
- `USER_STORE` - the user service endpoint (included from `github.com/sologenic/com-fs-user-model/`)
- `FEATURE_FLAG_STORE` - the feature flag service endpoint (included from `github.com/sologenic/com-fs-feature-flag-model/`)
- `ROLE_STORE` - the role service endpoint (included from `github.com/sologenic/com-fs-role-model/`)
- `ORGANIZATION_STORE` - the organization service endpoint (included from `github.com/sologenic/com-fs-organization-model/`)
- `WALLET_MIDDLEWARE` - the wallet middleware service endpoint (included from `github.com/sologenic/com-fs-wallet-middleware-model/`)
