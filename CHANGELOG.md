# Change Log

## Version 0.2

Consolidate both "single-entry" and "double-entry" modes into a single ledger format that breaks out the individual parts of a transaction into separate lines.

Specify that the document must be UTF-8 encoded.

Removes support for multiple languages in the header definitions.

Update column definition location to be after a single blank line, rather than declared in the type declaration.

Add Provenance header declaration.

Rename "Date & Time" column to "Timestamp".

Refinement of entry types with more definition on custom types.

Remove Exported from header declarations in favor of "Period start" and "Period end," which denote the export range of the data.

Clarification that Harmony CSV does not conform entirely to the RFC 4180 specification.


## Version 0.1

Initial specification.
