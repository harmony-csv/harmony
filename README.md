![](https://raw.githubusercontent.com/harmony-csv/harmony/master/harmony-logo.png)

Standardized import / export format for cryptocurrency transaction ledgers.

## Motivation

Harmony takes the guesswork out of creating an export format for account history. This standard enhances interoperability between producers and consumers of these exports, making "CSV wrangling" a simpler task.

The CSV data format enjoys near-universal support by spreadsheet apps and programming languages alike. The text-only contents of a Harmony file are both human-readable and machine-parseable. File versioning makes writing libraries easier, as each can target a specific range of versions for support.

## Who benefits from Harmony?

- Exchanges
- Wallets
- Service providers
- Users combining data from multiple sources

# The Format, Version 0.2

A Harmony file is UTF-8 encoded plain text consisting of a structured header area followed by normal CSV data contents. The CSV specification in [RFC 4180](https://tools.ietf.org/html/rfc4180) stipulates a single header row followed by rows containing one record each. The Harmony format deviates from this specification by creating a multi-row header area separated from the data contents by a single blank line. The header area follows the spirit of the CSV spec by using rows and columns separated by line breaks and commas, but the number of columns in the header area is not required to match the number of columns in the data payload. Implementations may safely pass the column definitions and transaction data portion to any compatible CSV parsing library.

The document contents are, in order: a type declaration, some optional header declarations, one blank line, the column definitions, and the transaction data.

    HarmonyCSV v0.2                       # Type declaration row
    Provenance, Interchange               # Header declaration (optional)
                                          # Single blank line
    Timestamp,  Transaction ID, ...       # Column definitions
    2010-01-01, 1,              ...       # Transaction data
    2010-02-02, 2,              ...       # ...

## Type Declaration

The first row of the document must contain the type declaration in exactly one cell. The format of this cell is a space-separated list of options, the first being the string `HarmonyCSV`.

    HarmonyCSV (option)(value) (option)(value) ...

As of version 0.2, there is only one option:

- **v** - The Harmony version of the document. Examples: `v0.2`, `v1`, `v1.2`

## Header Declarations

The header area of the document is all rows prior to the single blank row above the column definitions. Within this area, each institution may include arbitrary custom data. However, some values within this area are reserved for this specification. None of these header declarations are required.

When a cell within the header area contains one of the following strings exactly, the cell immediately following on the same row will contain the value of the declaration.

- **Provenance** - The institution that produced the report. This could be the originitaing exchange or another service provider.
- **Period start** - The starting timestamp of data export range, [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) format. This is the earliest possible timestamp of the data contents. The document must not contain data earlier than this timestamp. Example: for a one-year export of 2017 transaction history, this would be `2017-01-01T00:00:00Z`.
- **Period end** - The timestamp of the latest possible data within the document, ISO 8601. The document must not contain data after this timestamp. Example: `2017-12-31T23:59:59Z`

## Blank Line

A single blank line separates the header and data portions of the file. This line is "blank" in the CSV sense, in that it contains no data in any cell. As such, the line may contain commas, double-quotes, and whitespace followed by CRLF, but it may not contain actual cell contents, or it will be interpreted as part of the header.

Example of a valid "blank" line separating the header from the contents:

    "HarmonyCSV v0.2",""
    "",""
    "Transaction ID","Timestamp"

This example file's contents more closely align with the spirit of the CSV spec, requiring every row the have the same number of columns, hence the empty cells in the first and second rows. Each cell is surrounded by double quotes as permitted by RFC 4180. The second row is "blank" in the sense that a spreadsheet program would display empty cells.

## Column Definitions

The following column names are reserved to the Harmony CSV format. Other columns may be included by exporters while still complying with the specification. However, later versions of Harmony may conflict with external definitions.

| Column              | Required | Description |
| :------------------ | :------- | :---------- |
| Timestamp           | Yes | Date and time of the entry, in [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) format |
| Venue               | Yes | The execution location of the transaction |
| Type                | Yes | Ledger entry type and optional sub-type. See further explanation and types [below](#entry-types). |
| Amount              | Yes | The change in the asset balance |
| Asset               | Yes | The asset affected by this entry |
| Transaction&nbsp;ID | Yes | Some value that will uniquely identify this transaction to the venue, like a trade ID. All ledger entries with the same Venue, Transaction ID, and Instrument will be treated as belonging to the same transaction. Non-trade entries will not have an Instrument column value, so the Venue+Transaction ID must uniquely identify a transaction. |
| Reference&nbsp;ID   | No  | An optional unique identifier for a transaction, specific to the exporter's system |
| Balance             | No  | Running balance of the Asset after this entry takes effect |
| Order&nbsp;ID       | No  | Identifier that ties together multiple transactions |
| Account             | No  | Non-changing venue-specific account identifier, useful for denoting sub-accounts |
| Instrument          | No  | For trades, the thing being traded, in the form `BASE/QUOTE`. Example: `BTC/USD` |
| Network&nbsp;ID     | No  | The txid for on-chain transactions. For UTXO chains like Bitcoin, this ID may be in the `hash:index` format, which denotes the input or output index of this line item within the transaction. |
| Address             | No  | The wallet address spending or receiving on-chain funds |
| Note                | No  | A general-purpose, unstructured notes field |

### Entry Types

Each row represents a different part of a transaction. The type of each of these entries determines how they will be treated for accounting purposes. Top-level types may be further refined by declaring sub-types, separated with a colon `:`. Some sub-types are reserved here, but otherwise, there are no limitations on sub-type declarations.

The format for entry types should be lowercase, hyphenated strings joined by colons, matching the Regular Expression `/^([a-z0-9]+([-_:]?[a-z0-9]+)*)$/`. An example entry type could be:

    type:sub-type:detail-type

The following are reserved types and sub-types:

- `expense` - Non-fee general expenditure
- `fee` - Funds paid to the institution or network for the transaction
  - `fee:network` - Transaction fees for cryptocurrencies. Example: Gas on an Ethereum transaction.
  - `fee:trade` - Trade fees
  - `fee:transfer` - Fees paid to traditional institutions. Example: wire transfer fee
- `income`
  - `income:air-drop` - Tokens added to existing holders of another asset
  - `income:reward` - Income earned through cryptocurreny mining activity
- `loss` - Funds lost. Can sub-type as `loss:hack` or `loss:password` to specify in more detail.
- `tax` - Tax payment transactions
- `trade` - The sale or purchase of a trading instrument.
  - `trade:buy` - Indicates a bid that was filled
  - `trade:sell` - Indicates an ask that was filled
- `transfer` - Movement of funds in or out of an institution or wallet
  - `transfer:deposit`
  - `transfer:withdrawal` - Amount column should be negative, denoting a reduction in asset balance.

### Example File

This example file shows the following account activity expanded into line items, with the optional Balance column showing the running total of the entry asset.

- Deposits 1000 USD into Coinbase via `Wire-100` wire transfer
- Buy 0.10 Bitcoin for 900 USD (effective price of $9000) in trade `123456`
- Sell 0.05 Bitcoin for 1000 USD (effective price of $20,000) in trade `567890`
- Withdraw 0.099 Bitcoin, paying 0.001 transaction fee in on-chain transaction `abc123`

```
HarmonyCSV v0.2
Provenance, Interchange, https://interchangehq.com/
Period start, 2019-05-01 00:00:00 UTC, Period end, 2019-05-30 23:59:59 UTC

Timestamp,             Venue,     Type,          Amount,  Asset,  Transaction ID, Balance, Network ID
2018-05-01T00:00:00Z,  coinbase,  deposit,       1000,    USD,    Wire-100,       1000,
2018-05-02T00:00:00Z,  coinbase,  trade:buy,     0.10,    BTC,    123456,         0.10,
2018-05-02T00:00:00Z,  coinbase,  trade:buy,     -900,    USD,    123456,         100,
2018-05-03T00:00:00Z,  coinbase,  fee:exchange,  -9,      USD,    123456,         91,
2018-05-03T00:00:00Z,  coinbase,  trade:sell,    -0.05,   BTC,    567890,         0.05,
2018-05-03T00:00:00Z,  coinbase,  trade:sell,    1000,    USD,    567890,         1091,
2018-05-03T00:00:00Z,  coinbase,  fee:exchange,  -10,     USD,    567890,         1081,
2018-05-04T00:00:00Z,  coinbase,  withdrawal,    -0.049,  BTC,    abc123,         0.001,   abc123:0
2018-05-04T00:00:00Z,  coinbase,  fee:network,   -0.001,  BTC,    abc123,         0,
```
