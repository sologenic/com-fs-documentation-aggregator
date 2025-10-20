# Smart contract deployment service

The smart contract deployment service provides the following RESTFUL interface:

**Authenticated:**

* GET `/api/smartcontract/get` - retrieve a smart contract
* GET `/api/smartcontract/list` - retrieves a list of smart contracts
* POST `/api/smartcontract/deploy` - deploys a smart contract

## GET /api/smartcontract/get?name=someContractName

Retrieves the specified smart contract. Request must include an `Authorization` header with the `Authorization` token and the `Network` header.

The response returns the following fields:

- `metadata` - the metadata of the smart contract as defined in the metadata file
- `deployed` - the deployed instance of the smart contract as defined by the `SmartContractDetails` model
- `on_chain_code_id` - the current active code id the contract is using on the chain, may be nil if no contract is deployed (not yet instantiated)
- `latest_code_id` - the latest available code id, a mismatch between this and the `on_chain_code_id` indicates a new version is available

***Note: ActiveCodeID from `deployed` will be the same as `on_chain_code_id`. However, it is more reliable to use `on_chain_code_id` as it is fetched from the chain.***

### Example request

```bash
curl -X POST "https://comsolotex-admin.sologenic.org/api/smartcontract/get?name=hello-world-1" \
-H "Content-Type: application/json" \
-H "Network: mainnet" \ 
-H "Authorization:eyJQcml2YXRlS2......OGY0ZSJ9"
```

### Example response

```json5
{
    "metadata": {
        "contract_name": "hello-world-1",
        "version": "e5de9b77adc65cac1fddbb140b00fc90",
        "description": "A CosmWasm smart contract built using the Rust optimizer.",
        "repository": "https://github.com/sologenic/com-atg-sample-contract",
        "last_build": "2025-03-06T01:09:22.704943+00:00",
        "release_notes": "# Release Notes\n\n## Version e3b0c44298fc1c149afbf4c8996fb924\n- Initial release.\n- \n## Version 606386b328d90ca02a33dbb4a12c0cef\n- Fixed some issues.\n- Added migration function.\n"
    },
    "deployed": {
        "SmartContract": {
            "Name": "hello-world-1",
            "Version": "e5de9b77adc65cac1fddbb140b00fc90",
            "Address": "testcore13dlcgqv2dqlnsnww35wxgprpfeqjewmda60g2pq89qh2u37gkeyq0ela4u",
            "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
            "ActiveCodeID": 2188
        },
        "MetaData": {
            "Network": 2,
            "UpdatedAt": {
                "seconds": 1741292412,
                "nanos": 993577000
            },
            "CreatedAt": {
                "seconds": 1741292412,
                "nanos": 993576000
            }
        },
        "Audit": {
            "ChangedBy": "sg.org.testnet@gmail.com",
            "ChangedAt": {
                "seconds": 1741292412,
                "nanos": 993577000
            },
            "Reason": "Initial deployment"
        }
    },
    "on_chain_code_id": 2188,
    "latest_code_id": 2189
}
```

## GET /api/smartcontract/list

Retrieves a list of the metadata files for all smart contracts. Request must include an `Authorization` header with the `Authorization` token and the `Network` header.

### Example request

```bash
curl -X POST "https://comsolotex-admin.sologenic.org/api/smartcontract/list" \
-H "Content-Type: application/json" \
-H "Network: mainnet" \ 
-H "Authorization:eyJQcml2YXRlS2......OGY0ZSJ9"
```

### Example response

