# Order Management Service

The Order Management Service provides the following RESTFUL interface:

## Prerequisites for Trading (Asset Issuance and Whitelisting)

Before a user can place an order, two prerequisites must be satisfied automatically by the service:

### 1. Asset Issuance (Lazy Issuance)
- **Purpose**: Ensure the asset exists on the smart contract for trading
- **Check**: `IsIssuedInSmartContract` flag
- **Action**: If `false`, automatically issue the asset via smart contract `IssueAsset` function
- **Process**:
  - Calls smart contract with asset metadata (symbol, exchange, precision, description, version, etc.)
  - Funds transaction with `AssetIssuanceFund` (10 COREUM)
  - Updates asset record: `IsIssuedInSmartContract = true` and `Denom.Issuer = smartContractAddress`

### 2. User Whitelisting (Lazy Whitelisting)
- **Purpose**: Authorize user's wallet to trade the specific asset
- **Check**: Query `UserAssetList` for user/asset combination
- **Logic**:
  - **No record exists**: Proceed with automatic whitelisting
  - **Status = WHITELISTED**: User ready to trade
  - **Any other status**: Block trading (e.g., OUTDATED_VERSION, BLACKLISTED)
- **Action**: If whitelisting needed, automatically:
  - Set maximum hold limit on smart contract via `UpdateUser` message
  - Create `UserAssetList` record with `WHITELISTED` status

### Happy Path Workflow

```
1. User places an order via /order/create-tx endpoint

2. Asset Issuance Check (Lazy)
   ├─ Check: asset.AssetDetails.IsIssuedInSmartContract
   ├─ If FALSE → Issue asset on smart contract
   │  ├─ Call smart contract IssueAsset function
   │  ├─ Fund with AssetIssuanceFund (10 COREUM)
   │  └─ Update asset: IsIssuedInSmartContract = true
   └─ If TRUE → Continue

3. User Whitelisting Check (Lazy)
   ├─ Query UserAssetList for user + asset
   ├─ If NO RECORD → Whitelist user
   │  ├─ Call smart contract SetHoldLimit (max uint128)
   │  └─ Create UserAssetList with WHITELISTED status
   ├─ If WHITELISTED → Continue
   └─ If OTHER STATUS → Block trading (return error)

4. Order Processing
   ├─ Validate notional limits
   ├─ Check available funds
   ├─ Create smart contract order message
   └─ Return signed transaction for user
```

**Unauthenticated:**

User:

- GET `/api/ordermgt/user/balances/get?account_addr=...` - retrieves the spendable balances for the given account address in the given network

**Authenticated:**

Order:

- POST `/api/ordermgt/order/create-tx` - Create a transaction (buy/sell/cancel) to be signed by the user
- POST `/api/ordermgt/order/submit-tx` - Submit a signed transaction to the blockchain
- GET `/api/ordermgt/order/get` - Retrieve order information from smart contract

## POST /api/ordermgt/order/purchase

Creates and submits a purchase order to the blockchain network directly for testing purposes.

> **Note:**

- This is a test only endpoint and should not be used in production.
- The `sender_keyring_key` must correspond to a keyring record configured in `KEYRING_CONFIGS` environment variable
- The sender(buyer) must be registered and whitelisted against the asset

### Example request

```bash
curl -X POST "https://com-be-order-management-service-808447315415.us-central1.run.app/api/ordermgt/order/purchase" \
-H "Network: testnet" \
-H "OrganizationID: sologenic" \
-H "Authorization: Bearer FirebaseJWT" \
-d '{
    "sender_keyring_key": "testcore1hphhtkzz0tmxqwvn74gm8d3r9770a0gc50fcgc",
    "smart_contract_address": "testcore1et29cek95pl0zralsf43u4uply0g9nmxnj7fyt9yfy74spch7fpq3f8j0e",
    "denom": "mstr_v1-testcore1et29cek95pl0zralsf43u4uply0g9nmxnj7fyt9yfy74spch7fpq3f8j0e",
    "amount": "1",
    "amount_exp": "0",
    "instruction": {
        "limit_price": "250",
        "limit_price_exp": "0",
        "options": {
            "order_type": "limit",               // "limit" | "market | stoploss"
            "fill_or_kill": false,               // true | false
            "expires_at": "2024-11-29T23:59:59Z" // Optional ISO-8601 timestamp
        }
    },
    "hold": {
        "denom": "utestcore",
        "amount": "1000",
        "amount_exp": "1"
    }
}'
```

