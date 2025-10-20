# Adminnotification service

The admin notification service provides the following RESTFUL interface:

Authenticated:

* `GET /api/adminnotification/list?id=...&ts=...`: Retrieves all notifications for the current user up to the given id and timestamp. Returns a `more` to indicate if there is a next set.
* `POST /api/adminnotification/create`: Creates a new notification

Calling the authenticated endpoints without a valid session will result in a `401 Unauthorized` response.

## Architecture

The notification system is such that a message is manipulated at a minimal number of locations in the code base:

* Producers have to know the message requirements to be able to produce the messages: The producer creates a static message
* Consumers have to apply layout to the message: THe producer having to be informed already about the to be displayed values already provides all the information to be able to display the message => Consumers only have display rules, but do not retrieve any mode values from APIs. Display rules are rules which determine per usage where links relate to, images come from, etc.

The structure is then:

* Message listener/producer: Produces static message with point in time parameters, sends to the store for persistence.
* Notification service retrieves messages from the store and serves them out unmodified from the source.
* Consumer (FE, Telegram, Firebase) applies display rules.

## GET /api/adminnotification/list

Retrieves notifications based on the caller's role:

- **Sologenic Administrators**: Can retrieve all notifications (system-wide and all organization-specific).
- **Organization Administrators**: Can only retrieve notifications specific to their organization.

Returns a "More" to indicate if there is a next set.
The parameters from `More` are passed back in the call when the user clicks on the `more messages` location in the FE.
Request must include a `Network` header and an `Authorization` header.

### Example request

```bash
curl -X GET "https://comsolotex-admin.sologenic.org/api/adminnotification/list" \
-H "Content-Type: application/json" \
-H "Network: mainnet" \ 
-H "OrganizationID: ..." \
-H "Authorization: Bearer: ...."
```

### Example response

```json
{
    "Notification": [
        {
            "Type": 1,
            "Message": {
                "Format": 1,
                "Subject": "Maintenance Notification 2",
                "Body": "This is a maintenance notification. The service will be down on..."
            },
            "CreatedAt": {
                "seconds": 1740697822,
                "nanos": 775009000
            },
            "UpdatedAt": {
                "seconds": 1740697822,
                "nanos": 775009000
            },
            "NotificationID": 1740697822969213000,
            "From": "sg.be.test@gmail.com",
            "Importance": 1,
            "Target": [
                2
            ],
            "ExpiresAt": {
                "seconds": 1743205844,
                "nanos": 775010000
            },
            "ValidFrom": {
                "seconds": 1740697822,
                "nanos": 775011000
            },
            "Key": "Maintenance-1740697822775011000"
        },
        {
            "Type": 1,
            "Message": {
                "Format": 1,
                "Subject": "Maintenance Notification 1",
                "Body": "This is a maintenance notification. The service will be down on..."
            },
            "CreatedAt": {
                "seconds": 1740697819,
                "nanos": 147257000
            },
            "UpdatedAt": {
                "seconds": 1740697819,
                "nanos": 147258000
            },
            "NotificationID": 1740697819343267000,
            "From": "sg.be.test@gmail.com",
            "Importance": 1,
            "Target": [
                2
            ],
            "ExpiresAt": {
                "seconds": 1743205844,
                "nanos": 147258000
            },
            "ValidFrom": {
                "seconds": 1740697819,
                "nanos": 147259000
            },
            "Key": "Maintenance-1740697819147259000"
        }
    ]
}
```

in which more is the id of the first notification in the next set of notifications

An example call with the more parameter:

```bash
curl -X GET "https://comsolotex-admin.sologenic.org/api/adminnotification/list?id=1&ts=1234567890" \
-H "Content-Type: application/json" \
-H "Network: mainnet" \ 
-H "OrganizationID: ..." \
-H "Authorization: Bearer: ...."
```

## POST /api/adminnotification/create

Creates a new notification. The notification can be recipient specific (either account or organization specific) or for all users.

- **Sologenic Administrators**: 
  - **System-wide**: Create system-wide notifications (visible to all organizations)
  - **Sologenic org-scoped**: Create organization-specific notifications for the 'sologenic' organization by setting `"IsOrgScoped": true`
- **Organization Administrators**: 
  - **org-scoped** create organization-specific notifications (scoped to their organization only)

Request must include a `Network` header and an `Authorization` header.

### Example request

```bash
curl -X POST "https://comsolotex-admin.sologenic.org/api/adminnotification/create" \
-H "Content-Type: application/json" \
-H "Network: mainnet" \ 
-H "Authorization: Bearer: ...." \
-d '{
    "Notification_type": "Maintenance",
    "Message": {
        "Subject": "Scheduled Maintenance",
        "Body": "A system maintenance is scheduled for ... UTC. The system will be down for 1 hour."
    },
    "ExpiresAt": "2021-12-31T23:59:59Z"
}'
```

### Example: Sologenic Administrator creating organization-specific notification for `sologenic` organization

```bash
curl -X POST "https://comsolotex-admin.sologenic.org/api/adminnotification/create" \
-H "Content-Type: application/json" \
-H "Network: mainnet" \ 
-H "Authorization: Bearer: ...." \
-d '{
    "Notification_type": "Admin",
    "Message": {
        "Subject": "Internal Update",
        "Body": "This message is only for Sologenic organization users."
    },
    "IsOrgScoped": true,
    "ExpiresAt": "2021-12-31T23:59:59Z"
}'
```

Recipient specific notifications can be created by adding the `RecipientID` field to the message:

### Example request

```bash
curl -X POST "https://comsolotex-admin.sologenic.org/api/adminnotification/create" \
-H "Content-Type: application/json" \
-H "Network: mainnet" \
-H "OrganizationID: ..." \
-H "Authorization: Bearer: ...." \
-d '{
    "Notification_type": "Marketing",
    "Recipient_id": "c12e34f5-6789-0abc-def1-234567890abc",
    "Message": {
        "Subject": "Exclusive Offer",
        "Body": "Dear users, we have a special offer for you. Please check your account for more details."
    },
    "ExpiresAt": "2021-12-31T23:59:59Z"
}'
```

## Application start parameters

The application uses the following environment parameters:

* `HTTP_CONFIG` - the configuration for the http server (included from `github.com/sologenic/com-be-http-lib/`)
* `AUTH_FIREBASE_SERVICE` - the authentication service endpoint (included from `github.com/sologenic/com-fs-auth-firebase-model/`)
* `NOTIFICATION_STORE` - the notification service endpoint (included from `github.com/sologenic/com-fs-notification-model/`)
* `ACCOUNT_STORE` - the account service endpoint (included from `github.com/sologenic/com-fs-admin-account-model/`)
* `ROLE_STORE` - the role service endpoint (included from `github.com/sologenic/com-fs-admin-role-model/`)
* `FEATURE_FLAG_STORE` - the feature flag service endpoint (included from `github.com/sologenic/com-fs-admin-feature-flag-model/`)
* `ORGANIZATION_STORE` - the organization service endpoint (included from `github.com/sologenic/com-fs-admin-organization-model/`)
- `SOLOGENIC_ORGANIZATION_ID` - The organization id of the Sologenic Unified User, used to identify the Sologenic platform