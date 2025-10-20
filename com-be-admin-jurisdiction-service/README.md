# Admin Jurisdiction Service

The admin jurisdiction service provides the following RESTFUL interface:

> **Note:** All admin endpoints are protected by role-based access control and default to the `ORGANIZATION_ADMINISTRATOR` role. Permissions are managed dynamically by Organization administrators on the fly.

**Authenticated:**

* GET `/api/jurisdiction/get?id=...`: Returns a jurisdiction
* GET `/api/jurisdiction/list?filter=...`: Returns a list of all jurisdictions
* POST `/api/jurisdiction/create`: Creates a new jurisdiction
* PUT `/api/jurisdiction/update`: Updates an existing jurisdiction
* PUT `/api/jurisdiction/update/status?id=...&status=...&reason=...`: Update the status of a jurisdiction

## GET /api/jurisdiction/get?id=...

Returns a jurisdiction by its ID.

### Example call

```bash
curl -X GET \
  https://comsolotex-admin.sologenic.org/api/jurisdiction/get?id=8878d73b-edff-40a1-b6ce-43a4508aa96a \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ..." \
  -H "Network: testnet"
```

### Example response

```json
{
    "Jurisdiction": {
        "ID": "8878d73b-edff-40a1-b6ce-43a4508aa96a",
        "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
        "Name": "United States",
        "Description": "Federal jurisdiction of the United States of America",
        "ExternalID": "US-001",
        "Status": 2,
        "Country": {
            "Name": "United States",
            "Code": "USA"
        },
        "Regulators": [
            1,
            2
        ],
        "EffectiveFrom": {
            "seconds": 1672531200
        },
        "EffectiveTo": {
            "seconds": 1767225599
        }
    },
    "MetaData": {
        "Network": 2,
        "UpdatedAt": {
            "seconds": 1740516877,
            "nanos": 216014000
        },
        "CreatedAt": {
            "seconds": 1740511148,
            "nanos": 85177000
        }
    },
    "Audit": {
        "ChangedBy": "sg.org.testnet@gmail.com",
        "ChangedAt": {
            "seconds": 1740516877,
            "nanos": 216015000
        },
        "Reason": "Jurisdiction no longer supported"
    }
}
```

## GET /api/jurisdiction/list?filter=eyJSb2xlcyI6WzFd...someEncodedAccountFilter...iTGltaXQiOjIwfQ==

Returns a list of all assets for admins

> **Note**: The filter parameter must be base64 encoded JSON string containing the query parameters.

### Example call

```bash
curl -X GET \
  https://comsolotex-admin.sologenic.org/api/jurisdiction/list \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer: ...." \
  -H "Network: testnet"
```

### Example response

```json
[
    {
        "Jurisdiction": {
            "ID": "08af6b9f-9a9e-4689-8adb-5b4fe70fcddd",
            "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
            "Name": "Canada-British Columbia",
            "Description": "Provincial jurisdiction of British Columbia, Canada",
            "ExternalID": "CA-BC-001",
            "Status": 1,
            "Country": {
                "Name": "Canada",
                "Code": "CAN"
            },
            "Subdivision": {
                "Name": "British Columbia",
                "Code": "CA-BC"
            },
            "Regulators": [
                1,
                2
            ],
            "EffectiveFrom": {
                "seconds": 1672531200
            },
            "EffectiveTo": {
                "seconds": 1767225599
            }
        },
        "MetaData": {
            "Network": 2,
            "UpdatedAt": {
                "seconds": 1740518732,
                "nanos": 171974000
            },
            "CreatedAt": {
                "seconds": 1740518732,
                "nanos": 171972000
            }
        },
        "Audit": {
            "ChangedBy": "sg.org.testnet@gmail.com",
            "ChangedAt": {
                "seconds": 1740518732,
                "nanos": 171974000
            }
        }
    },
    {
        "Jurisdiction": {
            "ID": "8878d73b-edff-40a1-b6ce-43a4508aa96a",
            "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
            "Name": "United States",
            "Description": "Federal jurisdiction of the United States of America",
            "ExternalID": "US-001",
            "Status": 2,
            "Country": {
                "Name": "United States",
                "Code": "USA"
            },
            "Regulators": [
                1,
                2
            ],
            "EffectiveFrom": {
                "seconds": 1672531200
            },
            "EffectiveTo": {
                "seconds": 1767225599
            }
        },
        "MetaData": {
            "Network": 2,
            "UpdatedAt": {
                "seconds": 1740518012,
                "nanos": 266394000
            },
            "CreatedAt": {
                "seconds": 1740511148,
                "nanos": 85177000
            }
        },
        "Audit": {
            "ChangedBy": "sg.org.testnet@gmail.com",
            "ChangedAt": {
                "seconds": 1740518012,
                "nanos": 266395000
            },
            "Reason": "Update test"
        }
    }
]
```

