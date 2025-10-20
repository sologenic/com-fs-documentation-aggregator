# Admin Certificate Service

The Admin certificate service provides the following RESTFUL interface:

**Authenticated:**

* GET `/api/admincertificate/get?type=...`: Retrieves a certificate by type
* GET `/api/admincertificate/list`: Retrieves a list of all certificates for the organization
* POST `/api/emailtemplate/upsert`: Upsert a certificate

> **Note:** 
> - All admin endpoints are protected by role-based access control and default to the`ORGANIZATION_ADMINISTRATOR` role. Permissions are managed dynamically by Organization administrators on the fly.

## GET /api/admincertificate/get?type=1

Retrieves a certificate by type

### Example call

```bash
curl -X GET \
  https://comsolotex-admin.sologenic.org/api/admincertificate/get?type=1 \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ..." \
  -H "Network: testnet"
```

### Example response

```json
{
    "Certificate": {
        "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
        "Type": 1,
        "CredentialsJSON": "{\"type\":\"service_account\",\"project_id\":\"my-project-12345\",\"private_key_id\":\"abc123def456ghi789\",\"private_key\":\"-----BEGIN PRIVATE KEY-----\\nMIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQC7...\\n-----END PRIVATE KEY-----\\n\",\"client_email\":\"my-service-account@my-project-12345.iam.gserviceaccount.com\",\"client_id\":\"123456789012345678901\",\"auth_uri\":\"https://accounts.google.com/o/oauth2/auth\",\"token_uri\":\"https://oauth2.googleapis.com/token\",\"auth_provider_x509_cert_url\":\"https://www.googleapis.com/oauth2/v1/certs\",\"client_x509_cert_url\":\"https://www.googleapis.com/robot/v1/metadata/x509/my-service-account%40my-project-12345.iam.gserviceaccount.com\"}",
        "Description": "Storage access for production environment111"
    },
    "MetaData": {
        "UpdatedAt": {
            "seconds": 1748392726,
            "nanos": 950528000
        },
        "CreatedAt": {
            "seconds": 1748392214,
            "nanos": 950670000
        }
    },
    "Audit": {
        "ChangedBy": "sg.org.testnet@gmail.com",
        "ChangedAt": {
            "seconds": 1748392726,
            "nanos": 950552000
        }
    }
}
```
## GET /api/admincertificate/list

Retrieves a list of all certificates.

### Example call

```bash
curl -X GET \
  https://comsolotex-admin.sologenic.org/api/admincertificate/list \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ..." \
  -H "Network: testnet"
```

### Example response

```json
{
    "Certificates": [
        {
            "Certificate": {
                "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
                "Type": 1,
                "CredentialsJSON": "{\"type\":\"service_account\",\"project_id\":\"my-project-12345\",\"private_key_id\":\"abc123def456ghi789\",\"private_key\":\"-----BEGIN PRIVATE KEY-----\\nMIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQC7...\\n-----END PRIVATE KEY-----\\n\",\"client_email\":\"my-service-account@my-project-12345.iam.gserviceaccount.com\",\"client_id\":\"123456789012345678901\",\"auth_uri\":\"https://accounts.google.com/o/oauth2/auth\",\"token_uri\":\"https://oauth2.googleapis.com/token\",\"auth_provider_x509_cert_url\":\"https://www.googleapis.com/oauth2/v1/certs\",\"client_x509_cert_url\":\"https://www.googleapis.com/robot/v1/metadata/x509/my-service-account%40my-project-12345.iam.gserviceaccount.com\"}",
                "Description": "Storage access for production environment111"
            },
            "MetaData": {
                "UpdatedAt": {
                    "seconds": 1748392726,
                    "nanos": 950528000
                },
                "CreatedAt": {
                    "seconds": 1748392214,
                    "nanos": 950670000
                }
            },
            "Audit": {
                "ChangedBy": "sg.org.testnet@gmail.com",
                "ChangedAt": {
                    "seconds": 1748392726,
                    "nanos": 950552000
                }
            }
        },
        {
            "Certificate": {
                "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
                "Type": 2,
                "CredentialsJSON": "{\"type\":\"service_account\",\"project_id\":\"my-project-12345\",\"private_key_id\":\"abc123def456ghi789\",\"private_key\":\"-----BEGIN PRIVATE KEY-----\\nMIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQC7...\\n-----END PRIVATE KEY-----\\n\",\"client_email\":\"my-service-account@my-project-12345.iam.gserviceaccount.com\",\"client_id\":\"123456789012345678901\",\"auth_uri\":\"https://accounts.google.com/o/oauth2/auth\",\"token_uri\":\"https://oauth2.googleapis.com/token\",\"auth_provider_x509_cert_url\":\"https://www.googleapis.com/oauth2/v1/certs\",\"client_x509_cert_url\":\"https://www.googleapis.com/robot/v1/metadata/x509/my-service-account%40my-project-12345.iam.gserviceaccount.com\"}",
                "Description": "12332132"
            },
            "MetaData": {
                "UpdatedAt": {
                    "seconds": 1748392971,
                    "nanos": 381590000
                },
                "CreatedAt": {
                    "seconds": 1748392782,
                    "nanos": 740613000
                }
            },
            "Audit": {
                "ChangedBy": "sg.org.testnet@gmail.com",
                "ChangedAt": {
                    "seconds": 1748392971,
                    "nanos": 381591000
                }
            }
        }
    ]
}
```

## POST /api/admincertificate/upsert

Creates or updates a certificate.

### Example call

```bash
curl -X POST \
  https://comsolotex-admin.sologenic.org/api/admincertificate/upsert \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ..." \
  -H "Network: testnet" \
  -d '{
  "Certificate": {
    "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
    "Type": 2,
    "CredentialsJSON": "{\"type\":\"service_account\",\"project_id\":\"my-project-12345\",\"private_key_id\":\"abc123def456ghi789\",\"private_key\":\"-----BEGIN PRIVATE KEY-----\\nMIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQC7...\\n-----END PRIVATE KEY-----\\n\",\"client_email\":\"my-service-account@my-project-12345.iam.gserviceaccount.com\",\"client_id\":\"123456789012345678901\",\"auth_uri\":\"https://accounts.google.com/o/oauth2/auth\",\"token_uri\":\"https://oauth2.googleapis.com/token\",\"auth_provider_x509_cert_url\":\"https://www.googleapis.com/oauth2/v1/certs\",\"client_x509_cert_url\":\"https://www.googleapis.com/robot/v1/metadata/x509/my-service-account%40my-project-12345.iam.gserviceaccount.com\"}",
    "Description": "Storage access for production environment"
  }
}'
```

## Start parameters

The following start parameters are required:

* `ACCOUNT_STORE`: the account store endpoint (included from `github.com/sologenic/com-be-admin-account-store/`)
* `ROLE_STORE`: the role store endpoint (included from `github.com/sologenic/com-be-admin-role-store/`)
* `FEATURE_FLAG_STORE` - the feature flag service endpoint (included from `github.com/sologenic/fs-feature-flag-model/`)
* `AUTH_FIREBASE_SERVICE`: the firebase authentication service endpoint
* `HTTP_CONFIG`: the configuration for the http server (included from `github.com/sologenic/com-be-http-lib/`)
* `CERTIFICATE_STORE`: the certificate store endpoint (included from `github.com/sologenic/com-be-admin-certificate-store/`)
* `ORGANIZATION_STORE` - the organization service endpoint (included from `github.com/sologenic/com-fs-organization-store/`)
* `LOG_LEVEL` - the logging level for the application (e.g., "info", "debug", "warn", "error")