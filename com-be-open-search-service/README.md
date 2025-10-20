# Open Search Service

The Open Search service provides API interfaces for interacting with the Elasticsearch system.

## API Endpoints Overview

**Authentication Required:**

* GET `/api/search/assets?query=...&page=...&order_by=...&dir=...` - Search for assets
* GET `/api/search/orders?query=...&page=...&order_by=...&dir=...` - Search for orders

## Query Structure

The query parameter used in the search endpoints is a base64 encoded string representing `Query` objects as defined in the `fs-open-search-model` repository. The query structure is defined as follows:

```protobuf
message Query {
    string Field = 1;
    string Value = 2;
    Operator Operator = 3;
    FieldType FieldType = 4;
    repeated Query Query = 5;
}
```

For more details on the `Query` object structure and available operators, refer to `github.com/sologenic/fs-open-search-model`.

## Usage

To use the search endpoints, construct a `Query` object and encode it in base64 before passing it as the query parameter. The service's flexible design allows you to build complex queries tailored to your application's specific requirements.

### Field Names

When constructing queries, field names must match exactly as they appear in the protobuf model definitions. For nested fields, use dot notation to access the desired property. For example, when searching assets, use `AssetDetails.Status` to access the status field within the AssetDetails object. For flat structures, reference fields directly by their model name.

### Query Logic

>***Query objects seen below are simplified for clarity.***

**AND Logic** is achieved by placing multiple query conditions in a single array. All conditions must be satisfied for a match:

```ts
{
    Query: [
        { Field: "AssetDetails.OrganizationID", Value: "org-123" },
        { Field: "AssetDetails.Status", Value: "3" },
        { Field: "AssetDetails.Type", Value: "1" },
    ]
}
```

This searches for assets where the organization ID equals "org-123" **AND** the status equals 3 **AND** the type equals 1. All three conditions must be true.

**OR Logic** is implemented by nesting query conditions within a parent query. Only one of the nested conditions needs to be satisfied:

```ts
{
    Query: [
        {
            Field: "AssetDetails.OrganizationID",
            Value: "org-123",
            Query: [  // Nested queries create OR logic
                { Field: "AssetDetails.Status", Value: "3" },
                { Field: "AssetDetails.Type", Value: "1" },
            ],
        },
    ]
}
```

This searches for assets where the organization ID equals "org-123" **AND** either the status equals 3 **OR** the type equals 1. The parent condition must be true, but only one of the nested conditions needs to match.

**Security Note:** For security purposes, when account IDs or organization IDs need to be included in queries, the backend will automatically append these identifiers to your query rather than accepting them directly from the client. ***Therefore, omit these fields from the query you send.***

> ***For a more technical review of how the `AND` and `OR` logic works, refer to the `query` function (line 313) and the `queryParse` function (line 382) in `github.com/sologenic/be-open-search-store/store/store.go`***

### Sorting

Results can be sorted using the `order_by` and `dir` parameters. Use `order_by` to specify which field to sort by, and `dir` to control the direction (`0` for ascending, `1` for descending). If you specify a direction with `dir`, you MUST provide an `order_by` field. However, if you only specify the `order_by` field, a default sorting direction of descending will be applied. The field name must exactly match what's defined in the model for the data you're searching.

For timestamp fields (timestamppb), specify the subfield (`seconds` or `nanos`) in lowercase: `MetaData.UpdatedAt.seconds` or `CreatedAt.nanos`. This is required because our indexes store these subfields in lowercase, even though they appear as camelCase in the protobuf definitions.

### Pagination

Hits per page is always set to `20` for consistency. Page (`page`) is set to `0` by default.

## Parameters

- `query`: Base64-encoded `Query` object containing the search criteria.
- `page`: The page number for pagination (default is 0).
- `order_by`: The field name to order the results by (default is `UpdatedAt`). **Note:** The field must exist in the model definition for the data being searched. For example, if a field like `CreatedAt` doesn't exist in the model, sorting on that field will fail.
- `dir`: The sorting direction: `0` for asc, `1` for desc. Default is desc.

## GET `/api/search/assets?query=...&page=...&order_by=...&dir=...`

Search for assets using the provided query parameters and pagination.

### Example request

```bash
curl -X GET \
"https://comsolotex.sologenic.org/api/search/assets?query=Insib3Blc...R5cGUiOiAwfV19Ig==&page=1&order_by=UpdatedAt&dir=1" \
-H "Authorization: Bearer ..." \
-H "OrganizationID: ..." \
-H "Network: mainnet"
```

### Example response

