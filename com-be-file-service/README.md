# File service

The file service provides the following RESTFUL interface:

**Unauthenticated:**

* GET `/api/file/download?filename={filename_base64}`: Download file from the temporary or permanent path. Whether the file is temporary or commited is determined by the filename URL query.

**Authenticated:**

* POST `/api/file/upload`: Upload a file to the temporary location in the storage, returns temporary filename in base64.
* GET `/api/file/exist?filename{filename_base64}&commited={true_for_commited_file}`: Check if a file exists with the filename. Optional param "commited" defaults to false when omitted.

The service provides a grpc interface for being able to move files from temp to final location in a secure fashion with the permanent path and filename.

## Authenticated requests

All authenticated requests must include `Network`, `OrganizationID`, and `Authorization` in the header.

## GRPC interface

The grpc interface has a "commit" call which renames the file to the final name and places it on the correct location (as indicated in the call) on the cloud storage.

The commit call has a parameter "AllowOverwrite" as simple way to check if a file would be allowed to be replaced on the cloud storage.

The minimum requirement for the commit to work is:

* The unique temp filename as generated and returned in the POST of the file service
* permanent location(path) of the file with including the final name
* whether allowing overwriting file if the renamed file already exists in the permanent location

## Architecture notes

The filename provided for either temp files or final (commit) files is made unique bye either the file service or by the commit request:

- Cloud based temp files are made unique by the file service by pre-pending the `addressofuploader` & `network`, separated by `_`
- Commit (final) files are unique however managed by a naming schema from the application requesting the commit => In case of uncertainty set the flag `AllowOverwrite` to `false``

Garbage management on the cloud based tmp folder is provided by google cloud storage. All the files in the tmp location have the same garbage collection TTL (Time To Live) of 30 days after creation. This time is deemed sufficient to prevent most issues in normal file upload usage (files will mostly be processed in seconds to minutes, with an expected 99 percentile of 24hrs, and 99.99 percentile of 5 days).

## Unauthenticated endpoints

### Download

`GET /api/file/download?filename={filename_in_base64}`

Download a file from the storage. When the file exists, it gets returned as a stream. Client is required to get the file stream from the response and take further actions with it once confirming the status to be 200, whether to save it to the local disk, to pipe into another stream or to display it directly in case the content-type of response is "image".

Download is allowed for both the temporary and the committed file as long as the accurate temp filename or the full path of permanent path in the storage is passed as filename query. If the filename is parsable into the temporary file data structure defined by the service, the service considers it a request for the temporary file. If not, it will consider it to be a full path of the commited file in the permanent location.

#### Params

- `filename` _string, required_ - base64 string of the filename. 

*For a temporary file, it's the "FileName" returned by the upload endpoint. For a commited file, the decoded filename param will be in the format, "{app}/{internal_app_structure1}/{internal_app_structure2}/.../{final_file_name}. For instance, in case of NFT file, it could look like "nft/{collection_id}/{internal_id}" or something similar, identical to what the client passed as the "PermanentPath" param in the commit request.

#### Response

The response status code will be 200, however, it doesn't return any JSON body but contains the binary stream of file. For instance, in case of Golang, the response itself is io.Reader, and for JS/TS, a ReadableStream can be instantiated with the response.body.getReader() call. For the usage, refer to [this](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API/Using_readable_streams).

#### Example

```bash
curl --location 'https://comsolotex.sologenic.org/api/file/download?filename=bmZ0L3JFeXpwS3V2MXRnQVRFNkZadmdvYVh0Z3VYTU1qV3lxYlIvNDEwNDQ2ZTUtZTU5ZS00NTczLWE3ZDItYjJiZjc2MjI3NzVi' \
--header "OrganizationID: 72c4c072-2fe4-4f72-ae9d-d9d52a05fd71" \
--header "Network: mainnet"
```

## Authenticated endpoints

### Upload

`POST /api/file/upload`

Upload a file to the temporary location of the storage. It returns temp filename in base64 string. This temp filename must be included in the commit request as "TempFilename" param and "filename" URL query for calling download endpoint.

The request content-type must be in "multipart/form-data". Multiple attachments are not allowed.

#### Payload

- `file` _file, required_ - file attachment to upload

#### Response

```json5
{ 
    "Filename": "eyJBZGRyZXNzIjoickw1NHd6a25VWHhxaUM4VHpzNm16TGkzUUpUdFg1dVZLNiIsIk5ldHdvcmsiOiJtYWlubmV0IiwiVGVtcEZpbGVuYW1lIjoiYnV0dGVyZmx5XzMwMC5qcGciLCJFeHRlcm5hbEtleXMiOlt7IktleSI6ImNvbGxlY3Rpb25faWQiLCJWYWx1ZSI6InJFeXpwS3V2MXRnQVRFNkZadmdvYVh0Z3VYTU1qV3lxYlIifSx7IktleSI6Im5mdF91aWQiLCJWYWx1ZSI6IjQxMDQ0NmU1LWU1OWUtNDU3My1hN2QyLWIyYmY3NjIyNzc1YiJ9XX0="
}
```

#### Example

```bash
curl --location 'https://comsolotex.sologenic.org/api/file/upload' \
--header 'address: rL54wzknUXxqiC8Tzs6mzLi3QJTtX5uVK6' \
--header "Content-Type: multipart/form-data" \
--header 'Authorization: Bearer: ...' \
--header 'Network: mainnet' \
--header 'OrganizationID: 72c4c072-2fe4-4f72-ae9d-d9d52a05fd71' \
--form 'file=@"/Users/{user_name}/Downloads/{filename}"'
```

### Exist

`GET /api/file/exist?filename={filename_base64}&commited={true_for_commited_file}`

Check if a file is stored in the storage. Returns true if it's found by filename.

An optional param "commited" should be passed if the user wants to check if a commited file is found from the permanent location. If not passed, it's set to be false and searches the filename from the temporary location of the storage.

#### Param

- `filename` _string, required_ - filename in base64 string.
- `commited` _boolean, optional_ - true if the file is supposed to be already commited. When not provived, it defaults to false.

#### Response

```json5
{ 
    "Exist": true
}
```

#### Example

```bash
curl --location 'https://comsolotex.sologenic.org/api/file/exist?filename=bmZ0L3JFeXpwS3V2MXRnQVRFNkZadmdvYVh0Z3VYTU1qV3lxYlIvNDEwNDQ2ZTUtZTU5ZS00NTczLWE3ZDItYjJiZjc2MjI3NzVi&commited=true' \
--header 'address: rL54wzknUXxqiC8Tzs6mzLi3QJTtX5uVK6' \
--header 'Authorization: Bearer: ...' \
--header 'OrganizationID: 72c4c072-2fe4-4f72-ae9d-d9d52a05fd71' \
--header 'Network: mainnet' 
```

## GRPC Service

### Commit

Move an uploaded temp file to the permanent location of the storage with final name.

The temporary files are auto-deleted when TTL is reached, that is configured in the storage. When the temporary file is not found, it returns "FileFound" false with and emtpy string for "PublicURL" in the response, in which case the client is responsilbe to re-upload the file and commit, again.

#### Params

- `TempFileName`    _string, required_ - the temp filename to commit. Must be identical to "FileName" retured from the file service
- `PermanentPath`   _string, required_ - base64 string of the pernament path including final filename. Decode string will look like "{app}/{first_level_dir}/{secode_level_dir}/.../{final_filename}
- `AllowOverwrite`  _boolean, required_ - whether to overwrite the file or not, if the 'PermanentPath' already exists
- `Network`         _string, required_ -  network environment. One of "mainnet", "testnet" or "devnet"

#### Response

- `FileFound`  _boolean, required_ - if the temp filename is found from the storage.
- `PublicURL`  _string, required_ - publically accessable link of the file provided by the storage. Only accessible if the storage is configured for the public access. 
- `Network`    _string, required_ - network environment. One of "mainnet", "testnet" or "devnet"

## Testing FE

To run the testing FE, run following command in the app's root folder and open "localhost:3000" in the browser for the testing FE.

```bash
./bin/test.sh
```

The "PernamentFileName" input shouldn't be encoded in base64, but should be in a plain text, such as "test1/test11/test111", because the testing FE encodes it into base64 prior to including it to the request, so the tester doesn't have to take an extra step of enconding it manually.

The checkbox under the "Filename" input for Download must be checked if the download is for the temporary file that hasn't been commited, yet. Again, this is just for the testing FE. The service can differenciate whether it's for temporary or commited file download without an extra parameter.

## Application start parameters

* `ORGANIZATION_STORE`: Included from `github.com/sologenic/com-fs-organization-model`. For setup see `github.com/sologenic/com-fs-organization-model/client/README.md`
- `PROJECT_ID`: Google Cloud project ID
- `ROLE_STORE`: Role store endpoint
- `AUTH_FIREBASE_SERVICE`: Auth service endpoint
- `CREDENTIALS_LOCATION` : google cloud credentionals location
- `HTTP_CONFIG`: HTTP configuration

## Setup the temp bucket

The temp bucket has the unique property that it deletes files placed in there after 30 days, thus making it auto cleaning. If a user aborts a process in which files are to be permanently kept, and the process follows the ideas outlined in this readme, there is no cleanup required and garbage is automatically managed.

The bucket is a gcloud configuration which is as follows:

* CREATE bucket
* Set single region (no high availability required/lowest cost)
* Standard class
* Uniform access control (all data in this bucket is public, however with the hashing used it will be near impossible to guess what is in here (similar security level as what the ledger uses for the secret keys))
* Protection tools: None: We can loose the data and do not care about that: Data typically lives here for seconds to minutes, and a user can always re-upload it. Also the data is to be used in a public fashion, so no extra encryption is required.
* Uncheck the mark `Enforce public access prevention on this bucket` (this bucket is supposed to be publicly accessible, however will not be used in that style (it is a just in case))
* Confirm
* Click on the bucket
* Click permissions
* Click grant access
* Enter allUsers in the New Principals field
* Select Cloud Storage => Storage object viewer in the Select a role field
* Save
* Confirm `Allow public access`
* Select `Lifecycle`
* Select `Add a rule`
* Select `Delete object`
* Continue
* Select `Age`
* Enter `30` in `Days`
* Click `Continue`
* Click `Create`

There should now be a rule stating: `Delete object	30+ days since object was created`

## Adjust request body size

This application nginx as an Ingress controller. The default request body size is 1MB. To increase the request body size, the following configuration should be added to the file-service deployment file (file-service.yaml.erb).

```yaml
ingress:
  ...
  annotations:
    ...
    nginx.ingress.kubernetes.io/proxy-body-size: 100m # this is setting the request body size to 100MB
    ...
```