### Example response

```json5
{
  tx_hash: "7FFE0685ACDFB25D298E596F4C4812D89649B802CA103F95054FD5F0C092211D",
  order_details: {
    id: 211,
    creator: "testcore1hphhtkzz0tmxqwvn74gm8d3r9770a0gc50fcgc",
    denom: "mstr_v1-testcore1et29cek95pl0zralsf43u4uply0g9nmxnj7fyt9yfy74spch7fpq3f8j0e",
    amount: "1",
    amount_exp: "0",
    instruction: {
      limit_price: "250",
      limit_price_exp: "0",
      options: "eyJvcmRlcl90eXBlIjoibGltaXQiLCJmaWxsX29yX2tpbGwiOmZhbHNlLCJleHBpcmVzX2F0IjoiMjAyNC0xMS0yOVQyMzo1OTo1OVoifQ==",
    },
    hold: {
      denom: "utestcore",
      amount: "1000",
      amount_exp: "1",
    },
    funds_sent: {
      denom: "utestcore",
      amount: "10000",
    },
    order_type: "purchase",
    order_state: "open",
    payment_state: null,
    amount_executed: null,
    amount_executed_exp: null,
    used_funds_amount: null,
    used_funds_amount_exp: null,
    costs: null,
    costs_exp: null,
  },
}
```

## GET /api/ordermgt/user/balances/get?account_addr=...

Retrieves the spendable balances for the given account address in the given network

### Example request

```bash
curl -X GET "https://com-be-order-management-service-808447315415.us-central1.run.app/api/ordermgt/user/balances/get?account_addr=testcore1hphhtkzz0tmxqwvn74gm8d3r9770a0gc50fcgc" \
-H "Content-Type: application/json" \
-H "Network: testnet"
```

### Example response

```json5
{
  balances: [
    {
      denom: "mstr_v1-testcore1et29cek95pl0zralsf43u4uply0g9nmxnj7fyt9yfy74spch7fpq3f8j0e",
      amount: "2",
    },
    {
      denom: "tesla_v1-testcore1et29cek95pl0zralsf43u4uply0g9nmxnj7fyt9yfy74spch7fpq3f8j0e",
      amount: "17900",
    },
    {
      denom: "utestcore",
      amount: "494619593",
    },
  ],
}
```

## POST /api/ordermgt/order/create-tx

Creates a transaction for buy, sell, or cancel orders that needs to be signed by the user. This endpoint supports multiple order types and returns the transaction hash and the transaction to be signed.

> **Note:**

- This is an authenticated endpoint requiring a valid Firebase JWT token
- The user must be registered and have sufficient balances for trade orders
- Asset issuance and user whitelisting are handled automatically (lazy approach)
- The response includes a message hash and the complete transaction to be signed

### Example request for Purchase Order

