# Asset service

The Asset service provides the following RESTFUL interface for normal users:

### Normal User Interfaces:

**Authenticated:**

Asset:

* GET `/asset/get?asset_key=...`: Returns an asset  
* GET `/asset/list?offset=...&jurisdiction_ids=...&asset_type=...&exchange_ticker_symbol=...&exchange=...`: Returns a list of all available(listed, allowed) assets based on the user's jurisdiction 

User Asset List:

* GET `/asset/user/get?user_asset_list_key`=...: Returns a user asset list entry
* GET `/asset/user/list?account_id=...&offset=...&wallet=...&asset_key=...&status=...&visible=...`: returns a list of assets that the user has specifically bookmarked and whitelisted for viewing and trading
* POST `/asset/user/add`: Adds an asset to the user's list
* PUT `/asset/user/update/status?user_asset_list_key=...&status=...`: Updates the status of an asset in the user's list
* PUT `/asset/user/update/visible?user_asset_list_key=...&visible=false`: Updates the visibility of an asset in the user's list

## GET /asset/get?asset_key=...

Returns an asset if the user is authorized to view it based on their jurisdiction

> **Note**: An asset will only be returned if its status is `LISTED` and at least one of its JurisdictionIDs is permitted for the user, as determined by the user's allowed jurisdictions in the UserJurisdiction database.

### Example call

```bash
curl -X GET \
  https://com-be-asset-service-dfjiao-ijgao.a.run.app/asset/get?asset_key=BTC-34422ce6-7b51-4a15-accb-34959f39d8ca-1 \
    -H "Content-Type: application/json" \
    -H "Network: testnet" \ 
    -H "Authorization: Bearer: ...."
```

### Example response

```json
{
  "ID": "BTC-34422ce6-7b51-4a15-accb-34959f39d8ca-2",
  "OrganizationID": "34422ce6-7b51-4a15-accb-34959f39d8ca",
  "Status": 3,
  "JurisdictionIDs": [
    "j1",
    "j2",
    "j3"
  ],
  "Network": "testnet",
  "CreatedAt": {
    "seconds": 1724092846
  },
  "UpdatedAt": {
    "seconds": 1724268537
  },
  "Type": 1,
  "Symbol": "btc-test",
  "Currency": "BTC",
  "Version": "2",
  "Precision": 5,
  "Name": "BTC-test-asset2",
  "ExchangeTickerSymbol": "BTC",
  "Exchange": "copying",
  "Description": "BTC Test 6"
}
```

## GET /asset/list?offset=...&jurisdiction_ids=...&asset_type=...&exchange_ticker_symbol=...&exchange=...

Returns a list of all available (listed and allowed) assets that the user is authorized to view based on their jurisdiction.

> **Note**: Only assets with a status of `LISTED` and whose `JurisdictionIDs` include at least one of the user's permitted jurisdictions (allowed UserJurisdiction record) as retrieved from the `UserJurisdiction` database will be returned.

### Example call

```bash
curl -X GET \
  https://com-be-asset-service-dfjiao-ijgao.a.run.app/asset/list?&jurisdiction_ids=abc,bce,j1&exchange_ticker_symbol=BTC&exchange=copying&offset=0&asset_type=STOCK \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer: ...." \
  -H "Network: testnet"
```


### Example response

```json
[
  {
    "ID": "BTC-34422ce6-7b51-4a15-accb-34959f39d8ca-2",
    "OrganizationID": "34422ce6-7b51-4a15-accb-34959f39d8ca",
    "Status": 3,
    "JurisdictionIDs": [
      "j1",
      "j2",
      "j3"
    ],
    "Network": "testnet",
    "CreatedAt": {
      "seconds": 1724092846
    },
    "UpdatedAt": {
      "seconds": 1724268537
    },
    "Type": 1,
    "Symbol": "btc-test",
    "Currency": "BTC",
    "Version": "2",
    "Precision": 5,
    "Name": "BTC-test-asset2",
    "ExchangeTickerSymbol": "BTC",
    "Exchange": "copying",
    "Description": "BTC Test 6"
  }
]
```

## GET /asset/user/get?user_asset_list_key=...

Returns a user asset list entry

### Example call

```bash
curl -X GET \
  https://com-be-asset-service-dfjiao-ijgao.a.run.app/asset/user/get?user_asset_list_key=BTC-BTC-34422ce6-7b51-4a15-accb-34959f39d8ca-2-ddb9e00a-147b-4fdd-b9bc-0a79b01202d2 \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer: ...." 
    -H "Network: testnet" \ 
```

### Example response

