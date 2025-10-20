# Document service

The document service provides the following restful interfaces:

**Authenticated:**

* GET `/api/doc/get?md5sum=...` - Get a document by md5sum
* GET `/api/doc/list?filter=...`: Returns a list of documents based on the provided filter.
* PUT `/api/doc/sign`: Sign a document.

> **Note:** All authenticated endpoints require a valid Firebase token in Authorization header, the organization ID in the Organization header, and the network in the Network header. The token must be:
> - Obtained from the Firebase Authentication service
> - Prepended with `Bearer: ` e.g.`"Authorization: Bearer: eyJhb....`

## API endpoints details

### GET `/api/doc/get?md5sum=...`

Get a document by md5sum. *md5sum is required.*

#### Example request

```bash
curl -X GET \
"https://comsolotex.sologenic.org/api/doc/get?md5sum=..." \
-H "Authorization: Bearer: eyJhb....` \
-H "OrganizationID: ..." \
-H "Network: mainnet"
```

#### Example response

```json
{
    "Document": {
        "OrganizationID": "org123",
        "Name": "SampleDocument",
        "Version": "1.0.0",
        "Description": "This is a sample document",
        "File": {
            "Reference": "file-ref-123",
            "Extension": "pdf",
            "Name": "Sample Contract Document",
            "MD5SUM": "d41d8cd98f00b204e9800998ecf8427e"
        },
        "SignatureRequired": true,
        "Status": "ACTIVE"
    }
}
```

### GET `/api/doc/list?filter=...`

Get a list of documents based on the provided filter parameters.

#### Query Parameters

- `signature_required` *optional, boolean*: Filter documents by signature requirement
- `offset` *optional, integer*: Number of items to skip for pagination

#### Example request

```bash
curl -X GET "https://comsolotex.sologenic.org/api/doc/list?signature_required=true&offset=0" \
-H "Authorization: Bearer: eyJhb....` \
-H "OrganizationID: ..." \
-H "Network: mainnet"
```

#### Example response

```json
{
    "Documents": [
        {
            "OrganizationID": "org123",
            "Name": "SampleDocument",
            "Version": "1.0.0",
            "Description": "This is a sample document",
            "File": {
                "Reference": "file-ref-123",
                "Extension": "pdf",
                "Name": "Sample Contract Document",
                "MD5SUM": "d41d8cd98f00b204e9800998ecf8427e"
            },
            "SignatureRequired": true,
            "Status": "ACTIVE"
        }
    ],
    "Offset": 0
}
```

### PUT `/api/doc/sign`

Sign document(s). *References to the signed document(s) are stored in `User`.*

#### Example request

```bash
curl -X PUT "https://comsolotex.sologenic.org/api/doc/sign" \
-H "Authorization: Bearer: eyJhb....`" \
-H "OrganizationID: ..." \
-H "Network: mainnet" \
-H "Content-Type: application/json" \
-d '{
    "Name": "Terms and Conditions",
    "SignedVersion": "1.0.0",
    "DocumentState": 2,
    "FileMD5SUM": "d41d8cd98f00b204e9800998ecf8427e"
}'
```

## Application start parameters

The application uses the following environment parameters:

* `ORGANIZATION_STORE`: Included from `github.com/sologenic/com-fs-organization-model`. For setup see `github.com/sologenic/com-fs-organization-model/client/README.md`
* `HTTP_CONFIG` - the configuration for the http server (included from `github.com/sologenic/com-be-http-lib/`)
* `AUTH_FIREBASE_SERVICE` - the firebase authentication service endpoint
* `USER_STORE` - the user service endpoint (included from `github.com/sologenic/com-fs-user-model/`)
* `FEATURE_FLAG_STORE` - the feature flag service endpoint (included from `github.com/sologenic/com-fs-feature-flag-model/`)
* `ROLE_STORE` - the role service endpoint (included from `github.com/sologenic/com-fs-role-model/`)
* `DOCUMENT_STORE` - the document service endpoint (included from `github.com/sologenic/com-fs-document-model/`)
