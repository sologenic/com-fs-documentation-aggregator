# Admin comment service

The admin comment service provides the generic functionality to attach comments to any source type [github.com/sologenic/com-fs-notification-model](https://github.com/sologenic/com-fs-notification-model). This provides consistency between what can be commented and source types are used for rendering content. Essentially the source types are the different functions of the system.

The admin comment service is to facilitate the management of support system (chat and ticket) in the admin dashboard as well as admin delete actions of comments. For general usage, `com-be-comment-service` is to be used.

## API

**Authenticated:**

- GET `api/admincomment/get?status=...&reporter=...&account=...&source_type...&source=...&order=...` : List events by various filter criteria, oldest are first so that the oldest not processed comments get highest priority.
- GET `api/admincomment/count?source_type=...`: Count the number of comments for a source type and source (e.g. total count of comments). The assumption in this count is that it will be used on the single object page.
- POST `api/admincomment/add`: Add/create a comment or a comment on a comment
- PUT `api/admincomment/status`: sets status (read or unread, reviewed, admin deleted, etc.)
- DELETE `api/admincomment/delete`: Delete a comment or a file uploaded by a user

> **Note:**

- All authenticated requests must include `Network`, `OrganizationID`, and `Authorization` in the header.

### Pagination

All the list style endpoints (multiple results) have pagination options which are returned as a standard object in the response:

```json5
{
  "Offset": {The next offset to pass back},
}
```

The absence of the offset object means that there is no more data to be retrieved.

The offset parameter is for the GET endpoints with multiple results is:

`offset={offset}`

### Querying against an empty database

GET/COUNT does not work against an empty database by lack of any model for the database to reference to.
We have to add 1 comment into the system using wget/curl to avoid this empty database error (easier than catching it all over the place for a 10s situation only at launch of the comment system).

### List comments `GET /get`

The listing of comments is a main functionality to be able to view comments in different ways.
The endpoint is not authenticated, in contrary to the admin endpoint for listing comments. The admin endpoint differs in filter options and flexibility.

The endpoint accepts the following query parameters (`order` is optional for all queries):

- `source_type` and `order` (in which `order` is according to the `Order` enum)
- `source` and `order`
- `source_type`, `source` and `order`
- `account` and `order`
- `comment_ids` and `order`
- `status` and `order`
- `reporter` and `order`
- `source_type` and `source`

Other combinations are not allowed then the 8 variants above, are not allowed and can lead to unexpected results.

Order is optional.

The checks by the BE are not strict: Not all combinations are checked before execution.

If

- `source` is found, the source query runs
- `account` is found, the account query runs
- `comment_ids` is found, the comment_ids query runs

So if `source` and `account` are found, the `source` query runs (etc).

#### Examples

```bash
curl -H "Network: mainnet" \
-H "Content-Type: application/json" \
-X GET https://comsolotex-admin.sologenic.org/api/admincomment/get?account_id=12345&order=2
```

```bash
curl -H "Network: mainnet" \
-H "Content-Type: application/json" \
-X GET https://comsolotex-admin.sologenic.org/api/admincomment/get?comment_ids=WzEyMzQ1LDEyMzQ1NiwxMjM0NTY3XQo=
```

In which the comment_ids are base64 encoded (source value here was `[12345,123456,1234567]`).

#### Success response

The success response returns a list of comments.

```json
{
  "Offset": 1, // The offset to use for the next request
  "CommentResult": [
    {
      "Comments": [
        {
          "CommentID": "uuid string",
          "Account": "rhozSjrMoVBV8PuQAdyLro2FEF2uVBPfzw",
          "Network": 1,
          "Source": "1",
          "SourceType": 2,
          "CreatedAt": {
            "seconds": 1686172948,
            "nanos": 555295000
          },
          "UpdatedAt": {
            "seconds": 1686172948,
            "nanos": 555298000
          },
          "Content": "This is a comment",
          "Status": 6,
          "DeletionAccount": "rhozSjrMoVBV8PuQAdyLro2FEF2uVBPfzw"
        },
        {
          "CommentID": "uuid string",
          "Account": "rhozSjrMoVBV8PuQAdyLro2FEF2uVBPfzw",
          "Network": 1,
          "Source": "1",
          "SourceType": 2,
          "CreatedAt": {
            "seconds": 1686173092,
            "nanos": 411150000
          },
          "UpdatedAt": {
            "seconds": 1686173092,
            "nanos": 411151000
          },
          "Content": "This is a comment"
        },
        {
          "CommentID": "uuid string",
          "Account": "rhozSjrMoVBV8PuQAdyLro2FEF2uVBPfzw",
          "Network": 1,
          "Source": "1",
          "SourceType": 2,
          "CreatedAt": {
            "seconds": 1686173146,
            "nanos": 758432000
          },
          "UpdatedAt": {
            "seconds": 1686173146,
            "nanos": 758434000
          },
          "Content": "This is a comment",
          "Status": 5,
          "DeletionAccount": "rhozSjrMoVBV8PuQAdyLro2FEF2uVBPfzw"
        },
        {
          "CommentID": "uuid string",
          "Account": "rhozSjrMoVBV8PuQAdyLro2FEF2uVBPfzw",
          "Network": 1,
          "Source": "1",
          "SourceType": 2,
          "CreatedAt": {
            "seconds": 1686173680,
            "nanos": 532975000
          },
          "UpdatedAt": {
            "seconds": 1686173680,
            "nanos": 532977000
          },
          "Content": "This is a comment"
        }
      ]
    }
  ]
}
```

As might be visible is that comments which are marked for deletion (or reported) are still visible in the results.
The deletion and reporting is processed asynchronously. Once the deletion or reporting process is completed, the comments are or can be removed from the result set.

#### Error response

Code: 500 ACCOUNT PARAM INVALID
Code: 500 NETWORK PARAM INVALID
Code: 500 COMMENT_IDS PARAM INVALID
Code: 500 SOURCE_TYPE PARAM INVALID
Code: 500 ORDER PARAM INVALID
Code: 500 INVALID COMBINATION OF PARAMS

### Count comments `GET /count`

Count comments for a certain `source_type` and `source`
Count comments for a given `account`
Count comments associated with a given list of `comment_ids`

#### Examples

```bash
curl -H "Network: mainnet" \
-H "Content-Type: application/json" \
-X GET https://comsolotex-admin.sologenic.org/api/admincomment/count?source_type=NFT_ID&source=12345
```

#### Success response

The success response is an integer:

```json
{ "Counts": [{}] }
```

#### Error response

Code: 500 NETWORK PARAM INVALID
Code: 500 SOURCE_TYPE PARAM INVALID

#### Change comment status `POST /status`

The status of a comment is defined in the comment model:

```proto
enum Status {
    NOT_USED_STATUS = 0;
    NORMAL = 1;
    REVIEWED = 2;
    REVIEW_REQUESTED = 3;
    ADMIN_DELETED = 4;
    ADMIN_DELETED_CASCADE = 5;
    DELETED = 6; // Deleted by user or moderator
    DELETED_CASCADE = 7; // Deleted by moderator and all children are deleted as well
    READ = 8; // Read by admin (for support tickets)
}
```

Comments which get marked for review can be marked by the admin as `REVIEWED`, which is positive. Alternatively the admin can decide to delete the comment, which is negative.

The system uses the trigger kafka topic to delegate the report processing consequences to a sub process.

A negative review of a comment leads to the deletion of the comment.

On `ADMIN_DELETED` the admin will have the option to keep replies, the assumption is that the `KeepReplies` flag is set to false (too much work to review replies, plus when the comment is reaches the level of admin deletion agreement the comment is of such poor/unwanted content, that replies are most likely just people complaining about the comment).
The admin can change the reason before submitting the deletion.

The deletion of an event sets the ADMIN_DELETED status, plus reason on the comment.
Consequently the following processes are executed by a consumer on the trigger channel:

- Remove any uploads associated with the message
- Record deleted images in separate image log as MD5 of the image, reason and poster address
- Record deleted message in separate message log as MD5 of the message, reason and poster address

The message to the status endpoint is:

```go
type UpdateRequest struct {
  CommentID string
  Status    commentgrpc.Status // Enum from model
  Reason    *commentgrpc.Reason // Enum from model
}
```

In which the reason is required for status `ADMIN_DELETED`.
The reason on `REVIEWED` is reset by the BE to `NONE`.

##### Example

```bash
curl -H "Network: mainnet" \
-H "Authorization: Bearer: ..." \
-H "Content-Type: application/json" \
-H "OrganizationID: 72c4c072-2fe4-4f72-ae9d-d9d52a05fd71" \
-d '{"CommentID": "uuid string", "Status": 4, "Reason": 1}' \
-X PUT https://comsolotex-admin.sologenic.org/api/admincomment/status
```

#### Success response

`200OK`

#### Error responses

Code: 401 UNAUTHORIZED
Code: 500 ACCOUNT PARAM INVALID
Code: 500 NETWORK PARAM INVALID
Code: 500 COMMENT_ID PARAM INVALID
Code: 500 STATUS PARAM INVALID
Code: 500 REASON PARAM INVALID

###### Testnet and mainnet: A major issue

Pay special attention to the network here:
In production both testnet and mainnet are available, and the admin is required to review both.
At this moment no method for limiting the use of testnet is implemented or planned.

Since testnet does not have limitations with regards to availability of funds, funding the accounts, etc, testnet can be abused to spam the system and destroy the experience for users on Sologenic. Additional consideration need to be made to prevent this from happening.

##### Spamming of reviews

When a message is marked as `REVIEWED`, the reporting account gets an adjusted score to prevent spamming the admin with reviews of items which are within normal community standards.

This account property can be positive, 0 or negative:

- Positive: The user reports comments which are deleted by the admin
- Negative: The user reports more comments which are not deleted by the admin (so incorrect reports)
- Zero: There are no reports by the user in the last period or never

There is a background process resetting the user score from negative to 0 after a certain period of time.

Having a certain negative score can lead to banning the user from creating more reports or to banning the user completely from interactions with the social aspects of the system.

##### Future improvements

The collected MD5 hashes in the logs of removed images and messages can be used create a algorithm/model to automatically remove messages and images which are similar to the ones which are removed by the admin.

### Add comment `/add`

A typical comment consists out a body with some kind of embedded markup. The body is agnostic of language and can certain content associated with it like video or image. The FE component determines how this works, the BE service provides the endpoints to manage this.

The POST body to add a comment contains:

- SourceType: From the enum `SourceType` in the notification model [github.com/sologenic/com-fs-notification-model](https://github.com/sologenic/com-fs-notification-model).
- Source: The ID of the source type.
- Content: Content will contain the links to the uploaded images/videos excluding the domain (thus preventing linking to external resources).
- Tags: Tags are used to categorize the comment. e.g. `["Payment issue"]` for a support ticket.

The post body looks like:

```json5
{
  SourceType: 4,
  Source: "12345",
  Content: "This is a comment",
  Tags: ["tag1", "tag2"],
}
```

#### Success response

`200OK`

#### Error response

Code: 500 ACCOUNT PARAM INVALID
Code: 500 NETWORK PARAM INVALID
Code: 500 SOURCE TYPE INVALID
Code: 500 COMMENT INVALID (e.g. empty comment,and/or empty source)

```json5
{
  "errors": [
    {
      "name": "account.invalid" || "network.invalid" || "sourceType.invalid",
    }
  ]
}
```

#### Example

```bash
curl -H "Network: mainnet" \
-H "Authorization: Bearer: ..." \
-H "Content-Type: application/json" \
-H "OrganizationID: 72c4c072-2fe4-4f72-ae9d-d9d52a05fd71" \
-d '{"SourceType": 4, "Source": "12345", "Content": "This is a comment", Tags: ["tag1", "tag2"]}' \
-X POST https://comsolotex-admin.sologenic.org/api/admincomment/add
```

### Add comment to comment

The adding of a comment to a comment differs from the add comment in the fact that more information needs to be provided:

- SourceType: `1` (`COMMENT_ID`)
- Source: `ID of the comment the comment is added to`

The backend adds the `RootType` and `Root` against the `Source` to ensure that the comment is added to the correct object (and no cross commenting hacks are possible).

- RootType: The type of the root object (COMMENT_ID etc)
- Root: The ID of the root object (COMMENT_ID etc)

The `SourceType` and `Source` link the comment to the comment the reply is placed on.

The post body looks like:

```json5
{
  SourceType: 1,
  Source: "1234567890",
  Content: "This is a comment",
}
```

#### Success response

`200OK`

#### Error responses

Code: 401 UNAUTHORIZED
Code: 500 ACCOUNT PARAM INVALID
Code: 500 NETWORK PARAM INVALID
Code: 500 SOURCE TYPE INVALID

```json5
{
  "errors": [
    {
      "name": "account.invalid" || "network.invalid" || "sourceType.invalid",
    }
  ]
}
```

#### Example

A comment is added to a comment with ID `1234567890` which is a comment on an NFT with ID `12345`:

```bash
curl -H "Network: mainnet" \
-H "Authorization: Bearer: ..." \
-H "Content-Type: application/json" \
-H "OrganizationID: 72c4c072-2fe4-4f72-ae9d-d9d52a05fd71" \
-d '{"SourceType": 1,
"Source": "1234567890",
"Content": "This is a comment",
-X POST https://comsolotex-admin.sologenic.org/api/admincomment/add
```

### Add image/video to comment

The upload of an image or video is done in a separate call (refer to `com-be-file-service` for more info).
The FE pregenerates the URL in the to be created comment according to generating a signed URL for the upload. The signature is a UUID.
The BE in the upload process is provided with the same signature and will store the provided file under that signature.

### Delete a comment `/delete`

Only the owner of a comment can delete it. The system verifies ownership by checking that the accountID matches the account ID associated with the comment.

The service only sets the state of the comment to DELETED or DELETED_CASCADE.
The trigger channel will process the comment and remove it from the database if the rules are met:

- Cascading delete is executed only of ownership rules are met (account requesting delete is the owner of the root object)
- Cascading delete is ignored if the account requesting the delete is not the owner of the root object.
- Delete is ignored when the account requesting the delete is not the owner of the comment, or owner of the root object.

(FE should only allow accounts with certain ownership of either the comment of the root object (NFT etc) to submit deletions).

Since there can be several updates required to be executed based on the deletion, the comment is only marked with `Status=DELETED/DELETED_CASCADE`.
Secondary processes updating for example like counts for all users which liked the comment, adjusting account based reputation (both positive and negative), and others are executed in through the trigger topic based event.

The deletion of an event with sub-messages is handled in 2 different ways:

If a comment has replies, the user deleting the comment has the option to either:

1. Delete all replies attached to the comment
2. Keep the replies, in which case they will afterwards be shown as comments attached to the comment the deleted comment was a reply on, or if the deleted comment was a reply on the source (e.g. NFT etc) as replies on the source. In this scenario when the admin deletes the initial requested reviewed comment, the replies are marked as DELETED_CASCADE by the process and are processed as normal DELETION events. (e.g. the rules in this chapter apply, not the ADMIN_DELETED rules).

The comments displayed are filtered to show only comments with a NON-DELETED status.

The request body looks like:

```json5
{
  "CommentID": "uuid string",
  "KeepReplies": true || false
}
```

If KeepReplies is true, the replies are rebased.

If files are attached to the comment, the files are removed immediately from the bucket.

#### Success response

The success response is given in virtually all scenarios: If the commentID is invalid (for example), the system does not really care (2 deletions and one on a non-refreshed list: not a problem).

`200OK`

#### Error responses

Code: 401 UNAUTHORIZED
Code: 500 ACCOUNT PARAM INVALID
Code: 500 NETWORK PARAM INVALID
Code: 500 COMBINATION OF PARAMETERS INVALID
Code: 500 DELETE IN PROGRESS

#### Example

```bash
curl -H "Network: mainnet" \
-H "Authorization: Bearer: ..." \
-H "Content-Type: application/json" \
-H "OrganizationID: 72c4c072-2fe4-4f72-ae9d-d9d52a05fd71" \
-d '{"CommentID": "uuid string"}' \
-X DELETE https://comsolotex-admin.sologenic.org/api/admincomment/delete
```

## Application start parameters

The application uses the following environment parameters:

- `HTTP_CONFIG`: Included from `github.com/sologenic/com-be-http-lib/http/`. For setup see `github.com/sologenic/com-be-http-lib/http/README.md`
- `COMMENT_STORE_ENDPOINT`: Included from `github.com/sologenic/com-fs-comment-model`. For setup see `github.com/sologenic/com-fs-comment-model/client/README.md`
- `AUTH_FIREBASE_SERVICE`: Included from `github.com/sologenic/com-fs-auth-firebase-service`. For setup see `github.com/sologenic/com-fs-auth-firebase-service/README.md`
- `ACCOUNT_STORE`: Included from `github.com/sologenic/com-fs-account-model`. For setup see `github.com/sologenic/com-fs-account-model/client/README.md`
- `ROLE_STORE`: Included from `github.com/sologenic/com-fs-role-model`. For setup see `github.com/sologenic/com-fs-role-model/client/README.md`
- `FILE_STORE`: Included from `github.com/sologenic/com-fs-file-model`. For setup see `github.com/sologenic/com-fs-file-model/client/README.md`
* `ORGANIZATION_STORE` - the organization service endpoint (included from `github.com/sologenic/com-fs-organization-store/`)
- `FEATURE_FLAG_STORE` (optional if feature flags present): Included from `github.com/sologenic/com-fs-feature-flag-model`. For setup see `github.com/sologenic/com-fs-feature-flag-model/client/README.md`
