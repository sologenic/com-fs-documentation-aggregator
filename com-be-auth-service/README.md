# Auth service

The auth service implements the fs-auth-model and uses the fs-account-model to provide authentication and authorization for the other services.

The fs-auth-model contains client code for direct usage of the auth service.

The auth service has network dependencies and is connected to multiple cosmos networks at once.

The auth service depends on the gcloud datastore to remove tokens which are past the expiration date (env `EXPIRY` in days since creation).

## Token signing

The auth service depends on access to the correct coreum node associated with the network to validate the signature of the token. 
The coreum nodes are configured in the env variable                 

```json
{"NETWORKS": {"GRPC": [{"Network": "devnet","Host": "full-node.devnet-1.coreum.dev:9090"}]}}
```

In which networks is an array.

The application also requires the auth smart contract address (AUTH_ADDRESSES) to be able to retrieve the data from the blockchain:

```json
{"AuthAddresses": [{"Network": "devnet","Address": "123345"}]}
```

In which networks is an array.

## Service however no http API

The naming of this repo is a bit deviant from other repos with GRPC and HTTP restful interfaces:
This services provides a GRPC interface and is for internal usage only, however it is not a store, does not persist anything. 
To call it a store based on the non-external access and grpc interface however would also be misleading.

This is effectively the only internal service, so the exception on the otherwise well working naming convention in use.

