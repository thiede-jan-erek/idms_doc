



The upload feature of iDMS allows users to create new collections in their projects by uploading them directly. The upload is done via a simple web form where you select a project, fill in the necessary metadata fields, and directly upload files from your computer with a file browser. When you finalize the upload, the collection is automatically moved to the project.

## Workflow

Click on 'Dataset Uploads' (Figure: 1) in the left sidebar menu in iDMS. To start a new upload, then click on **New Upload** (Figure: 2) at the bottom of the same sidebar menu. You then go to a form where you select a target project, and enter a collection name and description (Figure: 3).

Clicking 'Save & Continue' (Figure: 4) will lead you to the next part of the form, where you select a [metadata schema](#metadata-schemata) and fill in the metadata. clicking 'Save' saves the metadata on the upload collection.

Next Select **Data Uploads** to [add your files](#adding-files) to the new collection, and finalize the upload by clicking Finalize (Figure: 6).

![](../../!assets/images/upload_feature_ngsweb_1.png)
![](../../!assets/images/upload_feature_ngsweb_2.png)

### Continuing an upload
All metadata and files of an unfinished upload are saved. You can safely log out of iDMS and continue your upload at a later time. To continue, Click on 'Dataset Uploads' (Figure: 1) in the sidebar menu, and select the dataset you want to continue on from the list of 'Pending uploads' (Figure: 3).

 > Note: This is particularly useful when a metadata schema is not yet available for your project, as it is required to select a metadata schema before finalizing an upload.

## Adding files
Adding files is done in the 'Data upload' tab of the UPLOAD SETTINGS sidebar menu (Figure: 5). Files of an unfinished upload are temporarily stored in your iRODS home folder: a subfolder with a strange name will be created automatically. Alternatively, you can directly put files in this iRODS folder with [:octicons-link-external-24: i-commands](https://docs.irods.org/master/icommands/user/#iput).

## Metadata Schemata
A metadata schema is required for your upload, because a dataset without (standardized) metadata is not [:octicons-link-external-24: FAIR](https://www.go-fair.org/fair-principles/).
All upload have the option to add **rivm.json** metadata. This form contains the metadata that is displayed in the central metadata catalogue (apollo.rivm.nl).
For more information about the connection between Apollo (the RIVM data catalogue) and iRods, see [here](/DMS/catalog_metadata).

A metadata schema consists of a set of field names together with the type of information these fields should contain. Which fields are required can depend on the values of other fields, so schemata can get quite complicated. Moreover, deciding which fields need to be present is requires domain knowledge of the datasets and technical know-how about metadata schemata. You can find examples of schemata here: [:octicons-link-external-24: /rivmZone/system/schemata/datasets](https://biorods.rivm.nl/collbrowser?path=/rivmZone/system/schemata/datasets). Download the json file, edit it to your project and upload it into your own project folder.

If you would like to use the upload feature of iDMS, and no schema has been configured for your project yet, please email us (onderzoeksendataloket@rivm.nl). We will then together construct a new metadata schema that fits your project.