# Auth service firebase

The firebase auth service provides functionality to interact with the Google Firebase authentication API.

## Usage

The usage is implemented into the http server such that the day to day development is the same as with any other auth service.

Functions:

* `ValidateToken`
* `Signout`
* `SignoutUserByEmail`

### ValidateToken

Validates a token and returns the user id (email address) if user is authenticated.
Token input expected is a firebase token prepended with `Bearer: `.

### Signout

Signout the user by using the token. Token input expected is a firebase token prepended with `Bearer: `.

Example of an authorization header:

```sh
curl -X POST "http://localhost:8080/somecall" \
    -H "network: devnet" \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer: eyJhb...."
```

### SignoutUserByEmail

Signout the user by using the email address. Token in the authrequest is set to to the email address of the user.

This is a method used in case a token is not present and we want a forced signout. This can also be used by backend functions to evict a user from the system (e.g. It makes it possible for the admin to signout a user, or for maintenance to signout all users).

## Test

There is a test program to test the service. The test program is a simple gRPC client that sends a token to the service and expects a response. The token is retrieved from the FE by inspecting the user object in the console.

## Start parameters

- `GRPC_PORT` - Port the service listens on for gRPC requests
- `FIREBASE_CONFIG` - Points to a secret with the json with the admin auth token (download in firebase console)
- `EXPIRE` - Expiry time in days for the token
- `ORGANIZATION_STORE`: Included from github.com/sologenic/com-fs-organization-model. For setup see github.com/sologenic/com-fs-organization-model/client/README.md