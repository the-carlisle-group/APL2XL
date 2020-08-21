# Style Guide
This file documents language and naming in the rest of the application

## General Principles
1. A primary requirement of this project is that there be as few external dependenceies as possible. The `.NET CORE` Zip library is the only admissible external dependency. 
2. All functions outside the `Build` directory must not cause mutation to the global scope. Generally mutation of objects outside of scope is considered bad style. 
3. Adhere to functional style where possible unless a reasonable justification is provided. 
4. Comments, and user and developer documentation should be provided for all new behavior. No changes will be accepted unless this requirement is met. Comments should be full and complete, and adhere to existing comment style.
5. The present commenting style uses `⍝.` to denote paragraphs to visually chunk comments in order that single line comments don't get confused with continued comments.  `⍝` may be used for single line isolated comments, or end of line comments.

## Domain Language:
|Name|Meaning|
|---|---|
|UC | upper case|
|lc| lower case|
|cti|cell to index conversion 'A1' ←→ 0 0|
|itc |index to cell conversion  0 0 ←→ 'A1'|
|S   |string formatting function|
|col | convert data type of an xml <c></c> tag|
|tag | create an xml tag|
|wb  | the internal representation of worksheet data|

## Components
These are the conventions used when referring to data and functions pertaining to these files.
When creating a workbook, there will be compenent properties such as `wb.WS` or `wb.SS` for each component. These properties are the data which is used to generate xml for these functions.
|Abbreviation| Meaning | Component Path|
|---|---|---|
|RR |Rels Root    |  ./_rels |
|RW| Rels Workbook|  ./xl/_rels/workbook.xml.rels|
|SS | SharedStrings|  ./xl/sharedStrings.xml|
|WB | Workbook|       ./xl/workbook.xml|
|WS|   Worksheet|      ./xl/worksheet.xml|

***NOTE: All components must minimally implement `Add`, `XML`, and `PATH` functions, and should include a `REL` function as necessary.***
|Behavior|Description|
|---|---|
|Path|Return the string path to the file|
|XML| Return a single string of XML to be written to file|
|Add| Add the necessary data to `wb` for future processing of the component part. Return data varies. Typically returns the `wb` after `wb` has been ammended.  |
|REL| Return the `rel` content for the respective `rel` file|


## DFNS over TRADFNS
All library functions should be defined as DFNS. All code that communicates outside of APL are defined as TRADFNS in order to handle mutation, file writing, etc. It is expected that all future functions should be defined as DFNS. 


## DFN Format
It is common to require multiple helper and formatting functions within a dfn. The format of a dfn within this project should be adhered to as necessary. See [Main/XL/WSAdd.aplf](./Main/XL/WSAdd.aplf) or [Main/XL/StyleXML.aplf](./Main/XL/StyleXML.aplf) for examples. 

```APL
comments - the leading line of a multi-line dfn must minimally include a definition of the arguments. See Main/Export.aplf for an expample. Additional comments are appreciated.

consts - all constant variale definitions

helpers - small one liner, or general purpose dfns used throughout this dfn

formatters - dfns used for formatting data (as in StyleADD)

business logic
```