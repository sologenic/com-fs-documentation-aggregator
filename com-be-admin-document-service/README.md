# Admin document service

The admin document service provides document lifecycle management with version control, status management for organizational compliance.

## Document Lifecycle Management

### Understanding Document Statuses
Documents have three states in the system:
- **UNPUBLISHED (Draft)**: Editable documents that can be modified freely, but not yet published(`Name` and `Version` are immutable)
- **ACTIVE**: Published documents that are current and may require user signatures depends on the `SignatureRequired` flag. Once published, documents cannot be modified except to mark them as outdated.
- **OUTDATED**: Archived documents preserved for historical reference but no longer considered current. They cannot be modified or reactivated.

### Document Management Workflow

1. **Create Draft Documents**: Upload PDF files and create document records with version, description, and signature requirement flag. Files are stored temporarily until published.

2. **Edit & Update**: Draft documents can be edited for metadata only - modify description or signature requirement flag. `Name`, `Version`, and `File` cannot be changed once created. To change a file, create a new version. Active and Outdated documents cannot be edited.

3. **Publish Workflow**: Publish draft documents to make them active. Each document name can have a maximum of 1 active version - publishing a new version automatically marks any existing active version with the same name as outdated.

4. **Status Management**: Documents transition through status(`UNPUBLISHED`, `ACTIVE`, `OUTDATED`). Once published, documents cannot be modified.

5. **Data Integrity Protection**: The system automatically resolves data integrity violations. If multiple active documents are detected for the same name (which should not occur under normal operation), all conflicting active documents are automatically set to OUTDATED status to maintain system consistency.

> **Note:** 
> - Once a document is published, its PDF file and signature requirements cannot be updated
> - Signature requirements are immutable after document creation for architectural consistency
> - To make changes to an active document, create a new version as a draft and publish it
> - Each document name can only have one active version at a time
> - The system includes auto-recovery mechanisms to handle data integrity violations gracefully

## API Endpoints Overview

**Authenticated:**

### General Document Management:
* GET `/api/admindoc/list?filter=...`: Returns a list of documents based on the provided filter.

### Draft Document Management: 
* POST `/api/admindoc/create/draft`: Creates a new draft document.
* PUT `/api/admindoc/update/draft`: Updates an existing draft document and only unpublished documents can be updated.
* PUT `/api/admindoc/publish`: Publishes the specified document, marking it as required for user signature and marking all previous versions as outdated.

### Active Document Management(Published documents):
* PUT `/api/admindoc/outdated`: Marks the specified document as outdated, preserving the document for historical reference.

> **Note:** All authenticated endpoints require a valid Firebase token in Authorization header, the organization ID in the Organization header, and the network in the Network header. The token must be:
> - Obtained from the Firebase Authentication service
> - Prepended with `Bearer: ` e.g.`"Authorization: Bearer: eyJhb....`

## API endpoints details

### GET /api/admindoc/list?filter=eyJSb2xlcyI6WzFd...someEncodedOrderQueryFilter...iTGltaXQiOjIwfQ==

Retrieve all documents based on the given `filter`, with pagination and an updated offset. The `filter` must be base64 encoded and included in the request query. If the limit and offset are not specified, the default limit is 20 and the offset is 0.

The `Filter` object is defined as follows(defined in `com-fs-document-model`):

```proto
message Filter {
    string OrganizationID = 1;
    optional string Name = 2;
    optional bool SignatureRequired = 3;
    optional DocumentStatus Status = 4;
    optional int32 Offset = 5;
    optional int32 Limit = 6;
}
```

### Example call

```bash
curl -X GET \
  https://comsolotex-admin.sologenic.org/api/admindoc/list?filter=ewogICJPcmdhbml6YXRpb25JRCI6ICI3MmM0YzA3Mi0yZmU0LTRmNzItYWU5ZC1kOWQ1MmEwNWZkNzEiLAogICJMaW1pdCI6IDEwMAp9 \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ..." \
  -H "OrganizationID: 72c4c072-2fe4-4f72-ae9d-d9d52a05fd71"
  -H "Network: testnet"
```

