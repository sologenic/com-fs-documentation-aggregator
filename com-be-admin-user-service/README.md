# Admin user service

The Admin User Service provides API interfaces that enable organization administrators to manage retail users within a multi-tier system.

All admin endpoints are protected by role-based access control and default to the`ORGANIZATION_ADMINISTRATOR` role. Permissions are managed dynamically by Organization administrators on the fly.

## API endpoints overview

**Authenticated:**

* GET `/api/adminuser/get?{user_id|external_id}=...` - retrieves the `User` information using either `UserID` or `ExternalUserID`
* GET `/api/adminuser/list?filter=...` - retrieves all `User` information, uses `Filter` to filter the results
* PUT `/api/adminuser/update` - updates the user information for a specified user within the organization
* PUT `/api/adminuser/update/status` - modifies the status of a specified user

> **Note:** All authenticated endpoints require a valid Firebase token in Authorization header. The token must be:
> - Obtained from the Firebase Authentication service
> - Prepended with `Bearer: ` e.g.`"Authorization: Bearer: eyJhb....`

## API endpoints details

### GET `/api/adminuser/get?{user_id|external_id}=...`

Retrieves the `User` information for the given `user_id` or `external_id`. The `User` information is returned as an `User` object as defined in the model. 
Request header must include `Network`, `OrganizationID`, and `Authorization` headers.

##### Example request

```bash
curl -X GET "https://comsolotex.sologenic.org/api/adminuser/get?user_id=456@gmail.com" \
-H "Content-Type: application/json" \
-H "Authorization: Bearer: ...." \
-H "Network: testnet" \
-H "OrganizationID: 72c4c072-2fe4-4f72-ae9d-d9d52a05fd71"
```

##### Example response

```json5
{
    "User": {
        "UserID": "456@gmail.com",
        "FirstName": "John",
        "LastName": "Doe",
        "Address": "123 Main Street, Apt 4B, Cityville, ST 12345",
        "Avatar": "https://example.com/avatars/johndoe.png",
        "Alias": "JD",
        "Description": "test",
        "Status": 2,
        "Wallets": [
            {
                "Address": "rU6K7V3Po4snVhBBaU29sesqs2qTQJWDw1",
                "Alias": "Primary Wallet",
                "Type": 3
            },
            {
                "Address": "rBKPS4oLSaV2KVVuHH8EpQqMGgGefGFRs9",
                "Alias": "Savings",
                "Type": 1
            }
        ],
        "Socials": [
            {
                "URL": "https://twitter.com/johndoe",
                "Type": 5
            },
            {
                "URL": "https://github.com/johndoe",
                "Type": 2
            },
            {
                "URL": "https://linkedin.com/in/johndoe",
                "Type": 9
            }
        ],
        "Language": {
            "Language": "en-US"
        },
        "ExternalUserID": "cbf6a315-3438-4ea0-a27b-3f0041ade118",
        "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
        "Role": 1
    },
    "MetaData": {
        "Network": 2,
        "UpdatedAt": {
            "seconds": 1741992596,
            "nanos": 71607000
        },
        "CreatedAt": {
            "seconds": 1741984486,
            "nanos": 16812090
        }
    },
    "Audit": {
        "ChangedBy": "sg.org.testnet@gmail.com",
        "ChangedAt": {
            "seconds": 1741992245,
            "nanos": 360512000
        }
    }
}
```

### GET /api/adminuser/users?filter=eyJSb2xlcyI6WzFd...someEncodedUserFilter...iTGltaXQiOjIwfQ==

Retrieve all `User` information based on the given `Filter`, with pagination and an updated offset. The `Filter` must be base64 encoded and included in the request query. If the limit and offset are not specified, the default limit is 20 and the offset is 0.
Request header must include `Network`, `OrganizationID`, and `Authorization` headers.

The `Filter` object is defined as follows:

```proto
message Filter {
    repeated string UserIDs = 1;
    optional Order Order = 2;
    optional int32 Offset = 3;
    optional int32 Limit = 4;
    optional metadata.Network Network = 5; 
    string OrganizationID = 6;
}
```

> **Note:** `OrganizationID` is a mandatory parameter that enforces proper organizational data boundaries, ensuring each tenant can only access records within their designated scope.

##### Example request

```bash
curl -X GET "https://comsolotex.sologenic.org/api/adminuser/users?filter=eyJVc2VySURzIjpbXX0=" \
-H "Content-Type: application/json" \
-H "Authorization: Bearer: ...." \
-H "Network: testnet"
-H "OrganizationID: 72c4c072-2fe4-4f72-ae9d-d9d52a05fd71"
```

##### Example response

