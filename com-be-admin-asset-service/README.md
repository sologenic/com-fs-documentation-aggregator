# Admin Asset service

The Asset service provides the following RESTFUL interface:

### Admin Asset Interfaces for Organizations:

> **Note:** All admin endpoints are protected by role-based access control and default to the `BROKER_ASSET_ADMINISTRATOR` role. Permissions are managed dynamically by Organization administrators on the fly.

**Authenticated:**

Asset Management:

- GET `/api/asset/list?offset=...&jurisdiction_ids=...&status=...&asset_type=...&exchange_ticker_symbol=...&exchange=...&symbol=...&version=...&issuer=...&industry=...`: Returns a list of all assets in the organization
- POST `/api/asset/create`: Creates a new asset for the organization(version 1)
- POST `/api/asset/create/version`: Creates a new version of an existing asset (version 2, 3, ~ 999)
- PUT `/api/asset/update`: Updates an existing asset
- PUT `/api/asset/update/status?asset_key=...&status=...&reason=...`: Updates the status of an asset

## GET /api/asset/list?offset=...&jurisdiction_ids=...&status=...&asset_type=...&exchange_ticker_symbol=...&exchange=...&symbol=...&version=...&issuer=...&industry=...

Returns a list of all assets in the organization

### Example call

```bash
curl -X GET \
  "https://com-be-admin-asset-service-dfjiao-ijgao.a.run.app/api/asset/list?\
jurisdiction_ids=8878d73b-edff-40a1-b6ce-43a4508aa96a,5bc32483-d6a6-3a8e-8a23-65c5e483b0f0&\
issuer=testcore1j974n26f48wgt4dpcxryrakrnkg433yznff29v&\
offset=0" \
  -H "Content-Type: application/json" \
  -H "Network: testnet" \
  -H "OrganizationID: 72c4c072-2fe4-4f72-ae9d-d9d52a05fd71" \
  -H "Authorization: Bearer: ...." \
```

### Example response

```json
{
  "Assets": [
    {
      "AssetDetails": {
        "ID": "dgly_1_72c4c072-2fe4-4f72-ae9d-d9d52a05fd71_testcore1j974n26f48wgt4dpcxryrakrnkg433yznff29v",
        "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
        "Status": 2,
        "Type": 9,
        "Denom": {
          "Currency": {
            "Symbol": "DGLY",
            "Version": "1"
          },
          "Subunit": "sudgly_1",
          "Description": "Digital Ally, Inc. Common Stock"
        },
        "SmartContractIssuerAddr": "testcore1j974n26f48wgt4dpcxryrakrnkg433yznff29v",
        "EquityDetails": {
          "ExchangeTickerSymbol": "DGLY",
          "Exchange": "NASDAQ",
          "MinTransactionAmount": 1,
          "TradingMarginPercentage": 0.1,
          "AssetMarginPercentage": 0.5
        },
        "FinancialProperties": {
          "Symbol": "DGLY",
          "Issuer": "testcore1j974n26f48wgt4dpcxryrakrnkg433yznff29v",
          "JurisdictionIDs": ["8878d73b-edff-40a1-b6ce-43a4508aa96a"],
          "Network": 2
        },
        "Description": {
          "Name": "Digital Ally, Inc. Common Stock"
        }
      },
      "MetaData": {
        "Network": 2,
        "UpdatedAt": {
          "seconds": 1759786658,
          "nanos": 453447000
        },
        "CreatedAt": {
          "seconds": 1759786658,
          "nanos": 453437000
        }
      },
      "Audit": {
        "ChangedBy": "API_TOKEN_72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
        "ChangedAt": {
          "seconds": 1759786658,
          "nanos": 453447000
        }
      }
    }
  ],
  "Offset": 20
}
```

## POST /api/asset/create

Creates a new asset

### Example call

```bash
curl -X POST "https://com-be-admin-asset-service-dfjiao-ijgao.a.run.app/api/asset/create" \
-H "Content-Type: application/json" \
-H "Network: testnet" \
-H "Authorization: Bearer: ...." \
-d '{
  "AssetDetails": {
    "OrganizationID": "215a551d-5691-91ce-f4a6-9284f40d1340",
    "JurisdictionIDs": [
      "8ac93283-d8a6-4a8e-8c01-67c5e106b0f0"
    ],
    "Type": 1,
    "Name": "Tesla",
    "ExchangeTickerSymbol": "TSLA",
    "Exchange": 1,
    "InternalDescription": "TSLA Token Version 2",
    "MinTransactionAmount": 1,
    "ExtraPercentage": 0.05,
    "Industry": 1,
    "Denom": {
      "Currency": {
        "Symbol": "TSLA",
        "Version": "2"
      },
      "Precision": 6,
      "Description": "Tesla TOKEN Version 2"
    }
  },
  "Audit": {
    "Reason": "new version"
  }
}'
```

### Example response

```json
{
  "Key": "aapl_1_215a551d-5691-91ce-f4a6-9284f40d1340"
}
```

## POST /api/asset/create/version

Creates a new version of an existing asset. The previous version will be automatically marked as outdated (status: OUTDATED_ASSET_VERSION).

> **Note:**
>
> - New version must be exactly 1 greater than the current active version
> - Only the following fields can be modified in the new version: `JurisdictionIDs`, `Name`, `InternalDescription`, `MinTransactionAmount`, `ExtraPercentage`, `Precision`, `Denom.Description`
> - When creating version N, version N-1 will be automatically marked as OUTDATED_ASSET_VERSION
> - Previous version must exist and be in LISTED status

