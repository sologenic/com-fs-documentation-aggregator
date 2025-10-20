# User service

The User Service provides API interfaces that manages (normal) users within an organization.

## TECHNICAL REQUIREMENTS

### This service has to be ALWAYS on to process the webhook data

## API endpoints overview

**Unauthenticated:**

- GET `/api/user/get/alias?filter=...` - retrieves an array of user aliases for a specified filter
- POST `/api/user/create` - creates a new user under an organization

**Authenticated:**

- GET `/api/user/get?user_id=...` - retrieves user information for a specified user
- PUT `/api/user/update` - updates the user information for a specified user within the organization
- PUT `/api/user/disable` - disables a user

> **Note:** All authenticated endpoints require a valid Firebase token in Authorization header. The token must be:
>
> - Obtained from the Firebase Authentication service
> - Prepended with `Bearer: ` e.g.`"Authorization: Bearer: eyJhb....`

## Authorization requirements

Access to authenticated endpoints requires that a user has the status `ACTIVE`.

## API endpoints details

**_Unauthenticated:_**

### GET /api/user/get/alias?filter=...

Retrieves an array of user alias objects, where each object contains an `ExternalUserID` and the corresponding `Alias`. The `filter` parameter is a base64-encoded JSON object (see `com-fs-user-model`) specifying filter criteria.

#### Example request

```bash
curl -X GET "https://comsolotex.sologenic.org/api/user/get/alias?filter=ewogICJFeHRlcm5hbFVzZXJJRHMiOiBbImY4YmY2OWFkLWVmOTgtNDJkNi05Y2QzLWJlZWE0ZDI4NTQyYSIsICIxNzgzODgwZC1lNjY3LTRjODgtYjU2NS1hNmE4ZmRmMzkyNWIiLCAiNWFlNGUwNjEtMzU0YS00NTlmLThmNGUtOWU5OTQxYjY1NDMwIl0sCiAgIk9yZ2FuaXphdGlvbklEIjogIjcyYzRjMDcyLTJmZTQtNGY3Mi1hZTlkLWQ5ZDUyYTA1ZmQ3MSIsCiAgIk5ldHdvcmsiOiAyCn0=" \
-H "Content-Type: application/json" \
-H "OrganizationID: 72c4c072-2fe4-4f72-ae9d-d9d52a05fd71" \
-H "Network: testnet"
```

#### Example response

```json5
[
  {
    "ExternalID": "00234946-010d-4c6e-abf1-dee6836c4e20",
    "Alias": "test",
  },
  {
    "ExternalID": "441e9c6e-f1b2-407d-a557-af89bcbc46a2",
    "Alias": "Verna",
  },
  {
    "ExternalID": "79f56dbc-d1d1-4448-9138-723f23ddae27",
    "Alias": "Joannie",
  },
  {
    "ExternalID": "e0c1672e-8a2d-4201-ac54-3043f847c19b",
    "Alias": "Justine",
  },
  {
    "ExternalID": "e605dbf1-db61-441e-8bff-df66d209a492",
    "Alias": "Sidney",
  },
]
```

### POST /api/user/create

Create a new user. `UserID` and `ExternalUserID` will be generated automatically. The request body must contain the `User` object as defined in the model. Request must include a `Network` header.

#### Example request

```bash
curl -X POST "https://comsolotex.sologenic.org/api/user/create" \
-H "Content-Type: application/json" \
-H "Network: mainnet" \
-H "OrganizationID: 72c4c072-2fe4-4f72-ae9d-d9d52a05fd71"
-d '{
    "UserID": "randy.lahey@gmail.com",
    "FirstName": "Randy",
    "LastName": "Lahey",
    "Address": "SomeAddress",
    "Avatar": "SomeAvatar",
    "Alias": "Bobandy",
    "Description": "Some description.",
    "Status": 1,
    "Wallets": [...],
    "Socials": [...],
    "Language": {...},
    "OrganizationID": "215a551d-...-9284f40d1340",
    "Employment": {...}
}'
```

#### Example response

```json5
{
  UserID: "sg.org.testnet11335@gmail.com",
  Network: 2,
}
```

**_Authenticated:_**

> **Note:** `OrganizationID` is a mandatory parameter that enforces proper organizational data boundaries, ensuring each tenant can only access records within their designated scope.

### GET /api/user/get?user_id=...

Retrieves user information for a specified `User`. The request must include the `UserID` parameter in the query string as `user_id`.
Request header must include `Network`, `OrganizationID`, and `Authorization` headers.

##### Example request

```bash
curl -X GET "https://comsolotex.sologenic.org/api/user/get?user_id=test@gmail.com" \
-H "Content-Type: application/json" \
-H "Authorization: Bearer: ...." \
-H "Network: mainnet"
```

##### Example response

