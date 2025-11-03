# Holdings service

The Holdings Service provides API interfaces that manages holding within an organization.

## API endpoints overview

**Authenticated:**

* GET `/api/holdings/get?denom=...` - retrieve a single holding for a user and denom
* GET `/api/holdings/list` - retrieve a list of holdings for a user, representing a user's portfolio
* GET `/api/holdings/graph?window=...&series=...` - retrieve portfolio performance over time as Asset Epoch Data (AEDs)

## GET `/api/holdings/get?denom=...`

Retrieves a single holding for a user and denom.

### Example request

```bash
curl -X GET \
"https://comsolotex.sologenic.org/api/holdings/get?denom=suaapl_1-testcore13s2mmgg4uu4fn8mue6s3lgn74jwdupndjtqah8uxufugtajkeq2qgznc28" \
-H "Authorization: Bearer ..." \
-H "OrganizationID: ..." \
-H "Network: mainnet"
```

### Example response

```json
{
    "UserID": "nick.luong@sologenic.org",
    "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
    "MetaData": {
        "Network": 2,
        "UpdatedAt": {
            "seconds": 1752596311,
            "nanos": 246733436
        },
        "CreatedAt": {
            "seconds": 1751895150,
            "nanos": 539847111
        }
    },
    "Denom": "suaapl_1-testcore13s2mmgg4uu4fn8mue6s3lgn74jwdupndjtqah8uxufugtajkeq2qgznc28",
    "Quantity": {
        "Value": 5
    },
    "CostBasis": {
        "Value": 92642,
        "Exp": -2
    },
    "Metrics": {
        "PortfolioPercentage": {
            "Value": 17,
            "Exp": -2
        },
        "CurrentValue": {
            "Value": 94080,
            "Exp": -2
        },
        "UnrealizedPnL": {
            "Value": 1438,
            "Exp": -2
        },
        "RealizedPnL": {
            "Value": -960,
            "Exp": -2
        },
        "AveragePrice": {
            "Value": 1852840,
            "Exp": -4
        },
        "CurrentPrice": {
            "Value": 18816,
            "Exp": -2
        }
    }
}
```

## GET `/api/holdings/list`

Retrieves a list of holdings for a user, representing a user's portfolio. The response is a `Portfolio` struct defined as follows:

```go
type Portfolio struct {
	UserID            string
	OrganizationID    string
	Network           metadata.Network
	Holdings          []*holdingsgrpc.Holding
	TotalMarketValue  shopdec.Decimal
	TotalCostBasis    shopdec.Decimal
	TotalUnrealizedPL shopdec.Decimal
	TotalRealizedPL   shopdec.Decimal
}
```

### Example request

```bash
curl -X GET \
"https://comsolotex.sologenic.org/api/holdings/list
-H "Authorization: Bearer ..." \
-H "OrganizationID: ..." \
-H "Network: mainnet"
```

### Example response

