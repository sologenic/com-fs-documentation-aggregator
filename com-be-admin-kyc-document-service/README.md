# Document service

The kyc admin document service provides the following restful interfaces:

- GET `api/adminkycdoc/get?id=...` - Get a document by ID
- GET `api/adminkycdoc/list?persona_id=...&user_id=...&inquery_id=...&type=...` - List documents belonging to a user using filters

> **Note:** All authenticated endpoints require a valid Firebase token in Authorization header, the organization ID in the Organization header, and the network in the Network header. The token must be:
>
> - Obtained from the Firebase Authentication service
> - Prepended with `Bearer: ` e.g.`"Authorization: Bearer: eyJhb....`

## API endpoints details

### GET api/adminkycdoc/get?id=...

Get a document by ID. This ID is an internal ID and not that of the Persona assigned ID.

#### Example request

```bash
curl -X GET \
"https://comsolotex-admin.sologenic.org/api/adminkycdoc/get?id=f47ac10b-58cc-4372-a567-0e02b2c3d479" \
-H "Authorization: Bearer ..." \
-H "OrganizationID: ..." \
-H "Network: mainnet"
```

#### Example response

```json
{
  "DocumentID": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "PersonaDocumentID": "doc_abc123def456",
  "InquiryID": "inq_xyz789uvw012",
  "UserID": "someUser@gmail.com",
  "FileName": "passport_front",
  "FilePath": "/path/to/document/in/storage",
  "Type": "passport",
  "OrganizationID": "some-org-id",
  "Network": 1,
  "CreatedAt": {
    "seconds": "1717027800",
    "nanos": 0
  },
  "ExpiresAt": {
    "seconds": "1874793800",
    "nanos": 0
  }
}
```

### GET api/adminkycdoc/list?persona_id=...&user_id=...&inquery_id=...&type=...

List documents based on filters. The filters can include:

- `persona_id`: The ID of the persona to which the documents belong.
- `user_id`: The ID of the user to which the documents belong.
- `inquery_id`: The ID of the inquiry to which the documents belong.
- `type`: The type of document.

#### Example request

```bash
curl -X GET \
"https://comsolotex-admin.sologenic.org/api/adminkycdoc/list?user_id=someUser@gmail.com" \
-H "Authorization: Bearer ..." \
-H "OrganizationID: ..." \
-H "Network: mainnet"
```

#### Example response

```json
{
  "KYCDocuments": [
    {
      "DocumentID": "b2c3d4e5-f6g7-8901-bcde-f23456789012",
      "PersonaDocumentID": "doc_def456ghi789",
      "InquiryID": "inq_xyz789uvw012",
      "UserID": "someUser@gmail.com",
      "FileName": "passport_front",
      "FilePath": "/path/to/document/in/storage",
      "Type": "passport",
      "OrganizationID": "some-org-id",
      "Network": 1,
      "CreatedAt": {
        "seconds": "1717027800",
        "nanos": 0
      },
      "ExpiresAt": {
        "seconds": "1874793800",
        "nanos": 0
      }
    },
    {
      "DocumentID": "c3d4e5f6-g7h8-9012-cdef-345678901234",
      "PersonaDocumentID": "doc_ghi789jkl012",
      "InquiryID": "inq_xyz789uvw012",
      "UserID": "someUser@gmail.com",
      "FileName": "passport_back",
      "FilePath": "/path/to/document/in/storage",
      "Type": "passport",
      "OrganizationID": "some-org-id",
      "Network": 1,
      "CreatedAt": {
        "seconds": "1717027820",
        "nanos": 0
      },
      "ExpiresAt": {
        "seconds": "1874793820",
        "nanos": 0
      }
    },
    {
      "DocumentID": "d4e5f6g7-h8i9-0123-def4-456789012345",
      "PersonaDocumentID": "doc_jkl012mno345",
      "InquiryID": "inq_xyz789uvw012",
      "UserID": "someUser@gmail.com",
      "FileName": "selfie_verification",
      "FilePath": "/path/to/document/in/storage",
      "Type": "passport",
      "OrganizationID": "some-org-id",
      "Network": 1,
      "CreatedAt": {
        "seconds": "1717027840",
        "nanos": 0
      },
      "ExpiresAt": {
        "seconds": "1874793840",
        "nanos": 0
      }
    }
  ],
  "Offset": 0
}
```

## Application start parameters

The application requires the following environment variables to be set:

- `AUTH_FIREBASE_SERVICE`: Included from `github.com/sologenic/com-fs-auth-firebase-service`. For setup see `github.com/sologenic/com-fs-auth-firebase-service/README.md`
- `KYC_DOCUMENT_STORE`: Included from `github.com/sologenic/com-fs-admin-kyc-document-model`. For setup see `github.com/sologenic/com-fs-admin-kyc-document-model/client/README.md`
- `ACCOUNT_STORE`: Included from `github.com/sologenic/com-fs-admin-account-model`. For setup see `github.com/sologenic/com-fs-admin-account-model/client/README.md`
- `ROLE_STORE`: Included from `github.com/sologenic/com-fs-role-model`. For setup see `github.com/sologenic/com-fs-role-model/client/README.md`
- `FEATURE_FLAG_STORE`: Included from `github.com/sologenic/com-fs-feature-flag-model`. For setup see `github.com/sologenic/com-fs-feature-flag-model/client/README.md`
- `ORGANIZATION_STORE` - Included from `github.com/sologenic/com-be-admin-certificate-store/`. For setup see `github.com/sologenic/com-be-admin-certificate-store/README.md`
- `CERTIFICATE_STORE`: Included from `github.com/sologenic/com-be-http-lib/http/`. For setup see `github.com/sologenic/com-be-http-lib/http/README.md`
- `HTTP_CONFIG`: Included from `github.com/sologenic/com-be-http-lib/http/`. For setup see `github.com/sologenic/com-be-http-lib/http/README.md`