```json5
{
    "Users": [
        {
            "User": {
                "UserID": "456@gmail.com",
                "FirstName": "John",
                "LastName": "Doe",
                "Address": "123 Main Street, Apt 4B, Cityville, ST 12345",
                "Avatar": "https://example.com/avatars/johndoe.png",
                "Alias": "JD",
                "Description": "Blockchain enthusiast and investor",
                "Status": 1,
                "Wallets": [
                    {
                        "Address": "rU6K7V3Po4snVhBBaU29sesqs2qTQJWDw1",
                        "Alias": "Primary Wallet",
                        "Type": 3
                    },
                    {
                        "Address": "rBKPS4oLSaV2KVVuHH8EpQqMGgGefGFRs9",
                        "Alias": "Savings",
                        "Type": 1
                    }
                ],
                "Socials": [
                    {
                        "URL": "https://twitter.com/johndoe",
                        "Type": 5
                    },
                    {
                        "URL": "https://github.com/johndoe",
                        "Type": 2
                    },
                    {
                        "URL": "https://linkedin.com/in/johndoe",
                        "Type": 9
                    }
                ],
                "Language": {
                    "Language": "en-US"
                },
                "ExternalUserID": "cbf6a315-3438-4ea0-a27b-3f0041ade118",
                "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71"
            },
            "MetaData": {
                "Network": 2,
                "UpdatedAt": {
                    "seconds": 1741984486,
                    "nanos": 16812450
                },
                "CreatedAt": {
                    "seconds": 1741984486,
                    "nanos": 16812090
                }
            },
            "Audit": {
                "ChangedAt": {
                    "seconds": 1741984486,
                    "nanos": 16812560
                }
            }
        }
}
```

### PUT /api/adminuser/update

Updates the user information for a specified `User`. The request body must contain the `User` object as defined in the model. 
Request header must include `Network`, `OrganizationID`, and `Authorization` headers.

> **Note:** Status won't be updated using this endpoint. Use the `/api/adminuser/status` endpoints for status updates.

#### Example request

```bash
curl -X POST "https://comsolotex.sologenic.org/api/adminuser/update" \
-H "Content-Type: application/json" \
-H "Network: mainnet" \ 
-H "Authorization: Bearer: ...." \
-H "OrganizationID: 72c4c072-2fe4-4f72-ae9d-d9d52a05fd71"
-d '{
    "User": {
        "UserID": "456@gmail.com",
        "FirstName": "John",
        "LastName": "Doe",
        "Address": "123 Main Street, Apt 4B, Cityville, ST 12345",
        "Avatar": "https://example.com/avatars/johndoe.png",
        "Alias": "JD",
        "Description": "test",
        "Status": 1,
        "Wallets": [
            {
                "Address": "rU6K7V3Po4snVhBBaU29sesqs2qTQJWDw1",
                "Alias": "Primary Wallet",
                "Type": 3
            },
            {
                "Address": "rBKPS4oLSaV2KVVuHH8EpQqMGgGefGFRs9",
                "Alias": "Savings",
                "Type": 1
            }
        ],
        "Socials": [
            {
                "URL": "https://twitter.com/johndoe",
                "Type": 5
            },
            {
                "URL": "https://github.com/johndoe",
                "Type": 2
            },
            {
                "URL": "https://linkedin.com/in/johndoe",
                "Type": 9
            }
        ],
        "Language": {
            "Language": "en-US"
        },
        "ExternalUserID": "cbf6a315-3438-4ea0-a27b-3f0041ade118",
        "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71"
    },
    "MetaData": {
        "Network": 2
    }
}'
```

#### Example response

```json5
{
    "UserID": "456@gmail.com",
    "Network": 2
}
```

### PUT /api/adminuser/status

Updates the status for an `User`. The request body must contain the `SetStatusMessage` object as defined in the model.
Request header must include `Network`, `OrganizationID`, and `Authorization` headers.

#### Example request

```bash
curl -X PUT "https://comsolotex.sologenic.org/api/adminuser/update/status" \
-H "Content-Type: application/json" \
-H "Authorization: Bearer: ...." \
-H "Network: testnet" \
-H "OrganizationID: 72c4c072-2fe4-4f72-ae9d-d9d52a05fd71"
-d '{
    "UserID": "456@gmail.com",
    "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
    "Status": 2,
    "Network": 2
}'
```

## Application start parameters

The application uses the following environment parameters:

* `HTTP_CONFIG` - the configuration for the http server (included from `github.com/sologenic/com-be-http-lib/`)
* `AUTH_FIREBASE_SERVICE` - the firebase authentication service endpoint
* `USER_STORE` - the user service endpoint (included from `github.com/sologenic/com-fs-user-model/`)
* `ACCOUNT_STORE` - the admin account service endpoint (included from `github.com/sologenic/com-fs-admin-account-model/`)
* `ROLE_STORE` - the admin role service endpoint (included from `github.com/sologenic/com-fs-admin-role-model/`)
* `FEATURE_FLAG_STORE` - the feature flag service endpoint (included from `github.com/sologenic/com-fs-feature-flag-model/`)
* `ORGANIZATION_STORE` - the organization service endpoint (included from `github.com/sologenic/com-fs-admin-organization-model/`)
