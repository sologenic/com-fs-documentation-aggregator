# KYC service

The `be-kyc-service` is designed to handle Know Your Customer (KYC) operations for Sologenic.com. This service manages KYC admin interface interactions and integrations with Sumsub for KYC verifications. 

The KYC service provides the following RESTFUL interface:

* GET `/kyc/get?external_user_id=...` - retrieves a KYC record by external user id
* GET `/kyc/count?status=...` - retrieves the count of KYC records categorized by their current status
* GET `/kyc/list?status=...` - retrieves a list of accounts
* POST `kyc/status` - updates the status of a KYC record. 

* POST `kyc/create` - (not implemented) Initiate KYC verification step on behalf of a user (User should initiate and process the KYC verification by themselves, so implementation will be considered upon request)

## GET /kyc/get?external_user_id=...

Retrieves a KYC record by external user id.

### Example call

```bash
curl -X GET \
  https://com-be-kyc-service-zk6dps4acq-uc.a.run.app/kyc/get?external_user_id=a9ab4580-7f8f-7889-2df6-d774ce8921a7 \
  -H "Content-Type: application/json" \
  -H "Authorization:eyJQcml2YXRlS2...someEncodedToken...OGY0ZSJ9" \
  -H "Network: testnet"
```

### Example response

Returns:

```json
{
  "AccountID": "0660ba9a-8423-9c53-eb0d-ae6ebb848544",
  "ExternalUserID": "a9ab4580-7f8f-7889-2df6-d774ce8921a7",
  "ParticipantID": "6675aa52-e05f-5b4f-a3ea-625858433d78",
  "KYCProvider": 1,
  "Status": 6,
  "AdminComment": "action required",
  "FirstSeen": {
    "seconds": 326,
    "nanos": 223
  },
  "UpdatedAt": {
    "seconds": 1720466106,
    "nanos": 46857000
  },
  "ApprovedAt": [
    {
      "seconds": 739,
      "nanos": 88
    },
    {
      "seconds": 825,
      "nanos": 296
    }
  ],
  "Documents": [
    "artificial intelligence",
    "Jamaican Dollar"
  ],
  "Details": {
    "SourceKey": "District Branding Handmade Wooden Car",
    "Email": "Hoyt.Borer@example.com",
    "Phone": "(890) 754-3139",
    "ApplicantInfo": {
      "FirstName": "Alanis",
      "LastName": "Hyatt",
      "DOB": "salmon Spring mobile",
      "Country": "Tokelau",
      "Addresses": [
        {
          "Country": "24/365",
          "Town": "azure Extended",
          "State": "Hawaii",
          "PostCode": "Liaison Kansas homogeneous",
          "Street": "Vandervort Crest",
          "SubStreet": "Accounts Cotton",
          "FlatNumber": "12267",
          "BuildingNumber": "459"
        },
        {
          "Contry": "Customer-focused",
          "Town": "payment",
          "State": "Kansas",
          "PostCode": "parse Handmade Frozen",
          "Street": "Ali Centers",
          "SubStreet": "Innovative",
          "FlatNumber": "49677",
          "BuildingNumber": "42452"
        }
      ]
    },
    "Review": {
      "ReviewID": "5f23aba3-045f-28b1-3742-0827ab07c379",
      "LevelName": "Fantastic Granite Tuna invoice",
      "ReviewResult": {
        "ReviewAnswer": 1,
        "RejectLabels": [
          "index Dynamic Berkshire",
          "red bypassing Unbranded Wooden Tuna"
        ],
        "ReviewResultType": 1,
        "ClientComment": "Turkey JBOD open-source",
        "ModerationComment": "Representative Investor Lake",
        "ButtonIDs": [
          "Usability driver",
          "sensor Industrial"
        ],
        "ReviewStatus": 3
      }
    }
  },
  "Network": "mainnet",
  "OrganizationID": "34422ce6-7b51-4a15-accb-34959f39d8ca"
}
```

## GET /kyc/count?status=...

Retrieves the count of KYC records categorized by their current status.

### Example call

```bash
curl -X GET \
  https://com-be-kyc-service-zk6dps4acq-uc.a.run.app/kyc/count?status=APPROVED \
  -H "Content-Type: application/json" \
  -H "Authorization:eyJQcml2YXRlS2...someEncodedToken...OGY0ZSJ9" \
  -H "Network: testnet"
```

### Example response

Returns:

```json
{
  "Count": 1,
  "KYCStatus": 3
}
```

## GET /kyc/list?status=...

Retrieves a list of accounts 

### Example call

```bash
curl -X GET \
  https://com-be-kyc-service-zk6dps4acq-uc.a.run.app/kyc/list?status=APPROVED \
  -H "Content-Type: application/json" \
  -H "Authorization:eyJQcml2YXRlS2...someEncodedToken...OGY0ZSJ9" \
  -H "Network: testnet"
```

### Example response

Returns:

```json
{
  "Accounts": [
    {
      "ID": "2461396b-8244-d8e2-4a25-bdb8d5b66710",
      "FirstName": "Karlie",
      "LastName": "Hills",
      "Address": "ABC2",
      "Wallets": [
        {
          "Address": "Awesome Tasty invoice1",
          "Alias": "Fresh back up1",
          "Type": 2
        },
        {
          "Address": "maroon Handmade Wooden Keyboard Borders1",
          "Alias": "bus program Versatile1",
          "Type": 3
        }
      ],
      "CreatedAt": {
        "seconds": 1720464107,
        "nanos": 757227000
      },
      "UpdatedAt": {
        "seconds": 694,
        "nanos": 721
      },
      "Socials": [
        {
          "URL": "https://drew.net"
        },
        {
          "URL": "https://ephraim.net"
        }
      ],
      "Avatar": "https://cloudflare-ipfs.com/ipfs/Qmd3W5DuhgHirLHGVixi6V76LhCkZUz6pnFt5AJBiyvHye/avatar/1005.jpg",
      "Alias": "ABC2",
      "Description": "invoice back up",
      "Network": "mainnet",
      "Status": 1,
      "Roles": [
        1
      ],
      "ExternalUserID": "6b5118f7-1912-5d6e-6b7f-c6036a517af1",
      "OrganizationID": "34422ce6-7b51-4a15-accb-34959f39d8ca"
    }
  ]
}
```

