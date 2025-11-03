# update service

The update service is a very special service: The service does not serve any data, or accepts any requests for data.
The service publishes to interested listeners that something changed.

The way it does this is by listening on a pubsub topic and publish the messages to the interested listeners.

## Type of messages

There are 2 generic types of messages:

* System style messages: Messages without an identifying key: For example: There is a new AMM Pool (or one was removed), which could be used to load a whole new set to discover and load the new data.
* Entity style messages: Messages with an identifying key: For example: The price of a token changed, or the balance of a wallet changed. This message contains the ID of the changed entity and can be filtered on by the ID, type and network.

## Pattern of messages

The update service operates using two distinct patterns for delivering updates to clients:

* Event-Driven Updates (Trigger-based): Event-Driven Updates deliver a message, containing only ID that clients must use to fetch complete data through additional API calls, primarily used for notifications, KYC updates, and comment replies.

* Time-Period Driven Updates: Time-Period Driven Updates deliver complete data objects at regular intervals through a polling mechanism, requiring no additional API calls from clients, primarily used for AED chart data that refreshes at configured intervals.

## Interaction with the websocket

Once connected to the websocket, the following messages can be send:

* `subscribe` - Subscribe to a topic. The topic can be a generic topic, or a topic with an entity ID. The topic can be a single topic, or a list of topics.
* `unsubscribe` - Unsubscribe from a topic. The topic can be a generic topic, or a topic with an entity ID. The topic can be a single topic, or a list of topics.
* `close` - Close the connection to the websocket. The server will close the connection in that scenario. This close method is preferred so that any subscriptions can be cleaned up instead of garbage collected.

## App internals

The application listens on a Pubsub topic on which the triggers to act on appear (the trigger topic `com-trigger`). This topic is used to manage multiple derived secondairy processes like notifications, or async data processing updates. The overall message processing has to be known by the listeners to know which messages to act on. A single trigger can however lead to different updates to be fires. For example a payment transaction would lead to 2 accountID updates. This model will evolve over time when all the components requiring updates to be fired (and remove all the long poll actions) are migrated to this model.

The app is running as a singleton application at the moment due to the way the pubsub consumer is implemented due to probably segment-io pubsub, and certainly due to the way the consumer is handled in the config:

* Config: The config can only have one consumer identifier (group id). This would mean that in a multi pod scenario a message read by one pod, would never be "seen" by the other pod(s) thus leading to gaps in the communication: We need more consumers and with that a different way of passing the identifier to the config. A fully dynamic identifier is however conflicting with existing applications and configurations which for parallel processing rely on this behaviour of reading the message only once.
* Reading of the whole topic or just start listening at the end of the topic: When a new consumer identifier (group id) is identified by pubsub on a given topic, the topic replays from the start. This is controllable, however the current implementation with Segment IO needs to be re-evaluated to see if it can be changed to read from the end of the topic (Rereading from the start is slow and could lead to >>minute long startup times without any messages being processed).

=> So for now there will be a singleton (single instance) of the app. This should be good enough for the start period in which we are processing only very specific messages. Later on with the full long poll method being replaced with specific messages over the websocket, this will have to be addressed.

The app has to retrieve additional information from certain stores to be able to fulfill all the functionalities. This might mainly be the case with:

* Trades (both normal and AMM): The trigger message contains the trade ID, while the account IDs of the affected users are required
* Notifications: Again we might want the account the message is for
* etc

Main risk is stability of the service when it is connected to many other services. If this turns out to be a problem, isolate the code in a listener (e.g. non-user facing service) and introduce a new pubsub topic to which that listener publishes messages: That way this update listener will only have to listen on one topic with complete data and thus not require all these connections. Another alternative would be to introduce a new message in general which just contains those keys however taht would make every sender responsible for being aware of all the changes which might occur in other places and make the messages tightly coupled (so certainly not the preferred method).

### Initial connection

On the initial connection to the websocket, the client gets a response `Connected`

The websocket is accessible on the following URL:

