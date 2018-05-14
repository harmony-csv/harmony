![](https://raw.githubusercontent.com/picksco/harmony/master/harmony-logo.png)

Standardized import / export format for cryptocurrency transaction ledgers.

## Motivation

Harmony takes the guesswork out of creating an export format for account history. This standard enhances interoperability between producers and consumers of these exports, making "CSV wrangling" a simpler task.

The basic requirements of the Harmony format are:

1. The source of the data is named somewhere within the file. For a single-source export, such as an exchange, this declaration can be made within the header. For exports containing history from multiple sources, such as from portfolio management tools, each row of the data must contain the venue for that transaction.
2. Transactions are double-entry, specifying the inbound and outbound amounts in separate columns.

## The Format

A Harmony file follows the simple CSV format described in [RFC 4180](https://tools.ietf.org/html/rfc4180).

### Type Declaration

The first cell of the document declares the format and options for the file. The format of this cell is a space-separated list of options, the first being the string `Harmony`.

```
Harmony (option)(value) (option)(value) ...
```

The options are as follows:
* **v** - The Harmony version of the document. Examples: `v0.1`, `v1`, `v1.2`
* **h** - The 1-indexed number of the column-definition row. The data content starts on the next row. Rows before this row are in the header area. Examples: `h1`, `h12`
* **l-** - Language of the document, following [RFC 5646](https://tools.ietf.org/html/rfc5646). Defaults to `en`. Examples: `l-en`, `l-zh-CN`

#### Examples

Version `0.1` file with column definitions on the third row of the document. English language assumed.

```
Harmony v0.1 h3
```

Version `1.2` file with column definitions on the tenth row of the document. Definitions in German.

```
Harmony v1.2 h10 l-de
```

### Header Declarations

The header area of the document is every row up to and including the row of column definitions. Within this area, each data provider may include arbitrary custom data. However, some values within this area are meaningful for this specification.

When a cell within the header area exactly contains one of the following strings, the cell immediately following on the same row will contain the value of the declaration.

* **Venue** - The source of all rows of data. Must be declared in the header or on each data row.
* **Exported** - The generation time stamp for this document.

### Minimum Required Column Definitions

* **Date & Time** - Time stamp of the entry
* **Venue** - If not specified in the header declarations, the execution location of the transaction. This value overrides the header declaration if both are present.
* **Type** - Transaction type, enumerated. Supported types are: `buy`, `sell`, `deposit`, `withdraw`
* **Outbound Asset** - The asset being removed from the account by this transaction.
* **Outbound Amount** - Quantity of Outbound Asset leaving the account.
* **Inbound Asset** - The asset being added to the account by this transaction.
* **Inbound Amount** - Quantity of Inbound Asset being added to the account.
* **Transaction ID** - Some value that will uniquely identify this transaction to the venue.

## Example File

```
Harmony v0.1 h5
Venue, mt-gox
Exported, 2013-06-01 00:00:00 UTC

Date & Time,             Type,     Outbound Asset, Outbound Amount, Inbound Asset, Inbound Amount, Transaction ID
2011-03-01 00:00:00 UTC, deposit,  ,               ,                USD,           1000,           Deposit Wire 100
2011-03-02 00:00:00 UTC, buy,      USD,            500,             BTC,           600,            123456
2013-04-01 00:00:00 UTC, sell,     BTC,            250,             USD,           62500,          567890
2013-05-01 00:00:00 UTC, withdraw, USD,            60000,           ,              ,               Withdraw Wire 200
```
