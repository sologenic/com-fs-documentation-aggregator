# Alert service

The alert service provides the following RESTful interfaces for managing price alerts:

- `POST /api/alert/add`: Creates a new price alert
- `DELETE /api/alert/delete`: Removes an existing alert owned by the authenticated user
- `GET /api/alert/get?asset_key=...`: Retrieves a specific alert by asset key

## POST /api/alert/add

Creates a new price alert for the specified asset.

### Parameters

The request body should contain a JSON object with the following fields:

- `AssetKey`: The unique identifier for the asset to monitor
- `TargetPrice`: The price at which the alert should trigger
- `Status`: The alert status (refer to the `AlertStatus` enum in `com-fs-alert-model` for valid values)

### Example curl POST

```bash
curl --request POST \
  --url http://localhost:8080/api/alert/add \
  --header 'Authorization: Bearer eyJhbGciOiJSUzI1NiIs...' \
  --header 'Content-Type: application/json' \
  --header 'Network: testnet' \
  --header 'organizationid: 72c4c072-2fe4-4f72-ae9d-d9d52a05fd71' \
  --data '{
    "AssetKey": "test-assetkey-1",
    "TargetPrice": 150.50,
    "Status": 1,
  }'
```

### Response

#### Success (201 Created)

```json
{
  "message": "alert created successfully"
}
```

#### Error (400 Bad Request)

```json
{
  "error": "price validation failed: target price 200.00 is 25.50% away from current price 150.00 (max allowed: 10.50%)"
}
```

#### Error (400 Bad Request - Missing Fields)

```json
{
  "error": "missing or invalid required fields"
}
```

## DELETE /api/alert/delete

Removes an existing alert. Only the owner of the alert (authenticated user) can delete it.

### Parameters

The request body should contain a JSON object with:

- `AssetKey`: The unique identifier of the alert to delete

### Example curl DELETE

```bash
curl --request DELETE \
  --url http://localhost:8080/api/alert/delete \
  --header 'Authorization: Bearer eyJhbGciOiJSUzI1NiIs...' \
  --header 'Content-Type: application/json' \
  --header 'Network: testnet' \
  --header 'organizationid: 72c4c072-2fe4-4f72-ae9d-d9d52a05fd71' \
  --data '{
    "AssetKey": "test-assetkey-1"
  }'
```

### Response

#### Success (200 OK)

```json
{
  "message": "alert deleted successfully"
}
```

#### Error (404 Not Found)

```json
{
  "error": "alert not found"
}
```

## GET /api/alert/get?asset_key=...

Retrieves a specific alert by its asset key. Only the owner of the alert can retrieve it.

### Parameters

- `asset_key` (query parameter): The unique identifier of the alert to retrieve

### Example curl GET

```bash
curl --request GET \
  --url 'http://localhost:8080/api/alert/get?asset_key=test-assetkey-1' \
  --header 'Authorization: Bearer eyJhbGciOiJSUzI1NiIs...' \
  --header 'Network: testnet' \
  --header 'organizationid: 72c4c072-2fe4-4f72-ae9d-d9d52a05fd71'
```

### Response

#### Success (200 OK)

```json
{
  "Alert": {
    "Account": "O7ksna16ZhYlSh6TPTeCJLbNQq73",
    "AssetKey": "test-assetkey-1",
    "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
    "TargetPrice": 150.5,
    "Status": 1
  },
  "Audit": {
    "ChangedAt": {
      "seconds": 1749580302,
      "nanos": 123456000
    },
    "ChangedBy": "O7ksna16ZhYlSh6TPTeCJLbNQq73"
  },
  "MetaData": {
    "Network": 1,
    "CreatedAt": {
      "seconds": 1749580302,
      "nanos": 123456000
    },
    "UpdatedAt": {
      "seconds": 1749580302,
      "nanos": 123456000
    },
    "UpdatedByAccount": "O7ksna16ZhYlSh6TPTeCJLbNQq73"
  }
}
```

#### Error (404 Not Found)

```json
{
  "error": "alert not found"
}
```

#### Error (400 Bad Request)

```json
{
  "error": "missing asset_key parameter"
}
```

## Application start parameters

The application requires the following environment variables to be set:

- `ALERT_STORE`: Connection string for the alert store service. Included from `github.com/sologenic/com-fs-alert-model`. For setup see `github.com/sologenic/com-fs-alert-model/client/README.md`
- `ORGANIZATION_STORE`: Included from `github.com/sologenic/com-fs-organization-model`. For setup see `github.com/sologenic/com-fs-organization-model/client/README.md`
- `AUTH_FIREBASE_SERVICE`: Connection string for Firebase authentication service. Included from `github.com/sologenic/com-fs-auth-firebase-model`. For setup see `github.com/sologenic/com-fs-auth-firebase-model/README.md`
- `ROLE_STORE`: Connection string for the role management service. Included from `github.com/sologenic/com-fs-role-model`. For setup see `github.com/sologenic/com-fs-role-model/client/README.md`
- `FEATURE_FLAG_STORE` - the feature flag service endpoint (included from `github.com/sologenic/com-fs-feature-flag-model/`)
- `USER_STORE`: Connection string for the user management service. Included from `github.com/sologenic/com-fs-user-model`. For setup see `github.com/sologenic/com-fs-user-model/client/README.md`
- `HTTP_CONFIG`: HTTP server configuration including port, CORS settings, and timeouts. Included from `github.com/sologenic/com-be-http-lib/http/`. For setup see `github.com/sologenic/com-be-http-lib/http/README.md`

### Example environment configuration

```bash
ALERT_STORE=localhost:50053
AUTH_FIREBASE_SERVICE=localhost:50070
ROLE_STORE=localhost:50066
USER_STORE=localhost:50049
HTTP_CONFIG='{"port": ":8080","cors": {"allowedOrigins":["http://localhost:3000"]},"timeouts": {"read": "10s","write": "10s","idle": "10s","shutdown": "10s"}}'
```
