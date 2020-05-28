# APL2XL 
APL2XL is a Dyalog APL library which exports APL arrays to Excel in the form of a .xlsx file. This library is a work in progress. 

To use the library, run the following git command from a known location:

```
git clone https://github.com/the-carlisle-group/APL2XL.git
```

From Dyalog
```
    ]link.create # 'c:\path\to\APL2XL\APLSource'
```
The rooth namespace `#` should now contain the `Main` namespace, and you can now follow the example usage below. 


## Example Usage
Other examples can be found in the `./APLSource/Demos/` directory of this project. 

Workbooks consist of Worksheets, and Worksheets consist of Ranges. Ranges, Worksheets and Workbooks are simple to define as Namespaces, and include the following properties

|Namespaces|Required Variables|Optional Variables|
|---|---|---|
|Ranges|Value, Address|NumberFormat, Font, Fill, Border, Address|
|Sheets|Name, Ranges|FreezePane|
|Workbook|Sheets, FileName||

Define range comprised of values starting at an address, and optional formatting. **Note: Currently, APL2XL assumes the shape of the namespace.Value as the shape of the data in Excel.**  This means that `range.Value←'simple depth 1 character vector'` will place each character into a single cell accross the row starting at `range.Adress`. If you would like to place a character vector into a single cell, enclose the character vector. Ex. `range.Value←⊂'this will occupy a single cell'`
```APL
⎕IO ⎕ML←0 1
r1             ←⎕NS''
r1.Value       ←⍪10000+⍳5
r1.NumberFormat←⊂'#,##0'           ⍝ 10,000
r1.Address     ←2 3               ⍝ or 'B3'

r2             ←⎕NS''
r2.Value       ←⍪⍳10
r2.NumberFormat←⊂'m/d/yyyy'
r2.Font        ←⊂11 1 'Broadway' 5         ⍝ enclosed size color name family
r2.Fill        ←⊂'solid' 'ff6699' '33cc33' ⍝ enclosed style foregroundColor backgoundColor
r2.Border      ←⊂5⍴(⊂'thin' '0000cc')      ⍝ enclosed 5 element vector of tuples containing ('thickness' 'hexcolor')       
r2.Address     ←'C1'
```

Place the Ranges into a Worksheet
```APL
s1←⎕NS''
s1.Name  ←'Sample1'
s1.Ranges←r1 r2
⍝ Optional property: sheet.FreezePane examples
⍝ s1.FreezePane 1 'rows'
⍝ s1.FreezePane 5 'columns'
```

Add the sheet to a workbook with a fully qualified path. Multiple worksheets can be included as a list, `wb.Sheets←s1 s2 s3...` **Note: path cannot be of the form `c:\{filename}` due to writing privileges.** Select a path of the form `c:\myfolder\myfile.xlsx`

```APL
wb←⎕NS''
wb.Sheets  ←s1 
wb.FileName←'c:\{path}\myfile.xlsx'

Main.Export wb
```

## Styling and Formats
This table is an exhaustive list of currently implemented styling features, and known missing style features. Existing styling currently applies styles to cells that exist in the range. Styles more granular than the cell level are not currently implemented. For specific information pertaining to a Style found in this table, see the following sections. 

|Style|Supported|Value|Usage|Note|
|---|---|---|---|---|
|Number Format|x|ExcelNumberFormat:'Character Vector'|range.NumberFormat←⊂'m/d/yyyy'||
|Font|x|Size:Integer Color:Integer Name:'Character Vector' Font-Family:Integer|range.Font←⊂1 1 1 'Broadway' 5||
|Fill|x|Pattern:'Character Vector' ForegroundColor:'HEXColor' BackgroundColor:'HEXColor'|range.Fill←⊂'solid' 'ff6699' '33cc33'||
|Border|x|5⍴(⊂Thickness:'Character Vector' Color:'HEXColor')|range.Border←⊂5⍴('thin' 1)('thick' '0000cc')||
|String Formatting| | | | Not Implemented|
|Table| | | | Not Implemented|

### Number Formats
Number Formatting is the simplest feature in this list. If the number format formula works in Excel, it will work here. 
|Example Formats|
|----|
|#,##0 |
|#,##0.00 |
|#,##0_);(#,##0) |
|#,##0_);\[Red\](#,##0) |
|#,##0.00_);(#,##0.00) |
|#,##0.00_);\[Red\](#,##0.00) |
|$#,##0_);($#,##0)|
|$#,##0_);\[Red\]($#,##0)|
|$#,##0.00_);($#,##0.00)|
|$#,##0.00_);\[Red\]($#,##0.00)|
|0%|
|0.00%|
|0.00E+00|
|##0.0E+0|
|# ?/?|
|# ??/??|
|m/d/yyyy|
|d-mmm-yy|
|d-mmm|
|mmm-yy|
|h:mm AM/PM|
|h:mm:ss AM/PM|
|h:mm|
|h:mm:ss|
|m/d/yyyy h:mm|
|mm:ss|
|mm:ss.0|
|@|
|\_($\* #,##0\_);\_($\* (#,##0);\_($\* "-"\_);\_(@\_)|
|\_(\* #,##0\_);\_(\* (#,##0);\_(\* "-"\_);_(@\_)|
|\_($\* #,##0.00\_);\_($\* (#,##0.00);\_($\* "-"??\_);\_(@\_)|
|\_(\* #,##0.00\_);\_(\* (#,##0.00);\_(\* "-"??_);\_(@\_)|

### Font
Font names and sizes are self-explanitory. What is not well understood is font-family<integer>.

Respecting Font Family, this number is absolutely required to match the font name in some way. It is unclear yet how to determine this. An urgent feature should be added to match common names with their appropriate font families. In the interim, simply follow these steps:

1. In Excel, create a single spreadsheet with a single cell value at cell 'A1'
2. Change to the font you wish to use
3. Save the Excel file somewhere you can find it
4. Use 7zip (or some other user zip library) to unzip your workbook
5. Open {workbook}/xl/style.xml and paste its contents into an [xml prettifier](https://www.samltool.com/prettyprint.php) to view its contents
6. Find the value `n` in the fonts tag `<fonts>...<font>...<family val="n">` that pertains to the font you selected
7. Use that value in your font definition: `range.Font←12 1 'Font Name' n` 

### Border
See `./APLSource/Demos/Chess.aplf` for an example of how to specify borders. 

Borders 5 element list relates to Left, Right, Top, Bottom, and Diagonal. Each border position is a tuple containing the type of border, and the color. There are many possible border types, although, not all have been documented. 

Currently documented values for border types
|Border Types|
|---|
|thick|
|thin|
|none|


### Fill
A ranges fill value is a 3 element vector: patternfill foreground background. Foreground and background are defined as the hexcode for the color you want. There are many patternfill types, although not all have been documented yet.

|Patternfill Types|
|----|
|none|
|solid|
|gray125|
|darkGray|
|mediumGray|
|lightGray|
|gray125|
|gray0625|
|darkHorizontal|
|darkHorizontal|
|darkVertical|
|darkDown|
|darkUp|
|darkGrid|
|darkTrellis|
|lightHorizontal|
|lightVertical|
|lightDown|
|lightUp|
|lightGrid|
|lightTrellis|




### Selecting Colors
Colors can be selected by copying the #FFFFFF color using a color picker [like this one.](https://www.w3schools.com/colors/colors_picker.asp)


Additional questions may be directed to nathan@dyalog.com