## POST /api/jurisdiction/create

Creates a new jurisdiction

> **Note**: 
> - The `Country.Code` must follow ISO 3166-1 alpha-3 standard (e.g., `USA` for United States, `CAN` for Canada)
> - The `Subdivision.Code` must follow ISO 3166-2 standard (e.g., `US-TX` for Texas, `CA-BC` for British Columbia)
> - If `Subdivision` is omitted, the jurisdiction is considered country-wide

### Example call

```bash
curl -X POST \
  https://comsolotex-admin.sologenic.org/api/jurisdiction/create \
  -H "Content-Type: application/json" \
  -H "Network: testnet" \
  -H "Authorization: Bearer ..." \
  -d '{
  "Jurisdiction": {
    "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
    "Name": "Canada-British Columbia",
    "Description": "Provincial jurisdiction of British Columbia, Canada",
    "ExternalID": "CA-BC-001",
    "Status": 1,
    "Country": {
        "Name": "Canada",
        "Code": "CAN"
    },
    "Subdivision": {
        "Name": "British Columbia",
        "Code": "CA-BC"
    },
    "Regulators": [1, 2],
    "EffectiveFrom": "2023-01-01T00:00:00Z",
    "EffectiveTo": "2025-12-31T23:59:59Z"
  }
}'
```

### Example response

```json
{
  "ID": "08af6b9f-9a9e-4689-8adb-5b4fe70fcddd"
}
```

## PUT /api/jurisdiction/update

Updates an existing jurisdiction

> **Note**: 
- The following fields are immutable and will not be updated:: `CreatedAt`, `Network`, `ID`, `OrganizationID`, `ExternalID`, `Country`, `Subdivision` 
- `Status` can only be updated through the `/api/jurisdiction/update/status` endpoint


### Example call

```bash
curl -X PUT \
  https://comsolotex-admin.sologenic.org/api/jurisdiction/update \
  -H "Content-Type: application/json" \
  -H "Network: testnet" \
  -H "Authorization: Bearer ..." \
  -d '{
  "Jurisdiction": {
    "ID": "8878d73b-edff-40a1-b6ce-43a4508aa96a",
    "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
    "Name": "United States",
    "Description": "Federal jurisdiction of the United States of America",
    "ExternalID": "US-001",
    "Country": {
        "Name": "United States",
        "Code": "USA"
    },
    "Regulators": [1, 2],
    "EffectiveFrom": "2023-01-01T00:00:00Z",
    "EffectiveTo": "2025-12-31T23:59:59Z"
  },
  "Audit": {
    "Reason": "Update test"
  }
}'
```

### Example response

```json
{
  "ID": "8878d73b-edff-40a1-b6ce-43a4508aa96a"
}
```

## PUT /api/jurisdiction/update/status?id=...&status=...&reason=...

Updates the status of an existing jurisdiction.

### Example call

```bash
curl -X PUT \
  'https://comsolotex-admin.sologenic.org/api/jurisdiction/update/status?id=8878d73b-edff-40a1-b6ce-43a4508aa96a&status=2&reason=Jurisdiction%20no%20longer%20supported' \
  -H "Content-Type: application/json" \
  -H "Network: testnet" \
  -H "Authorization: Bearer ..."
```

### Example response

```json
{
  "ID": "8878d73b-edff-40a1-b6ce-43a4508aa96a"
}
```

## Start parameters

The following start parameters are required:

* `ACCOUNT_STORE`: the account store endpoint (included from `github.com/sologenic/com-be-admin-account-store/`)
* `ASSET_STORE`: the asset store endpoint (included from `github.com/sologenic/com-be-asset-store/`)
* `JURISDICTION_STORE`: the jurisdiction store endpoint (included from `github.com/sologenic/com-be-jurisdiction-store/`)
* `FEATURE_FLAG_STORE` - the feature flag service endpoint (included from `github.com/sologenic/fs-feature-flag-model/`)
* `ORGANIZATION_STORE` - the organization service endpoint (included from `github.com/sologenic/com-fs-organization-store/`)
* `AUTH_FIREBASE_SERVICE`: the firebase authentication service endpoint
* `ROLE_STORE` - the role service endpoint (included from `github.com/sologenic/com-fs-admin-role-model/`)
* `HTTP_CONFIG`: the configuration for the http server (included from `github.com/sologenic/com-be-http-lib/`)
