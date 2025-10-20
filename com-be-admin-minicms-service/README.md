# Admin MiniCMS Service

The Admin MiniCMS Service provides lightweight content management functionality, enabling clients to retrieve active content blocks configured for specific pages and organizations.

## API Endpoints Overview

**_Authenticated:_**

> **Note:** `OrganizationID` is a mandatory parameter that enforces proper organizational data boundaries, ensuring each tenant can only access records within their designated scope.

- `GET /api/adminminicms/get`  
  Retrieves active content for a given identifier, organization, and language.
- `GET /api/adminminicms/list`  
  Lists content with pagination support. The list can be filtered by identifier. Includes support for `limit` and `offset` parameters to control the number of results returned.
- `POST /api/adminminicms/create`
  Creates a new content block.

- `PUT /api/adminminicms/update`  
  Updates an existing content block.

> **Note:** All endpoints require a valid Firebase token in the `Authorization` header:
>
> - Token must be obtained from Firebase Authentication
> - The header must be prepended with `Bearer: `  
>   e.g. `"Authorization: Bearer: eyJhb...."`

### `GET /api/adminminicms/get`

Retrieves a single piece of active content for the requested ContentID.

#### Required Headers

- `Authorization`: Firebase Bearer token
- `OrganizationID`: UUID string
- `Network`: string (e.g., `mainnet`, `testnet`)

#### Required Query Parameters

- `content_id`: A string representing the ID of the content to be retrieved.

### Example Request

```bash
curl -X GET "https://comsolotex.sologenic.org/api/adminminicms/get?content_id=1234-5678" \
-H "Authorization: Bearer: ...." \
-H "OrganizationID: 215a551d-...-9284f40d1340" \
-H "Network: mainnet"
```

### Example Response

```json
{
  "Content": {
    "ContentID": "1234-5678",
    "BackgroundImage": {
      "Reference": "cms/images/marketing_banner",
      "Extension": "jpg",
      "Name": "banner"
    },
    "Title": "Welcome to Solotex",
    "Content": "Asset tokenization and trading platform",
    "CallToAction": "https://solotex.com",
    "ActiveFrom": {
      "Seconds": 1741900000
    },
    "ActiveUntil": {
      "Seconds": 1745500000
    },
    "Status": true,
    "Identifier": "markets_top_left_block",
    "Language": {
      "Language": "{...}"
    },
    "OrganizationID": "215a551d-...-9284f40d1340"
  },
  "MetaData": {
    "CreatedAt": {
      "Seconds": 1741890000
    },
    "UpdatedAt": {
      "Seconds": 1741980000
    },
    "Network": 2
  },
  "Audit": {
    "ChangedAt": {
      "Seconds": 1741983951
    }
  }
}
```

### `GET /api/adminminicms/list`

Lists content with pagination support. The list can be filtered by identifier. Includes support for `limit` and `offset` parameters to control the number of results returned.

#### Required Headers

- `Authorization`: Firebase Bearer Token
- `OrganizationID`: UUID string
- `Network`: string (e.g., `mainnet`, `testnet`)

#### Query Parameters

- `identifier`: the identifier string of the content
- `limit`: (optional) integer, default is 20
- `offset`: (optional) integer, default is 0

The endpoint will retrieve a list of content based on the provided filters. If the limit and offset are not specified, the default values are `Limit = 20` and `Offset = 0`. Request header must include `Network` and `OrganizationID` headers.

### Example Request

```bash
curl -X GET "https://comsolotex.sologenic.org/api/adminminicms/list?identifier=...&offset=...&limit=..." \
-H "Authorization: Bearer: ...." \
-H "Content-Type: application/json" \
-H "OrganizationID: 215a551d-...-9284f40d1340" \
-H "Network: mainnet"
```

### Example Response

```json
{
  "Content": [
    {
      "ContentID": "1234-5678",
      "BackgroundImage": {
        "Reference": "cms/images/marketing_banner",
        "Extension": "jpg",
        "Name": "banner"
      },
      "Title": "Welcome to Solotex",
      "Content": "Asset tokenization and trading platform",
      "CallToAction": "https://solotex.com",
      "ActiveFrom": {
        "Seconds": 1741900000
      },
      "ActiveUntil": {
        "Seconds": 1745500000
      },
      "Status": true,
      "Identifier": "markets_bottom_left_block",
      "Language": {
        "Language": "en-US"
      },
      "OrganizationID": "215a551d-...-9284f40d1340"
    },
    {
      "ContentID": "5678-1234",
      "BackgroundImage": {
        "Reference": "cms/images/another_banner",
        "Extension": "jpg",
        "Name": "banner"
      },
      "Title": "Welcome to Solotex",
      "Content": "Asset tokenization and trading platform",
      "CallToAction": "https://solotex.com",
      "ActiveFrom": {
        "Seconds": 1741900000
      },
      "ActiveUntil": {
        "Seconds": 1745500000
      },
      "Status": true,
      "Identifier": "markets_bottom_left_block",
      "Language": {
        "Language": "{...}"
      },
      "OrganizationID": "215a551d-...-9284f40d1340"
    }
  ],
  "MetaData": {
    "CreatedAt": {
      "Seconds": 1741890000
    },
    "UpdatedAt": {
      "Seconds": 1741980000
    },
    "Network": 2
  },
  "Audit": {
    "ChangedAt": {
      "Seconds": 1741983951
    }
  }
}
```

