# Dev Checklist
In order to aid your learning and developing experience, this checklist is provided to guide you through developing within this project, while providing paths to resources when and as needed. As a contributor, it is expected that you understand OOXML and the concepts of SpreadsheetML. A useful introduction can be found here: http://officeopenxml.com/anatomyofOOXML-xlsx.php . Happy hacking!


To gain the most benefit from this checklist, it is recommended that you follow it as a procedure, in order. Step 5 must be completed before contributions is accepted and merged into the project. 
1. Read the Style Guide
2. Read the DevGuide document
3. When adding a new component
    - Add new property name to `Main/XL/WB.aplf` to process user input as an intermediate data representation
    - Add new `Main/XL/<Component><Behavior>.aplf` for all behaviors ADD, PATH, REL (if necessary), and XML
    - Transform the data in `ADD`, and compile the data to XML inside `<Component>XML.aplf`
    - Call your new `<Component>ADD.aplf` where necessary from within the existing data flow
    - Call your new `<Component>XML.aplf` from within `Build/CompileXML.aplf`
    - If your component has a relational part for the workbook .rels file, call your `<Component>REL.aplf` function from within the `RWXML.aplf` function. All `<Component>REL.aplf` files call `RWADD` in order to add the data to the workbook level .rels component XML.

4. When extending an existing component
    - Look to the `Main/XL/<Component><Behavior>.aplf` file related to your task
    - Read the comments.
    - READ THE COMMENTS.
    - *READ THE COMMENTS.*
    - ***READ THE COMMENTS!*** 
    - Refer to the DevGuide document related to `Main/XL/<Component><Behavior>`
    - Integrate new behavior into existing data flow
    - Add new behavior if necessary, and then integrate
5. Before Committing
    - Add comments in the existing style. Comments should leave nothing to interpretation
    - Add a new section to the DevGuide document
    - Add a Demo demonstrating usage of the new behavior
6. If all else fails
    - Submit an issue to the github repository 