```js
wss://api.sologenic.org/api/v1/update/ws
```

### Keep alive

The nginx has a timeout, which is about 10s (TODO: Verify). To prevent disconnects, the client has to send a keep alive message about every 5s. The keep alive message is a string:

```js
ping
```

The server responds with a "pong `{instanceID}`" message.
The instanceID is stable for the duration of the server side running. If there is a crash or if there are multiple instances, the instanceID will change. The client can, when the instanceID changes, to resend the subscriptions.

### Subscribe to a message (or topic)

The topics are enumerated in the `com-fs-update-model`. The subscription messages are also defined in the model. The choice for a separate model was made based on the previously mentioned stability risk (if this risk is real, than we do not have to refactor all, but can just implement a result pubsub topic containing the required messages as defined in this `com-fs-update-model`).

A subscription consists of sending a message on the websocket with the following structure:

For subscription to an sub system:

```json
{
    "Action": 0, // SUBSCRIBE
    "Subscription": {
        "Network": 1,
        "OrganizationID": "024e6e3e-59e6-4037-be4d-bfbb3894dbeb",
        "Method": 3
    }
}
```

This would represent subscribing to hear if there is a new reply on the user's support ticket.

Subscription to an entity:

```json
{
    "Action": 0, // SUBSCRIBE
    "Subscription": {
        "Network": 1,
        "OrganizationID": "024e6e3e-59e6-4037-be4d-bfbb3894dbeb",
        "Method": 2,
        "ID": "user_email@gmail.com"
    }
}
```

This would represent a subscription to a specific AMM pool and getting a signal if there is a change on that pool.

The ID in the subscription is suggested by the enum name used: For example AMM_TRADE_FOR_ACCOUNT indicates with the name that you would apply an account ID to the subscription.
(P.S. You might notice that (at this moment) there is no AMM_TRADE enum selection: AMM_POOL already indicated that there is an update, so AMM_TRADE would be redundant. All functions will be analyzed to limit redundant messages).

***Important:*** The actual messages use enums (integers). The package for this is provided in `fs-update-model`.

### Unsubscribe from a message (or topic)

The unsubscribe works the same as the subscribe, however replace the action with `unsubscribe`.

### Close the connection

The close is the following json:

```json
{
    "Action": 2, // CLOSE
}
```

which will close the websocket at the server side.

## Messages send to the client

The messages send to the client are in structure the same as the messages send to the server.
The differences are:

* `Action`: `response` (Enum value) 
* `ID`: Always present since now the BE has to tell which ID should be used for a refresh (or if there is more FE logic: Which ID the potential refresh is related to)

A sample response:

```json
{
    "Action": 3, // RESPONSE
    "Subscription": {
        "Network": 1,
        "Method": 2,
        "OrganizationID": "024e6e3e-59e6-4037-be4d-bfbb3894dbeb",
        "ID": "FOO_XRP_r123456"
    }
}
```

## Application start parameters

- `ORGANIZATION_STORE`: the organization service endpoint (included from `github.com/sologenic/com-fs-admin-organization-model`).
- `COMMENT_STORE`: the comment service endpoint (included from `github.com/sologenic/com-fs-comment-model`).
- `AED_STORE`: the aed endpoint (included from `github.com/sologenic/com-fs-aed-model`).
- `ASSET_STORE` - the asset service endpoint (included from `github.com/sologenic/com-fs-asset-model/`)
- `USER_STORE` - the user service endpoint (included from `github.com/sologenic/com-fs-user-model/`)
- `HTTP_CONFIG` - The configuration for the http server

For listener:
- `PROJECT_ID` - The project id of the google cloud project
- `HTTP_PORT` - The port the http server health probe listens on
- `CREDENTIALS_LOCATION` - The location of the google cloud credentials file with access credentials for the datastore
- `APP_NAME` - The name of the application for identification purposes
- `GRPC_APPEND` - The suffix to append to gRPC service names
- `SUB_CONFIG` - PubSub subscription configuration JSON containing subscription id, topic, endpoint, etc.