```json
{
    "UserID": "nick.luong@sologenic.org",
    "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
    "Network": 2,
    "Holdings": [
        {
            "UserID": "nick.luong@sologenic.org",
            "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
            "MetaData": {
                "Network": 2,
                "UpdatedAt": {
                    "seconds": 1752614489,
                    "nanos": 504990000
                },
                "CreatedAt": {
                    "seconds": 1752614489,
                    "nanos": 505001000
                }
            },
            "Denom": "suusdc_1-testcore13s2mmgg4uu4fn8mue6s3lgn74jwdupndjtqah8uxufugtajkeq2qgznc28",
            "Quantity": {
                "Value": 1,
                "Exp": 3
            },
            "CostBasis": {
                "Value": 1,
                "Exp": 3
            },
            "Metrics": {
                "PortfolioPercentage": {
                    "Value": 18,
                    "Exp": -2
                },
                "CurrentValue": {
                    "Value": 1,
                    "Exp": 3
                },
                "UnrealizedPnL": {
                    "Exp": 3
                },
                "RealizedPnL": {},
                "AveragePrice": {
                    "Value": 10000,
                    "Exp": -4
                },
                "CurrentPrice": {
                    "Value": 1
                }
            }
        },
        {
            "UserID": "nick.luong@sologenic.org",
            "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
            "MetaData": {
                "Network": 2,
                "UpdatedAt": {
                    "seconds": 1752609173,
                    "nanos": 956769000
                },
                "CreatedAt": {
                    "seconds": 1752609173,
                    "nanos": 956779000
                }
            },
            "Denom": "suwusdc_1-testcore13s2mmgg4uu4fn8mue6s3lgn74jwdupndjtqah8uxufugtajkeq2qgznc28",
            "Quantity": {
                "Value": 1,
                "Exp": 3
            },
            "CostBasis": {
                "Value": 1,
                "Exp": 3
            },
            "Metrics": {
                "PortfolioPercentage": {
                    "Value": 18,
                    "Exp": -2
                },
                "CurrentValue": {
                    "Value": 1,
                    "Exp": 3
                },
                "UnrealizedPnL": {
                    "Exp": 3
                },
                "RealizedPnL": {},
                "AveragePrice": {
                    "Value": 10000,
                    "Exp": -4
                },
                "CurrentPrice": {
                    "Value": 1
                }
            }
        },
        // ...
    ],
    "TotalMarketValue": {
        "Value": 569590,
        "Exp": -2
    },
    "TotalCostBasis": {
        "Value": 575530,
        "Exp": -2
    },
    "TotalUnrealizedPL": {
        "Value": -5940,
        "Exp": -2
    },
    "TotalRealizedPL": {
        "Value": -6648,
        "Exp": -2
    }
}
```

### How It Works

The graph endpoint calculates portfolio performance by:

1. **Getting Current Holdings**: Fetches the user's current asset holdings to determine which assets to track
2. **Fetching Market Data**: Retrieves daily closing prices for each asset in the user's portfolio
3. **Calculating Portfolio Value**: For each time bucket, multiplies current holdings by historical closing prices to get total portfolio value
4. **Returning AEDs**: Creates Asset Epoch Data entries representing portfolio value over time

### Response Format

Each AED in the response represents the user's total portfolio value at a specific point in time:

- **`Symbol`**: Format is `{baseCurrency}:{userID}_{organizationID}` (e.g., "USD:user123_org123")
- **`Timestamp`**: Start of the time bucket based on the requested window
- **`Value.Field`**: Always "CLOSE" representing the portfolio closing value for that window
- **`Value.Float64Val`**: Total portfolio value in base currency at that timestamp
- **`Series`**: Always "USER_PERFORMANCE" indicating this is user-specific performance data

### Use Cases

- **Portfolio Performance Charts**: Display how portfolio value changes over time
- **Performance Analytics**: Calculate returns, volatility, and other performance metrics
- **Historical Analysis**: Compare portfolio performance across different time windows
- **Risk Assessment**: Analyze portfolio value fluctuations for risk management

## Calculation Logic

The Holdings Service performs real-time calculations for financial metrics using data from multiple sources:

### Data Sources

- **Blockchain Wallet**: Current asset quantities (real-time)
- **Holdings Store**: Cost basis and historical trade data (stored)
- **AED Store**: Current market prices (real-time)
- **Data Warehouse**: Portfolio percentages (pre-calculated)

### Holding-Level Calculations

Each holding's metrics are calculated dynamically when requested:

```go
// From blockchain wallet
currentQuantity := walletAsset.SymbolAmount

// From holdings store  
costBasis := storedHolding.CostBasis

// From AED store (latest market data)
currentPrice := aed.Close // Latest closing price

// Calculated fields
currentValue := currentQuantity * currentPrice
totalReturn := currentValue - costBasis  
averagePrice := costBasis / currentQuantity
```

**Field Descriptions:**

