# Introduction

This document serves as an introduction to the project architecture. This document is a prerequesite for maintaining and extending the library. 


`APL2XL` is an APL library that is used to export APL arrays as Excel Workbooks. Excel Workbooks are built upon the OOXML specification. Anyone intending to extend or maintain this library would do well to become familiar with OOXML, and the various "parts" as they are defined. [This page](http://officeopenxml.com/anatomyofOOXML-xlsx.php) contains sufficient information to get started.  

General Style Rules:

- .NET core's Zip library is the only acceptable dependency. All code which introduces further dependencies must be announced, and approved.
- `APL2XL` must be compatible with both `link` and `AcreTools`
- `APL2XL` extensions must be `(⎕IO ⎕ML)←0 1` compliant. 
- `APL2XL` functions must be written as Dfns.
- `APL2XL` must be importable irrespective of context.
- `APL2XL` must maintain a functional style unless not possible to do so. Object-oriented code should be considered undesirable. 
- All tests executed by `APL2XL/APLSource/Tests/RunAll.aplf` must pass before a merge request or a commit will be accepted into the master branch. 
- New features, extensions, modifications, or maintenance must maintain all preexisting functionality. 


## Project Layout
`APL2XL` is a collection of APL namespaces which can be imported into an existing namespace, making the library portable. `APL2XL/APLSource` contains the following folders:

|Folder|Description|
|---|---|
|Demos|Example usage of the library and API|
|Docs|documentation not ready to be published to the project wiki on github|
|Main|All APL code relevant to APL2XL|
|Tests|Functions used to test project code found in Main|

`./APLSource` also includes the `quadVars.apln` file which is used by `AcreTools` to set environment variables when the project is loaded into the namespace. 

### Demos
New features can be unintuitive for those unfamiliar with `OOXML`, therefore a folder containing demonstrations is included with the library. Should a user want to know how to use any part of the language, they should be able to find an example in the `Demos` directory. Any new feature added should contain a brief end-to-end demonstrating usage of said feature. 

### Docs
This folder should contain only backup of externally published documentation, or documentation that is under development not ready to be externally published or published onto the github project wiki. 

### Main
Main contains the following folders:
|Folder|Description|
|---|---|
|Build|Functions for compiling XML, and exporting XML to files|
|Utility|Generic functions not specific to the domain of XML generation for Excel Workbooks|
|XL|Functions for generating XML, and managing data before XML generation|
|Root Files|Public API|

### Build
Build will only contain functions which manage generating, writing, and zipping Excel workbooks. Build should be the only directory in the entire project which contains Tradfns. That isn't to say place all Tradfns here, but that the project expects new functions to be defined as dfns. 

When adding new functionality, you must call relevant `{Part}XML` and `{Part}PATH` files here. Most Parts have been added, but should there be a feature requiring an as yet unspecified part, See `Main.Build.CompileXML`.

### Utility
There exists only one function presently within the Utility directory. `hex` is a function found in the `dfns` workspace, and is placed here for convenience. 

### XL
If you are extending or maintaining this library, this is where new functionality is to be added, and where each part of an Excel Workbook directory is managed. The naming convention for files is as follows:

|Pattern|Description|
|---|---|
|{Part}PATH|Function must return the name of a file which will be used for writing XML for this OOXML part. These are functions to be called, since several OOXML parts require generating several files of specific names, such as worksheets. See `Main.XL.WSPATH`|
|{Part}XML|Function must return XML pertaining to be written to a single file, or a vector of XML to be written to multiple files. See `Main.XL.SSXML` for an example of a single files XML. See `Main.XL.WSXML` for an example of multiple XML file output. |
|{Part}Add|Function will manage data structures which will be used in the `Main.Build` phase of compiling XML files. These functions are defined when direct XML compilation is not possible, such as with the Shared Strings part, or the Styles part. If you are extending the library and have the need to track data accross a workbook like the Shared Strings part, or Styles, define a `{Part}Add` function as necessary. |
|{Part}REL|Function must add the relationships where necessary. OOXML builds relationships at 2 primary levels, the Workbook level, and the root level. The root builds relationships between the workbook and OOXML parts, and remains mainly static. The Workbook level relationships must contain pointers to all files. If you are building a feature which defines an as yet unspecified Part, you must create a `{Part}REL` function which will add it's list of relationships to the relevant REL data structures found in `Main.XL.WB`. See `Main.XL.RWAdd`.|
|{Part}{Other}|These are functions which relate to a given part, but are not generic accross all parts like previously described. `WSViews` builds the `<sheetViews>` tag for each Worksheet. `WSNames` returns the list of names of worksheets after they have been collected, but isn't necessary for other parts. |

Definitions:

|Part Abbreviation|Definition|
|---|---|
|App|XML which will be written to the `{outputFile}/docPropx/app.xml` file|
|Core|XML which will be written to the `{outputFile}/docProps/core.xml` file|
|CT|Content Types, XML which will be written to the `{outputFile}/[Content_Types].xml` file|
|RR|Rel Root, or Root level relationship Part, XML which will be written to the `{outputFile}/_rels/.rels` file|
|RW|Rel Workbook, or Workbook level relationship Part, XML which will be written to the `{outputFile}/xl/_rels/workbook.xml.rels` file|
|SS|Shared Strings, XML which will be written to the `{outputFile}/xl/sharedStrings.xml` file|
|Style|Style, XML which will be written to the `{outputFile}/xl/styles.xml` file|
|WB|Workbook, XML which will be written to the `{outputFile}/xl/workbook.xml` file|
|WS|Worksheet, XML which will be written to each of the `{outputFile}/xl/worksheets/sheet{n}.xml` files|

### Main/{Files}

The files contained within main that are not compartmented into other directories are intended to be the public API for this library.

### Main.Export

`Main.Export` is the primary public API function which allows a user to export an APL namespace as an Excel Workbook. Export loops over each range inside of each worksheet, and calls functions internal to `Main.XL` and `Main.Build` to convert a workbook Namespace into a `.xlsx` zip file. 

`Main.Export` defines a `⎕Signal` which in the event of any file read/write error will `⎕Trap` in `Main.Build.CompileXML`. 

### Main.PatternFills
Simple function which returns some example fill styles.

### Main.GetErrorMessage
This function should be moved to another directory, and is not part of the public API.

### Main.NextCol/Main.NextRow
These functions take a Range namespace as a right argument, and returns the next available row or column available after the argument in a given orientation.

### Main.NumberFormats 
Simple function returning example Number Formats.

### Main.quadVars.apln
Set global configuration for the `Main` namespace.

### Main/stylegiude.info
Some abbreviation defininitions, and should be useful in understanding abbreviated names throughout the project. 

### Tests/Utility
Contains helper functions for writing tests and test cases. There are a number of tests which use `OLEClient` for the purpose of opening Excel files created using Main.Export, and reading back the contents to verify output correctness. Several functions within `Test/Utility` encapsulate `OLEClient` so it doesn't pollute test code. `Tests.Utility.TestCase` is an operator used to wrap a test case in error protection. This could also be used to wrap reporting behavior in future. 

### Tests.RunAll
Run all test cases located in `APLSource/Tests`. Tests cases are files who's names begin with the word `Test` with correct casing, and which define Dfns which return a boolean vector result. 

Should you wish to define new test cases, create a file, who's name conforms with `Test{myfile}.aplf`, which returns a boolean vector result. Inside of your Dfn, define any number of functions, each of which should be passed as left operand of `Tests.Utility.TestCase`. Any arguments for your function can be passed as right argument to your new testcase function. 