### `POST /api/adminminicms/create`

Creates a new content block. The request body must contain the content in JSON format with the shape of the content proto.

#### Required Headers

- `Authorization`: Firebase Bearer Token
- `OrganizationID`: UUID string
- `Network`: string (e.g., `mainnet`, `testnet`)

#### Request Body

The request body must be a JSON object with the shape of the `Content` as defined by the model. The `ContentID` field has to be empty as it will be generated on creation.

### Example Request

```bash
curl -X POST "https://comsolotex.sologenic.org/api/adminminicms/create" \
-H "Content-Type: application/json" \
-H "Authorization: Bearer: ...." \
-H "OrganizationID: 215a551d-...-9284f40d1340" \
-H "Network: mainnet"
-d '{
  "Content": {
    "ContentDetails": {
      "BackgroundImage": {
        "Reference": "cms/images/marketing_banner",
        "Extension": "jpg",
        "Name": "banner"
      },
      "Title": "Welcome to Solotex",
      "Content": "Asset tokenization and trading platform",
      "CallToAction": "https://solotex.com",
      "ActiveFrom": {
        "Seconds": 1741900000
      },
      "ActiveUntil": {
        "Seconds": 1745500000
      },
      "Status": true,
      "Identifier": "markets_top_left_block",
      "Language": {
        "Language": "{...}"
      },
      "OrganizationID": "215a551d-...-9284f40d1340"
    }
  }
}'
```

### Example Response

```json
{
  "ContentID": "1234-5678"
}
```

### `PUT /api/adminminicms/update`

Updates an existing content block. The request body must contain the `Content` in JSON format as defined by the model.

#### Required Headers

- `Authorization`: Firebase Bearer Token
- `OrganizationID`: UUID string
- `Network`: string (e.g., `mainnet`, `testnet`)

#### Request Body

The request body must be a JSON object with the shape of the `Content` as defined by the model. The `ContentID` field must be set to the ID of the content to be updated.

### Example Request

```bash
curl -X PUT "https://comsolotex.sologenic.org/api/adminminicms/update" \
-H "Content-Type: application/json" \
-H "Authorization: Bearer: ...." \
-H "OrganizationID: 215a551d-...-9284f40d1340" \
-H "Network: mainnet"
-d '{
  "Content": {
    "ContentID": "1234-5678",
    "ContentDetails": {
      "BackgroundImage": {
        "Reference": "cms/images/marketing_banner",
        "Extension": "jpg",
        "Name": "banner"
      },
      "Title": "Welcome to Solotex",
      "Content": "Asset tokenization and trading platform",
      "CallToAction": "https://solotex.com",
      "ActiveFrom": {
        "Seconds": 1741900000
      },
      "ActiveUntil": {
        "Seconds": 1745500000
      },
      "Status": true,
      "Identifier": "markets_top_left_block",
      "Language": {
        "Language": "{...}"
      },
      "OrganizationID": "215a551d-...-9284f40d1340"
    }
  },
  "Audit": {...},
  "MetaData": {...}
}'
```

## Content Activation Logic (only affects user facing endpoints; admin returns all content)

The MiniCMS service implements a content activation logic to ensure that only relevant and timely content is served to users. The criteria for content to be considered "active" are as follows:

- `Status` is `true`
- Current UTC time is after `ActiveFrom` (if set)
- Current UTC time is before `ActiveUntil` (if set)

This ensures that time-sensitive marketing content or banners are only returned when they are intended to be live. Admins however can retrieve all content regardless of its activation status.

## Application Start Parameters

The MiniCMS service uses the following environment variables to start:

- `HTTP_CONFIG` - the configuration for the http server (included from `github.com/sologenic/com-be-http-lib/`)
- `FILE_STORE`: Included from `github.com/sologenic/com-fs-file-model`. For setup see `github.com/sologenic/com-be-file-store/`
- `MINICMS_STORE`: The MiniCMS service endpoint (included from `github.com/sologenic/com-be-minicms-store/`)
- `AUTH_FIREBASE_SERVICE` - the firebase authentication service endpoint
- `ACCOUNT_STORE` - the admin account service endpoint (included from `github.com/sologenic/com-be-admin-account-store/`)
- `ROLE_STORE` - the admin role service endpoint (included from `github.com/sologenic/com-be-role-store/`)
- `FEATURE_FLAG_STORE` - the feature flag service endpoint (included from `github.com/sologenic/com-be-feature-flag-store/`)
* `ORGANIZATION_STORE` - the organization service endpoint (included from `github.com/sologenic/com-fs-organization-store/`)
- `CREDENTIALS_LOCATION` - Location of the service account credentials file in json format
- `PROJECT_ID` - The project id of the project the storage is running in
- `CERTIFICATE_STORE` - The location of the certificate store (required for multi-tenant support)