```json5
[
    {
        "contract_name": "hello-world-1",
        "version": "e5de9b77adc65cac1fddbb140b00fc90",
        "description": "A CosmWasm smart contract built using the Rust optimizer.",
        "repository": "https://github.com/sologenic/com-atg-sample-contract",
        "last_build": "2025-03-05T23:34:49.153283+00:00",
        "release_notes": "# Release Notes\n\n## Version e3b0c44298fc1c149afbf4c8996fb924\n- Initial release.\n- \n## Version 606386b328d90ca02a33dbb4a12c0cef\n- Fixed some issues.\n- Added migration function.\n"
    },
    {
        "contract_name": "hello-world-2",
        "version": "e5de9b77adc65cac1fddbb140b00fc90",
        "description": "A CosmWasm smart contract built using the Rust optimizer.",
        "repository": "https://github.com/sologenic/com-atg-sample-contract",
        "last_build": "2025-03-05T23:34:57.628786+00:00",
        "release_notes": "# Release Notes\n\n## Version e3b0c44298fc1c149afbf4c8996fb924\n- Initial release.\n- \n## Version 606386b328d90ca02a33dbb4a12c0cef\n- Fixed some issues.\n- Added migration function.\n"
    },
    {
        "contract_name": "hello-world-3",
        "version": "e5de9b77adc65cac1fddbb140b00fc90",
        "description": "A CosmWasm smart contract built using the Rust optimizer.",
        "repository": "https://github.com/sologenic/com-atg-sample-contract",
        "last_build": "2025-03-05T23:35:14.377590+00:00",
        "release_notes": "# Release Notes\n\n## Version e3b0c44298fc1c149afbf4c8996fb924\n- Initial release.\n- \n## Version 606386b328d90ca02a33dbb4a12c0cef\n- Fixed some issues.\n- Added migration function.\n"
    }
]
```

## POST /api/smartcontract/deploy

Deploys the given smart contract. Request must include an `Authorization` header with the `Authorization` token and the `Network` header.

In the request body, the `SmartContract` object must include the following fields:

- `Name` - the name of the smart contract
- `OrganizationID` - the organization ID
- `ActiveCodeID` - the active code ID

`Address` is optional since it will only be available after instantiation. However, it is required if the smart contract is to be migrated.

The `Audit` object is optional, but if included, should only contain the `Reason` field.

Address of the newly deployed smart contract is returned upon a successful deployment.

### Example request

```bash
curl -X POST "https://comsolotex-admin.sologenic.org/api/smartcontract/deploy" \
-H "Content-Type: application/json" \
-H "Network: mainnet" \ 
-H "Authorization:eyJQcml2YXRlS2......OGY0ZSJ9" \
-d '{
    "SmartContract": {
        "Name": "hello-world-1",
        "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71"
        "ActiveCodeID": 2182
    },
    "Audit": {
        "Reason": "Some reason."
    }
}'
```

### Example response

```json5
"testcore1ppsa8yzg6x53h67c9y04wsw5unk7xmgmgaz4e64nhrwk8gj9722quytmmh"
```

## Application start parameters

The application uses the following environment parameters:

* `HTTP_CONFIG` - the configuration for the http server (included from `github.com/sologenic/com-be-http-lib/`)
* `ACCOUNT_STORE` - the account service endpoint (included from `github.com/sologenic/com-fs-admin-account-model/`)
* `FEATURE_FLAG_STORE` - the feature flag service endpoint (included from `github.com/sologenic/com-fs-admin-feature-flag-model/`)
* `ORGANIZATION_STORE` - the organization service endpoint (included from `github.com/sologenic/com-fs-admin-organization-model/`)
* `SMART_CONTRACT_STORE` - the smart contract service endpoint (included from `github.com/sologenic/com-fs-admin-smart-contract-model/`)
* `ROLE_STORE` - the role service endpoint (included from `github.com/sologenic/com-fs-admin-role-model/`)
* `AUTH_FIREBASE_SERVICE` - the authentication service endpoint (included from `github.com/sologenic/com-fs-auth-firebase-model/`)
* `NETWORKS` - networks config for coreum clients
* `PROJECT_ID` - the project ID for the storage bucket
* `CREDENTIALS_LOCATION` - certificate with gcloud storage read rights
* `GRPC_APPEND` - segment of the service URL that follows the `service` keyword (i.e. given the URL `https://com-be-my-service-dfjiao-ijgao.a.run.app`, extract the portion after `service` including the `-`, in this case `dfjiao-ijgao.a.run.app`)
