![](https://raw.githubusercontent.com/picksco/harmony/master/harmony-logo.png)

Standardized import / export format for cryptocurrency transaction ledgers.

## Motivation

Harmony takes the guesswork out of creating an export format for account history. This standard enhances interoperability between producers and consumers of these exports, making "CSV wrangling" a simpler task.

The basic requirements of the Harmony format are:

1. The source of the data is named somewhere within the file. For a single-source export, such as an exchange, this declaration can be made within the header. For exports containing history from multiple sources, such as from portfolio management tools, each row of the data must contain the venue for that transaction.
2. Transactions can be entered as single-entry, which makes trade history exports more readable; or double-entry, specifying the inbound and outbound amounts in separate columns.

## Objectives

The Harmony standard should be:

- Easily readable by a human
- Adaptable to different languages
- Versatile enough to account for most transactional use cases
- Upgradeable

## Who is this for?

- Exchanges
- Wallet providers
- Service providers

# The Format, Version 0.1

A contents of a Harmony file are in CSV format as described in [RFC 4180](https://tools.ietf.org/html/rfc4180). Implementors may look for language-specific CSV libraries that conform this RFC.

## Type Declaration

The first row of the document must contain one type declaration cell, which specifies the options for the file. The format of this cell is a space-separated list of options, the first being the string `Harmony`. This declaration may be safely placed as the last cell of the first row.

    Harmony (option)(value) (option)(value) ...

The options are as follows:

- **v** - The Harmony version of the document. Examples: `v0.1`, `v1`, `v1.2`
- **h** - The 1-indexed number of the column-definition row. The data content starts on the next row. Rows before this row are in the header area. Defaults to `h3`. Examples: `h1`, `h12`
- **a** - The accounting methodology used in the document. Valid options are `1` for Single-Entry and `2` for Double-Entry. Defaults to `2`. Examples: `a1`, `a2`
- **l-** - Language of the document, following [RFC 5646](https://tools.ietf.org/html/rfc5646). Defaults to `en`. Examples: `l-en`, `l-zh-CN`

### Examples

Version `0.1` file. Assumed defaults: column definitions on the third row of the document, double-entry accounting, English language.

    Harmony v0.1

Version `1.2` file with column definitions on the tenth row of the document. Single-entry accounting. Definitions in German.

    Harmony v1.2 h10 a1 l-de

## Header Declarations

The header area of the document is every row up to and including the row of column definitions. Within this area, each data provider may include arbitrary custom data. However, some values within this area are meaningful for this specification.

When a cell within the header area exactly contains one of the following strings, the cell immediately following on the same row will contain the value of the declaration.

- **Venue** - The source of all rows of data. Must be declared either here in the header or on each data row.
- **Exported** - The generation time stamp for this document.

## Required Column Definitions

- **Date & Time** - Time stamp of the entry, in [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) format.
- **Venue** - If not specified in the header declarations, the execution location of the transaction. This value overrides the header declaration if both are present.
- **Type** - Transaction type, enumerated. Supported types are: `buy`, `sell`, `deposit`, `withdraw`
- **Transaction ID** - Some value that will uniquely identify this transaction to the venue.

## Double-Entry Mode Required Column Definitions

- **Inbound Asset** - The asset being added to the account by this transaction.
- **Inbound Amount** - Quantity of Inbound Asset being added to the account. No negative values.
- **Outbound Asset** - The asset being removed from the account by this transaction.
- **Outbound Amount** - Quantity of Outbound Asset leaving the account. No negative values.

## Single-Entry Mode Required Column Definitions

- **Asset** - What asset is being transacted. For trades, this is the trading pair in the format `BASE/QUOTE`. For deposit and withdraw, the single currency.
- **Price** - The execution price for trades, as the amount of the `QUOTE` asset paid for each unit of the `BASE` asset
- **Amount** - How much of the `BASE` asset was transacted, non-negative.

## Optional Column Definitions

- **Fee Asset** - The asset being spent as payment for the transaction. For on-chain transactions, this may represent the network fee when the Network Fee Asset column is not defined.
- **Fee Amount** - Quantity of the Fee Asset being spent.
- **Network Fee Asset** - The asset being spent as payment for an on-chain transaction.
- **Network Fee Amount** - Quantity of the Network Fee Asset being spent.
- **Tax Asset** - The asset being used to pay taxes on the transaction.
- **Tax Amount** - Quantity of the Tax Asset being spent.
- **Total** - *Single-entry only* - The total amount of `QUOTE` asset involved in the transaction
- **Balance [ASSET]** - The amount of the ASSET in the account after this transaction takes effect. Files may include as many Balance columns as needed as long as they all represent different assets. Examples: `Balance USD`, `Balance BTC`
- **Network ID** - The on-chain transaction identifier.
- **From ID** - An identifier indicating from where the transaction originated, possibly an on-chain address.
- **To ID** - An identifier indicating from where the transaction was sent, possibly an on-chain address.
- **Trade Order Type** - The type of order execution method utilized for a given transaction. Examples: `market`, `limit`, `stop`, `auction`, `block`
- **Notes** - A general-purpose, unstructured field.

## Example File

    Harmony v0.1 h5
    Venue, mt-gox
    Exported, 2013-06-01 00:00:00 UTC
    
    Date & Time,             Type,     Outbound Asset, Outbound Amount, Inbound Asset, Inbound Amount, Transaction ID
    2011-03-01 00:00:00 UTC, deposit,  ,               ,                USD,           1000,           Deposit Wire 100
    2011-03-02 00:00:00 UTC, buy,      USD,            500,             BTC,           600,            123456
    2013-04-01 00:00:00 UTC, sell,     BTC,            250,             USD,           62500,          567890
    2013-05-01 00:00:00 UTC, withdraw, USD,            60000,           ,              ,               Withdraw Wire 200

## What's missing

- **Cost-basis reporting** - While cost-basis should be included as an optional column, should the identifier of the source funds spent be required alongside? There is an inherent issue with this however, in the event that more than one source, each with a different cost-basis, was used.
- **Derivatives**
- **Short Selling**