```bash
curl -X POST "http://localhost:8080/api/ordermgt/order/create-tx" \
-H "Content-Type: application/json" \
-H "Network: testnet" \
-H "OrganizationID: sologenic" \
-H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsImtpZCI6IjkyZTg4M2NjNDY2M2E2MzMyYWRhNmJjMWU0N2YzZmY1ZTRjOGI1ZDciLCJ0eXAiOiJKV1QifQ.eyJpc3MiOiJodHRwczovL3NlY3VyZXRva2VuLmdvb2dsZS5jb20vc2ctZGV4LXN0YWdlLTMyNjgxNyIsImF1ZCI6InNnLWRleC1zdGFnZS0zMjY4MTciLCJhdXRoX3RpbWUiOjE3NTYyNDU4NzUsInVzZXJfaWQiOiJLWGxDN0pzUW1YUDNBZVQ0MDR1R3RwSlRJQloyIiwic3ViIjoiS1hsQzdKc1FtWFAzQWVUNDA0dUd0cEpUSUJaMiIsImlhdCI6MTc1NjI0NTg3NSwiZXhwIjoxNzU2MjQ5NDc1LCJlbWFpbCI6InNvbG9nZW5pYy5hdXRvbWF0ZWRAbWFpbGluYXRvci5jb20iLCJlbWFpbF92ZXJpZmllZCI6dHJ1ZSwiZmlyZWJhc2UiOnsiaWRlbnRpdGllcyI6eyJlbWFpbCI6WyJzb2xvZ2VuaWMuYXV0b21hdGVkQG1haWxpbmF0b3IuY29tIl19LCJzaWduX2luX3Byb3ZpZGVyIjoicGFzc3dvcmQifX0.aV4nb6E4ZonDt8oh9I4sok0rG7mk8T2jPLouc4BzCmRPRSF2yWgH958dITMYCzhUFkdwOXcsv73hue_WxlQapS5grqYuiaCEQxtW88LAhuQGRKyGDKmPWUUAOXdw1fI1z9T937ulxy1j4RmPUju4KYNc5irvPbD-owShl9gcicU84iFtfS5JgZGpzUogfzT1OAj2zT0EmSdbfZXf81r27JepDdy33QVWG84bJxTPey5awOIxFmuR5g8N7DJUSF6auB1jGm5-kKZJCzFdU_9X7PbPr4RgP5tyvJUEu-IoQM1uYTfEb6iUJ5wUjOE_E682ziFYnUmC6lpec1HNvHWcYA" \
-d '{
    "order_type": "purchase",
    "denom": "suamzn_1-testcore13s2mmgg4uu4fn8mue6s3lgn74jwdupndjtqah8uxufugtajkeq2qgznc28",
    "amount": "1",
    "amount_exp": "0",
    "instruction": {
        "limit_price": "250",
        "limit_price_exp": "0",
        "options": {
            "order_type": "limit",
            "fill_or_kill": false,
            "expires_at": "2024-11-29T23:59:59Z"
        }
    },
    "hold": {
        "denom": "utestcore",
        "amount": "1000",
        "amount_exp": "1"
    }
}'
```

### Example response

```json5
{
  msg_hash: "a1b2c3d4e5f6789012345678901234567890abcdef",
  tx_to_sign: {
    sender: "testcore1hphhtkzz0tmxqwvn74gm8d3r9770a0gc50fcgc",
    contract: "testcore1et29cek95pl0zralsf43u4uply0g9nmxnj7fyt9yfy74spch7fpq3f8j0e",
    msg: "base64EncodedMessage",
    funds: [
      {
        denom: "utestcore",
        amount: "10000",
      },
    ],
  },
}
```

## POST /api/ordermgt/order/submit-tx

Submits a signed transaction to the blockchain. This endpoint accepts the signed transaction bytes and broadcasts them to the network.

> **Note:**

- This is an authenticated endpoint requiring a valid Firebase JWT token
- The transaction must be properly signed before submission
- Returns the transaction hash and order details upon successful submission

### Example request

```bash
curl -X POST "https://com-be-order-management-service-808447315415.us-central1.run.app/api/ordermgt/order/submit-tx" \
-H "Content-Type: application/json" \
-H "Network: testnet" \
-H "Authorization: Bearer FirebaseJWT" \
-d '{
    "TX": "base64EncodedSignedTransaction"
}'
```

### Example response

```json5
{
  tx_hash: "7FFE0685ACDFB25D298E596F4C4812D89649B802CA103F95054FD5F0C092211D",
  order_details: {
    id: 211,
    creator: "testcore1hphhtkzz0tmxqwvn74gm8d3r9770a0gc50fcgc",
    denom: "mstr_v1-testcore1et29cek95pl0zralsf43u4uply0g9nmxnj7fyt9yfy74spch7fpq3f8j0e",
    amount: "1",
    amount_exp: "0",
    instruction: {
      limit_price: "250",
      limit_price_exp: "0",
      options: "eyJvcmRlcl90eXBlIjoibGltaXQiLCJmaWxsX29yX2tpbGwiOmZhbHNlLCJleHBpcmVzX2F0IjoiMjAyNC0xMS0yOVQyMzo1OTo1OVoifQ==",
    },
    hold: {
      denom: "utestcore",
      amount: "1000",
      amount_exp: "1",
    },
    funds_sent: {
      denom: "utestcore",
      amount: "10000",
    },
    order_type: "purchase",
    order_state: "open",
    payment_state: null,
    amount_executed: null,
    amount_executed_exp: null,
    used_funds_amount: null,
    used_funds_amount_exp: null,
    costs: null,
    costs_exp: null,
  },
}
```

