# Dev Checklist
In order to aid your learning and developing experience, this checklist is provided to guide you through developing within this project, while providing paths to resources when and as needed. As a contributor, it is expected that you understand OOXML and the concepts of SpreadsheetML. A useful introduction can be found here: http://officeopenxml.com/anatomyofOOXML-xlsx.php . Happy hacking!


This checklist should be followed and step 5 completed before contributions will be accepted and merged into the project. 
1. Read the Style Guide
2. Read the DevGuide document
3. When adding a new component
    - Add new property name to `Main/XL/WB.aplf`
    - Add new `Main/XL/<Component><Behavior>.aplf` for all behaviors ADD, PATH, REL, and XML
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
6. When strugging to find adequate information related to this project
    - Submit an issue to the github repository 