```json
{
    "Results": [
        {
            "Data": "{\"AssetDetails\":{\"ID\":\"gnhm_1_9a81fae8-f245-4c47-be94-b7d683dff6ef_testcore1mp52apfwjgpd4fkqqf8lkysy9pwnhkm9kxqrvp\",\"OrganizationID\":\"9a81fae8-f245-4c47-be94-b7d683dff6ef\",\"Status\":2,\"Type\":1,\"Name\":\"Trinidad and Tobago Dollar\",\"ExchangeTickerSymbol\":\"TRIN\",\"Exchange\":2,\"InternalDescription\":\"Compatible sustainable monitoring\",\"MinTransactionAmount\":68566,\"TradingMarginPercentage\":0.213,\"LogoFile\":{\"Reference\":\"testnet/logo/9a81fae8-f245-4c47-be94-b7d683dff6ef/TRIN_2\",\"Extension\":\"image/jpeg\",\"Name\":\"logoo.jpeg\"},\"AssetMarginPercentage\":0.03,\"Denom\":{\"Currency\":{\"Symbol\":\"GNHM\",\"Version\":\"1\"},\"Subunit\":\"sugnhm_1\",\"Precision\":4,\"Description\":\"A withdrawal of TOP 498.30 occurred at Gleason, Gulgowski and Rolfson using a card ending in ****5722 for account ***9041.\"},\"SmartContractIssuerAddr\":\"testcore1mp52apfwjgpd4fkqqf8lkysy9pwnhkm9kxqrvp\"},\"MetaData\":{\"Network\":2,\"UpdatedAt\":{\"seconds\":1752219576,\"nanos\":920473992},\"CreatedAt\":{\"seconds\":1752219576,\"nanos\":920450906}},\"Audit\":{\"ChangedBy\":\"iso-admin@mailinator.org\",\"ChangedAt\":{\"seconds\":1752219576,\"nanos\":920474402}}}",
            "CreatedAt": {
                "nanos": 368224824
            },
            "UpdatedAt": {
                "nanos": 368224954
            },
            "Key": "gnhm_1_9a81fae8-f245-4c47-be94-b7d683dff6ef_testcore1mp52apfwjgpd4fkqqf8lkysy9pwnhkm9kxqrvp",
            "Network": "testnet"
        },
        {
            "Data": "{\"AssetDetails\":{\"ID\":\"nwyb_1_9a81fae8-f245-4c47-be94-b7d683dff6ef_testcore1mp52apfwjgpd4fkqqf8lkysy9pwnhkm9kxqrvp\",\"OrganizationID\":\"9a81fae8-f245-4c47-be94-b7d683dff6ef\",\"Status\":3,\"JurisdictionIDs\":[\"08318fb5-a698-48e0-b970-96ca7ec8ea77\"],\"Type\":3,\"Name\":\"Afghani\",\"ExchangeTickerSymbol\":\"TRIN\",\"Exchange\":2,\"InternalDescription\":\"A withdrawal for SEK 82.07 was made at Dickinson, Herzog and Hodkiewicz via card ending ****2036 on account ***7954.\",\"MinTransactionAmount\":43208,\"TradingMarginPercentage\":0.24989999999999998,\"LogoFile\":{\"Reference\":\"testnet/logo/9a81fae8-f245-4c47-be94-b7d683dff6ef/TRIN_2\",\"Extension\":\"image/jpeg\",\"Name\":\"logoo.jpeg\"},\"AssetMarginPercentage\":0.05,\"Denom\":{\"Currency\":{\"Symbol\":\"NWYB\",\"Version\":\"1\"},\"Subunit\":\"sunwyb_1\",\"Precision\":6,\"Description\":\"A payment for MKD 719.80 was made at Zulauf Inc via card ending ****3597 on account ***6552.\"},\"SmartContractIssuerAddr\":\"testcore1mp52apfwjgpd4fkqqf8lkysy9pwnhkm9kxqrvp\"},\"MetaData\":{\"Network\":2,\"UpdatedAt\":{\"seconds\":1752247313,\"nanos\":595891949},\"CreatedAt\":{\"seconds\":1750957277,\"nanos\":238914594}},\"Audit\":{\"ChangedBy\":\"iso-admin@mailinator.org\",\"ChangedAt\":{\"seconds\":1752247313,\"nanos\":595892129},\"Reason\":\"invoice at Kautzer Group with a card ending in ****4467 for BAM 647.21 from account ***8349.\"}}",
            "CreatedAt": {
                "nanos": 269335443
            },
            "UpdatedAt": {
                "nanos": 269335633
            },
            "Key": "nwyb_1_9a81fae8-f245-4c47-be94-b7d683dff6ef_testcore1mp52apfwjgpd4fkqqf8lkysy9pwnhkm9kxqrvp",
            "Network": "testnet"
        },
        {
            "Data": "{\"AssetDetails\":{\"ID\":\"ojka_1_9a81fae8-f245-4c47-be94-b7d683dff6ef_testcore1mp52apfwjgpd4fkqqf8lkysy9pwnhkm9kxqrvp\",\"OrganizationID\":\"9a81fae8-f245-4c47-be94-b7d683dff6ef\",\"Status\":2,\"Type\":1,\"Name\":\"Swedish Krona\",\"ExchangeTickerSymbol\":\"SWED\",\"Exchange\":1,\"InternalDescription\":\"Triple-buffered homogeneous framework\",\"MinTransactionAmount\":56748,\"TradingMarginPercentage\":0.175,\"LogoFile\":{\"Reference\":\"testnet/logo/9a81fae8-f245-4c47-be94-b7d683dff6ef/SWED_1\",\"Extension\":\"image/jpeg\",\"Name\":\"logoo.jpeg\"},\"AssetMarginPercentage\":0.01,\"Denom\":{\"Currency\":{\"Symbol\":\"OJKA\",\"Version\":\"1\"},\"Subunit\":\"suojka_1\",\"Precision\":3,\"Description\":\"Your invoice of CNY 979.10 at Barrows Inc was successful. Charged via card ****3967 to account ***4133.\"},\"SmartContractIssuerAddr\":\"testcore1mp52apfwjgpd4fkqqf8lkysy9pwnhkm9kxqrvp\"},\"MetaData\":{\"Network\":2,\"UpdatedAt\":{\"seconds\":1752249670,\"nanos\":220152191},\"CreatedAt\":{\"seconds\":1752249670,\"nanos\":220124578}},\"Audit\":{\"ChangedBy\":\"iso-admin@mailinator.org\",\"ChangedAt\":{\"seconds\":1752249670,\"nanos\":220152591}}}",
            "CreatedAt": {
                "nanos": 65622455
            },
            "UpdatedAt": {
                "nanos": 65622665
            },
            "Key": "ojka_1_9a81fae8-f245-4c47-be94-b7d683dff6ef_testcore1mp52apfwjgpd4fkqqf8lkysy9pwnhkm9kxqrvp",
            "Network": "testnet"
        },
        // ...
    ],
    "Total": 100
}
```

