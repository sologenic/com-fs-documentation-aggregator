# Admin feature flag service

The admin feature flag service provides the following RESTFUL interface:

**Authenticated:**

* GET `/api/featureflag/get?featureflag_name=...` - retrieves the `FeatureFlag` information for the given `FeatureFlagName`
* GET `/api/featureflag/list?filter=...` - retrieves all `FeatureFlags` information, uses `Filter` to filter the results
* PUT `/api/featureflag/update` - updates a `FeatureFlag` for the given `FeatureFlagName`

## GET /api/featureflag/get?featureflag_name=...

Retrieves the `FeatureFlag` information for the given `featureFlagName`. The `FeatureFlag` information is returned as an `FeatureFlag` object as defined in the model. Request must include a `Network` header and an `Authorization` header.

TODO: finish once data is available

### Example request

```bash
curl -X GET "" \
-H "Content-Type: application/json" \
-H "Authorization:eyJQcml2YXRlS2......OGY0ZSJ9" \
-H "Network: mainnet"
```

### Example response

```json5
```

## Application start parameters

The application uses the following environment parameters:

* `HTTP_CONFIG` - the configuration for the http server (included from `github.com/sologenic/com-be-http-lib/`)
* `AUTH_SERVICE` - the authentication service endpoint (included from `github.com/sologenic/com-fs-auth-model/`)
* `ROLE_STORE` - the role store service endpoint (included from `github.com/sologenic/com-fs-role-model/`)
* `FEATURE_FLAG_STORE` - the feature flag service endpoint (included from `github.com/sologenic/com-fs-feature-flag-model/`)
* `GRPC_APPEND` - segment of the service URL that follows the `service` keyword (i.e. given the URL `https://com-be-my-service-dfjiao-ijgao.a.run.app`, extract the portion after `service` including the `-`, in this case `dfjiao-ijgao.a.run.app`)
* `ORGANIZATION_STORE` - the organization service endpoint (included from `github.com/sologenic/com-fs-organization-store/`)