### Example call

```bash
curl -X POST "https://com-be-admin-asset-service-dfjiao-ijgao.a.run.app/api/asset/create/version" \
-H "Content-Type: application/json" \
-H "Network: testnet" \
-H "Authorization: Bearer: ...." \
-d '{
  "AssetDetails": {
    "OrganizationID": "215a551d-5691-91ce-f4a6-9284f40d1340",
    "JurisdictionIDs": [
      "8ac93283-d8a6-4a8e-8c01-67c5e106b0f0"
    ],
    "Type": 1,
    "Name": "Tesla",
    "ExchangeTickerSymbol": "TSLA",
    "Exchange": 1,
    "InternalDescription": "TSLA Token Version 3",
    "MinTransactionAmount": 1,
    "ExtraPercentage": 0.1,
    "Industry": 4,
    "Denom": {
      "Currency": {
        "Symbol": "TSLA",
        "Version": "3" // must be exactly 1 greater than the current active version
      },
      "Precision": 6,
      "Description": "Tesla TOKEN Version 3"
    }
  },
}'
```

### Example response

```json
{
  "Key": "tsla_3_215a551d-5691-91ce-f4a6-9284f40d1340"
}
```

## PUT /api/asset/update

Updates an existing asset

> **Note**:

- The following fields will not be updated: `CreatedAt`, `Network`, `OrganizationID`, `Type`, `Currency`, `Subunit`, `Precision`, `Description`
- `Status` can only be updated through the `/api/asset/update/status` endpoint

### Example call

```bash
curl -X PUT "https://com-be-admin-asset-service-dfjiao-ijgao.a.run.app/api/asset/update" \
-H "Content-Type: application/json" \
-H "Network: testnet" \
-H "Authorization: Bearer: ...." \
-d '{
  "AssetDetails": {
    "ID": "aapl_1-215a551d-5691-91ce-f4a6-9284f40d1340",
    "OrganizationID": "215a551d-5691-91ce-f4a6-9284f40d1340",
    "JurisdictionIDs": [
      "8ac93283-d8a6-4a8e-8c01-67c5e106b0f0"
    ],
    "Status": 2,
    "Name": "APPLE",
    "ExchangeTickerSymbol": "AAPL",
    "Exchange": 1,
    "InternalDescription": "APPLE Token Version 1 updated",
    "MinTransactionAmount": 1,
    "ExtraPercentage": 0.2,
    "Industry": 3,
    "Denom": {
      "Currency": {
        "Symbol": "AAPL",
        "Version": "1"
      }
    }
  },
  "Audit": {
    "Reason": "Update Test"
  }
}'
```

### Example response

```json
{
  "Key": "aapl_1_215a551d-5691-91ce-f4a6-9284f40d1340"
}
```

## PUT /api/asset/update/status?asset_key=...&status=...&reason=...

Updates the status of an asset

### Example call

```bash
curl -X PUT "https://com-be-admin-asset-service-dfjiao-ijgao.a.run.app/api/asset/update/status?asset_key=aapl_1-215a551d-5691-91ce-f4a6-9284f40d1340&status=2&reason=Asset%20no%20longer%20supported" \
-H "Content-Type: application/json" \
-H "Network: testnet" \
-H "Authorization: Bearer: ...." \
```

### Example response

```json
{
  "Key": "aapl_1_215a551d-5691-91ce-f4a6-9284f40d1340"
}
```

## Start parameters

The following start parameters are required:

### Service Endpoints

- `HTTP_CONFIG`: the configuration for the http server (included from `github.com/sologenic/com-be-http-lib/`)
- `AUTH_FIREBASE_SERVICE`: the firebase authentication service endpoint
- `ACCOUNT_STORE`: the admin account store endpoint (included from `github.com/sologenic/com-be-admin-account-store/`)
- `FEATURE_FLAG_STORE`: the feature flag store endpoint (included from `github.com/sologenic/com-be-feature-flag-store/`)
- `FILE_STORE`: the file store endpoint (included from `github.com/sologenic/com-be-file-store/`)
- `ROLE_STORE`: the role store endpoint (included from `github.com/sologenic/com-be-admin-role-store/`)
- `ORGANIZATION_STORE`: the organization store endpoint (included from `github.com/sologenic/com-be-organization-store/`)
- `ASSET_STORE`: the asset store endpoint (included from `github.com/sologenic/com-be-asset-store/`)
- `CERTIFICATE_STORE`: the certificate store endpoint

### Smart Contract Configuration

- `REGISTRY_CONTRACT_ADDRESS`: the registry smart contract address on the Coreum network
- `ATG_BROKER_CONTRACT_ADDRESS`: the main smart contract address of the ATG Broker contract on the Coreum network
- `ASSET_EXTENSION_CODE`: the asset extension code for Coreum assets
- `ORDER_HUB_CONTRACT_ADDRESS`: the order hub smart contract address on the Coreum network
- `CROWDFUND_CONTRACT_CODE_ID`: the crowdfunding smart contract code ID on the Coreum network

### Blockchain Configuration

- `KEYRING_CONFIGS`: JSON array of keyring configurations containing network, keyname, and mnemonic for smart contract interactions
- `NETWORKS`: JSON object containing WebSocket and gRPC network configurations for Coreum blockchain connectivity

### Application Configuration

- `LOG_LEVEL`: the logging level for the application (e.g., "info", "debug", "error")
