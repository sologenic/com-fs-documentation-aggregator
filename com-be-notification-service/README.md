// TODO: update after implementation

# Notification service

The notification service provides the following RESTFUL interface:

* GET `/api/notification/unread`: Returns if the user has unread notifications
* GET `/api/notification/topauth?type=admin|marketing|maintenance`: Retrieves all unread admin/marketing/maintenance notifications for the current user
* GET `/api/notification/list?id=...&ts=...`: Retrieves all notifications for the current user up to the given id and timestamp. Returns a `more` to indicate if there is a next set.
* PUT `/api/notification/read`: Mark(s) notification(s) with a specific key(s) as read
* PUT `/api/notification/readall`: Marks all notifications as read

Calling the authenticated endpoints without a valid session will result in a 403 (TODO: verify) response.

## Architecture

The notification system is such that a message is manipulated at a minimal number of locations in the code base:

* Producers have to know the message requirements to be able to produce the messages: The producer creates a static message
* Consumers have to apply layout to the message: THe producer having to be informed already about the to be displayed values already provides all the information to be able to display the message => Consumers only have display rules, but do not retrieve any mode values from APIs. Display rules are rules which determine per usage where links relate to, images come from, etc.

The structure is then:

* Message listener/producer: Produces static message with point in time parameters, sends to the store for persistence.
* Notification service retrieves messages from the store and serves them out unmodified from the source.
* Consumer (FE, Telegram, Firebase) applies display rules.

### Parameters

The functional message structure is used to be able to store a server side fully static message.
There is a strong preference on pre-processing to promote simplicity of the individual components to have only interfaces which are real dependencies, not occasional dependencies.

Parameters are dynamic values that are parsed into the message. The parameters are defined in the proto file.

While not present in the current model, investigation has been done to see if the message can be post processed. Some post processing can be applied by the FE, and is potentially portable to the BE.
In any scenario the number of interfaces for post-processing has to be limited to prevent changing the post-processing service (notification service) for every message thus to prevent coupling the message layout into both the pre-processing service (message producers like the notification payment service) and post processing service.
The goal of this structure is to be able to only create producers and have a stable post-processing service which does not require a change per message.

#### FromTo

`FromTo` has, if present, the following rules:
`FromTo` is always accompanied by `Source` and `SourceType`
`FromTo` is only used to render URLs or other in application identifiers for navigation
`DataType`, `Label` and `Value` can be present (however are optional and for other rendering reasons)

`FromTo` has possible values:

* `FROM`
* `TO`
* `CONTENT`

These wordings are choosen like this so that different rendering (top to bottom, or in a different language from right to left) are not yet opiniated into the descriptor.

### i18n & display rules

The notifications schema is driven by protos containing the enums, i18n, mark read, display, targeting and pagination requirements.

Special is the i18n part:
If the `Body` and/or `Subject` uses `{{PARAMETER}}` placeholders then the body/subject is an i18n styled string.

The i18n key for a subject is `{{Type}}_subject` in which `Type` is the string representation of the `Type` value in the `NotificationType` enum.

If subject is absent then the `Type` is used to get the key from the i18n file. Virtually no messages have a body: Body is used mainly in person to person or unstructured (e.g. marketing) communication.

Whether the message is in i18n or not, there can always be parameters to be parsed into the message. The assumption is that certain quick messages (emergencies?) will not have translations by definition, and we do not want to have to place all messages in the translation files.

The FE is responsible to apply the i18n translations for the notifications inbox, however the BE is responsible for translating the messages in the notifications to the user's language for notifications using for example firebase, telegram, email, etc.

Since there will not always be translations available, the base language is English, and a translation file for this should always be present.

The translations files will be shared between BE and FE to prevent having to maintain multiple translation files and to reduce errors.

Since languages have different grammar, and Sologenic has to replace certain place holders like `{{amount}}` with the actual amount, translations are not straight forward.

A typical line in i18n file looks like:

```r
SOMENOTIFICATION: "You have received {{amount}} {{currency}}"
```

In which the order of the text and values can be different in different languages. For example in Dutch:

```r
SOMENOTIFICATION: "Je hebt  {{currency}} {{amount}} ontvangen"
```

The i18n lines in itself will not contain any layout related information/formatting. This is such taht the i18n files in itself can be re-used to output not just HTML but also plain text, JSON, Markdown, etc. and thus be consumable by any application.

Since the BE needs to store the values for the placeholders at the moment of creation of the message, a dynamic return format for the message is required so the the parsing application can parse the message and replace the placeholders with the correct values.

To do this a key/value approach is used.

A typical translation instruction looks like:

```r
"Parameters": [
    {
        "Label": "changedBy",
        "Value": "{{ some persons alias }}",
        "SourceType": "Collection ID| NFT ID| Account ID| Etc",
        "Source": "address of alias",
        "DataType": "string",
        "FromTo": "From",
    },
    {
        "Label": "currency",
        "Value": "XRP",
        "DataType": "string"
    },
    {
        "Label": "price",
        "Value": "0.5",
        "DataType": "number"
    },
    {
        "Label": "upOrDown",
        "Value": "https://sologenic.org/assets/images/arrow-down.svg",
        "DataType": "image"
    }
]
```

Parameter values are always strings and do not carry any semantic meaning. The message formatting used determines how the value is used and will be displayed in the end.
Parameter keys are in the notification proto as an enum so that there can not be any spelling differences between the producer of the data and the final parser.

Parameters are always populated by the BE and are not available for the maintenance/marketing messages.

The parameters contain render instructions for the message consumer which can have a different representation in different consumers, and might not be parsed by certain consumers at all.