- **`CurrentValue`**: `Quantity × CurrentPrice` - The current market value of the holding
- **`TotalReturn`**: `CurrentValue - CostBasis` - Unrealized profit/loss on the position
- **`AveragePrice`**: `CostBasis ÷ Quantity` - Average price paid per unit
- **`CurrentPrice`**: Latest market price from AED service
- **`PortfolioPercentage`**: `CurrentValue ÷ TotalPortfolioValue` - Percentage of this holding in the total portfolio

### Portfolio-Level Calculations

Portfolio totals are aggregated from all individual holdings:

```go
totalValue := sum(holding.CurrentValue for all holdings)
totalCostBasis := sum(holding.CostBasis for all holdings)
totalUnrealizedPL := totalValue - totalCostBasis
```

**Field Descriptions:**

- **`TotalMarketValue`**: Sum of all holdings' current values
- **`TotalCostBasis`**: Sum of all holdings' cost basis
- **`TotalUnrealizedPL`**: Total unrealized profit/loss across portfolio
- **`TotalRealizedPL`**: Realized gains/losses from completed trades (stored)

### Calculation Flow

1. **Get Wallet Assets**: Fetch current quantities from blockchain
2. **Get Stored Data**: Retrieve cost basis and portfolio percentages
3. **Get Market Prices**: Fetch latest prices from AED service
4. **Calculate Metrics**: Compute all derived fields server-side
5. **Aggregate Portfolio**: Sum individual holding values for portfolio totals

### Precision & Accuracy

- All calculations use `decimal.Decimal` for financial precision
- No floating-point arithmetic to avoid rounding errors
- Quantities adjusted for asset-specific decimal precision
- Prices formatted with appropriate decimal places

## GET `/api/holdings/graph`

Retrieves portfolio performance over time as Asset Epoch Data (AEDs). This endpoint returns the user's total portfolio value calculated at regular time intervals (buckets) over a specified date range.

### Query Parameters

- `window` (required): time range for historical data. Supported values depend on series type.
- `series` (optional): data series type. Defaults to `USER_PERFORMANCE`. Supported values:
  - `USER_PERFORMANCE`: Portfolio performance data (minimum 1 month window)
  - `INTERNAL_TRADES`: Internal trade data (minimum 1 day window)

#### Window Options by Series

**USER_PERFORMANCE Series:**
- `1mo`: 1 month  
- `3mo`: 3 months
- `6mo`: 6 months
- `1y`: 1 year
- `ytd`: year-to-date (from Jan 1st of the current year)
- `all`: all available data (up to 10 years)

**INTERNAL_TRADES (crypto) Series:**
- `1d`: 1 day (hourly buckets)
- `1w`: 1 week (daily buckets)
- Plus all USER_PERFORMANCE windows above

### Example request

```bash
curl -X GET \
"https://comsolotex.sologenic.org/api/holdings/graph?window=1mo&series=3" \
-H "Authorization: Bearer ..." \
-H "OrganizationID: ..." \
-H "Network: mainnet"
```

### Example response