```json
{
  "AccountID": "ddb9e00a-147b-4fdd-b9bc-0a79b01202d2",
  "Wallet": "123",
  "AssetKey": "BTC-34422ce6-7b51-4a15-accb-34959f39d8ca-2",
  "Status": 3,
  "Network": "testnet",
  "Visible": true,
  "CreatedAt": {
    "seconds": 1724199574
  },
  "UpdatedAt": {
    "seconds": 1724263252
  }
}
```

## GET /asset/user/list?account_id=...&offset=...&wallet=...&asset_key=...&status=...&visible=...

Returns a list of assets that the user has specifically bookmarked and whitelisted for viewing and trading

### Example call

```bash
curl -X GET \
  https://com-be-asset-service-dfjiao-ijgao.a.run.app/asset/user/list?account_id=ddb9e00a-147b-4fdd-b9bc-0a79b01202d2&visible=true&status=LISTED&wallet=123&offset=0&asset_key=BTC-34422ce6-7b51-4a15-accb-34959f39d8ca-3 \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer: ...."
    -H "Network: testnet" \ 
```

### Example response

```json
[
  {
    "AccountID": "ddb9e00a-147b-4fdd-b9bc-0a79b01202d2",
    "Wallet": "123",
    "AssetKey": "BTC-34422ce6-7b51-4a15-accb-34959f39d8ca-3",
    "Status": 4,
    "Network": "testnet",
    "Visible": true,
    "CreatedAt": {
      "seconds": 1724198166
    }
  }
]
```

## POST /asset/user/add

Adds an asset to the user's list

### Example call

```bash
curl -X POST "https://com-be-asset-service-dfjiao-ijgao.a.run.app/asset/user/add" \
-H "Content-Type: application/json" \
-H "Authorization: Bearer: ...." \
-H "Network: testnet" \
-d '{
  "AccountID": "ddb9e00a-147b-4fdd-b9bc-0a79b01202d2",
  "Wallet": "123",
  "AssetKey": "BTC-34422ce6-7b51-4a15-accb-34959f39d8ca-2",
  "Status": 3,
  "Network": "testnet",
  "Visible": true
}'
```

### Example response

```json
{
  "Key": "BTC-34422ce6-7b51-4a15-accb-34959f39d8ca-2-ddb9e00a-147b-4fdd-b9bc-0a79b01202d2"
}
```

## PUT /asset/user/update/status?user_asset_list_key=...&status=...

Updates the status of an asset in the user's list

### Example call

```bash
curl -X PUT "https://com-be-asset-service-dfjiao-ijgao.a.run.app/asset/user/update/status?user_asset_list_key=BTC-34422ce6-7b51-4a15-accb-34959f39d8ca-2-ddb9e00a-147b-4fdd-b9bc-0a79b01202d2&status=WHITELISTING_REQUESTED" \
-H "Content-Type: application/json"
-H "Authorization: Bearer: ...." \
-H "Network: testnet"
```

### Example response

```json
{
  "Key": "BTC-34422ce6-7b51-4a15-accb-34959f39d8ca-2-ddb9e00a-147b-4fdd-b9bc-0a79b01202d2"
}
```

## PUT /asset/user/update/visible?user_asset_list_key=...&visible=false

Updates the visibility of an asset in the user's list

### Example call

```bash
curl -X PUT "https://com-be-asset-service-dfjiao-ijgao.a.run.app/asset/user/update/visible?user_asset_list_key=BTC-34422ce6-7b51-4a15-accb-34959f39d8ca-2-ddb9e00a-147b-4fdd-b9bc-0a79b01202d2&visible=true" \
-H "Content-Type: application/json"
-H "Authorization: Bearer: ...." \
-H "Network: testnet"
```

### Example response

```json
{
  "Key": "BTC-34422ce6-7b51-4a15-accb-34959f39d8ca-2-ddb9e00a-147b-4fdd-b9bc-0a79b01202d2"
}
```

## Start parameters

The following start parameters are required:

* `HTTP_CONFIG`: the configuration for the http server (included from `github.com/sologenic/com-be-http-lib/`)
* `AUTH_FIREBASE_SERVICE`: the firebase authentication service endpoint
* `ADMIN_ACCOUNT_STORE`: the account store endpoint (included from `github.com/sologenic/com-be-admin-account-store/`)
* `ACCOUNT_STORE`: the admin account store endpoint (included from `github.com/sologenic/com-be-account-store/`)
* `ASSET_STORE`: the asset store endpoint (included from `github.com/sologenic/com-be-asset-store/`)
* `JURISDICTION_STORE`: the jurisdiction store endpoint (included from `github.com/sologenic/com-be-jurisdiction-store/`)
* `ORGANIZATION_STORE`: Included from github.com/sologenic/com-fs-organization-model. For setup see github.com/sologenic/com-fs-organization-model/client/README.md
