# Contact List service

The contact list service provides the following restful interfaces:

* `POST /marketing/email/upsert`: Upsert an email into the contact list

## POST /marketing/email/upsert

Upsert an email into the contact list.

### Request body

```json
{
    "Email": "test@test.com",
    "Name": "Test User",
    "CompanyName": "Test Company",
    "URL": "https://test.com",
}
```

Response is always 200OK.

### Example call

```bash
curl -X POST https://apiv2.sologenic.com/api/marketing/email/upsert \
-H "Content-Type: application/json" \
-d '{"Email": "test@test.com", "Name": "Test User", "CompanyName": "Test Company", "URL": "https://test.com"}'
```

## Application start parameters

The application requires the following environment variables to be set:

* `AUTH_FIREBASE_SERVICE`: Included from `github.com/sologenic/com-fs-auth-firebase-service`. For setup see `github.com/sologenic/com-fs-auth-firebase-service/README.md`
* `CONTACT_LIST_STORE`: Included from `github.com/sologenic/com-fs-contact-list-model`. For setup see `github.com/sologenic/com-fs-contact-list-model/client/README.md`
* `USER_STORE`: Included from `github.com/sologenic/com-fs-user-model`. For setup see `github.com/sologenic/com-fs-user-model/client/README.md`
* `ROLE_STORE`: Included from `github.com/sologenic/com-fs-role-model`. For setup see `github.com/sologenic/com-fs-role-model/client/README.md`
* `HTTP_CONFIG`: Included from `github.com/sologenic/com-be-http-lib/http/`. For setup see `github.com/sologenic/com-be-http-lib/http/README.md`
* `FEATURE_FLAG_STORE`: Included from `github.com/sologenic/com-fs-feature-flag-model`. For setup see `github.com/sologenic/com-fs-feature-flag-model/client/README.md`
* `ORGANIZATION_STORE`: Included from `github.com/sologenic/com-fs-organization-model`. For setup see `github.com/sologenic/com-fs-organization-model/client/README.md`