### Example response
```json
{
    "Documents": [
        {
            "Document": {
                "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
                "Name": "Regulatory",
                "Version": "v2",
                "Description": "Draft",
                "File": {
                    "Reference": "eyJBZGRyZXNzIjoic2cub3JnLnRlc3RuZXRAZ21haWwuY29tIiwiTmV0d29yayI6MiwiVGVtcEZpbGVuYW1lIjoiUmVndWxhdG9yeSBUZXN0XzIucGRmIiwiVGltZXN0YW1wIjoxNzQ4OTg3MjE0Njg3ODIwMDAwfQ==.pdf",
                    "Extension": "pdf",
                    "Name": "Regulatory Test_2.pdf",
                    "MD5SUM": "b1a2c3d4e5f6g7h8i9j0k1l2m3n4o5p6"
                },
                "SignatureRequired": true,
                "Status": 1
            },
            "MetaData": {
                "UpdatedAt": {
                    "seconds": 1748987215,
                    "nanos": 802662000
                },
                "CreatedAt": {
                    "seconds": 1748987215,
                    "nanos": 802661000
                }
            },
            "Audit": {
                "ChangedBy": "sg.org.testnet@gmail.com",
                "ChangedAt": {
                    "seconds": 1748987215,
                    "nanos": 802663000
                }
            }
        },
        {
            "Document": {
                "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
                "Name": "Regulatory",
                "Version": "v1",
                "Description": "Test",
                "File": {
                    "Reference": "testnet/document/72c4c072-2fe4-4f72-ae9d-d9d52a05fd71/Regulatory_v1",
                    "Extension": "pdf",
                    "Name": "Regulatory",
                    "MD5SUM": "c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8"
                },
                "SignatureRequired": true,
                "Status": 2
            },
            "MetaData": {
                "UpdatedAt": {
                    "seconds": 1748987082,
                    "nanos": 626485000
                },
                "CreatedAt": {
                    "seconds": 1748987069,
                    "nanos": 212503000
                }
            },
            "Audit": {
                "ChangedBy": "sg.org.testnet@gmail.com",
                "ChangedAt": {
                    "seconds": 1748987082,
                    "nanos": 626486000
                }
            }
        },
        {
            "Document": {
                "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
                "Name": "Privacy",
                "Version": "1.2",
                "Description": "Test",
                "File": {
                    "Reference": "testnet/document/72c4c072-2fe4-4f72-ae9d-d9d52a05fd71/Privacy_1.2",
                    "Extension": "pdf",
                    "Name": "Privacy",
                    "MD5SUM": "d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t"
                },
                "SignatureRequired": true,
                "Status": 2
            },
            "MetaData": {
                "UpdatedAt": {
                    "seconds": 1748986761,
                    "nanos": 431254000
                },
                "CreatedAt": {
                    "seconds": 1748986752,
                    "nanos": 664446000
                }
            },
            "Audit": {
                "ChangedBy": "sg.org.testnet@gmail.com",
                "ChangedAt": {
                    "seconds": 1748986761,
                    "nanos": 431255000
                }
            }
        },
        {
            "Document": {
                "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
                "Name": "Privacy",
                "Version": "1.1",
                "Description": "Test",
                "File": {
                    "Reference": "testnet/document/72c4c072-2fe4-4f72-ae9d-d9d52a05fd71/Privacy_1.1",
                    "Extension": "pdf",
                    "Name": "Privacy",
                    "MD5SUM": "e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0u1"
                },
                "SignatureRequired": true,
                "Status": 3
            },
            "MetaData": {
                "UpdatedAt": {
                    "seconds": 1748986761,
                    "nanos": 51447000
                },
                "CreatedAt": {
                    "seconds": 1748986665,
                    "nanos": 860168000
                }
            },
            "Audit": {
                "ChangedBy": "sg.org.testnet@gmail.com",
                "ChangedAt": {
                    "seconds": 1748986761,
                    "nanos": 51448000
                }
            }
        },
        {
            "Document": {
                "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
                "Name": "Privacy",
                "Version": "1.0",
                "Description": "Test",
                "File": {
                    "Reference": "testnet/document/72c4c072-2fe4-4f72-ae9d-d9d52a05fd71/Privacy_1.0",
                    "Extension": "pdf",
                    "Name": "Privacy",
                    "MD5SUM": "f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0u1v2"
                },
                "SignatureRequired": true,
                "Status": 3
            },
            "MetaData": {
                "UpdatedAt": {
                    "seconds": 1748986674,
                    "nanos": 555895000
                },
                "CreatedAt": {
                    "seconds": 1748986622,
                    "nanos": 903508000
                }
            },
            "Audit": {
                "ChangedBy": "sg.org.testnet@gmail.com",
                "ChangedAt": {
                    "seconds": 1748986674,
                    "nanos": 555896000
                }
            }
        }
    ]
}
```

## POST /api/admindoc/create/draft

Create a new draft document.

> **Note**: 
> - If a document with the same `Name`, and `Version` combination already exists, the request will be rejected 

### Example call

```bash
curl -X POST "https://comsolotex-admin.sologenic.org/api/admindoc/create/draft" \
-H "Content-Type: application/json" \
-H "Network: testnet" \ 
-H "OrganizationID: 72c4c072-2fe4-4f72-ae9d-d9d52a05fd71" \
-H "Authorization: Bearer: ...." \
-d '{
  "Document": {
    "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
    "Name": "Margin Agreement",
    "Version": "1.0.0",
    "Description": "Margin Agreement for testing purposes",
    "File": {
      "Reference":"eyJBZGRyZXNzIjoiaGVhdmVuOTFAaG90bWFpbC5jb20iLCJOZXR3b3JrIjoyLCJUZW1wRmlsZW5hbWUiOiJhYWEuanBnIiwiVGltZXN0YW1wIjoxNzQzNTU0NDQwMDU2NjYzNjM4fQ==.pdf",
      "Extension": "pdf",
      "Name": "Margin Agreement",
      "MD5SUM": "3a43056cb0bae9fed74dffaf9b3f7d73"
    },
    "SignatureRequired": true
  }
}'
```

## PUT /api/admindoc/update/draft

