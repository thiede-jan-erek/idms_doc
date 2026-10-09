An iRODS project is a group of people managing and contributing to a collection of data sets which are analysed in a similar way with (automated) pipelines.

> If some data should be analysed really differently from other data, you may need to create multiple projects. Similarly, if not all data have the same data compliance or data storing/archiving strategy, it may also be better to create multiple projects. Conversely, projects should not be split if this is not necessary. If you have any questions about this, reach out to [us](home).

## Creating a project
To create a new project, contact one of us (see the [home page](home)), and we will create one for you. We will need you to provide us with the following necessary information:
 - a clear and discernible project name. No spaces or special characters are allowed, except for underscores. Maximum of 24 characters. We advice not to use group/institu names or very generic names. 
 - a list of [managers](#users-and-managers) of the project. Managers can add users. 
 - a primary contact person who is responsible for or owner of the data (stored in the contacts list and in the metadata of the project)
 - a [Data Management Plan](https://rivmnl.sharepoint.com/sites/HomeBase_Onderwerpen/SitePages/Data-Management-Plan.aspx) (DMP). A DMP is often already available for newly initiated projects, as it is a mandatory requirement for starting any data-related work. If this is not available, we need to know atleast the archiving time.
    - This is so we know how to store/archive the data and for how long
 - [metadata schemata](/DMS/Manual-upload-in-iDMS#metadata-schemata) for the datasets (description of the minimal metadata on each collection)
    - For FAIRness of the data
    - Findability in the metadata catalogue
      - Fill the **Project Metadata** form _rivm.json_ for your project, and the metadata will be harvested by the catalogue (see [here](/DMS/catalog_metadata) for more information).
    - So uploading the data with metadata can be done via a [metadata json form](/DMS/Manual-upload-in-iDMS) on iDMS.
    - If you know .json format, you can look at examples files in [iDMS](biorods.rivm.nl) > Collection Browser > system > schemata > datasets > example_project (/rivmZone/system/schemata/datasets/example_project).

The [Datastewards](http://wiki.rivm.nl/inwiki/bin/view/Onderzoeks+en+Dataloket/Contact+Datastewards) can help you to fill in a DMP. For the metadata schema, you can get help both from us (technical and bio-informatics specific metadata) and from the datastewards (general research metadata).

## Configuration options

### Public Data
Data in this project should be available for reuse within the RIVM. Public data can be viewed and downloaded by all iDMS users.

### Pipelines
Pipelines in the DMS can be started automatically when new collections arrive in the project. For more information about configuring these so-called processgroups, see [this page](DMS/pipeline_automation).

### Project Metadata
- rivm.json: Metadata for the central RIVM catalogue (apollo.rivm.nl). After filling the required minimal set of fields the project metadata will be harvested by the catalogue (see [here](/DMS/catalog_metadata) for more information).
- project.json: general metadata for the project.

### Dataset Default Metadata
This tab can be used to prefill the metadata json files used for dataset uploads with default information.
- rivm.json: Metadata for the central RIVM catalogue (apollo.rivm.nl). After filling the required minimal set of fields in dataset uploads the dataset metadata will be harvested by the catalogue.
- default.json: prefilled metadata template for metadata to be added to dataset uploads.  Multiple json files can be specified here, and selected and filled during a data upload: [Data Uploads](DMS/Manual-upload-in-iDMS).

### Users and managers
Users of a project can add data to the project and set/modify metadata of collections and data objects in the collection. They can also (manually) start processes and process groups on collections in the project.

> Note: Any iRODS user can read all metadata, but reading data is restricted to a project's users.

A project's managers can additionally change the settings of the project. This includes adding and removing people to the list of users and managers, changing the list of contact persons, and [modifying the (default) processgroup](DMS/pipeline_automation).

### Optional: Contacts
The 'Contacts' tab of the [project settings page](http://biorods.rivm.nl/projects/), shows a list of people that can be contacted for more information about the project. There is currently no functionality of the DMS/iDMS coupled to this list.
