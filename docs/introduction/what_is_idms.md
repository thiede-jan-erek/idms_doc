# iRODS Data management system (iDMS)

## Introduction
For unstructured data, a data management system (DMS) is the ideal solution for secure and FAIR storage. One of the primary functions of a DMS is to link unstructured data with metadata within an organized storage structure. At the moment, the DMS we describe here is mainly used for bio-informatics data. However, it is not exclusive to this type of data and can be used for all unstructered data. 

The DMS consists of multiple components, serving different goals. Although iRODS is only one of these components, we often refer to the whole DMS as iRODS.


## Functions
The idea of the DMS is to store data in such a way that they can be easily found and accessed, and to make it easy to reproducibly analyse the data.

To find and access the data, we use the iRODS system for storing data together with metadata and iDMS as a user friendly interface. The metadata includes the project to which the dataset belongs, the provenance information such as which other datasets, and what software/git repo (with tag) was used to create the data. iRODS uses a 'virtual' location for the data, which makes the data easier to access even when it is archived on tape at Surf.

Data sets are saved in [projects](../../user-manual/starting_a_new_project/). A project can be configured so it is only accessible to certain users. This uses the RIVM Active Directory (authenticate and authorize), so the usernames/passwords are the same as those for other RIVM services such as webmail. For each project, you can also configure a set of default pipeline(s), which is automatically run on each new dataset by the jobengine.


## Components
We briefly describe only the main components here. For a more complete picture of the architecture, see the [developers overview](../../../developers/jobengine/). As a user of the system, you will most likely only interact directly with the iRODS and the iDMS component.

### iRODS
iRODS is a system for storing and virtualizing the data together with metadata. It should be possible from different applications or platforms to connect to iRODS directly and access the data and metadata. 


### iDMS
iDMS is browser based user interface for the DMS, accessed through :octicons-link-external-24:https://biorods.rivm.nl. It can be used to monitor processes in the DMS, and to make some manual changes. For example, you can: 

- view all [:octicons-link-external-24:running pipelines](http://biorods.rivm.nl/jobs/) of one of your projects.
- [:octicons-link-external-24:search](http://biorods.rivm.nl/collbrowser/search) through all your collections and filter by metadata.
- configure new projects and [pipelines](../../user-manual/pipeline_automation/).
- [(re)start analysis pipelines](../../user-manual/pipeline_automation#starting-a-pipeline-manually) with a specific version of your code.
- visually trace the lineage of your dataset back to the point the (raw) data entered iRODS in the [:octicons-link-external-24:Collection Browser](http://biorods.rivm.nl/collbrowser/)
- upload data with metadata via a simple [online form](../../user-manual/manual_upload_in_idms).
- etc.

### Jobengine
The jobengine is the system that runs analysis pipelines for datasets on the High-performance Computing Cluster (HPC). Jobs can be started manually from iDMS, or automatically when a new collection is added to a project. The status of a job/pipeline can be monitored in iDMS as well. 

A more technical schema of the jobengine can be found in the [developers overview](../../../developers/jobengine/)
