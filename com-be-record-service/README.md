# Record service

The Record Service provides API interfaces to retrieve users' records, such as tax reports and financial statements.

## API endpoints overview

**Authenticated:**

* GET `/api/record/get?type=...&period_id=...` - retrieve a single record for a user
* GET `/api/record/list?filter=...` - retrieve a list of records for a user, uses `Filter` to filter the results

> **Note:** All authenticated endpoints require a valid Firebase token in Authorization header, the organization ID in the Organization header, and the network in the Network header. The token must be:
> - Obtained from the Firebase Authentication service
> - Prepended with `Bearer: ` e.g.`"Authorization: Bearer: eyJhb....`

## API endpoints details

### GET `/api/record/get?type=...&period_id=...`

Retrieve a single record for a user. All query parameters are required.

#### Query Parameters

- `type` The type of record (e.g., 1)
- `period_id` The period identifier (e.g., "2023", "2023-Q1")

#### Example request

```bash
curl -X GET \
"https://comsolotex.sologenic.org/api/record/get?type=1&period_id=2024" \
-H "Authorization: Bearer: eyJhb...." \
-H "OrganizationID: 72c4c072-2fe4-4f72-ae9d-d9d52a05fd71" \
-H "Network: testnet"
```

#### Example response

```json
{
    "Record": {
        "UserID": "nick.luong@sologenic.org",
        "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
        "Title": "Tax Summary Report - 2024",
        "Description": "Annual tax summary report for tax year 2024",
        "Type": 1,
        "Period": 7,
        "PeriodIdentifier": "2024",
        "PeriodFrom": {
            "seconds": 1704067200
        },
        "PeriodTo": {
            "seconds": 1735689599
        },
        "File": {
            "Reference": "testnet/record/Tax_Slip",
            "Extension": "pdf",
            "Name": "Tax_Slip"
        }
    },
    "MetaData": {
        "Network": 2,
        "UpdatedAt": {
            "seconds": 1751326196,
            "nanos": 347829000
        },
        "CreatedAt": {
            "seconds": 1751326196,
            "nanos": 347827000
        }
    },
    "Audit": {
        "ChangedBy": "SYSTEM",
        "ChangedAt": {
            "seconds": 1751326196,
            "nanos": 347830000
        }
    }
}
```

### GET `/api/record/list?filter=...`

Retrieve a list of records for a user. `filter` query parameters are optional and `filter` must be base64 encoded JSON string(refer to `com-fs-record-model` for details).

#### Example request

```bash
curl -X GET \
"https://comsolotex.sologenic.org/api/record/list" \
-H "Authorization: Bearer: eyJhb...." \
-H "OrganizationID: 72c4c072-2fe4-4f72-ae9d-d9d52a05fd71" \
-H "Network: testnet"
```

#### Example response

```json
{
    "Records": [
        {
            "Record": {
                "UserID": "nick.luong@sologenic.org",
                "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
                "Title": "Tax Summary Report - 2024",
                "Description": "Annual tax summary report for tax year 2024",
                "Type": 1,
                "Period": 7,
                "PeriodIdentifier": "2024",
                "PeriodFrom": {
                    "seconds": 1704067200
                },
                "PeriodTo": {
                    "seconds": 1735689599
                },
                "File": {
                    "Reference": "testnet/record/Tax_Slip",
                    "Extension": "pdf",
                    "Name": "Tax_Slip"
                }
            },
            "MetaData": {
                "Network": 2,
                "UpdatedAt": {
                    "seconds": 1751326196,
                    "nanos": 347829000
                },
                "CreatedAt": {
                    "seconds": 1751326196,
                    "nanos": 347827000
                }
            },
            "Audit": {
                "ChangedBy": "SYSTEM",
                "ChangedAt": {
                    "seconds": 1751326196,
                    "nanos": 347830000
                }
            }
        },
        {
            "Record": {
                "UserID": "nick.luong@sologenic.org",
                "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
                "Title": "Account Statement - 2025-05",
                "Description": "Monthly account statement for period 2025-05",
                "Type": 2,
                "Period": 3,
                "PeriodIdentifier": "2025-05",
                "PeriodFrom": {
                    "seconds": 1746057600
                },
                "PeriodTo": {
                    "seconds": 1748735999
                },
                "File": {
                    "Reference": "testnet/record/Statement",
                    "Extension": "pdf",
                    "Name": "Statement"
                }
            },
            "MetaData": {
                "Network": 2,
                "UpdatedAt": {
                    "seconds": 1751326196,
                    "nanos": 722188000
                },
                "CreatedAt": {
                    "seconds": 1751326196,
                    "nanos": 722188000
                }
            },
            "Audit": {
                "ChangedBy": "SYSTEM",
                "ChangedAt": {
                    "seconds": 1751326196,
                    "nanos": 722189000
                }
            }
        }
    ]
}
```

## Application start parameters

The application uses the following environment parameters:

* `HTTP_CONFIG` - the configuration for the http server (included from `github.com/sologenic/com-be-http-lib/`)
* `AUTH_FIREBASE_SERVICE` - the firebase authentication service endpoint
* `USER_STORE` - the user service endpoint (included from `github.com/sologenic/com-fs-user-model/`)
* `FEATURE_FLAG_STORE` - the feature flag service endpoint (included from `github.com/sologenic/com-fs-feature-flag-model/`)
* `ROLE_STORE` - the role service endpoint (included from `github.com/sologenic/com-fs-role-model/`)
* `RECORD_STORE` - the record service endpoint (included from `github.com/sologenic/com-fs-record-model/`)
* `ORGANIZATION_STORE`: the organization store endpoint (included from `github.com/sologenic/com-be-organization-model/`)