(Unpublished documents only)

Update an existing draft document. Only metadata updates are allowed - files cannot be changed.

> **Note**: 
> - Only `UNPUBLISHED` documents(drafts) can be updated
> - `Name`, `Version`, and `File` cannot be changed once a document is created
> - Only `Description` and `SignatureRequired` can be updated
> - To change a file, create a new version of the document

### Example call

```bash
curl -X PUT "https://comsolotex-admin.sologenic.org/api/admindoc/update/draft" \
-H "Content-Type: application/json" \
-H "Network: testnet" \
-H "OrganizationID: 72c4c072-2fe4-4f72-ae9d-d9d52a05fd71" \
-H "Authorization: Bearer: ...." \
-d '{
  "Document": {
    "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
    "Name": "Privacy Policy",
    "Version": "2.0.0",
    "Description": "Updated privacy policy with GDPR compliance",
    "File": {
      "MD5SUM": "a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6"
    },
    "SignatureRequired": false,
    "Status": 1
  }
}'
```

## PUT /api/admindoc/publish

(Unpublished documents only)

Publish a draft document, making it active and automatically marking any existing active version with the same name as outdated.

> **Note**: 
> - Only `UNPUBLISHED` documents(drafts) can be published
> - Publishing automatically outdates any existing active document with the same name
> - The file reference is updated to permanent storage during publishing
> - Only one active document per name is allowed at any time

### Example call

```bash
curl -X PUT "https://comsolotex-admin.sologenic.org/api/admindoc/publish" \
-H "Content-Type: application/json" \
-H "Network: testnet" \
-H "OrganizationID: 72c4c072-2fe4-4f72-ae9d-d9d52a05fd71" \
-H "Authorization: Bearer: ...." \
-d '{
  "Document": {
    "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
    "Name": "Terms of Service",
    "Version": "3.1.0",
    "File": {
      "Reference": "eyJBZGRyZXNzIjoiaGVhdmVuOTFAaG90bWFpbC5jb20iLCJOZXR3b3JrIjoyLCJUZW1wRmlsZW5hbWUiOiJhYWEuanBnIiwiVGltZXN0YW1wIjoxNzQzNTU0NDQwMDU2NjYzNjM4fQ==.pdf",
      "Extension": "pdf",
      "Name": "Terms of Service Final",
      "MD5SUM": "z9y8x7w6v5u4t3s2r1q0p9o8n7m6l5k4"
    },
    "SignatureRequired": false,
    "Status": 1
  }
}'
```

## PUT /api/admindoc/outdated

(Active documents only)

Manually mark an active document as outdated. It will preserve the document for historical reference but will no longer be considered active.

> **Note**: 
> - Only `Active` documents can be marked as outdated
> - This action is irreversible - outdated documents cannot be reactivated

### Example call

```bash
curl -X PUT "https://comsolotex-admin.sologenic.org/api/admindoc/outdated" \
-H "Content-Type: application/json" \
-H "Network: testnet" \
-H "OrganizationID: 72c4c072-2fe4-4f72-ae9d-d9d52a05fd71" \
-H "Authorization: Bearer: ...." \
-d '{
  "Document": {
    "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
    "Name": "Terms of Service",
    "Version": "3.1.0",
    "File": {
      "Reference": "testnet/document/72c4c072-2fe4-4f72-ae9d-d9d52a05fd71/Terms_of_Service_3.1.0",
      "Extension": "pdf",
      "Name": "Terms of Service Final",
      "MD5SUM": "z9y8x7w6v5u4t3s2r1q0p9o8n7m6l5k4"
    },
    "SignatureRequired": false,
    "Status": 2
  }
}'
```

## Application start parameters

The application requires the following environment variables to be set:

* `AUTH_FIREBASE_SERVICE`: Included from `github.com/sologenic/com-fs-auth-firebase-service`. For setup see `github.com/sologenic/com-fs-auth-firebase-service/README.md`
* `DOCUMENT_STORE`: Included from `github.com/sologenic/com-fs-document-model`. For setup see `github.com/sologenic/com-fs-document-model/client/README.md`
* `ACCOUNT_STORE`: Included from `github.com/sologenic/com-fs-admin-account-model`. For setup see `github.com/sologenic/com-fs-admin-account-model/client/README.md`
* `ROLE_STORE`: Included from `github.com/sologenic/com-fs-role-model`. For setup see `github.com/sologenic/com-fs-role-model/client/README.md`
* `FEATURE_FLAG_STORE`: Included from `github.com/sologenic/com-fs-feature-flag-model`. For setup see `github.com/sologenic/com-fs-feature-flag-model/client/README.md`
* `FILE_STORE`: Included from `github.com/sologenic/com-fs-feature-flag-model`. For setup see `github.com/sologenic/com-fs-feature-flag-model/client/README.md`
* `ORGANIZATION_STORE` - the organization service endpoint (included from `github.com/sologenic/com-fs-organization-store/`)
* `HTTP_CONFIG`: Included from `github.com/sologenic/com-be-http-lib/http/`. For setup see `github.com/sologenic/com-be-http-lib/http/README.md`