## POST /kyc/status

Updates the status of a KYC record, and optionally adds an admin comment. This endpoint is intended for use by an KYC admin to update the status of a KYC record. An admin can only change the KYC status to one of the allowed statuses.

The status can be updated to one of the following values:
* APPROVED (2)
  - Authorizes an account by an admin, bypassing the KYC process and marking the status as approved without completing the KYC process

  - > **WARNING:** This should be used with caution as this overrides standard KYC processes internally and in exceptional cases only.

* ADMIN_DENIED (3)
  - Allows an admin to deny an account regardless of the KYC application status. This action is based on the admin's discretion and is performed internally. It does not reflect on Sumsub's dashboard (we do not make any calls to block or deactivate the application to Sumsub) to maintain a "single source of truth" for account management.
  Reference: https://docs.sumsub.com/reference/deactivate-applicant-profile

* RE_REQUESTED (5)
  - Triggers a re-request of KYC record in the cases where KYC process is stuck or an applicant makes a support request to re-pass verification from scratch with new documents. This will reset the profile and roll the KYC status back to 'init' and deactivate any previous documents.

  - > **WARNING:** Attempting to re-request KYC for an applicant identified with fraudulent patterns will result in a error(fraudlent pattern check is injected in the handler). Ensure the applicant's record is free of such labels: FORGERY, SELFIE_MISMATCH, BLACKLIST, BLOCKLIST, INCONSISTENT_PROFILE, FRAUDULENT_PATTERNS.   
  Reference: https://docs.sumsub.com/reference/reset-applicant

* NOT_PROCESSABLE_FOREVER (7)
  - Marks a specific KYC record as unprocessable and records the reason. This status is used when a record cannot be processed without further action by the user or admin.

* FIX_REQUIRED (8)
  - Sends a ticket to notify the dev team to address and fix a "stuck" record associated with the given external_user_id, prompting necessary actions to resolve the issue in the KYC provider's dashboard.

### Example call

```bash
curl -X POST \
  https://com-be-kyc-service-zk6dps4acq-uc.a.run.app/kyc/status \
  -H "Content-Type: application/json" \
  -H "Authorization:eyJQcml2YXRlS2...someEncodedToken...OGY0ZSJ9" \
  -H "Network: testnet"
  -d '{
    "externalUserID": "e1aad668-3af1-a4b4-557e-05ae7ca21d72",
    "kycStatus": "RE_REQUESTED",
    "adminComment": "user's first name has changed, re-request KYC process"
}'
```

### Example response

Returns:

```json
{
  "UpdatedAt": {
    "seconds": 1720469737,
    "nanos": 116377000
  }
}
```

## Application start parameters

The application uses the following environment parameters:

- `ORGANIZATION_STORE`: Included from `github.com/sologenic/com-fs-organization-model`. For setup see `github.com/sologenic/com-fs-organization-model/client/README.md`
- `AUTH_SERVICE` - Auth GRPC service endpoint
- `ACCOUNT_STORE` - Account GRPC service endpoint
- `OPEN_SEARCH_STORE` - Open search GRPC service endpoint
- `HTTP_CONFIG` - The configuration for the http server


TODO: Remove the initial draft below once the service is fully implemented
```
Admin Account Management: Admins should be able to create and authorize other admin or user accounts.
Integration with Frontend: The frontend should provide interfaces for these admin functionalities, meaning that the FE should have forms and views where admins can perform these actions.

// TODO: delete the following section once the service is fully implemented (note keeping for FE reference)
GET `/kyc/count?status=...`
* admin can get a count of "stuck" records
  - will be displayed on the overview page

GET `/kyc/list?status=...`
* admin can get an overview of all the users
  - some records will be marked as "Record not processable"
  - admin can filter this overview on status of the KYC records to show "stuck" records
  - list should be ordered by updated datetime desc
  - admin gets a displayed count of "stuck" records

POST `kyc/mark/{id}`
* admin is able to mark a record as "Not processable forever" 
  - this should now give the admin an overview of the actual "stuck" record

POST `/kyc/notif/{id}`
* admin can send a ticket to the developers to notify them of a "stuck" record
  - click a button to send a ticket

POST `kyc/reprocess/{ids}`
* admin can trigger the reprocessing of "stuck" records
  - this can be for a single record, or bulk reprocessing
  - done by clicking either one of the two buttons: "reprocess record" or "reprocess all stuck records"
  - the `id` parameter will be a base64 encoded array of record ids

POST `kyc/create`
* admin can setup a new account on behalf of a user
  - although the functionality is the same as provided in the account service, this action is done by the KYC admin, and therefore should be available in the KYC service

POST `kyc/authorize/{id}` - authorizes an account
* admin can authorize a user account
  
// TODO: consider: does this need to be a manual process, where the admin manually notifies the user (by clicking a button to notify for example)
* POST `kyc/notify/{id}` - sends a notification to the user when their KYC process is completed

GET `/user/{query}` => will most likely be implemented in the account service
* view user details
  - can filter based on name/ status
  - will use elastic search
```