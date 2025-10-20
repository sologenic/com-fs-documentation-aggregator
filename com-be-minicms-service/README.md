# MiniCMS Service

The MiniCMS Service provides lightweight content management functionality, enabling clients to retrieve active content blocks configured for specific pages and organizations.

## API Endpoints Overview

**_Unauthenticated:_**

> **Note:** `OrganizationID` is a mandatory parameter that enforces proper organizational data boundaries, ensuring each tenant can only access records within their designated scope.
> **Note:** The `Network` header is required to specify the network context (e.g., mainnet, testnet) for the request.

- `GET /api/minicms/get`  
  Retrieves a single piece of active content for a given identifier, organization, and language.
- `GET /api/minicms/list`  
  Lists content with pagination support. The list can be filtered by identifier. Includes support for `limit` and `offset` parameters to control the number of results returned.

### `GET /api/minicms/get`

Retrieves a single piece of active content for the requested ContentID.

#### Required Headers

- `OrganizationID`: UUID string
- `Network`: string (e.g., `mainnet`, `testnet`)

#### Required Query Parameters

- `content_id`: A string representing the ID of the content to be retrieved.

### Example Request

```bash
curl -X GET "https://comsolotex.sologenic.org/api/minicms/get?content_id=1234-5678" \
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
      "seconds": 1741900000
    },
    "ActiveUntil": {
      "seconds": 1745500000
    },
    "Status": true,
    "Identifier": "markets_top_left_block",
    "Language": {
      "Language": "en-US"
    },
    "OrganizationID": "215a551d-...-9284f40d1340"
  },
  "MetaData": {
    "CreatedAt": {
      "seconds": 1741890000
    },
    "UpdatedAt": {
      "seconds": 1741980000
    },
    "Network": 2
  },
  "Audit": {
    "ChangedAt": {
      "seconds": 1741983951
    }
  }
}
```

### `GET /api/minicms/list`

Lists content with pagination support. The list can be filtered by identifier. Includes support for `limit` and `offset` parameters to control the number of results returned.

#### Required Headers

- `OrganizationID`: UUID string
- `Network`: string (e.g., `mainnet`, `testnet`)

#### Query Parameters

- `identifier`: the identifier string of the content
- `limit`: (optional) integer, default is 20
- `offset`: (optional) integer, default is 0

The endpoint will retrieve a list of content based on the provided filters. If the limit and offset are not specified, the default values are `Limit = 20` and `Offset = 0`. Request header must include `Network` and `OrganizationID` headers.

### Example Request

```bash
curl -X GET "https://comsolotex.sologenic.org/api/minicms/list?identifier=...&offset=...&limit=..." \
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
        "seconds": 1741900000
      },
      "ActiveUntil": {
        "seconds": 1745500000
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
        "seconds": 1741900000
      },
      "ActiveUntil": {
        "seconds": 1745500000
      },
      "Status": true,
      "Identifier": "markets_bottom_left_block",
      "Language": {
        "Language": "en-US"
      },
      "OrganizationID": "215a551d-...-9284f40d1340"
    }
  ],
  "MetaData": {
    "CreatedAt": {
      "seconds": 1741890000
    },
    "UpdatedAt": {
      "seconds": 1741980000
    },
    "Network": 2
  },
  "Audit": {
    "ChangedAt": {
      "seconds": 1741983951
    }
  }
}
```

## Content Activation Logic

Content will only be returned if the following conditions are true:

- `Status` is `true`
- Current UTC time is after `ActiveFrom` (if set)
- Current UTC time is before `ActiveUntil` (if set)

This ensures that time-sensitive marketing content or banners are only returned when they are intended to be live.

## Data Model (Proto)

### ContentDetails

```proto
message ContentDetails {
  File BackgroundImage = 1;
  string Title = 2;
  string Content = 3;
  string CallToAction = 4;
  google.protobuf.Timestamp ActiveFrom = 5;
  google.protobuf.Timestamp ActiveUntil = 6;
  bool Status = 7;
  string Identifier = 8;
  language.Language Language = 9;
  string OrganizationID = 10;
}
```

### File

```proto
message File {
  string Reference = 1;
  string Extension = 2;
  optional string Name = 3;
}
```

## Application Start Parameters

The MiniCMS service uses the following environment variables to start:

- `HTTP_CONFIG`: Included from `github.com/sologenic/com-be-http-lib/http/`. For setup see `github.com/sologenic/com-be-http-lib/http/README.md`
- `AUTH_FIREBASE_SERVICE`: Included from `github.com/sologenic/com-fs-auth-firebase-service`. For setup see `github.com/sologenic/com-fs-auth-firebase-service/README.md`
- `ROLE_STORE`: Included from `github.com/sologenic/com-fs-role-model`. For setup see `github.com/sologenic/com-fs-role-model/client/README.md`
- `FEATURE_FLAG_STORE` (optional if feature flags present): Included from `github.com/sologenic/com-fs-feature-flag-model`. For setup see `github.com/sologenic/com-fs-feature-flag-model/client/README.md`
- `ORGANIZATION_STORE`: Included from github.com/sologenic/com-fs-organization-model. For setup see github.com/sologenic/com-fs-organization-model/client/README.