```json
{
    "Portfolio": [
        {
            "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
            "Symbol": "suwusdc_1-testcore13s2mmgg4uu4fn8mue6s3lgn74jwdupndjtqah8uxufugtajkeq2qgznc28:nick.luong@sologenic.org_72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
            "Timestamp": {
                "seconds": 1750464000
            },
            "Period": {
                "Type": 3,
                "Duration": 1
            },
            "MetaData": {
                "Network": 2
            },
            "Value": [
                {
                    "Field": 4,
                    "Float64Val": 2000
                }
            ]
        },
        {
            "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
            "Symbol": "suwusdc_1-testcore13s2mmgg4uu4fn8mue6s3lgn74jwdupndjtqah8uxufugtajkeq2qgznc28:nick.luong@sologenic.org_72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
            "Timestamp": {
                "seconds": 1750550400
            },
            "Period": {
                "Type": 3,
                "Duration": 1
            },
            "MetaData": {
                "Network": 2
            },
            "Value": [
                {
                    "Field": 4,
                    "Float64Val": 2000
                }
            ]
        },
        // ...
    ],
    "Denoms": {
        "suaapl_1-testcore13s2mmgg4uu4fn8mue6s3lgn74jwdupndjtqah8uxufugtajkeq2qgznc28": [
            {
                "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
                "Symbol": "suwusdc_1-testcore13s2mmgg4uu4fn8mue6s3lgn74jwdupndjtqah8uxufugtajkeq2qgznc28:suaapl_1-testcore13s2mmgg4uu4fn8mue6s3lgn74jwdupndjtqah8uxufugtajkeq2qgznc28",
                "Timestamp": {
                    "seconds": 1750636800
                },
                "Period": {
                    "Type": 3,
                    "Duration": 1
                },
                "MetaData": {
                    "Network": 2
                },
                "Value": [
                    {
                        "Field": 4,
                        "Float64Val": 69305.8811425729
                    }
                ]
            },
            {
                "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
                "Symbol": "suwusdc_1-testcore13s2mmgg4uu4fn8mue6s3lgn74jwdupndjtqah8uxufugtajkeq2qgznc28:suaapl_1-testcore13s2mmgg4uu4fn8mue6s3lgn74jwdupndjtqah8uxufugtajkeq2qgznc28",
                "Timestamp": {
                    "seconds": 1750723200
                },
                "Period": {
                    "Type": 3,
                    "Duration": 1
                },
                "MetaData": {
                    "Network": 2
                },
                "Value": [
                    {
                        "Field": 4,
                        "Float64Val": 844498.1881709549
                    }
                ]
            },
            // ...
        ],
        "suamzn_1-testcore13s2mmgg4uu4fn8mue6s3lgn74jwdupndjtqah8uxufugtajkeq2qgznc28": [
            {
                "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
                "Symbol": "suwusdc_1-testcore13s2mmgg4uu4fn8mue6s3lgn74jwdupndjtqah8uxufugtajkeq2qgznc28:suamzn_1-testcore13s2mmgg4uu4fn8mue6s3lgn74jwdupndjtqah8uxufugtajkeq2qgznc28",
                "Timestamp": {
                    "seconds": 1750636800
                },
                "Period": {
                    "Type": 3,
                    "Duration": 1
                },
                "MetaData": {
                    "Network": 2
                },
                "Value": [
                    {
                        "Field": 4,
                        "Float64Val": 8232.345233867627
                    }
                ]
            },
            {
                "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
                "Symbol": "suwusdc_1-testcore13s2mmgg4uu4fn8mue6s3lgn74jwdupndjtqah8uxufugtajkeq2qgznc28:suamzn_1-testcore13s2mmgg4uu4fn8mue6s3lgn74jwdupndjtqah8uxufugtajkeq2qgznc28",
                "Timestamp": {
                    "seconds": 1750723200
                },
                "Period": {
                    "Type": 3,
                    "Duration": 1
                },
                "MetaData": {
                    "Network": 2
                },
                "Value": [
                    {
                        "Field": 4,
                        "Float64Val": 591.4655475771873
                    }
                ]
            },
            // ...
        ],
        "suusdc_1-testcore13s2mmgg4uu4fn8mue6s3lgn74jwdupndjtqah8uxufugtajkeq2qgznc28": [
            {
                "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
                "Symbol": "suwusdc_1-testcore13s2mmgg4uu4fn8mue6s3lgn74jwdupndjtqah8uxufugtajkeq2qgznc28:suusdc_1-testcore13s2mmgg4uu4fn8mue6s3lgn74jwdupndjtqah8uxufugtajkeq2qgznc28",
                "Timestamp": {
                    "seconds": 1750464000
                },
                "Period": {
                    "Type": 3,
                    "Duration": 1
                },
                "MetaData": {
                    "Network": 2
                },
                "Value": [
                    {
                        "Field": 4,
                        "Float64Val": 1000
                    }
                ]
            },
            {
                "OrganizationID": "72c4c072-2fe4-4f72-ae9d-d9d52a05fd71",
                "Symbol": "suwusdc_1-testcore13s2mmgg4uu4fn8mue6s3lgn74jwdupndjtqah8uxufugtajkeq2qgznc28:suusdc_1-testcore13s2mmgg4uu4fn8mue6s3lgn74jwdupndjtqah8uxufugtajkeq2qgznc28",
                "Timestamp": {
                    "seconds": 1750550400
                },
                "Period": {
                    "Type": 3,
                    "Duration": 1
                },
                "MetaData": {
                    "Network": 2
                },
                "Value": [
                    {
                        "Field": 4,
                        "Float64Val": 1000
                    }
                ]
            },
            // ...
        ],
        // ...
    }
}
```

