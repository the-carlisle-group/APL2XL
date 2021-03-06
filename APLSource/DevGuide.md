## Intent
The purpose of this document is to enable collaborative and open-source contributions to this project. Collaborators should be familiar with using `APL2XL` and OOXML before contributing. See the [README](../README.md) for more information about `APL2XL` usage. See the [DevGuide](./DevGuide.md) and the [StyleGuide](./StyleGuide.md) for more help getting started. 

## Contents

1. [Getting Started](#Getting-Started)
    - [Data Flow](#Data-Flow)
    - [From Entry Point to Excel Workbook](#From-Entry-Point-to-Excel-Workbook)
2. [Entry Point](#Entry-Point)
3. [Main.Export](#Main.Export)
4. [Main/XL](#Main/XL)
5. [Extending Sheet-Global Properties Of Worksheets](#Extending-Sheet-Global-Properties-Of-Worksheets)
6. [Extending Cell-Wise Properties](#Extending-Cell-Wise-Properties)
7. [Extending Shared Strings](#Extending-Shared-Strings)
8. [Extending Styles](#Extending-Styles)
9. [Extending Workbook/Root Relationships](#Extending-Workbook/Root-Relationships)
10. [Extending Themes/App/Core/[ContentType]/_rel (root)/Not Implemented](#Extending-Themes/App/Core/ContentType/rel-root/Not-Implemented)
11. [Build](#Build)
    - Build.CompileXML
    - Build.CreateXLSXDirectory
    - Build.WriteXML

### Build.Zip

## Getting Started
This document presumes that you understand OOXML and the concepts of SpreadsheetML. A useful introduction can be found here: http://officeopenxml.com/anatomyofOOXML-xlsx.php . This document will occasionally explain relevant concepts to OOXML, but mostly contains information on developing and extending `APL2XL`.

### Data Flow

`APL2XL` is designed as a stateless data flow tool, which accepts data formatted into namespaces, and outputs a `.xlsx` file. This format enables users to collect multiple namespaces intended to be converted to `.xlsx` files simultaneously without dealing with residual state. This meets a requirement of the original specification and must not be altered. `APL2XL` functions must not contain residual state that persists between uses of the tool. 

When using `APL2XL`, the user first creates `Range` namespaces. The `Range` namespaces is then added to a `Sheet` namespace. The `Sheet` namespaces can contain multiple ranges. The `Sheet` namespaces are then added to a `Workbook` namespace. Each of these types of namesapces contain variable names, which follow PascalCase convention of naming. Names should be full length names, with no abbreviations, and clearly correspond to the resulting behavior in some way. Variables can be assigned values for cells, styling for cell ranges, global properties of cells, sheets, or workbooks respectively, and can potentially be used for providing custom functions for customized processing if necessary in the future. 

When extending `APL2XL` it is likely that you will either be using the existing namespace variable definitions, or if additional data is required to add your feature, you can create additional names, and define the format of the data as necessary. When adding new names, [Main/XL/WB.aplf](./Main/XL/WB.aplf) must be updated with an empty representation or default values of the new name.. If the new data a cell-wise relationship in a `Range`, the data should conform to the same shape of the cell data. 

### From Entry Point to Excel Workbook
Once a user has collected their data into their respective workbooks, the user calls `Main.Export myWorkbook` to kick off the data flow. A `wb` is created as an intermediate data representation to collect data into their respective component parts. Functions contained in `Main/XL` of the format `<Component>Add.aplf` accept data for the respective component, and add data to the `wb` object. The `<Component>XML` files generate the necessary xml text based on the data contained with the `wb` object. 

`Main.Export` can be reduced to the following steps: 
1. Generate an empty `wb` intermediate data representation object
2. Transform the user data into intermediate data representation
3. Call `Build.CompileXML`.     
    - This function calls every `<Component>XML.aplf` function, gathers the file paths, and writes the generated XML to their respective files. 
    - Once the files are written, the containing temporary fold is zipped, and saved to a default path as a `.xlsx` file.




## Entry Point
The only public function available at this time is found in [Main/Export.aplf](./Main/Export.aplf).
```APL
     ⍵                                  ←→ namespace containing:
     ⍵.FileName                         ←→ the name to be written to, either relative or fully qualified
     ⍵.Sheets                           ←→ array of "worksheet" namespaces
     ⍵.Sheets[n]                        ←→ namespace containing:
     ⍵.Sheets[n].Name                   ←→ name of a worksheet found in a workbook
     ⍵.Sheets[n].Ranges                 ←→ array of value ranges and formatting information
     ⍵.Sheets[n].Ranges[n]              ←→ namespace containing:
     ⍵.Sheets[n].Ranges[n].NumberFormat ←→ apply numberformat to range
     ⍵.Sheets[n].Ranges[n].Value        ←→ values in the range
     ⍵.Sheets[n].Ranges[n].Address      ←→ address can be either indices i.e. ←→ 0 0 (at cell A1) or cell string i.e. ←→ 'A1' (at index 0 0)
```
`Main.Export` expects a namespace that contains the properties `FileName` and `Sheets`. `Sheets` is a vector of `Sheet` namespaces which require `Name` and `Ranges`. `Ranges` is a vector of `Range` namespaces which require `NumberFormat`, `Value`, and `Address`. This is not a complete list of all available options, but are a complete list of all required names in order to successfully process the namespace `⍵` into a `.xlsx` file. 

### Main.Export
`Main.Export` normalizes the incoming namespaces with a call to `##.Main.WB`. Any time `wb` is referenced from henceforth refers to the resulting data structure from this function. 

`Main.Export` defines 2 primary dfns. `sheets` and `ranges` are used to normalize and preprocess data so that it can be further processed without needing to bother with edge cases later on. Inside of both of the aforementioned dfns are calls to functions located within `Main/XL`. 

`Main/XL` contains all functions used for processing the provided data into XML text. 

### Main/XL
This folder contains a number of utility functions, which provide some quality of life benefits throughout the application. These functions should be used instead of using custom definitions inside `Main/XL` components. 
|Name|Use|
|---|---|
|lc| lower case normalize|
|UC| upper case normalize|
|WB| normalize the users input data for more effective process of component parts|
|S| a custom string formatter to make some string formatting easier to read|
|tag| simple xml tag generator. will be deprecated, and new features should use  ⎕XML|
|cti| cell to index. Convert a text cell address into APL indices 'A1' ←→ 0 0|
|itc| index to cell. Convert APL indices to text cell Addresses 0 0 ←→ 'A1'|

The reamining functions defined within `Main/XL` use the name of the relating OOXML component as prefix, followed by a description of the behavior. 

|Name |OOXML Component|
|---|---|
|App|docProps/app.xml|
|Core|docProps/core.xml|
|CT|outputFile/[Content_Types].xml|
|RR|_rels/.rels|
|RW|xl/_rels/workbook.xml.rels|
|Style|xl/styles.xml|
|SS|xl/sharedStrings.xml|
|Theme|xl/theme/theme1.xml|
|WB|xl/workbook.xml|
|WS|xl/worksheets/sheet{n}.xml where `n` is the sheet number|

Every component requires a `PATH`, a `REL`, and `XML`. Other names may be used or defined as necessary. For instance `WSProps` and `WSNames` are useful definitions to prevent unnccessary repetition throughout `Main/XL`. When defining a new component, or extending existing components, the following naming conventions are used. If you want to know where to look to extend existing behavior, begin here. 
|Description|Behavior|
|---|---|
|Add|Format the data to be processed in the corresponding `<Component>XML.aplf` function and Add  the data to `wb`|
|PATH| Return the Component file path|
|REL| Generate the data for the `rel` XML "part" of the corresponding `rel` file. For workbooks and global rels `_rels/.rels`. For Component parts within a workbook: `xl/_rels/workbook.xml.rels`|
|XML| Process the `wb` for the Componant, returning the final XML to be exported to file|

### Extending Sheet-Global Properties Of Worksheets
The `Main.Export` function calls the dfn `sheets` for each sheet in the namespace `⍵`. Any additional feature that requires extending Sheet-Global should normalize the representation of the required data inside this `sheets` function. The worksheet is then passed to the function [Main/XL/WSProps.aplf](./Main/XL/WSProps.aplf)

Extending Sheet-Global requires extension of the `WSProps` function. Portions of the `<sheetFormatPr>` "part" are order sensitive. Any modification should not break existing features, and should also respect the existing order of output of existing child components. 

### Extending Cell-Wise Properties
The dfn `sheets` located inside `Main.Export` calls the dfn `ranges` for each range in each sheet. Any additional feature that requires extending Cell-Wise properties requires extension [Main/XL/WSAdd.aplf](./Main/XL/WSAdd.aplf) and [Main/XL/WSXML.aplf](./Main/XL/WSXML.aplf). The primary property used for building worksheets is `wb.WS`. Shape is an `n 5` matrix, where each rows columns are defined as follows. `WSAdd` expects a vector of these values:
```APL
⍵[0] ←→ sheetname
⍵[1] ←→ addr, can be a range ex. 'A6:C12;D19;f30'
⍵[2] ←→ style (should be a reference to a workbook style)
⍵[3] ←→ cell value
⍵[4] ←→ is this cell a member of a merged range
```

|Function|Description|
|---|---|
|`WSAdd`|Add the incoming data from the `Main.Export` `ranges` dfn to `wb.WS`. Modification of [Main/XL/WSAdd.aplf](./Main/XL/WSAdd.aplf) is only necessary when custom preprocessing is required for the cell properties. Cell data should be normalized before being passed to `WSAdd`. For instance: Data Types must be converted to an Excel recognized data type based on business rules. This is handled within `WSAdd` so as not to clutter the `ranges` function. |
|`WSXML`|`WSXML` groups the data collected in  `wb.WS` by sheetname. `WSXML` performs array operations to format the cell properties for an entire sheet simultaneously. If new properties must be added or modified, the comments found within the function should help to guide you.|

### Extending Shared Strings
The shared strings component of a `.xlsx` file contains the count of each unique string found within an entire workbook, shared accross all worksheets. [Main/XL/WSAdd.aplf](./Main/XL/WSAdd.aplf) calls [Main/XL/SSAdd.aplf](./Main/XL/SSAdd.aplf) in order to collect all unique strings, and returns the index of unique strings. `SSXML` generates the xml for `xl/sharedStrings.xml`. This behavior is currently very simple, and requires little modification. 

### Extending Styles
For each style part, there exists an id, and for each unique combination of those styles, there exists a record in the `wb.styles.cellXfs` table relating a collection of style ids to that unique combination. Additionally, there is a table/vector for each style type. Number Format, Fill, Border, and other styles all have their own dedicated table/vector which contains the cell map for that style type for each cell.

If a cell is styled to be Red and Bold, that is one unique combination. If another cell is ONLY Red, that is another unique combination. Each cell inside a worksheet contains an id related to a record in the `wb.styles.cellXfs` table which identifies the unique combination of all styles for that cell. 

|Function|Description|
|---|---|
|`Main/XL/StyleAdd`|This function is called from within `WSAdd` and generates the content for the `cellXfs` table. This table collects unique styles for each worksheet. The function returns indices to these styles. |
|`Main/XL/StyleXML`|Several formatter functions are defined for each different part of a single style. A single style is comprised of a sequence of indices for each part of a style. The `cellXfs` table contains a record for each unique combination which is assigned and ID. This function transforms style table into XML, and also generates the required XML for the `cellXfs` table.|

### Extending Workbook/Root Relationships
If your new component has a woorkbook or root level relationship, it must be added to the respective relationship file. Add your `<Component>REL.aplf` call inside of [./Main/XL/RWRelXML.aplf](./Main/XL/RWRelXML.aplf) to ensure that the XML returned from your function is added to the workbook level rel file to be generated.

Root relationships are not implemented as at present, only default relationships exist. 

It is not anticipated that the `RWAdd` should be modified at this time.


### Extending Themes/App/Core/ContentType/rel root/Not Implemented
These components simply return default xml content for each component file because the files are required to compile.


## Build
When adding new components, `Build.CompileXML.aplf` must call the new XML generation function for that component, along with a PATH function which returns the path to that file. 

The remainder of the Build functions are not anticipated to require extension for the life of this project. For the sake of completeness, descriptions of the Build functions are included in this document. 

### Build.CompileXML
Call each XML and PATH functions for each component. This function generates a temporary directory, calls `Build.WriteXML`, `Build.Zip`, and then removes the temporary directory. 

### Build.CreateXLSXDirectory
Create the folder hierarchy within the temporary directory in which to place the `.xml` files.

### Build.WriteXML
Write XML text to a single output file. 

### Build.Zip
Contains the only external dependency of the project. This function imports the `.NET CORE` compression library, zips the temporary directory once all `.xml` files have been written.

