# Metadata for Apollo

Projects and Datasets in iRods can be registered automatically in the RIVM data catalog [Apollo](https://apollo.rivm.nl/).
Apollo is connected to iRods through the so-called "harvester", which collects metadata from iRods and sends it to Apollo.
The harvester only collects the metadata that is added specifically for the catalog: all metadata filled in through the metadata editor using the **rivm** schema.

## How To

Project metadata for the catalog can be added through the Project page on the Project Metadata tab ([see how](../adding_metadata_to_a_project)).
![](../../!assets/images/catalog_metadata_project.png)

For a dataset, catalog metadata is added during the upload process by selecting the **rivm schema** for the metadata ([see how](../manual_upload_in_idms))
![](../../!assets/images/catalog_metadata_upload.png)


To determine in what dataverse (i.e. department/centre/domain) the metadata must be placed, we require the project to have a department configured.
All datasets within the project will be published in the dataverse for that department.
![](../../!assets/images/catalog_metadata_department.png)


If all metadata is filled in correctly and a **department** is set for the project, then the harvester will collect and update your metadata each night.
This should result in a new draft record in Apollo, which will have to be reviewed before publication.
There are designated reviewers for each department, who should get notified automatically that there is a new draft record to review.

## When to

If your project is one where data is handled by automated pipelines, it is probably enough to set your metadata at the project level.
If you regularly upload data that was made with different scripts (or even manually) and each dataset clearly has its own distinct metadata, then it is recommended to set your metadata on the dataset level.

## Advanced usage

If you set your metadata yourself through iRods i-commands and you want to set catalog metadata, you will need to use a json2avu conversion.
As the metadata for the catalog is nested, it is typically handled in nested json-format, and needs to be converted to AVUs for storage in iRods.
Please contact us for more information if you intend to use this advanced feature!