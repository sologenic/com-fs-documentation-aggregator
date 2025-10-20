# AED service

The asset epoch data service provides the following restful interfaces:

* `GET /aed/chart?symbol=...&period=...&offset...&backfill=...`: Returns a chart for the for the given symbol and period
* `GET /aed/tickers?symbols=...`: Retrieve the ticker for a given symbol

## Offset

A call returning a list of values can return an offset. Use this offset in the subsequent call to get the next set of data.
If there is no more data to be returned, the offset is not returned or set to 0.

## GET /aed/chart?symbol=...&period=...&from=...&to=...&backfill=...

Load the price chart data for the symbol for a given period.

***NOTE: For user performance graph, use the `/graph` endpoint in `com-be-holdings-service`. If `Series` for `USER_PERFORMANCE` (3) is requested to this service, it will be rejected.***

### Parameters

* `symbol`: The symbol (pair) to load the chart data for
* `series`: The series of the aed data to load (refer to the `Series` enum in `com-fs-aed-model` for a list of valid series)
* `period`: The period to load the chart data for. Valid values are: 1minute, 3minute, 5minute, 15minute, 30minute, 1hour, 3hour, 6hour, 12hour, 1day, 3day, 1week (as `"1m","3m","5m","15m","30m","1h","3h","6h","12h","1d","3d","1w"`)
* `from`: from time stamp (seconds since epoch)
* `to`: to time stamp (seconds since epoch)
* `backfill`: If set to `true` the service will backfill until the from if there was no exact record at the requested from time, that is it will fill in the first gap. Default is `true` because this is the most likely usage. Note that `backfill` is only effective when `allowcache` is also set to `true`.
* `allowcache`: If set to `true` the service will use the in-memory cache in the ohlc store.
It will fill in the middle and end gaps with the last valid data point. Additionally, the first gap will be filled if `backfill` is set to `true`. If `allowcache` is `false`, it will return data directly from the database without filling any gaps, which may result in sparse data.

The data returned is sparse, meaning that if there is no data for a given interval, the interval is not returned, and that rendinering code will have to expand the data.
This will guarantee the smallest sets of data going over the wire (fast), and smaller caches which are easier to manage.

### Example curl GET

```bash
curl -X GET \
"https://comsolotex.sologenic.org/api/aed/chart?symbol=suwusdc_1-testcore13s2mmgg4uu4fn8mue6s3lgn74jwdupndjtqah8uxufugtajkeq2qgznc28:sumsft_1-testcore13s2mmgg4uu4fn8mue6s3lgn74jwdupndjtqah8uxufugtajkeq2qgznc28&series=2&period=5m&from=1743687000&to=1743701400" \
-H "Authorization: Bearer ..." \
-H "OrganizationID: ..." \
-H "Network: mainnet"
```

### Response
  
```json
{
    "AEDs": [
        {
            "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
            "Symbol": "suwusdc_1-testcore13s2mmgg4uu4fn8mue6s3lgn74jwdupndjtqah8uxufugtajkeq2qgznc28:sumsft_1-testcore13s2mmgg4uu4fn8mue6s3lgn74jwdupndjtqah8uxufugtajkeq2qgznc28",
            "Timestamp": {
                "seconds": 1743687000
            },
            "Period": {
                "Type": 1,
                "Duration": 5
            },
            "MetaData": {
                "Network": 2,
                "UpdatedAt": {
                    "seconds": 1745972243,
                    "nanos": 177630000
                },
                "CreatedAt": {
                    "seconds": 1745972242,
                    "nanos": 122181000
                }
            },
            "Value": [
                {
                    "Field": 1,
                    "Float64Val": 320.65819262974753
                },
                {
                    "Field": 2,
                    "Float64Val": 321.00428602312036
                },
                {
                    "Field": 3,
                    "Float64Val": 319.6604219196984
                },
                {
                    "Field": 4,
                    "Float64Val": 319.6604219196984
                },
                {
                    "Field": 5,
                    "Float64Val": 25.906950396334995
                },
                {
                    "Field": 6,
                    "Int64Val": 5
                },
                {
                    "Field": 8,
                    "Float64Val": 2397453164397.7383
                },
                {
                    "Field": 9,
                    "Float64Val": 5.158400859304637
                },
                {
                    "Field": 10,
                    "Float64Val": 28.2097248007264
                },
                {
                    "Field": 11,
                    "Float64Val": 0.5204631011271763
                },
                {
                    "Field": 12,
                    "Int64Val": 1743687000
                },
                {
                    "Field": 13,
                    "Int64Val": 1743687240
                }
            ],
            "Series": 2
        },
        {
            "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
            "Symbol": "suwusdc_1-testcore13s2mmgg4uu4fn8mue6s3lgn74jwdupndjtqah8uxufugtajkeq2qgznc28:sumsft_1-testcore13s2mmgg4uu4fn8mue6s3lgn74jwdupndjtqah8uxufugtajkeq2qgznc28",
            "Timestamp": {
                "seconds": 1743687300
            },
            "Period": {
                "Type": 1,
                "Duration": 5
            },
            "MetaData": {
                "Network": 2,
                "UpdatedAt": {
                    "seconds": 1745972244,
                    "nanos": 807874000
                },
                "CreatedAt": {
                    "seconds": 1745972243,
                    "nanos": 580059000
                }
            },
            "Value": [
                {
                    "Field": 1,
                    "Float64Val": 319.5315898274127
                },
                {
                    "Field": 2,
                    "Float64Val": 319.70894400381826
                },
                {
                    "Field": 3,
                    "Float64Val": 318.697741396257
                },
                {
                    "Field": 4,
                    "Float64Val": 319.18733795815683
                },
                {
                    "Field": 5,
                    "Float64Val": 30.049074476775246
                },
                {
                    "Field": 6,
                    "Int64Val": 5
                },
                {
                    "Field": 8,
                    "Float64Val": 2393905034686.1763
                },
                {
                    "Field": 9,
                    "Float64Val": 5.373756432510579
                },
                {
                    "Field": 10,
                    "Float64Val": 29.66575234790458
                },
                {
                    "Field": 11,
                    "Float64Val": 0.6820568237845774
                },
                {
                    "Field": 12,
                    "Int64Val": 1743687300
                },
                {
                    "Field": 13,
                    "Int64Val": 1743687540
                }
            ],
            "Series": 2
        },
        {
            "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
            "Symbol": "suwusdc_1-testcore13s2mmgg4uu4fn8mue6s3lgn74jwdupndjtqah8uxufugtajkeq2qgznc28:sumsft_1-testcore13s2mmgg4uu4fn8mue6s3lgn74jwdupndjtqah8uxufugtajkeq2qgznc28",
            "Timestamp": {
                "seconds": 1743687600
            },
            "Period": {
                "Type": 1,
                "Duration": 5
            },
            "MetaData": {...},
            "Value": [...],
            "Series": 2
        },
                {
            "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
            "Symbol": "suwusdc_1-testcore13s2mmgg4uu4fn8mue6s3lgn74jwdupndjtqah8uxufugtajkeq2qgznc28:sumsft_1-testcore13s2mmgg4uu4fn8mue6s3lgn74jwdupndjtqah8uxufugtajkeq2qgznc28",
            "Timestamp": {
                "seconds": 1743687900
            },
            "Period": {
                "Type": 1,
                "Duration": 5
            },
            "MetaData": {...},
            "Value": [...],
            "Series": 2
        },
        ...
    ]
}
```