## GET `/api/search/orders?query=...&page=...&order_by=...&dir=...`

Search for orders using the provided query parameters and pagination.

### Example request

```bash
curl -X GET \
"https://comsolotex.sologenic.org/api/search/orders?query=Insib3Blc...R5cGUiOiAwfV19Ig==&page=2" \
-H "Authorization: Bearer ..." \
-H "OrganizationID: ..." \
-H "Network: mainnet"
```

### Example response

```json
{
    "Results": [
        {
            "Data": "{\"Network\":2,\"SmartContractAddr\":\"testcore13s2mmgg4uu4...qah8uxufugtajkeq2qgznc28\",\"Instruction\":{\"OrderID\":1234,\"Creator\":\"testcore15ms2zan4z3y9d29rxsyan5v5vecc04ycts2c85\",\"Denom\":\"suaapl_1-testcore13s2mmgg4uu4...qah8uxufugtajkeq2qgznc28\",\"Amount\":100,\"AmountExp\":2,\"LimitPrice\":15000,\"LimitPriceExp\":2,\"OrderDetailType\":2,\"OrderType\":1},\"CreatedAt\":{\"seconds\":1752191886,\"nanos\":414533000},\"UpdatedAt\":{\"seconds\":1752191889,\"nanos\":202828000},\"TransactionType\":1,\"TXID\":\"someTXID-123456-test\",\"GasFee\":1000000,\"DetectedAt\":{\"seconds\":1752191886,\"nanos\":414534000},\"Height\":123456,\"InternalOrderState\":6,\"BrokerOrderDetails\":{\"BrokerAssignedID\":\"someBrokerID-123456-test\",\"ClientOrderID\":{\"Network\":2,\"SmartContractAddr\":\"testcore13s2mmgg4uu4...qah8uxufugtajkeq2qgznc28\",\"OrderID\":123},\"SubmittedAt\":{\"seconds\":1752191886,\"nanos\":414521000},\"FilledAt\":{\"seconds\":1752191886,\"nanos\":414522000},\"ExpiredAt\":{\"seconds\":1752191886,\"nanos\":414523000},\"CancelledAt\":{\"seconds\":1752191886,\"nanos\":414524000},\"AssetID\":\"someAssetID-123456-test\",\"Symbol\":\"suaapl_1-testcore13s2mmgg4uu4...qah8uxufugtajkeq2qgznc28\",\"AssetClass\":1,\"OrderClass\":1,\"Type\":1,\"Side\":1,\"TimeInForce\":1,\"CreatedAt\":{\"seconds\":1752191886,\"nanos\":414524000},\"UpdatedAt\":{\"seconds\":1752191886,\"nanos\":414525000},\"Status\":2,\"PartialPrice\":{\"Value\":15000,\"Exp\":2},\"PartialQty\":{\"Value\":100,\"Exp\":2}},\"InstanceID\":\"someInstanceID-123456-test\",\"BlockTime\":{\"seconds\":1752191886,\"nanos\":414535000},\"Sequence\":123,\"OrganizationID\":\"72c4c072-2fe4-4f72-ae9d-d9d52a05fd71\",\"UserID\":\"test@test.com\"}",
            "CreatedAt": {
                "nanos": 771019217
            },
            "UpdatedAt": {
                "nanos": 771019507
            },
            "Key": "someTXID-123456-test",
            "Network": "testnet"
        },
        {
            "Data": "{\"Network\":2,\"SmartContractAddr\":\"testcore13s2mmgg4uu4...qah8uxufugtajkeq2qgznc28\",\"Instruction\":{\"OrderID\":2345,\"Creator\":\"testcore15ms2zan4z3y9d29rxsyan5v5vecc04ycts2c85\",\"Denom\":\"suamzn_1-testcore13s2mmgg4uu4...qah8uxufugtajkeq2qgznc28\",\"Amount\":100,\"AmountExp\":2,\"LimitPrice\":15000,\"LimitPriceExp\":2,\"OrderDetailType\":2,\"OrderType\":1},\"CreatedAt\":{\"seconds\":1752192138,\"nanos\":173216000},\"UpdatedAt\":{\"seconds\":1752192144,\"nanos\":110307000},\"TransactionType\":1,\"TXID\":\"someTXID-123456-test2\",\"GasFee\":1000000,\"DetectedAt\":{\"seconds\":1752192138,\"nanos\":173217000},\"Height\":123456,\"InternalOrderState\":6,\"BrokerOrderDetails\":{\"BrokerAssignedID\":\"someBrokerID-123456-test2\",\"ClientOrderID\":{\"Network\":2,\"SmartContractAddr\":\"testcore13s2mmgg4uu4...qah8uxufugtajkeq2qgznc28\",\"OrderID\":456},\"SubmittedAt\":{\"seconds\":1752192138,\"nanos\":173210000},\"FilledAt\":{\"seconds\":1752192138,\"nanos\":173211000},\"ExpiredAt\":{\"seconds\":1752192138,\"nanos\":173212000},\"CancelledAt\":{\"seconds\":1752192138,\"nanos\":173212000},\"AssetID\":\"someAssetID-123456-test2\",\"Symbol\":\"suamzn_1-testcore13s2mmgg4uu4...qah8uxufugtajkeq2qgznc28\",\"AssetClass\":1,\"OrderClass\":1,\"Type\":1,\"Side\":1,\"TimeInForce\":1,\"CreatedAt\":{\"seconds\":1752192138,\"nanos\":173213000},\"UpdatedAt\":{\"seconds\":1752192138,\"nanos\":173213000},\"Status\":2,\"PartialPrice\":{\"Value\":15000,\"Exp\":2},\"PartialQty\":{\"Value\":100,\"Exp\":2}},\"InstanceID\":\"someInstanceID-123456-test2\",\"BlockTime\":{\"seconds\":1752192138,\"nanos\":173218000},\"Sequence\":123,\"OrganizationID\":\"72c4c072-2fe4-4f72-ae9d-d9d52a05fd71\",\"UserID\":\"test@test.com\"}",
            "CreatedAt": {
                "nanos": 20664297
            },
            "UpdatedAt": {
                "nanos": 20664797
            },
            "Key": "someTXID-123456-test2",
            "Network": "testnet"
        }
        // ...
    ],
    "Total": 100
}
```

