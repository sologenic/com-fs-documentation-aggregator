# com-be-reference-service

Generic reference tracking system

## Basic functionalities

The service stores and retrieves references for different subsystems. The base concepts, while they could be used in a different context (not advised), are:

- Follow: Follow a user or system: E.g. follow the Market index
- Like: Like an (granular/lowlevel) object: E.g. like an NFT, Token
- Visit: A page
- Watch: An (group/high level) object: E.g. watch a collection
- Vote: Vote on an object: E.g. vote on a poll with either an upvote or a downvote

## Enums vs strings

The enum values are passed as their integer values. This can be managed by the implementer by importing hte model and using the constants in there.

### Follow/Like/Watch/Vote

Follow/Like/Watch/Vote is a boolean/binary approach: A user either has a record or not.

On storage level that means:

- Some key is being {actioned} by some other key at time X

The removal of a Follow/Like/Watch/Vote leads to removal of that given record.

The Follow/Like/Watch/Vote is defined in an enum in `com-fs-reference-model` and can be freely extended to more values: The term `Follow/Like/Watch/Vote` is only relevant for discussions, notifications, or subsystems putting a meaning on such a term, but has no other meaning in the reference system itself.
The enum is used to differentiate between the data for systems: This is crucial: Using a `marketindex_follow` in NFT marketplace context, will lead to the NFT marketplace generating broken links (for example).

### Visit

Visit is a page visit. It behaves different than `follow/like/watch`:
Where like, follow and watch are boolean/binary in behaviour: You follow or do not any longer follow (or have never followed) another user, a visit is permanent.
Visit however only tracks the _last_ visit and keeps this forever (or for how long we do not delete visits).

For example:

User A visits Page D (at 13:00)

And later:

User A visits Page D (at 13:05) => Only this event is kept.

The timestamps `CreatedAt` and `UpdatedAt` are used indicate the difference in time between the visits.

## API

The API is as follows:

Unauthenticated:

- GET /list?reference_type=...&account=...&reference_keys= : Returns list of type for account
- GET /count?reference_type=...&account=... : Returns the count of type for account
- GET /alltime?reference_type=...&reference_ids=... : Returns the count for the given type and references in an array
- GET /votecount?reference_type=...&reference_ids=... : Returns the count for the given type and references in an array grouped by the vote (up,down,none) value

Authenticated:

- POST /add : Add a reference
- DELETE /delete : Delete a reference
- POST /vote : Vote on an object

### List `GET /list?reference_type=...&account=...&reference_keys=...&before_reference_id=...`

- `reference_type` _required_ - The type as string as indicated in the enum. Casing same as in the enum (backend checks the value to be exact)
- `account` _required_ - xrp address of the account that the list of reference_type is to be retrieved from
- `reference_keys` _optional_ - Retrieve if the Reference_keys for the given type by the account
- `before_reference_id` _optional_ - if specified, returns only references for the given type in which the `reference_id` is less than the specified value

`reference_keys`: The reference_ids of the type of which the dev wants to know if there is a `type` from the account on those given reference_ids, provided as base64 encoded json array.

Json array:

```json
[
  "00082EE0A19D0322C71F859F8C7A7D8584D234E97DB949C3B72E91A300000008",
  "00082EE0A19D0322C71F859F8C7A7D8584D234E97DB949C3B72E91A300000008",
  "00082EE0A19D0322C71F859F8C7A7D8584D234E97DB949C3B72E91A300000008",
  "00082EE0A19D0322C71F859F8C7A7D8584D234E97DB949C3B72E91A300000008",
  "00082EE0A19D0322C71F859F8C7A7D8584D234E97DB949C3B72E91A300000008"
]
```

in base64 encoding:

