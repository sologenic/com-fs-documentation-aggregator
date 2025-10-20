# Comment service

The comment service provides the generic functionality to attach comments to any source type [github.com/sologenic/com-fs-notification-model](https://github.com/sologenic/com-fs-notification-model). This provides consistency between what can be commented and source types are used for rendering content. Essentially the source types are the different functions of the system.

Besides adding comments to a source type, the comment service also provides the functionality to add comments to other comments. This allows for a threaded discussion. 

## API

**Unauthenticated (Public comments visible to all users):**

* GET `api/comment/get?source_type=...&source...&comment_ids=...&account_id=...&order=...`: Get comments for source type, source, comment_ids, account (see below for allowed combinations)
  
* GET `api/comment/count?source_type=...&source=...`: Count the number of comments for a source type and source (e.g. total count of comments). The assumption in this count is that it will be used on the single object page.
* GET `api/comment/count?account_id=...`: Count the number of comments for a user
* GET `api/comment/count?comment_ids=...`: Get counts for sub comment(s). Multiple IDs are provided in a base64 encoded json array, max 20 ids.

> **Note:** 
> - Unauthenticated endpoints will be used for public comments visible to all users.
> - Sensitive source types (such as support tickets) are automatically blocked in unauthenticated endpoints for security reasons. These can only be accessed through the authenticated endpoints by the comment owner.
> - ⚠️ **For maintainers**: When adding new sensitive source types to the system, you MUST update the `sensitiveSourceTypes` variable in `comment.go` to include these types in the security check.

**Authenticated (Private comments visible only to the user, admins):**

* GET `api/comment/auth/get?source_type=...&order=...`: Get comments for source type, source, comment_ids, account (see below for allowed combinations)

* GET `api/comment/auth/get?source_type=...&order=...`: Get all root comments created by the authenticated account for a certain source type. Ordered by newest first
* POST `api/comment/auth/add`: Add/create a comment or a comment on a comment
* DELETE `api/comment/auth/delete`: Delete a comment, used for direct owner of message deletion
* PUT `api/comment/auth/close`: Close a comment
* PUT `api/comment/auth/new-reply`: Set the status of a comment to `NEW_REPLY` to signal that a support ticket needs further admin attention after receiving a reply.

> **Note:** 
> - All authenticated requests must include `Network`, `OrganizationID`, and `Authorization` in the header.
> - Users can only access and modify their own comments. Admin operations are handled through separate service(com-be-admin-comment-service)

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

### Note about enums

The ENUM values in the documentation are represented in the string values for readability. In the actual communication between FE and BE they are the enum integer values.

### Querying against an empty database

GET/COUNT does not work against an empty database by lack of any model for the database to reference to.
We have to add 1 comment into the system using wget/curl to avoid this empty database error (easier than catching it all over the place for a 10s situation only at launch of the comment system).

### List comments `GET /get`

The listing of comments is a main functionality to be able to view comments in different ways.
The endpoint is not authenticated, in contrary to the admin endpoint for listing comments. The admin endpoint differs in filter options and flexibility.

The endpoint accepts the following query parameters:

* `source_type`, and `order` in which `source_type` is defined in parameters.SourceType [parameters](https://github.com/sologenic/com-fs-notification-model/parameters) and `order` is according to the `Order` enum
* `source_type`, `source` and `order` in which `source_type` is defined in parameters.SourceType [parameters](https://github.com/sologenic/com-fs-notification-model/parameters) and `order` is according to the `Order` enum
* `account` and `order`
* `comment_ids`

Other combinations are not allowed then the 4 variants above, are not allowed and can lead to unexpected results.

Order is optional.

The checks by the BE are not strict: Not all combinations are checked before execution.

If 

* `source_type` is found, the source_type query runs
* `account` is found, the account query runs
* `comment_ids` is found, the comment_ids query runs

So if `source_type` and `account` are found, the `source_type` query runs (etc).

#### Examples

```bash
curl -H "Network: mainnet" \
-H "Content-Type: application/json" \
-X GET https://comsolotex.sologenic.org/api/comment/get?source_type=2&source=1234ABCD&order=1
```

```bash
curl -H "Network: mainnet" \
-H "Content-Type: application/json" \
-X GET https://comsolotex.sologenic.org/api/comment/get?source_type=2&order=1
```

```bash
curl -H "Network: mainnet" \
-H "Content-Type: application/json" \
-X GET https://comsolotex.sologenic.org/api/comment/get?account_id=abc@gmail.com&order=1
```

```bash
curl -H "Network: mainnet" \
-H "Content-Type: application/json" \
-X GET https://comsolotex.sologenic.org/api/comment/get?comment_ids=WzEyMzQ1LDEyMzQ1NiwxMjM0NTY3XQo=
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
                    "Content": "This is a comment",
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
                    "DeletionAccount": "rhozSjrMoVBV8PuQAdyLro2FEF2uVBPfzw",
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

Count comments for a certain `source_type`
Count comments for a certain `source_type` and `source`
Count comments for a given `account`
Count comments associated with a given list of `comment_ids`

#### Examples

```bash
curl -H "Network: mainnet" \
-H "Content-Type: application/json" \
-X GET https://comsolotex.sologenic.org/api/comment/count?source_type=2&source=12345
```

#### Success response

The success response is an integer:

```json
{"Counts":[{}]}

```

#### Error response

Code: 500 NETWORK PARAM INVALID
Code: 500 SOURCE_TYPE PARAM INVALID

### Get Comments `/auth/get`

Get comments with authentication, supporting the same four query patterns as the public endpoint: by source_type, by source_type+source, by account, or by comment_ids. 
Unlike the public endpoint, authenticated users can access their own sensitive content (like support tickets).

#### Examples

```bash
curl -H "Network: mainnet" \
-H "Authorization: Bearer: ..." \
-H "Content-Type: application/json" \
-H "OrganizationID: 72c4c072-2fe4-4f72-ae9d-d9d52a05fd71" \
-X GET https://comsolotex.sologenic.org/api/comment/auth/get?source_type=4&order=1
```

#### Success response

The success response returns a list of comments.

```json
{
    "Offset": 1, // The offset to use for the next request
    "CommentResult": [
        {
            "Comments": [
                {
                    "CommentID": "62e3f1fc-fc12-45c6-982b-4455934a02ef",
                    "AccountID": "sg.be.org.test@gmail.com",
                    "MetaData": {
                        "Network": 2,
                        "UpdatedAt": {
                            "seconds": 1742502788,
                            "nanos": 274733605
                        },
                        "CreatedAt": {
                            "seconds": 1742502788,
                            "nanos": 250151933
                        }
                    },
                    "Source": "04f51a34-fbe8-4d02-97f7-8a4be2d4c4dc",
                    "SourceType": 4,
                    "Content": "testtest account issue22",
                    "Root": "04f51a34-fbe8-4d02-97f7-8a4be2d4c4dc",
                    "RootType": 4,
                    "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
                    "Tags": [
                        "Account issue"
                    ],
                    "Files": [
                        {
                            "Reference": "7a39bf30-3b1c-4538-94ed-f76210b7c973",
                            "Extension": "image/png",
                            "Name": "Screenshot 2025-03-26 at 12.19.59.png"
                        }
                    ],
                    "Status": 1
                },
            ]
        }
    ]
}
```

### Add comment `/add`

A typical comment consists out a body with some kind of embedded markup. The body is agnostic of language and can certain content associated with it like video or image. The FE component determines how this works, and media files must be uploaded separately via com-be-file-service, with the resulting links included in the Content field.

The POST body to add a comment contains:

* SourceType: From the enum `SourceType` in the notification model [github.com/sologenic/com-fs-notification-model](https://github.com/sologenic/com-fs-notification-model).
* Source: The ID of the source type.
* Content: Content will contain the links to the uploaded images/videos excluding the domain (thus preventing linking to external resources).
* Tags: Tags are used to categorize the comment. e.g. `["Payment issue"]` for a support ticket.

The post body looks like:

```json5
{
  "SourceType": 4,
  "Source": "12345",
  "Content": "This is a comment",
  "Tags": ["tag1", "tag2"],
  "Files": [
    {
      "Reference": "a5dd1be5-b93a-46fc-98e0-0c20f64955f1",
      "Extension": "image/png",
      "Name": "Screenshot 2025-03-26 at 12.19.34.png"
    },
    {
      "Reference": "c76d4aff-ac34-4057-a8f2-216aa1ff5344",
      "Extension": "image/png",
      "Name": "Screenshot 2025-03-26 at 12.19.44.png"
    }
  ]
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
-X POST https://comsolotex.sologenic.org/api/comment/auth/add
```

### Add comment to comment

The adding of a comment to a comment differs from the add comment in the fact that more information needs to be provided:

* SourceType: `1` (`COMMENT_ID`)
* Source: `Source of the comment the comment is added to`

The backend adds the `RootType` and `Root` against the `Source` to ensure that the comment is added to the correct object (and no cross commenting hacks are possible).
* RootType: The type of the root object (NFT_ID etc). RootType is of type SourceType.
* Root: The ID of the root object (NFT_ID etc)

The addition of the `RootType` and `Root` makes it possible to count all the comments on a given object without having to iterate over all the comments in the comment tree.
There is no information to count all the replies on replies in a comment tree without iterating over the tree itself.

The `SourceType` and `Source` link the comment to the comment the reply is placed on.

The post body looks like:

```json5
{
  "SourceType": 1,
  "Source": "1234567890",
  "Content": "This is a comment",
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
-X POST https://comsolotex.sologenic.org/api/comment/auth/add
```

### Delete a comment `/delete`

Only the owner of a comment can delete it. The system verifies ownership by checking that the accountID matches the account ID associated with the comment.

The service only sets the state of the comment to DELETED or DELETED_CASCADE.
The trigger channel will process the comment and remove it from the database if the rules are met:
* Cascading delete is executed only of ownership rules are met (account requesting delete is the owner of the root object)
* Cascading delete is ignored if the account requesting the delete is not the owner of the root object.
* Delete is ignored when the account requesting the delete is not the owner of the comment, or owner of the root object.

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
-X DELETE https://comsolotex.sologenic.org/api/comment/auth/delete
```

#### Close a comment `PUT /close`

Closes a comment for a user. The endpoint hardcodes the status to `REVIEWED` to prevent the user from updating the status of the comment to anything else. A user cannot reopen a comment once it is closed.

The message to the `/close` endpoint is:

```go
type StatusUpdateRequest struct {
	CommentID string
}
```

##### Example

```bash
curl -H "Network: testnet" \
-H "Authorization: Bearer: ..." \
-H "Content-Type: application/json" \
-H "OrganizationID: 72c4c072-2fe4-4f72-ae9d-d9d52a05fd71" \
-d '{"CommentID": "uuid string"}' \
-X PUT https://comsolotex.sologenic.org/api/comment/auth/close
```

#### Success response

`200OK`

#### New reply `PUT /new-reply`

The new reply endpoint is used to set the status of a comment to `NEW_REPLY` to signal that a support ticket needs further admin attention after receiving a reply. This is used in the support ticket system.

The message to the `/new-reply` endpoint is:

```go
type StatusUpdateRequest struct {
    CommentID string
}
```

##### Example

```bash
curl -H "Network: testnet" \
-H "Authorization: Bearer: ..." \
-H "Content-Type: application/json" \
-H "OrganizationID: 72c4c072-2fe4-4f72-ae9d-d9d52a05fd71" \
-d '{"CommentID": "uuid string"}' \
-X PUT https://comsolotex.sologenic.org/api/comment/auth/new-reply
```

#### Success response

`200OK`

## Application start parameters

The application uses the following environment parameters:

* `ORGANIZATION_STORE`: Included from `github.com/sologenic/com-fs-organization-model`. For setup see `github.com/sologenic/com-fs-organization-model/client/README.md`
* `HTTP_CONFIG`: Included from `github.com/sologenic/com-be-http-lib/http/`. For setup see `github.com/sologenic/com-be-http-lib/http/README.md`
* `COMMENT_STORE`: Included from `github.com/sologenic/com-fs-comment-model`. For setup see `github.com/sologenic/com-fs-comment-model/client/README.md`
* `AUTH_FIREBASE_SERVICE`: Included from `github.com/sologenic/com-fs-auth-firebase-service`. For setup see `github.com/sologenic/com-fs-auth-firebase-service/README.md`
* `USER_STORE`: Included from `github.com/sologenic/com-fs-user-model`. For setup see `github.com/sologenic/com-fs-user-model/client/README.md`
* `ROLE_STORE`: Included from `github.com/sologenic/com-fs-role-model`. For setup see `github.com/sologenic/com-fs-role-model/client/README.md`
* `FEATURE_FLAG_STORE`: Included from `github.com/sologenic/com-fs-feature-flag-model`. For setup see `github.com/sologenic/com-fs-feature-flag-model/client/README.md`
* `FILE_STORE`: Included from `github.com/sologenic/com-fs-file-model`. For setup see `github.com/sologenic/com-fs-file-model/client/README.md`
