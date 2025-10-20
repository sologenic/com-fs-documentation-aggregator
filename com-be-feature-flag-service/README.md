# Feature flag service

The feature flag (user facing) service provides the following RESTFUL interface:

**Unauthenticated:**

* GET `/api/featureflag/list` - retrieves all active `FeatureFlags`

## GET /api/featureflag/list

Retrieves the `FeatureFlags` information. The `FeatureFlag` information is returned as an `FeatureFlags` object as defined in the model. Request must include a `Network` and `OrganizationID` as headers.

TODO: below will be true after caching is introduced:
The fetched `FeatureFlags` is to be cached for a period of time to reduce the number of requests to the feature flag service.  Once the cache expires, FE will call the service again to get the latest `FeatureFlags`.

It is important to note that we deliberately do not return any information about disabled or future features, to ensure they remain undiscoverable. This design helps prevent potential malicious activities from inferring upcoming features or unannounced products by examining API responses, as well as to avoid unintended marketing leaks, where the presence of new feature flags could spoil announcements or roadmaps.

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

* `ORGANIZATION_STORE`: Included from `github.com/sologenic/com-fs-organization-model`. For setup see `github.com/sologenic/com-fs-organization-model/client/README.md`
* `HTTP_CONFIG` - the configuration for the http server (included from `github.com/sologenic/com-be-http-lib/`)
* `FEATURE_FLAG_STORE` - the feature flag service endpoint (included from `github.com/sologenic/com-fs-feature-flag-model/`)
* `GRPC_APPEND` - segment of the service URL that follows the `service` keyword (i.e. given the URL `https://com-be-my-service-dfjiao-ijgao.a.run.app`, extract the portion after `service` including the `-`, in this case `dfjiao-ijgao.a.run.app`)