## GET /aed/tickers?symbols=...

### Parameters

* `symbols` *required max 20 symbols* - a base64 encoded array of symbols for which tickers should be fetched. While there is no direct limit in the application, it is advised to keep the total length of the URL below 2kb (2048bytes) to prevent the browser from failing to send the url.
* `series` - the series of the aed data to load (refer to the `Series` enum in `com-fs-aed-model` for a list of valid series).

### Response

```json5
{
    "tickers": [
        {
            "Symbol": "suwusdc_1-testcore13s2mmgg4uu4fn8mue6s3lgn74jwdupndjtqah8uxufugtajkeq2qgznc28:suaapl_1-testcore13s2mmgg4uu4fn8mue6s3lgn74jwdupndjtqah8uxufugtajkeq2qgznc28",
            "OpenTime": 1746125992,
            "CloseTime": 1746212392,
            "OpenPrice": 169.00978514008708,
            "HighPrice": 169.00978514008708,
            "LowPrice": 169.00978514008708,
            "LastPrice": 169.00978514008708,
            "FirstPrice": 0,
            "Volume": 0,
            "InvertedVolume": 0,
            "Inverted": false,
            "MarketCap": 2525374784819.8413,
            "EPS": 5.46566917649556,
            "PERatio": 27.81760241146561,
            "Yield": 0.593260650413186
        }
    ]
}
```

### Example

```bash
curl -X "GET" \
"https://comsolotex.sologenic.org/api/aed/tickers?symbols=WyJzdXd1c2RjXzEtdGVzdGNvcmUxM3MybW1nZzR1dTRmbjhtdWU2czNsZ243NGp3ZHVwbmRqdHFhaDh1eHVmdWd0YWprZXEycWd6bmMyODpzdWFhcGxfMS10ZXN0Y29yZTEzczJtbWdnNHV1NGZuOG11ZTZzM2xnbjc0andkdXBuZGp0cWFoOHV4dWZ1Z3RhamtlcTJxZ3puYzI4Il0=&series=2" \
-H "Authorization: Bearer ..." \
-H "OrganizationID: ..." \
-H "Network: mainnet"
```

## Application start parameters

The application requires the following environment variables to be set:

* `AUTH_FIREBASE_SERVICE`: Included from `github.com/sologenic/com-fs-auth-firebase-service`. For setup see `github.com/sologenic/com-fs-auth-firebase-service/README.md`
* `AED_STORE`: Included from `github.com/sologenic/com-fs-aed-model`. For setup see `github.com/sologenic/com-fs-aed-model/client/README.md`
* `ASSET_STORE`: Included from `github.com/sologenic/com-fs-asset-model`. For setup see `github.com/sologenic/com-fs-asset-model/client/README.md`
* `USER_STORE`: Included from `github.com/sologenic/com-fs-user-model`. For setup see `github.com/sologenic/com-fs-user-model/client/README.md`
* `ROLE_STORE`: Included from `github.com/sologenic/com-fs-role-model`. For setup see `github.com/sologenic/com-fs-role-model/client/README.md`
* `HTTP_CONFIG`: Included from `github.com/sologenic/com-be-http-lib/http/`. For setup see `github.com/sologenic/com-be-http-lib/http/README.md`
* `FEATURE_FLAG_STORE`: Included from `github.com/sologenic/com-fs-feature-flag-model`. For setup see `github.com/sologenic/com-fs-feature-flag-model/client/README.md`
* `ORGANIZATION_STORE`: Included from `github.com/sologenic/com-fs-organization-model`. For setup see `github.com/sologenic/com-fs-organization-model/client/README.md`