### Data Points & Time Buckets

Each AED in the response represents the total portfolio value calculated for a specific time bucket within the requested time range. The bucketing strategy depends on the window parameter:

**Hourly Buckets (short-term windows):**
- **Windows**: `1d` (future crypto support)
- One data point per hour for 24 total points

**Daily Buckets (most windows):**
- **Windows**: `1w`, `1mo`, `3mo`, `6mo`, `1y`, `ytd`
- **Bucket Size**: Daily intervals (1 day each)
- **Response**: One AED per day showing portfolio value at the start of each day
- **Example**: `window=1mo` returns ~30 AEDs (one per day for the last month)
- **Example**: `window=1w` returns ~7 AEDs (one per day for the last week)

**Weekly Buckets (long windows):**
- **Windows**: `all`
- **Bucket Size**: Weekly intervals (7 days each, starting Monday)
- **Response**: One AED per week showing portfolio value at the start of each week
- **Example**: `window=all` returns weekly snapshots over up to 10 years of data

**Time Range Calculation:**
- **`1d`**: Last 24 hours from now (hourly bucket)
- **`1w`**: Last 7 days from today (daily bucket)
- **`1mo`**: Last 1 month from today (daily bucket)
- **`3mo`**: Last 3 months from today (daily bucket)
- **`6mo`**: Last 6 months from today (daily bucket)
- **`1y`**: Last 1 year from today (daily bucket)
- **`ytd`**: From January 1st of current year to today (daily bucket)
- **`all`**: Last 10 years from today (weekly bucket)

**Important Notes:**
- All timestamps are normalized to the beginning of the day (00:00:00 UTC)
- Portfolio values are calculated using market closing prices for each time bucket
- The system uses the user's current holdings to determine which assets to track historically
- Missing data points indicate windows where no portfolio value could be calculated (e.g., no market data available)

## Application start parameters

- `AED_STORE` - the asset exchange details service endpoint (included from `github.com/sologenic/com-fs-aed-model/`)
- `HOLDINGS_STORE` - the holdings service endpoint (included from `github.com/sologenic/com-fs-holdings-model/`)
- `USER_STORE` - the user service endpoint (included from `github.com/sologenic/com-fs-user-model/`)
- `ROLE_STORE` - the role service endpoint (included from `github.com/sologenic/com-fs-role-model/`)
- `FEATURE_FLAG_STORE` - the feature flag service endpoint (included from `github.com/sologenic/com-fs-feature-flag-model/`)
- `AUTH_FIREBASE_SERVICE` - the firebase authentication service endpoint (included from `github.com/sologenic/com-fs-auth-firebase-model/`)
- `ORGANIZATION_STORE` - the organization service endpoint (included from `github.com/sologenic/com-fs-admin-organization-model/`)
- `HTTP_CONFIG` - the configuration for the http server (included from `github.com/sologenic/com-be-http-lib/`)
- `BASE_CURRENCY` - the base currency configuration
