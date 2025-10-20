# com-be-admin-jurisdiction-asset-mgmt-service

The Admin Jurisdiction and Asset Management Service provides the necessary technical interfaces with the blockchain to manage whitelists on smart tokens and asset whitelists in the broker smart contract. The service also handles KYC (Know Your Customer) related changes, ensuring that users' jurisdictional data is up to date and correctly managed within the system.

## User Jurisdiction Management

Monitors KYC changes and updates UserJurisdiction records accordingly, including the addition or invalidation(NOT_ALLOWED) of jurisdictions.

## Whitelist Management

Adds wallets to the whitelist, enabling direct trading of assets without interacting with the broker smart contract, thereby improving system efficiency.

## Asset Management

Interacts with the broker smart contract to create or update assets as needed when handling whitelist requests.

## Start parameters
- `ACCOUNT_STORE` - The account store service endpoint
- `JURISDICTION_STORE` - The jurisdiction store service endpoint
- `ASSET_STORE` - The asset store service endpoint
- `PROJECT_ID` - The project id of the google cloud project
- `CREDENTIALS_LOCATION` - The location of the google cloud credentials file with access credentials for the datastore
- `HTTP_CONFIG` - The http configuration for the service