## GET /api/ordermgt/order/get?order_id=...&smart_contract_addr=...

Retrieves the order information for the given order ID from the given smart contract address.

> **Note:**

- This is an authenticated endpoint requiring a valid Firebase JWT token
- Requires both order_id and smart_contract_addr query parameters

### Example request

```bash
curl -X GET "https://com-be-order-management-service-808447315415.us-central1.run.app/api/ordermgt/order/get?order_id=201&smart_contract_addr=testcore1et29cek95pl0zralsf43u4uply0g9nmxnj7fyt9yfy74spch7fpq3f8j0e" \
-H "Content-Type: application/json" \
-H "Network: testnet" \
-H "Authorization: Bearer FirebaseJWT"
```

### Example response

```json5
{
  id: 201,
  creator: "testcore1hphhtkzz0tmxqwvn74gm8d3r9770a0gc50fcgc",
  denom: "mstr_v1-testcore1et29cek95pl0zralsf43u4uply0g9nmxnj7fyt9yfy74spch7fpq3f8j0e",
  amount: "1",
  amount_exp: "0",
  instruction: {
    limit_price: "250",
    limit_price_exp: "0",
    options: "eyJleHBpcmVzX2F0IjoiMjAyNC0xMS0yOVQyMzo1OTo1OVoiLCJvcmRlcl90eXBlIjoibGltaXQiLCJmaWxsX29yX2tpbGwiOmZhbHNlfQ==",
  },
  hold: {
    denom: "utestcore",
    amount: "1000",
    amount_exp: "1",
  },
  funds_sent: {
    denom: "utestcore",
    amount: "10000",
  },
  order_type: "purchase",
  order_state: "open",
  payment_state: null,
  amount_executed: null,
  amount_executed_exp: null,
  used_funds_amount: null,
  used_funds_amount_exp: null,
  costs: null,
  costs_exp: null,
}
```

## Application start parameters

The application uses the following environment parameters:

- `ACCOUNT_STORE` - the address for the account store service (e.g., "localhost:50051")
- `USER_STORE` - the address for the user store service (e.g., "localhost:50049")
- `ROLE_STORE` - the address for the role store service (e.g., "localhost:50066")
- `FEATURE_FLAG_STORE` - the address for the feature flag store service (e.g., "localhost:50061")
- `AUTH_FIREBASE_SERVICE` - the address for the Firebase authentication service (e.g., "localhost:50075")
- `OPEN_SEARCH_STORE` - the address for the OpenSearch store service (e.g., "localhost:50057")
- `ASSET_STORE` - the address for the asset store service (e.g., "localhost:50056")
- `WALLET_MIDDLEWARE` - the address for the wallet middleware service (e.g., "localhost:50088")
- `ORGANIZATION_STORE` - the address for the organization store service (e.g., "localhost:50062")
- `HTTP_CONFIG` - the configuration for the http server (JSON format with port, CORS settings, and timeouts)
- `NETWORKS` - the network configurations for WebSocket and gRPC connections (JSON format with network details)
- `SMART_CONTRACT_ADDRESS` - the smart contract address to interact with for order operations
- `SMART_CONTRACT_ISSUER_ADDRESS` - the smart contract issuer address
- `LOG_LEVEL` - the log level (e.g., "info", "debug", "error")
- `WUSDC_DENOM` - The denom of the WUSDC currency
- `COREUM_DENOM` - The denom of the Coreum currency for token issuance (e.g., "utestcore")
- `HASHING_SALT` - The salt for the hashing of the message to validate is the same
- `KEYRING_CONFIGS` - JSON array of keyring configurations for different networks and key names
