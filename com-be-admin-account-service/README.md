# Admin account service

The Admin Account Service provides API interfaces that manages users, and their roles within a multi-tier system. All organization related operation will be provided by `com-be-admin-organization-service` separately.

In ATG, users are categorized into these primary account types:

**High-level Account Types**

1. **Sologenic Administrator (SOLOGENIC_ADMINISTRATOR)**:
    Sologenic Administrators provide initial setup such as the creation and management of organizations, and assign initial Organization Administrators, allowing organizations to manage their structure and user management.
   - Creates new organizations
   - Creates initial organization administrators
   - Views list of organizations
   - Cannot access or modify org users, end users or organization details
   - Creates new sologenic administrators

2. **Admins within organization**:

   * **Organization Administrator (ORGANIZATION_ADMINISTRATOR)**  
     Organization Administrators have complete authority within their assigned organization, managing all aspects from user accounts to organizational settings.  
     - Manages all aspects within their organization  
     - Creates a new account (organization's employee) and assigns roles such as KYC_ADMINISTRATOR, Role_BROKER_ASSET_ADMINISTRATOR  
     - Updates organization details  
     - Assigns and updates user roles within the organization  
     - Manages rolepaths
     - Manages users within their organization (retrieving, updating, disabling accounts)

   * **KYC Administrator (KYC_ADMINISTRATOR)**  
     KYC Administrators are responsible for managing KYC processes within their organization, ensuring that all end users are compliant with KYC requirements.  
     - Manages KYC processes within their organization  
     - KYC processes are handled in a separate KYC service

   * **Broker Asset Administrator (BROKER_ASSET_ADMINISTRATOR)**  
     Broker Asset Administrators are responsible for managing assets within their organization, including the creation and management of assets.  
     - Manages assets within their organization
     - Creates and manages assets
     - Asset management is handled in a separate asset service

3. **End Users (NORMAL_USER)**:
    - Fully managed by Organization Administrators, with no direct access to the admin service
    - Operate under organization management
    - KYC will be managed by the KYC Administrator independently in the separate KYC service
 
**Business Logic**

- **Sologenic Administrators**:
Managing the entire platform's structure, which includes creating new organizations and assigning initial Organization Administrators to those organizations. They play a critical role in onboarding new entities and ensuring that the administrative hierarchy is properly established.

- **Organization Administrators** 
Responsible for the internal management of their organizations. This includes updating organizational details such as names and addresses, managing other types of admins by creating new user(organization's employee) or modifying existing ones, and assigning or updating user roles to align with the organization's needs including assigning KYC admin roles, and more. Also, they can manage end users within their organization, including retrieving, updating, and disabling accounts.

> View [Sequence Diagram](https://github.com/sologenic/be-sequence-diagram/blob/main/src/admin-account-service/workflow.svg) for a visual representation of the workflow.

## Primary Sologenic Administrator Creation(optional)

The application can be configured with a primary Sologenic Administrator account at startup. When configured, this account will be created automatically (if it doesn't exist) and granted the Sologenic admin's privileges.

### Configuration

Set the `SOLOGENIC_ADMINISTRATOR` environment variable with a JSON string containing:
- `accountID`: Email address of the primary administrator
- `network`: Network where the account will be created ("mainnet", "testnet", or "devnet")

Example configuration:
```bash
SOLOGENIC_ADMINISTRATOR='{"accountID":"sg.primary.admin@gmail.com","network":"mainnet"}'
```

### Creating Additional Sologenic Administrators

Once at least one Sologenic Administrator exists in the system, they can create additional Sologenic Administrator accounts through the `/api/account/create-sologenic-admin` endpoint. These accounts will:
- Be created on the same network as specified in the request
- Have the SOLOGENIC_ADMINISTRATOR role

> **Note:** Only existing Sologenic Administrators can create new Sologenic Administrator accounts, and they can only create them on networks they have access to.

## API Endpoints Overview

### Sologenic Administrator Interfaces:

**Authenticated:**

Organization Onboarding:

* POST `/api/account/create-org-admin` - creates a new organization admin for newly registered organization and assign the organization admin roles

Sologenic Administrator Management:
* POST `/api/account/create-sologenic-admin` - creates a new Sologenic Administrator
* PUT `/api/account/update-sologenic-admin` - updates a Sologenic Administrator account
* PUT `/api/account/update-sologenic-admin/roles` - updates the roles of a specified Sologenic Administrator account
* PUT `/api/account/update-sologenic-admin/status` - updates the status of a Sologenic Administrator account

### Organization Administrator Interfaces:

**Authenticated:**

User Management:

* GET `/api/account/get?account_id=...` - retrieves the `Account` information for the given `AccountID`
* GET `/api/account/accounts?filter=...` - retrieves all `Account` information, uses `AccountFilter` to filter the results
* PUT `/api/account/update` - updates the account information for a specified account within the organization
* PUT `/api/account/update/roles` - updates the roles of a specified account within the organization.
* PUT `/api/account/update/status` - modifies the status of a specified account.

Internal Organization Management:

* POST `/api/account/create` - create a new account under their organization, with the `NORMAL_USER` role assigned by default.

Rolepath Management: 

* GET `/api/account/rolepaths` - retrieves all `Rolepath` information for the current organization and network
* PUT `/api/account/rolepaths` - updates the rolepath for an organization and network

> **Note:** All authenticated endpoints require a valid Firebase token in Authorization header. The token must be:
> - Obtained from the Firebase Authentication service
> - Prepended with `Bearer: ` e.g.`"Authorization: Bearer: eyJhb....`

## Authorization Check Route by Role

By calling injected authorization check route(`/isauth`) for each authenticated route, UI components can be rendered dynamically based on the user's role.

Each authenticated route has two corresponding routes:

1. **Main Route**: `{basepath}/{path}`
   - Used for actual HTTP operations
   - Returns 401 if unauthorized
   - Example: `/api/account/get`

2. **Authorization Check Route**: `{basepath}/isauth/{path}`
   - Checks if user has permission
   - Always returns 200 OK with authorization status
   - Useful for UI rendering decisions
   - Example: `/api/account/isauth/get`

### Example response for Authorization Check Route

```json5
{
    "Authorized": true,
    "Roles": [3, 7]
}
```

## API Endpoints Details

### POST /api/account/create-org-admin

(Sologenic Administrator Only)

Creates a new organization admin for the newly registered organization and assigns the organization admin roles. 
This endpoint will only be available to Sologenic administrators. `AccountID` and `ExternalUserID` will be generated automatically. The request body must contain the `Account` object as defined in the model. Request must include a `Network` header and an `Authorization` header.

> **Note:** New organization admins will be created with the same network of the sologenic admin who creates them.

#### Example request

```bash
curl -X POST "https://comsolotex-admin.sologenic.org/api/account/create-org-admin" \
-H "Content-Type: application/json" \
-H "Network: mainnet" \ 
-H "Authorization: Bearer: ...." \
-d '{
    "FirstName": "Billy",
    "LastName": "Bob",
    "Address": "SomeAddress",
    "Avatar": "someAvatar",
    "Alias": "BillyBob",
    "Description": "1st admin for new org",
    "Status": 1,
    "OrganizationID": "783174ea-d891-4c0c-9c69-c733850ea391"
}'
```

#### Example response

```json5
{
  "AccountID": "billy@abc.org",
  "Network": 1
}
```

### POST /api/account/create-sologenic-admin

(Sologenic Administrator Only)

Creates a new Sologenic Administrator account. This endpoint is only available to Sologenic Administrators. The account will be created (or updated) and its role escalated to `SOLOGENIC_ADMINISTRATOR`.

> **Note:** New Sologenic admins will be created with the same network of the sologenic admin who creates them.

#### Example request

```bash
curl -X POST "https://comsolotex-admin.sologenic.org/api/account/create-sologenic-admin" \
-H "Content-Type: application/json" \
-H "Network: mainnet" \
-H "Authorization: Bearer: ..." \
-d '{
    "AccountID": "sologenic.admin@sologenic.org",
    "FirstName": "Martin",
    "LastName": "Kim",
    "Description": "Sologenic admin"
}'

#### Example response

```json5
{
  "AccountID": "sologenic.admin@sologenic.org",
  "Network": 1
}
```

### PUT /api/account/update-sologenic-admin

(Sologenic Administrator Only)

Updates a Sologenic Administrator account. This endpoint allows Sologenic Administrators to update account information for other Sologenic Administrators. The request body must contain the `Account` object as defined in the model. Request must include a `Network` header and an `Authorization` header.

> **Note:** Role and Status won't be updated using this endpoint. Use the `/api/account/update-sologenic-admin/status` endpoint for status updates. Roles for Sologenic Administrators are managed separately through the primary admin system.

#### Example request

```bash
curl -X PUT "https://comsolotex-admin.sologenic.org/api/account/update-sologenic-admin" \
-H "Content-Type: application/json" \
-H "Network: mainnet" \
-H "Authorization: Bearer: ..." \
-d '{
    "AccountID": "sologenic.admin@sologenic.org",
    "FirstName": "Martin",
    "LastName": "Kim",
    "Address": "UpdatedAddress",
    "Avatar": "updatedAvatar",
    "Alias": "UpdatedAlias",
    "Description": "Updated Sologenic admin description",
    "Network": 1
}'
```

#### Example response

```json5
{
  "AccountID": "sologenic.admin@sologenic.org",
  "Network": 1
}
```

### PUT /api/account/update-sologenic-admin/roles

(Sologenic Administrator Only)

Updates the roles for an `Account`. The request body must contain the `SetRolesMessage` object as defined in the model. Request must include a `Network` header and an `Authorization` header.

#### Example request

```bash
curl -X PUT "https://comsolotex-admin.sologenic.org/api/account/update/roles" \
-H "Content-Type: application/json" \
-H "Authorization: Bearer: ...." \
-H "Network: testnet" \
-d '{
    "AccountID": "billy@abc.org",
    "Roles": [4, 6],
    "Network": 2
}'
```


### PUT /api/account/update-sologenic-admin/status

(Sologenic Administrator Only)

Updates the status of a Sologenic Administrator account. This endpoint allows Sologenic Administrators to activate, deactivate, or change the status of other Sologenic Administrator accounts. The request body must contain the `SetStatusMessage` object as defined in the model. Request must include a `Network` header and an `Authorization` header.

#### Example request

```bash
curl -X PUT "https://comsolotex-admin.sologenic.org/api/account/update-sologenic-admin/status" \
-H "Content-Type: application/json" \
-H "Network: mainnet" \
-H "Authorization: Bearer: ..." \
-d '{
    "AccountID": "sologenic.admin@sologenic.org",
    "Status": 1,
    "Network": 1
}'
```

#### Example response

```json5
{
  "AccountID": "sologenic.admin@sologenic.org",
  "Network": 1
}
```

### GET /api/account/get?account_id=...

Retrieves the `Account` information for the given `account_id`. The `Account` information is returned as an `Account` object as defined in the model. Request must include a `Network` header and an `Authorization` header.

##### Example request

```bash
curl -X GET "https://comsolotex-admin.sologenic.org/api/account/get?account_id=derrick@sologenic.org" \
-H "Content-Type: application/json" \
-H "Authorization: Bearer: ...." \
-H "Network: testnet"
```

##### Example response

```json5
{
    "AccountID": "derrick@sologenic.org",
    "FirstName": "Derrick",
    "LastName": "Quigley",
    "Address": "ABC3",
    "Wallets": [
        {
            "Address": "Awesome Tasty invoice1",
            "Alias": "Fresh back up1",
            "Type": 2
        },
        {
            "Address": "maroon Handmade Wooden Keyboard Borders1",
            "Alias": "bus program Versatile1",
            "Type": 3
        }
    ],
    "CreatedAt": {
        "seconds": 1720469429,
        "nanos": 503021000
    },
    "UpdatedAt": {
        "seconds": 694,
        "nanos": 721
    },
    "Socials": [
        {
            "URL": "http://lurline.name"
        },
        {
            "URL": "https://maryam.biz"
        }
    ],
    "Avatar": "https://cloudflare-ipfs.com/ipfs/Qmd3W5DuhgHirLHGVixi6V76LhCkZUz6pnFt5AJBiyvHye/avatar/493.jpg",
    "Alias": "ABC3",
    "Description": "invoice back up",
    "Network": 2,
    "Status": 1,
    "Roles": [
        1
    ],
    "ExternalUserID": "976e4b26-9a82-3353-c598-fff0c7ff2f32",
    "OrganizationID": "34422ce6-7b51-4a15-accb-34959f39d8ca"
}
```

### GET /api/account/accounts?filter=eyJSb2xlcyI6WzFd...someEncodedAccountFilter...iTGltaXQiOjIwfQ==

(Sologenic Administrator and Organization Administrator Only)

Retrieve all `Account` information based on the given `AccountFilter`, with pagination and an updated offset. The `AccountFilter` must be base64 encoded and included in the request query. If the limit and offset are not specified, the default limit is 20 and the offset is 0. Request must include a `Network` header and an `Authorization` header.

The `AccountFilter` object is defined as follows:

```proto
message AccountFilter {
    optional Role Role = 1;
    repeated string AccountIDs = 2;
    optional Order Order = 3;
    optional int32 Offset = 4;
    optional int32 Limit = 5;
    string Network = 6;
}
```

##### Example request

```bash
curl -X GET "https://comsolotex-admin.sologenic.org/api/account/accounts?filter=eyJSb2xlcyI6WzFdLCJBY2NvdW50SURzIjpbImYyNmY4NjJjLTBjNzAtYzYxYi0yNmFmLTczZDhlM2ZlZDg3ZiIsImY0YmVjMGZiLWZjMGItNTE4OS1jYzNkLWVlZGQwZDE0ZTNiOCIsIjk3MGIxYTY5LTA5NGEtOGZlZi04Y2FmLTI4MzMxYjJmNTU0MCJdLCJPcmRlciI6e30sIk9mZnNldCI6MSwiTGltaXQiOjIwfQ==" \
-H "Content-Type: application/json" \
-H "Authorization: Bearer: ...." \
-H "network: testnet"
```

##### Example response

```json5
{
    "Accounts": [
        {
            "AccountID": "cecilia@abc.org",
            "FirstName": "Cecilia",
            "LastName": "Mueller",
            "Address": "HBTestAddress",
            "Wallets": [
                {
                    "Address": "Future disintermediate",
                    "Alias": "cross-media Jordanian Dinar Internal",
                    "Type": 1
                },
                {
                    "Address": "Computers deposit TCP",
                    "Alias": "Clothing, Books \u0026 Sports Jewelery \u0026 Electronics",
                    "Type": 3
                }
            ],
            "CreatedAt": {
                "seconds": 1720564151,
                "nanos": 172969000
            },
            "UpdatedAt": {
                "seconds": 613,
                "nanos": 765
            },
            "Socials": [
                {
                    "URL": "http://samson.net",
                    "Type": 2
                },
                {
                    "URL": "http://celine.org",
                    "Type": 1
                }
            ],
            "Avatar": "https://cloudflare-ipfs.com/ipfs/Qmd3W5DuhgHirLHGVixi6V76LhCkZUz6pnFt5AJBiyvHye/avatar/616.jpg",
            "Alias": "feed",
            "Description": "Dynamic Isle",
            "Network": 2,
            "Status": 1,
            "Roles": [
                1,
                2
            ],
            "ExternalUserID": "2802f6b7-52ec-354a-5da7-33c0995ca972",
            "OrganizationID": "34422ce6-7b51-4a15-accb-34959f39d8ca"
        },
        {
            "AccountID": "admden@abc.org",
            "FirstName": "Camden",
            "LastName": "Larkin",
            "Address": "Arkansas1",
            "Wallets": [
                {
                    "Address": "Future disintermediate",
                    "Alias": "cross-media Jordanian Dinar Internal",
                    "Type": 1
                },
                {
                    "Address": "Computers deposit TCP",
                    "Alias": "Clothing, Books \u0026 Sports Jewelery \u0026 Electronics",
                    "Type": 3
                }
            ],
            "CreatedAt": {
                "seconds": 1720564067,
                "nanos": 201675000
            },
            "UpdatedAt": {
                "seconds": 613,
                "nanos": 765
            },
            "Socials": [
                {
                    "URL": "https://rowena.biz",
                    "Type": 2
                },
                {
                    "URL": "https://cortez.biz",
                    "Type": 1
                }
            ],
            "Avatar": "https://cloudflare-ipfs.com/ipfs/Qmd3W5DuhgHirLHGVixi6V76LhCkZUz6pnFt5AJBiyvHye/avatar/683.jpg",
            "Alias": "feed",
            "Description": "Dynamic Isle",
            "Network": 2,
            "Status": 2,
            "Roles": [
                1
            ],
            "ExternalUserID": "e1aad668-3af1-a4b4-557e-05ae7ca21d72",
            "OrganizationID": "34422ce6-7b51-4a15-accb-34959f39d8ca"
        }
    ],
    "Offset": 0
}
```

### POST /api/account/create

(Organization Administrator Only)

Creates a new account in the given organization. `AccountID` and `ExternalUserID` will be generated automatically. By default, the account will be created with the `NORMAL_USER` role. The `Account` object is to be provided in the request body as defined in the model. Request must include a `Network` header and an `Authorization` header.

#### Example request

```bash
curl -X POST "https://comsolotex-admin.sologenic.org/api/account/create" \
-H "Content-Type: application/json" \
-H "Network: testnet" \ 
-H "Authorization: Bearer: ...." \
-d '{
    "AccountID": "billy.bob@abc.org",
    "FirstName": "Billy",
    "LastName": "Bob",
    "Address": "SomeAddress",
    "Avatar": "someAvatar",
    "Alias": "BillyBob",
    "Description": "some description",
    "Status": 1,
    "OrganizationID": "34422ce6-7b51-4a15-accb-34959f39d8ca"
}'
```

#### Example response

```json5
{
  "AccountID": "billy.bob@abc.org",
  "Network": 2
}
```

### PUT /api/account/update

(Organization Administrator Only)

Updates the account information for a specified account within the organization. The request body must contain the `Account` object as defined in the model. Request must include a `Network` header and an `Authorization` header.

> **Note:** Role and Status won't be updated using this endpoint. Use the `/api/account/update/roles` and `/api/account/update/status` endpoints for these operations.

#### Example request

```bash
curl -X POST "https://comsolotex-admin.sologenic.org/api/account/update" \
-H "Content-Type: application/json" \
-H "Network: mainnet" \ 
-H "Authorization: Bearer: ...." \
-d '{
    "AccountID": "billy.bob@abc.org",
    "FirstName": "Billy",
    "LastName": "Bob",
    "Address": "SomeAddress",
    "Avatar": "someAvatar",
    "Alias": "BillyBob",
    "Description": "some description",
    "Network": 1,
    "ExternalUserID": "022a60d0-c78e-4eb5-92be-e417a466acee",
    "OrganizationID": "215a551d-5691-91ce-f4a6-9284f40d1340"
}'
```

#### Example response

```json5
{
  "AccountID": "billy.bob@abc.org",
  "Network": 1
}
```

### PUT /api/account/update/roles

(Organization Administrator Only)

Updates the roles for an `Account`. The request body must contain the `SetRolesMessage` object as defined in the model. Request must include a `Network` header and an `Authorization` header.

> **Note:** Including the `SOLOGENIC_ADMINISTRATOR` role will result in an error.

#### Example request

```bash
curl -X PUT "https://comsolotex-admin.sologenic.org/api/account/update/roles" \
-H "Content-Type: application/json" \
-H "Authorization: Bearer: ...." \
-H "Network: testnet" \
-d '{
    "AccountID": "billy@abc.org",
    "Roles": [1, 2],
    "Network": 1
}'
```

### PUT /api/account/update/status

(Organization Administrator Only)

Updates the status for an `Account`. The request body must contain the `SetStatusMessage` object as defined in the model. Request must include a `Network` header and an `Authorization` header.

#### Example request

```bash
curl -X PUT "https://comsolotex-admin.sologenic.org/api/account/update/status" \
-H "Content-Type: application/json" \
-H "Authorization: Bearer: ...." \
-H "Network: testnet" \
-d '{
    "AccountID": "billy@abc.org",
    "Status": 1,
    "Network": 1
}'
```

### GET /api/account/rolepaths

(Organization Administrator Only)

Retrieves all `Rolepath` information for the current organization and network. Request must include a `Network` header and an `Authorization` header.

#### Example request

```bash
curl -X GET "https://comsolotex-admin.sologenic.org/api/account/rolepaths" \
-H "Content-Type: application/json" \
-H "Network: mainnet" \
-H "Authorization: Bearer: ...."
```

#### Example response

```json
{
  "RolePaths": {
    "RolePaths": [
      {
        "Path": "create",
        "BasePath": "account",
        "Method": 1,
        "OrganizationID": "215a551d-5691-91ce-f4a6-9284f40d1340",
        "Role": [
            4,
            7
        ],
        "CreatedAt": {
            "seconds": 1732910539,
            "nanos": 358492000
        },
        "UpdatedAt": {
            "seconds": 1732910539,
            "nanos": 358493000
        },
        "Network": 1
      }
    ]
  }
}
```

### PUT /api/account/rolepaths

(Organization Administrator Only)

Updates the rolepath for an organization and network. The request body must contain the `RolePaths` object as defined in the model. Request must include a `Network` header and an `Authorization` header.

> **Note:** Organization Administrators can create or update roles within their privilege level but cannot create or update role paths that require higher privileges than their own (e.g., roles requiring Sologenic Administrator privileges).

#### Example request

```bash
curl -X PUT "https://comsolotex-admin.sologenic.org/api/account/rolepaths" \
-H "Content-Type: application/json" \
-H "Network: testnet" \
-H "Authorization: Bearer: ...." \
-d '{
    "Path": "create",
    "BasePath": "account",
    "Method": 1,
    "Role": [3,4,7],
    "Description": "Update role testing",
}'
```

## Application start parameters

The application uses the following environment parameters:

* `HTTP_CONFIG` - the configuration for the http server (included from `github.com/sologenic/be-http-lib/`)
* `AUTH_FIREBASE_SERVICE` - the firebase authentication service endpoint
* `ACCOUNT_STORE` - the account service endpoint (included from `github.com/sologenic/com-fs-admin-account-model/`)
* `FEATURE_FLAG_STORE` - the feature flag service endpoint (included from `github.com/sologenic/fs-feature-flag-model/`)
* `ORGANIZATION_STORE` - the organization service endpoint (included from `github.com/sologenic/fs-admin-organization-model/`)
* `ROLE_STORE` - the role service endpoint (included from `github.com/sologenic/com-fs-admin-role-model/`)
* `SOLOGENIC_ADMINISTRATOR` - (optional) If set with a JSON object containing accountID and network (e.g., `{"accountID":"admin@sologenic.org","network":"mainnet"}`), that account is injected into the database as a Sologenic Administrator avoiding all security checks (This is to setup the application initially)