* `DataType`: String, number, url, image. Always present. Used for formatting purposes only;
* `FromTo`: Possible values: From, To. Optional. Present on max 2 provided parameters, with max 1 `From` and max 1 `To`. Always accompanied by the `Target` and `ImageValue`. Used for rendering purposes as abstraction of any information to be used in a From/To formatting approach like the left and right hand icons in the web design;
* `SourceType`: Indicator of type of ID to be found in `Source`. Only present if `Source` is present. Used for rendering purposes as abstraction for links, images, and other rule based determinations of FE or message make-up;
* `Source`: The root address of the value (Value could be the name of the collection, nft, some address or anything else). The `Source` is used to generate a rul or image location for the given value.

***Note on `Value` and `Source`***: `Value` is a parsed value for which `Source` was used. `Value` is essentially the display value, however in many cases the `Value` and `Source` will have the same data (e.g. a ledger address). Since `Value` can have parsed data, never use `Value` for any logic, always use `Source` for logic.

All values are always represented by the enum as defined in the proto files.

## GET /api/notification/topauth

Retrieves all unread admin/marketing/maintenance notifications for the current user.
The type parameter is optional can be one or more of the values:

* admin
* marketing
* maintenance

### Example curl GET

```bash
curl -X GET "https://comsolotex.sologenic.org/api/notification/topauth?type=Admin,Marketing" \
-H "Content-Type: application/json" \
-H "Network: mainnet" \ 
-H "OrganizationID: ..." \
-H "Authorization: Bearer: ...."
```

Returns:

```json
{
    "Notifications": [
        {
            ...
            "Created_at": "2019-01-01T00:00:00.000Z",
            "Read": false,
            "To": "accountid",
            "From": "accountid",
            "Type": "admin/etc: 27 types, enum in proto",
        }
    ]
}
```

## GET /api/notification/unread

Returns if the user has unread notifications.
Uses the address in the header to identify the user. The address is present when the user has a wallet connected.

### Example curl GET

```bash
curl -X GET "https://comsolotex.sologenic.org/api/notification/unread" \
-H "Network: mainnet" \ 
-H "OrganizationID: ..." \
-H "Authorization: Bearer: ...."
```

Returns:

```json
{
    "unread": true/false
}
```

## GET /api/notification/list

Retrieves all notifications for the current user up to the given id. Returns a "More" to indicate if there is a next set.
The parameters from `More` are passed back in the call when the user clicks on the `more messages` location in the FE.

### Example curl GET

```bash
curl -X GET "https://comsolotex.sologenic.org/api/notification/list" \
-H "Network: mainnet" \ 
-H "OrganizationID: ..." \
-H "Authorization: Bearer: ...."
```

Returns:

```json
{
    "Notifications": [
        {
            ...
        }
    ],
    "More": { 
        "ID": "1",
        "TS": "nano timestamp"
    }
}
```

in which more is the id of the first notification in the next set of notifications

An example call with the more parameter:

```bash
curl -X GET "https://comsolotex.sologenic.org/api/notification/list?id=1&ts=1234567890" \
-H "Network: mainnet" \ 
-H "OrganizationID: ..." \
-H "Authorization: Bearer: ...."
```

## PUT /api/notification/read

Marks the notifications with the given keys as read.

*Important note:* An admin/marketing/maintenance notification has 1 id for all users. This is done so that a message sent by the admin/marketing/maintenance team is shown at once to all users. When the message is marked as read, the backend takes care that the message is marked read for the given user only.
For practical purposes the list of keys should not exceed 100 keys. See the readall chapter in case more drastic read actions are required.

Since marking messages read takes time, it is adviced that the FE marks the messages read by themselves to prevent having to wait for the backend to respond.
The backend response can be a pre-emptive 200OK (e.g. The data is being processed, might not be done yet. It can just be an "I heard you, will do" response).

### Example put

```bash
curl -X PUT "https://comsolotex.sologenic.org/api/notification/read" \
-H "Network: mainnet" \ 
-H "OrganizationID: ..." \
-H "Authorization: Bearer: ...." \
-d '{"Key":["abc","def"]}'
```

## PUT /api/notification/readall

Marks all notifications as read.
See *important note* in the read chapter with regards to admin/marketing/maintenance notifications.

The service responds with a 200OK if the request is valid. The actual processing can take significant time. The FE should mark the messages read by themselves and block the `read all` button to prevent the BE from getting multiple read all requests.

### Example put

```bash
curl -X PUT "https://comsolotex.sologenic.org/api/notification/readall" \
-H "Network: mainnet" \ 
-H "OrganizationID: ..." \
-H "Authorization: Bearer: ...."
```

## Application start parameters

The application uses the following environment parameters:

* `ORGANIZATION_STORE`: Included from `github.com/sologenic/com-fs-organization-model`. For setup see `github.com/sologenic/com-fs-organization-model/client/README.md`
* `HTTP_CONFIG` - the configuration for the http server (included from `github.com/sologenic/com-be-http-lib/`)
* `AUTH_FIREBASE_SERVICE` - the firebase authentication service endpoint
* `USER_STORE` - the user service endpoint (included from `github.com/sologenic/com-fs-user-model/`)
* `FEATURE_FLAG_STORE` - the feature flag service endpoint (included from `github.com/sologenic/com-fs-feature-flag-model/`)
* `ROLE_STORE` - the role service endpoint (included from `github.com/sologenic/com-fs-role-model/`)
* `NOTIFICATION_STORE` - the notification store service endpoint (included from `github.com/sologenic/com-fs-notification-model/`)
