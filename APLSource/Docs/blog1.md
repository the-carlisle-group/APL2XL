## Hello, APL2XL
As many of you may be aware, Paul Mansour has sponsored the development of an open source APL to Excel conversion library. First presented at the Dyalog User Meeting 2019 by Nathan Rogers, the project can now be easily obtained from Github. Because `APL2XL` is an open source library, anyone may utilize the library for their Dyalog projects, or contribute to the project's development by modifying the project and submitting pull requests through Github. Future blog posts will provide complete documentation for contributing to `APL2XL`, but this blog post will serve as an introduction for users of the library. 

## Where to find help
If you experience any errors or issues when following this article, or when using the library, please use the [github repository issues page](https://github.com/the-carlisle-group/APL2XL/issues) to log any issues, feature requests, or questions. 

## Downloading and Installing
It is recommended that you install Git, and the Git toolset to download and manage repositories from Github, but no knowledge of Github is necessary to obtain and use this project. To help users with little or no knowledge of Github acquire and utilize `APL2XL` this section will be broken down into 2 sections. Section 1 will be for those who don't know how, or don't wish to use Git. Section 2 will cover how to acquire this library through Git. To continue without Git [Click Here](#Download-and-install-without-Git), To continue using Git [Click Here](#Download-and-install-with-Git)

### Download and install without Git
1. Visit the [APL2XL Repository Page on Github](https://github.com/the-carlisle-group/APL2XL). 
2. On the right had portion of the screen, locate a green button labeled "Clone or Download ▼".  A context menu will appear.
3. Within the context menu, Click "Download Zip".
4. Locate the downloaded folder, and using a zip library, such as (7zip)[https://www.7-zip.org/], right click the file, and unzip its contents.
5. Place the output directory containing the unzipped library somewhere that is easy to locate. I placed mine in `C:\Users\{user}\Documents\APL2XL`
6. With a Dyalog session open, type the following user command `]link.create # 'C:\Users\{user}\Documents\APL2XL\APLSource'` (be sure to replace {user} with the appropriate username)
7. Test that the library is working by running a Demo:
```APL
  Demos.Chess''
C:/Users/{user}/AppData/Local/Temp/chess.xlsx
```
8. Exporting a workbook using APL2XL returns the output file name. Use `]open C:/Users/{user}/AppData/Local/Temp/chess.xlsx` to open the output file. 
9. You can now create your own workbooks. Skip to the [Exporting example](#Exporting-a-simple-Workbook).


### Download and install with Git
There are 2 ways you can manage a git repository. One approach is to clone a repository to your local machine using the repository as a standalone project, or if you already have a git project in which you wish to nest the another git repository, you can begin by deleting the `.git` folder found within the new dependency repository, and move the directory into your existing repository. The second approach is to use git submodules, designed specifically for managing dependency repositories. The second approach is the recommended approach for managing nested repositories, such as dependencies, but an example of installing `APL2XL` using git submodules is beyond the scope of this blog post, and should probably be the topic of a future blog post. If you wish to look into using git submodules for your git projects, [this article](https://labvolution.com/using-git-submodules/) is a useful and simple introduction to the subject. 

#### Clone and install the repository
1. Open a terminal prompt (linux or windows)
2. `cd` to a known directory where you would like to clone `APL2XL
3. Run the following command: `c:\Users\{user}\Documents> git clone https://github.com/the-carlisle-group/APL2XL.git` 
4. With a Dyalog session open, type the following user command `]link.create # 'C:\Users\{user}\Documents\APL2XL\APLSource'` (be sure to replace {user} with the appropriate username)
5. Test that the library is working by running a Demo:
```
  Demos.Chess''
C:/Users/{user}/AppData/Local/Temp/chess.xlsx
```
6. Exporting a workbook using APL2XL returns the output file name. Use `]open C:/Users/{user}/AppData/Local/Temp/chess.xlsx` to open the output file. 
7. You can now create your own workbooks. Skip to the [Exporting example](#Exporting-a-simple-Workbook).

If you intend to use `APL2XL` as a library within an existing project, and your project is also a git repository, you may wish to place the `APL2XL` directory within your git repository. This isn't possible due to nesting git repositories. If you wish to do so anyone, you **can** follow the next step, but it is recommended instead that you use a git submodule instead. The drawback of manually installing a git repository as follows is that you remove the repository metadata, making the repository just a dumb directory. In other words, if when there are updates to `APL2XL`, you will need to manually cloan and move the directory, overwriting the old one, as in the next example. If you use a subdirectory, you can simply perform a `git pull` as you would with any git repository. The following is just for completeness of this example. Please look into git submodules if you need to nest this project within another repository.

Within every git repository is a `.git` directory in the root of that repository. 
1. With a prompt open, `cd` to the `APL2XL` repository
2. (linux) Run `rm -rf ./.git` 
3. (windows) Run 
```
rmdir /s .git
.git, Are you sure (Y/N)? Y
```
4. `mv` the `APL2XL` directory into your project where necessary


## Exporting a simple Workbook
Now that you have tested the library by successfully running one of the included Demos, it's time to understand how to export your own Excel Workbooks. To Export, we first begin by defining a required set of namespaces, required or optional variables within the namespaces, and passing the collection of namespaces along the Export function. 

Required Namespaces:
- Workbook
- Sheet
- Range


### Creating a simple Range Namespace
A Range namespace is a conceptual value within a single Excel Worksheet. Ranges consist of a namespace containing the following variables:
|Name|Description|Required|
|---|---|---|
|Address|Cell Address: Ex. `'A1' 'B4' 'CZ88'`|✓|
|Value|The value seen inside the cell beginning at the Address, with the ⍴Value. The values contained must conform to `∧/{(≡⍵)∊⍳2}¨Value`. Only numerical and textual data are supported at the time of this writing, and only arrays that conform to `2≤≢⍴Value`. |✓|
|NumberFormat|NumberFormat properties for the Range|✗|
|Border|Border properties for the Range|✗|
|Fill|Fill style properties for the Range|✗|
|Font|Font style properties for the Range|✗|

This example demonstrates the minimum required variables for a Range namespaces, to help you get a sense for exporting 
```APL
    range1←⎕NS''
    range1.Address←'A1'
    range1.Value←(20 2⍴'Hello' 'APL2XL'),⍳20 
```

### Creating a simple Sheet Namespace
Worksheet namespaces contain all of the ranges which will define the cell contents of a worksheet, the name of a worksheet, and other optional configuration variables. 

|Name|Description|Required|
|---|---|---|
|Name|Name for the worksheet|✓|
|Ranges|Vector of Range namespaces|✓|
|FreezePane|Configure freeze pane settings |✗|

```APL
    sheet1←⎕NS''
    sheet1.Name←'Example Worksheet'
    sheet1.Ranges←range1
```

### Creating the Workbook Namepsace
Workbook namespaces contain all of the sheets which will define the the list of worksheets contained within a workbook, and the file name of a workbook.

|Name|Description|Required|
|---|---|---|
|FileName|Name for the workbook. A fully qualified path is required. If only the file name is provided, it will write the output file to the current directory in which Dyalog is running. |✓|
|Sheets|Vector of Sheet namespaces|✓|
```APL
    workbook←⎕NS''
    workbook.FileName←'MyFile.xlsx'
    workbook.Sheets←sheet1
```

### Export
There must be at least one Range in a Sheet, at least one Sheet in a Workbook, and at least one Workbook in order to successfully Export. When all combined, you should be able to run the complete example by pasting the following code block into a Dyalog session. First clear your workspace the paste the following (***NOTE:* be sure to use your own local paths for lines 2 and 3 in this example!!!!)**.

```APL
    output_filename←'C:\Users\{user name}\Documents\MyAPL2XLFile.xlsx'
    ]link.create # 'C:\{path to your local repository}\APL2XL\APLSource'
    range1←⎕NS''
    range1.Address←'A1'
    range1.Value←(20 2⍴'Hello' 'APL2XL'),⍳20 
    sheet1←⎕NS''
    sheet1.Name←'Example Worksheet'
    sheet1.Ranges←range1
    workbook←⎕NS''
    workbook.FileName←output_filename
    workbook.Sheets←sheet1
    Main.Export workbook
```

Use `]open` on the ouput of Main.Export to view the output file in Excel. An error you're likely to encounter early on is when attempting to run `Main.Export` several times on the same workbook. If you've already run `Main.Export`, and have opened the workbook file, make sure that the workbook file is closed before attempting to Export the same workbook again. Congratulations, you've just created your very first workbook! 

For more complex examples, please take a look at the Demos folder included with the repository. There are examples for how to accomplish complex borders in `Demos.Chess`, there's an example of several worksheets within `Demos.MonthlyCaldendars`, and you can find an example of using the FreezePane settings in `Demos.FreezePane`. You can also take a look at the [github repository ReadMe](https://github.com/the-carlisle-group/APL2XL/blob/master/README.md#Example-Usage) for complete documentation on styles, and the format of style arguments. Additional documentation can also be found on the [github repository wiki](https://github.com/the-carlisle-group/APL2XL/wiki), as more documentation will be added over the lifespan of the project. As previously stated, if you have any issues using the library, please use the [github repository issues page](https://github.com/the-carlisle-group/APL2XL/issues) to log any issues, feature requests, or questions. 