## GET `/api/search/trades?query=...&page=...&order_by=...&dir=...`

Search for trades using the provided query parameters and pagination.

### Example request

```bash
curl -X GET \
"https://comsolotex.sologenic.org/api/search/trades?query=Insib3Blc...R5cGUiOiAwfV19Ig==&page=2" \
-H "Authorization: Bearer ..." \
-H "OrganizationID: ..." \
-H "Network: mainnet"
```

### Example response

```json
{
    "Results": [
        {
            "Data": "{\"UserID\":\"nick.luong@sologenic.org\",\"OrderKey\":\"1234-testcore13s2mmgg4uu4f...ah8uxufugtajkeq2qgznc28-2\",\"Sequence\":123,\"Amount\":{\"Value\":100,\"Exp\":2},\"Price\":1500000,\"Denom1\":{\"Currency\":{\"Symbol\":\"WUSDC\",\"Version\":\"1\"},\"Subunit\":\"suwusdc_1\",\"Issuer\":\"testcore13s2mmgg4uu4f...ah8uxufugtajkeq2qgznc28\"},\"Denom2\":{\"Currency\":{\"Symbol\":\"AAPL\",\"Version\":\"1\"},\"Subunit\":\"suaapl_1\",\"Issuer\":\"testcore13s2mmgg4uu4f...ah8uxufugtajkeq2qgznc28\"},\"Side\":1,\"BlockTime\":{\"seconds\":1752191886,\"nanos\":414535000},\"OrganizationID\":\"72c4c072-2fe4-4f72-ae9d-d9d52a05fd71\",\"MetaData\":{\"Network\":2,\"UpdatedAt\":{\"seconds\":1752191889,\"nanos\":202828000},\"CreatedAt\":{\"seconds\":1752191886,\"nanos\":414533000}},\"TXID\":\"someTXID-123456-test\",\"BlockHeight\":123456,\"Enriched\":true,\"Processed\":true,\"Status\":6}",
            "CreatedAt": {
                "nanos": 601903226
            },
            "UpdatedAt": {
                "nanos": 601903466
            },
            "Key": "someTXID-123456-test",
            "Network": "testnet"
        },
        {
            "Data": "{\"UserID\":\"nick.luong@sologenic.org\",\"OrderKey\":\"2345-testcore13s2mmgg4uu4f...ah8uxufugtajkeq2qgznc28-2\",\"Sequence\":123,\"Amount\":{\"Value\":100,\"Exp\":2},\"Price\":1500000,\"Denom1\":{\"Currency\":{\"Symbol\":\"WUSDC\",\"Version\":\"1\"},\"Subunit\":\"suwusdc_1\",\"Issuer\":\"testcore13s2mmgg4uu4f...ah8uxufugtajkeq2qgznc28\"},\"Denom2\":{\"Currency\":{\"Symbol\":\"AMZN\",\"Version\":\"1\"},\"Subunit\":\"suamzn_1\",\"Issuer\":\"testcore13s2mmgg4uu4f...ah8uxufugtajkeq2qgznc28\"},\"Side\":1,\"BlockTime\":{\"seconds\":1752192138,\"nanos\":173218000},\"OrganizationID\":\"72c4c072-2fe4-4f72-ae9d-d9d52a05fd71\",\"MetaData\":{\"Network\":2,\"UpdatedAt\":{\"seconds\":1752192144,\"nanos\":110307000},\"CreatedAt\":{\"seconds\":1752192138,\"nanos\":173216000}},\"TXID\":\"someTXID-123456-test2\",\"BlockHeight\":123456,\"Enriched\":true,\"Processed\":true,\"Status\":6}",
            "CreatedAt": {
                "nanos": 943458957
            },
            "UpdatedAt": {
                "nanos": 943459137
            },
            "Key": "someTXID-123456-test2",
            "Network": "testnet"
        }
        // ...
    ],
    "Total": 100
}
```

## Application start parameters

The application uses the following environment parameters:

- `ORGANIZATION_STORE`: Included from `github.com/sologenic/com-fs-organization-model`. For setup see `github.com/sologenic/com-fs-organization-model/client/README.md`
- `USER_STORE` - the user service endpoint (included from `github.com/sologenic/com-fs-user-model/`)
- `ROLE_STORE` - the role service endpoint (included from `github.com/sologenic/com-fs-role-model/`)
- `FEATURE_FLAG_STORE` - the feature flag service endpoint (included from `github.com/sologenic/com-fs-feature-flag-model/`)
- `AUTH_FIREBASE_SERVICE` - the firebase authentication service endpoint (included from `github.com/sologenic/com-fs-auth-firebase-model/`)
- `OPEN_SEARCH_STORE` - the open search service endpoint (included from `github.com/sologenic/fs-open-search-model/`)
- `HTTP_CONFIG` - the configuration for the http server (included from `github.com/sologenic/com-be-http-lib/`)
- `SOLOGENIC_ORGANIZATION_ID` - the organization ID for Sologenic users (e.g., `sologenic`)