```json5
{
  "User": {
    "UserID": "test@gmail.com",
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
        "Type": 3,
      },
      {
        "Address": "rBKPS4oLSaV2KVVuHH8EpQqMGgGefGFRs9",
        "Alias": "Savings",
        "Type": 1,
      },
    ],
    "Socials": [
      {
        "URL": "https://twitter.com/johndoe",
        "Type": 5,
      },
      {
        "URL": "https://github.com/johndoe",
        "Type": 2,
      },
      {
        "URL": "https://linkedin.com/in/johndoe",
        "Type": 9,
      },
    ],
    "Language": {
      "Language": "en-US",
    },
    "ExternalUserID": "3492068a-c20f-4f66-8bfa-7f751057109b",
    "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
  },
  "MetaData": {
    "Network": 2,
    "UpdatedAt": {
      "seconds": 1741983951,
      "nanos": 82486205,
    },
    "CreatedAt": {
      "seconds": 1741983951,
      "nanos": 82485825,
    },
  },
  "Audit": {
    "ChangedAt": {
      "seconds": 1741983951,
      "nanos": 82486435,
    },
  },
}
```

### PUT /api/user/update

Updates the user information for a specified `User`. The request body must contain the `User` object as defined in the model.
Request header must include `Network`, `OrganizationID`, and `Authorization` headers.

##### Update constraints

For security reasons, certain fields are considered "immutable" and will be preserved from the existing record, even if new values are provided in the update request:

Immutable fields:

- `OrganizationID`: organization scope cannot be altered
- `ExternalUserID`: system-assigned identifier cannot be altered
- `MetaData.CreatedAt`: original creation timestamp is preserved
- `MetaData.Network`: network on the account cannot be altered
- `User.Role`: always enforced as `NORMAL_USER`

> **Note:** Status won't be updated using this endpoint. Use the `/api/user/status` endpoints for status updates.

#### Example request

```bash
curl -X PUT "https://comsolotex.sologenic.org/api/user/update" \
-H "Content-Type: application/json" \
-H "Network: mainnet" \
-H "Authorization: Bearer: ...." \
-d '{
    "User": {
        "FirstName": "Randy",
        "LastName": "Lahey",
        "Address": "SomeOtherAddress",
        "Avatar": "SomeOtherAvatar",
        "Alias": "Bobandy",
        "Description": "Some other description.",
        "UserStatus": 1,
        "Wallets": [
            {
                "Address": "215a551d-...-9284f40d1340",
                "Alias": "someAlias",
                "Type": 1
            },
            {...}
        ],
        "Socials": [...],
        "Language": {...},
        "ExternalUserID": "g4h5j3k2-...-3k4j5h6g7j8",
        "OrganizationID": "215a551d-...-9284f40d1340",
        "Employment": {...}
    },
    "Audit": {...},
    "MetaData": {...}
}'
```

#### Example response

```json5
{
  UserID: "lahey@abc.com",
  Network: 1,
}
```

### PUT /api/user/disable

Allows a user to disable their own account.
Request header must include `Network`, `OrganizationID`, and `Authorization` headers.

> **_Note:_** This endpoint performs self-disabling operations only. The authenticated user can only disable their own account, not other users' accounts.
> Only a valid admin can re-enable a disabled account (refer to `com-be-admin-user-service` for more information).

#### Example request

```bash
curl -X PUT "https://comsolotex.sologenic.org/api/user/disable" \
-H "Content-Type: application/json" \
-H "Authorization: Bearer: ...." \
-H "Network: mainnet"
```

### POST /api/kyc/start

Allows user to start the process of KYC with Organization KYC Provider
Request header must include `Network`, `OrganizationID` headers.

#### Example request

```bash
curl -X POST "https://comsolotex.sologenic.org/api/kyc/start" \
-H "Content-Type: application/json" \
-H "Authorization: Bearer ..." \
-H "Network: mainnet"
```

#### Example response

```json5

```

### POST /api/kyc/webhook

Allows KYC Provider to post events regarding inquiries and kyc processes of users

#### Example request

```bash
curl -X POST "https://comsolotex.sologenic.org/api/kyc/webhook" \
-H "Content-Type: application/json" \
-H "Authorization: Bearer ..." \
-H "Network: mainnet"
```

> **Note**: The service config needs to be always on for the go routines used to process the data to execute reliably

## Application start parameters

The application uses the following environment parameters:

- `AUTH_FIREBASE_SERVICE` - the firebase authentication service endpoint
- `USER_STORE` - the user store endpoint (included from `github.com/sologenic/com-fs-user-model/`)
- `FEATURE_FLAG_STORE` - the feature flag store endpoint (included from `github.com/sologenic/com-fs-feature-flag-model/`)
- `ROLE_STORE` - the role store endpoint (included from `github.com/sologenic/com-fs-role-model/`)
- `KYC_DOCUMENT_STORE` - the kyc document store endpoint (included from `github.com/sologenic/com-fs-kyc-document-model/`)
- `TRADE_PROFILE_STORE` - the trade profile store endpoint (included from `github.com/sologenic/com-fs-trade-profile-model/`)
- `CERTIFICATE_STORE` - the certificate store endpoint (included from `github.com/sologenic/com-fs-certificate-model/`)
- `HTTP_CONFIG` - the configuration for the http server (included from `github.com/sologenic/com-be-http-lib/`)
- `CREDENTIALS_LOCATION` - the credentials location for the gcloud datastore
- `PROJECT_ID` - the project id for the gcloud datastore
- `REDIRECT_URL` - the redirect URI for KYC to redirect to after user process completion
- `ORGANIZATION_STORE` - the organization store endpoint (included from `github.com/sologenic/com-fs-organization-model`)
- `PARENT_ORGANIZATION_ID` - the TX organization id for account cloning
