## Metadata on Projects and Datasets

Once a project is started [(see how)](../starting_a_new_project), it is time to add metadata.

Metadata can apply to three levels:

1. RIVM level: General metadata applying to all datasets or projects in the RIVM. This metadata is automatically harvested by the metadata catalogue
2. Project level: General metadata applying to all projects, for example project responsible.
3. Project Specific level: Specific metadata only for your project.

There are two types of metadata on a project, with two corresponding tabs in the Projects page. Both pages display the available metadata schemata on the left and the metadata editor tool on the right of the screen.

They can be found in the Project Details page in 2 tabs:
![](../../!assets/images/project_metadata_tabs.png){width=675 height=165}

> Note: Only **Managers** of a project can add or edit metadata. <br/>     Entered metadata will be visible for all users of iDMS, so also users that are not a project user.

### Project Metadata
Project metadata is metadata about the project. It can be used for making your project better findable, or [FAIR](https://www.go-fair.org/fair-principles/).
The **rivm** schema contains metadata used in the metadata catalogue.
For more information about the connection between Apollo (the RIVM data catalogue) and iRods, see [here](../catalog_metadata).

### Dataset Default Metadata
Dataset Default metadata can be used as a template for metadata of uploaded datasets. There are no required fields for schemata in this tab, because metadata can differ per uploaded dataset


## The metadata editor

![](../../!assets/images/metadata_editor/metadata_editor.jpg)

This editor can be used whenever metadata needs to be edited, on Projects, or Dataset Defaults. On the left hand side the available schemata are shown. If you are manager of a project you can edit this metadata.
Select a schema by clicking the select box (1).
Open the selected schema by clicking the display field with the name (2).

The form will be displayed in the editor window (3)

![](../../!assets/images/metadata_editor/rivm_schema.jpg)

- The selected schema is highligted (1)
- Required fields are indicated with an asterix (*)
- The small triangle in the right corner of the field (2) indicates a Dropdown with multiple options to select from.
- The blue [+] button (3) can be used to initialize a field, or add multiple fields if required.
- The red [x] button removes the field
- Date fields have three fields, with placeholders 'Year', 'Month' and 'Day'.
- Errors in fields are displayed on top of the form.

After filling the required information press 'Save' (5) to store the metadata on iDMS. It is stored on the collection corresponding with the project or dataset uploaded.

> !! Beware: unselecting a schema in the metadata editor wil remove the metadata from the collection (after being warned in the iDMS)


If you have any questions about this, reach out to onderzoeksendataloket@rivm.nl.