```base64
WyIwMDA4MkVFMEExOUQwMzIyQzcxRjg1OUY4QzdBN0Q4NTg0RDIzNEU5N0RCOTQ5QzNCNzJFOTFBMzAwMDAwMDA4IiwiMDAwODJFRTBBMTlEMDMyMkM3MUY4NTlGOEM3QTdEODU4NEQyMzRFOTdEQjk0OUMzQjcyRTkxQTMwMDAwMDAwOCIsIjAwMDgyRUUwQTE5RDAzMjJDNzFGODU5RjhDN0E3RDg1ODREMjM0RTk3REI5NDlDM0I3MkU5MUEzMDAwMDAwMDgiLCIwMDA4MkVFMEExOUQwMzIyQzcxRjg1OUY4QzdBN0Q4NTg0RDIzNEU5N0RCOTQ5QzNCNzJFOTFBMzAwMDAwMDA4IiwiMDAwODJFRTBBMTlEMDMyMkM3MUY4NTlGOEM3QTdEODU4NEQyMzRFOTdEQjk0OUMzQjcyRTkxQTMwMDAwMDAwOCJd
```

With a maximum of 20 reference_keys in a single array to prevent overflow of the URL. This now encoded data + other parts of the URL maxes out at about 1850 characters, which below the limit of 2MB for Chrome/Edge, and way below the limits of Firefox and Opera, see [reference](https://mywebshosting.com/what-is-the-maximum-url-length-limit-in-browsers/).

The use of the reference_key in the given context means that the FE usually already has a very valid key available to start a query into the reference system, and does not first have to lookup internal reference ids with an initial (open) query to the reference system.

Unknown keys are not added to the result set as empty values (query for single unknwon key will show an `{}` as response).

Duplicate keys are de-duplicated in the result set.

#### Success Response

```json
{
  "References": [
    {
      "Reference": {
        "ReferenceID": 1747426781762399450,
        "Account": "dev.sologenic@gmail.com",
        "ReferenceType": 1,
        "ReferenceKey": "11",
        "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71"
      },
      "MetaData": {
        "Network": 2,
        "UpdatedAt": {
          "seconds": 1747426781,
          "nanos": 762400271
        },
        "CreatedAt": {
          "seconds": 1747426781,
          "nanos": 633295001
        },
        "UpdatedByAccount": "dev.sologenic@gmail.com"
      },
      "Audit": {
        "ChangedBy": "dev.sologenic@gmail.com",
        "ChangedAt": {
          "seconds": 1747426781,
          "nanos": 762399961
        }
      }
    }
  ]
}
```

#### Error Response

Code: 422 ACCOUNT PARAM INVALID

```json5
{
  errors: [
    {
      name: "account.invalid",
    },
  ],
}
```

Code: 422 REFERENCE_TYPE PARAM INVALID

```json5
{
  errors: [
    {
      name: "reference_type.invalid",
    },
  ],
}
```

#### Example

Without pagination (e.g. initial call)

```bash
curl -H "Network: testnet" \
"https://comsolotex.sologenic.org/api/reference/list?reference_type=3&account=rHfvKjmoHZtsgFzwa3xcuERnhN1GgXgSCp"
```

With pagination:

```bash
curl -H "Network: testnet" \
"https://comsolotex.sologenic.org/api/reference/list?reference_type=3&account=rHfvKjmoHZtsgFzwa3xcuERnhN1GgXgSCp&before_reference_id=150"
```

Example with reference keys:

00082EE0A19D0322C71F859F8C7A7D8584D234E97DB949C3B72E91A300000008

### Count reference_type by account `GET /count?reference_type=...&account=...`

- `reference_type` _required_ - The type as string as indicated in the enum. Casing same as in the enum (backend checks the value to be exact)
- `account` _required_ - xrp address of the account that the count of reference_type is to be retrieved from

#### Success Response

`int`

#### Error Response

Code: 422 ACCOUNT PARAM INVALID

```json5
{
  errors: [
    {
      name: "account.invalid",
    },
  ],
}
```

Code: 422 REFERENCE_TYPE PARAM INVALID

```json5
{
  errors: [
    {
      name: "reference_type.invalid",
    },
  ],
}
```

#### Example

```bash
curl -H "Network: testnet" \
"https://comsolotex.sologenic.org/api/reference/count?reference_type=3&account=rHfvKjmoHZtsgFzwa3xcuERnhN1GgXgSCp"
```

Returns:

`5`

### `POST /add`

There are 2 types of adding a reference to the reference system:

- A Key value pair: For example a NFT and a NFT ID
- A system level reference: For example `ido_follow` with then as ReferenceKey the same as the ReferenceType `ido_follow`

#### Payload

```json
{
  "ReferenceKey": "00082EE0A19D0322C71F859F8C7A7D8584D234E97DB949C3B72E91A300000008",
  "ReferenceType": 3
}
```

Or

```json
{
  "ReferenceKey": "ido_follow",
  "ReferenceType": 2
}
```

#### Success Response

`HTTP 200 OK`

#### Error Response

Code: 422 ACCOUNT PARAM INVALID

```json5
{
  errors: [
    {
      name: "account.invalid",
    },
  ],
}
```

#### Example

```bash
curl -X POST "https://comsolotex.sologenic.org/api/reference/add" \
    -H "Content-Type: application/json" -H "Authorization: 120003228000000024015BAD19201B0161A2D268400000000000000C732102CE2925AD304D2CF04C4E4EB8F335BA0F87C7914D642C0B26F73209B4B585CC657446304402205B5649AF1A08AC2BDDDFCE2385E58DD5113079E579FD8C2BF9AEE795B114298F022019C5E5996B8248B4B3A1F5AA88A39A8AA89670E38430413C928A6F0A3C323AD081140AC546D09CEB1DCE1ECAC0BCD7810EC244F40E9BF9EA7D217369676E5F696E5F5F5F323032312D31312D33302032303A32333A30302E303030E1EA7D2461633232656164632D383738382D346132382D393536392D356230636461646564656637E1F1" \
    -H "Network: testnet" \
    -d '{"ReferenceKey": "00082EE0A19D0322C71F859F8C7A7D8584D234E97DB949C3B72E91A300000008",
         "ReferenceType": 3,
        }'
```

### `DELETE /delete`

Deletes a reference or a vote.

```json
{
  "ReferenceKey": "23456",
  "ReferenceType": 3
}
```

#### Success Response

`HTTP 200 OK`

#### Error Response

```json5
{
  errors: [
    {
      name: "account.invalid",
    },
  ],
}
```

#### Example delete

```bash
curl -X DELETE "https://comsolotex.sologenic.org/api/v1/nft-marketplace/nfts/likes" \
    -H "Content-Type: application/json" -H "Authorization: 120003228000000024015BAD19201B0161A2D268400000000000000C732102CE2925AD304D2CF04C4E4EB8F335BA0F87C7914D642C0B26F73209B4B585CC657446304402205B5649AF1A08AC2BDDDFCE2385E58DD5113079E579FD8C2BF9AEE795B114298F022019C5E5996B8248B4B3A1F5AA88A39A8AA89670E38430413C928A6F0A3C323AD081140AC546D09CEB1DCE1ECAC0BCD7810EC244F40E9BF9EA7D217369676E5F696E5F5F5F323032312D31312D33302032303A32333A30302E303030E1EA7D2461633232656164632D383738382D346132382D393536392D356230636461646564656637E1F1" \
    -H "Network: testnet" \
    -d '{"ReferenceKey": "00082EE0A19D0322C71F859F8C7A7D8584D234E97DB949C3B72E91A300000008"
          "ReferenceType": 3,
        }'
```

### `POST /vote`

The vote is nearly the same as the add.

#### Payload

```json
{
  "ReferenceKey": "00082EE0A19D0322C71F859F8C7A7D8584D234E97DB949C3B72E91A300000008",
  "ReferenceType": 2,
  "VoteType": 1
}
```

### All time counts

All time counts are provided from the BE (as opposed to Algolia). This enables the system provide real time counts for nft_likes and others without having to update Algolia at high frequency.

`GET /alltime?reference_type=...&reference_keys=...`

#### URL Params

- `reference_type` _required_
- `reference_keys` _required_ - json array of reference_keys, max number of ids 20

reference_keys are encoded the same as in chapter **List** (base64 encoded json array)

#### Example all time counts

```bash
curl -H "Network: testnet" \
  "https://comsolotex.sologenic.org/api/reference/alltime?reference_type=nft_likes&reference_keys=W3siUmVmZXJlbmNlS2V5IjoiMDAwODJFRTBBMjE5RDAzMjJDNzFGODU5RjhDN0E3RDg1ODRENjM0RTk3REI5NDlDM0I3MkU5MUEzMDAwMDAwMDgiLCJSZWZlcmVuY2VUeXBlIjoibmZ0X2xpa2VzIn1d"
```

Returns:

```json
{
    "Counts": [
        {
            "Count": 1,
            "ReferenceType": 1,
            "ReferenceKey": "00082EE0A19D0322C71F859F8C7A7D8584D234E97DB949C3B72E91A300000008",
            "Network": "mainnet"
        }
    ]
},
{...}
]
```

### Votecount `GET /votecount`

Returns the votecount for one or more reference_keys.

#### URL Params

- `reference_type` _required_
- `reference_keys` _required_ - json array of reference_keys, max number of ids 20

```json
{
  "UpVotes": [
    {
      "Count": 1,
      "ReferenceType": 9,
      "ReferenceKey": "1",
      "Network": "mainnet"
    }
  ],
  "DownVotes": []
}
```

In which the lack of DownVotes means that there are no downvotes.

#### Example votecount

```bash
curl -H "Network: testnet" \
  "https://comsolotex.sologenic.org/api/reference/votecount?reference_type=4&reference_keys=WyIxIl0="
```

### Get by Reference Key `GET /get?reference_key=...`

Returns a single reference record matching the provided reference key and network.

- `reference_key` _required_ - The unique key of the reference to retrieve (string, not base64 encoded)
- `Network` header is required (e.g. `mainnet`, `testnet`)
- `Authorization` header is required (e.g. `Bearer: Qdfjm13-fame02d1...)

#### Success Response

```json
{
  "Reference": {
    "ReferenceID": 1747426781762399450,
    "Account": "dev.sologenic@gmail.com",
    "ReferenceType": 1,
    "ReferenceKey": "11",
    "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71"
  },
  "MetaData": {
    "Network": 2,
    "UpdatedAt": {
      "seconds": 1747426781,
      "nanos": 762400271
    },
    "CreatedAt": {
      "seconds": 1747426781,
      "nanos": 633295001
    },
    "UpdatedByAccount": "dev.sologenic@gmail.com"
  },
  "Audit": {
    "ChangedBy": "dev.sologenic@gmail.com",
    "ChangedAt": {
      "seconds": 1747426781,
      "nanos": 762399961
    }
  }
}
```

#### Error Response

Code: 422 REFERENCE_KEY PARAM INVALID

```json5
{
  errors: [
    {
      name: "reference_key.invalid",
    },
  ],
}
```

#### Example

```bash
curl -H "Network: testnet" \
  "https://comsolotex.sologenic.org/api/reference/getByKey?reference_key=00082EE0A19D0322C71F859F8C7A7D8584D234E97DB949C3B72E91A300000008"
```

## Application start parameters

- `ORGANIZATION_STORE`: Included from `github.com/sologenic/com-fs-organization-model`. For setup see `github.com/sologenic/com-fs-organization-model/client/README.md`
- `REFERENCE_STORE_ENDPOINT` - See [github.com/sologenic/com-fs-reference-store/client](github.com/sologenic/com-fs-reference-store/client)
- `AUTH_SERVICE_ENDPOINT` - See [github.com/sologenic/com-fs-auth-model/client/go](github.com/sologenic/com-fs-auth-model/client/go)) in the new (non-deprecated) format
- `HTTP_CONFIG` - Authorized and unauthorized setup (see [github.com/sologenic/be-services/go/pkg/http](github.com/sologenic/be-services/go/pkg/http))
- `ACCOUNT_STORE` - See [github.com/sologenic/fs-account-store](github.com/sologenic/fs-account-store)
