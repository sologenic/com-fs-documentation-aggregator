# Organization service

The organization service provides the following RESTFUL interface:

**Authenticated:**

Organization - Organization admin:

- GET `/api/organization/get` - retrieves the `Organization` information for the organization to which the caller belongs
- PUT `/api/organization/update` - updates the `Organization` information for the organization to which the caller belongs

Organization - Sologenic admin:

- GET `/api/organization/list?offset=...` - retrieves a list of all `Organization`
- POST `/api/organization/create` - create a new organization

## GET /api/organization/get

Retrieves the `Organization` information for the organization to which the caller belongs. Request must include an `Authorization` header with the `Authorization` token and the `Network` header.

### Example request

```bash
curl -X GET "https://comsolotex-admin.sologenic.org/api/organization/get" \
-H "Content-Type: application/json" \
-H "Network: mainnet" \
-H "Authorization:eyJQcml2YXRlS2......OGY0ZSJ9" \
```

### Example response

```json5
{
  OrganizationID: "96b5e611-4bc7-4967-b5f9-7aee68aea158",
  Name: "atg-org2",
  Description: "atg test org2",
  Logo: "http://tyrique.name",
  URL: "https://hudson.com",
  CreatedAt: {
    seconds: 1722367552,
  },
  UpdatedAt: {
    seconds: 1722369970,
  },
}
```

## PUT /api/organization/update

Updates the `Organization` information for the organization to which the caller belongs. The request body must contain the `Organization` object as defined in the model. Request must include a `Network` header and an `Authorization` header.

### Example request

```bash
curl -X PUT "https://comsolotex-admin.sologenic.org/api/organization/update" \
-H "Content-Type: application/json" \
-H "Network: mainnet" \
-H "Authorization:eyJQcml2YXRlS2...someEncodedToken...OGY0ZSJ9" \
-d '{
    "OrganizationID": "34422ce6-7b51-4a15-accb-34959f39d8ca",
    "Name": "com-atg",
    "Description": "atg test org",
    "Logo": "http://coty.name.com",
    "URL": "https://pete.info",
}'
```

### Example response

```json5
{
  OrganizationID: "34422ce6-7b51-4a15-accb-34959f39d8ca",
}
```

## GET /api/organization/list?offset=...&limit=...

Retrieves a list of all `Organization`. Request must include a `Network` header and an `Authorization` header.

## Example request

```bash
curl -X GET "https://comsolotex-admin.sologenic.org/api/organization/list?offset=0&limit=10" \
-H "Content-Type: application/json" \
-H "Network: mainnet" \
-H "Authorization:eyJQcml2YXRlS2......OGY0ZSJ9" \
```

## Example response

```json5
{
  Organizations: [
    {
      OrganizationID: "34422ce6-7b51-4a15-accb-34959f39d8ca",
      Name: "com-atg-org 1",
      Description: "atg-test org1",
      Logo: "http://coty.name.com",
      URL: "https://pete.info",
      CreatedAt: {
        seconds: 1722035004,
      },
      UpdatedAt: {
        seconds: 1722622853,
      },
    },
    {
      OrganizationID: "783174ea-d891-4c0c-9c69-c733850ea391",
      Name: "New ORG 2",
      Description: "Test Register ",
      Logo: "http://abc.name.com",
      URL: "https://cde.info",
      CreatedAt: {
        seconds: 1722635122,
      },
      UpdatedAt: {
        seconds: 1722635122,
      },
    },
  ],
  Offset: 0,
}
```

## POST /api/organization/create

Creates a new organization. This endpoint will only be available to Sologenic administrators. The request body must contain the `Organization` object as defined in the model. `OrganizationID` be generated automatically. Request must include a `Network` header and an `Authorization` header.

### Example request

```bash
curl -X POST "https://comsolotex-admin.sologenic.org/api/organization/create" \
-H "Content-Type: application/json" \
-H "Network: mainnet" \
-H "Authorization:eyJQcml2YXRlS2......OGY0ZSJ9" \
-d '{
    "Name": "New ORG 2",
    "Description": "Test Create",
    "Logo": "http://abc.name.com",
    "URL": "https://cde.info"
    "AdminEmail": "someEmail@some.com",
}'
```

### Example response

```json5
{
  OrganizationID: "783174ea-d891-4c0c-9c69-c733850ea391",
}
```

## Application start parameters

The application uses the following environment parameters:

- `HTTP_CONFIG` - the configuration for the http server (included from `github.com/sologenic/com-be-http-lib/`)
- `AUTH_FIREBASE_SERVICE` - the authentication service endpoint (included from `github.com/sologenic/com-fs-auth-firebase-model/`)
- `ACCOUNT_STORE` - the account service endpoint (included from `github.com/sologenic/com-fs-admin-account-model/`)
- `ROLE_STORE` - the role service endpoint (included from `github.com/sologenic/com-fs-admin-role-model/`)
- `FEATURE_FLAG_STORE` - the feature flag service endpoint (included from `github.com/sologenic/com-fs-admin-feature-flag-model/`)
- `ORGANIZATION_STORE` - the organization service endpoint (included from `github.com/sologenic/com-fs-admin-organization-model/`)
- `GRPC_APPEND` - segment of the service URL that follows the `service` keyword (i.e. given the URL `https://com-be-my-service-dfjiao-ijgao.a.run.app`, extract the portion after `service` including the `-`, in this case `dfjiao-ijgao.a.run.app`)
- `SOLOGENIC_ADMINISTRATOR` - If set with a JSON string containing `accountID` and `network` (e.g. `{"accountID":"admin@example.com","network":"testnet"}`), that account is injected into the database as a Sologenic organizational administrator, bypassing all security checks. This
- `COMPLIANCE_MANAGER_CODE_ID` - Contract code id of the compliance manager contract in the Coreum network
