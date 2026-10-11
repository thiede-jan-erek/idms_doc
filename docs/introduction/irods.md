# Data management system (DMS)

## Introduction
For unstructured data, a data management system (DMS) is the ideal solution for secure and FAIR storage. One of the primary functions of a DMS is to link unstructured data with metadata within an organized storage structure. At the moment, the DMS we describe here is mainly used for bio-informatics data. However, it is not exclusive to this type of data and can be used for all unstructered data. 

The DMS consists of multiple components, serving different goals. Although iRODS is only one of these components, we often refer to the whole DMS as iRODS.


## Functions
The idea of the DMS is to store data in such a way that they can be easily found and accessed, and to make it easy to reproducibly analyse the data.

To find and access the data, we use the [iRODS](#irods) system for storing data together with metadata and [iDMS](#idms) as a user friendly interface. The metadata includes the project to which the dataset belongs, the provenance information such as which other datasets, and what software/git repo (with tag) was used to create the data. iRODS uses a 'virtual' location for the data, which makes the data easier to access even when it is archived on tape at Surf.

Data sets are saved in [projects](DMS/starting-a-new-project). A project can be configured so it is only accessible to certain users. This uses the RIVM Active Directory (authenticate and authorize), so the usernames/passwords are the same as those for other RIVM services such as webmail. For each project, you can also configure a set of default pipeline(s), which is automatically run on each new dataset by the [jobengine](#jobengine).


## Components
We briefly describe only the main components here. For a more complete picture of the architecture, see the [development wiki](https://gitlab.rivm.nl/bioinformatics/DevOps/-/wikis/dms#architecture-overview). As a user of the system, you will most likely only interact directly with the iRODS and the iDMS component.

### iRODS
iRODS is a system for storing and virtualizing the data together with metadata. It should be possible from different applications or platforms to connect to iRODS directly and access the data and metadata. 

## Accessing data
You can access the data via the:

 - [command line, iget](tools/use-iget-to-get-data-from-irods) from within the Wetenschappelijke Werkomgeving using [i-commands](https://docs.irods.org/master/icommands/user/). These are similar to regular terminal commands, but for the virtualized file system of iRODS. For example `ls` lists all files and folders in your current working directory, and `ils` lists all iRODS collections (folders) and data objects (files) in your current iRODS directory.
 - [iDMS](#idms). 
 - WebDAV. read only browsing of iRODS files and directories (no metadata) at https://rivm-bioir-l03p.rivm.ssc-campus.nl. This can be accessed through iDMS as well: when viewing a collection, simply click the folder icon in the top right of the screen.
 - [Windows Explorer](tools/access-irods-from-windows-explorer) "This PC/iRODS projects"
 - [Mount on head node: `mount_irods` command](tools/mount_irods)
 - NFSRods
 - directly from Python with [this client](https://github.com/irods/python-irodsclient), see [this page](DMS/using-irods-data-directly-in-code) for more information and an example.
 - directly from R with [this package](https://github.com/jspijker/ricmd)
 - [directly from Snakemake with this plugin](https://github.com/snakemake/snakemake-storage-plugin-irods)
 - [Other iRODS clients](https://irods.org/clients)


> :bulb: We are actively working on guides to use data directly form iRODS (on the LSF) from your scripts or workflows.


## Accessing metadata
- `imeta`, example:
```
$ imeta ls -C /rivmZone/projects/adhoc
AVUs defined for collection /rivmZone/projects/adhoc:
attribute: coll_description
value: Adhoc data, visible for everyone
units: 
----
attribute: ngsweb::sort_order
value: create_time
units: 
----
attribute: ngsweb::sort_reverse
value: true
units: 
----
attribute: projectID
value: adhoc
units: 
----
attribute: projectresponsible
value: beldvdm
units: 


$ imeta ls -C /rivmZone/projects/adhoc/240723_VH01799_93_AAFW77WM5_0004| head -20
AVUs defined for collection /rivmZone/projects/adhoc/240723_VH01799_93_AAFW77WM5_0004:
attribute: complete
value: true
units: 
----
attribute: projectID
value: adhoc
units: 
----
attribute: sequencing::brand
value: Illumina
units: 
----
attribute: sequencing::host
value: 131.224.198.19
units: 
----
attribute: sequencing::platform
value: nextseq
units: 
...
```

- [accessing metadata from Python](https://github.com/irods/python-irodsclient)
- [accessing metadata from R](https://github.com/jspijker/ricmd)
- viewing metadata in biorods: eg https://biorods.rivm.nl/collbrowser?path=/rivmZone/projects/adhoc/240723_VH01799_93_AAFW77WM5_0004
![image](uploads/cebcbffcbbec02210e7f546cdc87a2ec/image.png)
- query metadata: [example 1: find a sample number](https://gitlab.rivm.nl/bioinformatics/docs/-/wikis/tools/iquest) or [example 2: find parent collection](https://gitlab.rivm.nl/bioinformatics/docs/-/wikis/irods_parent_col)

### iDMS
iDMS is browser based user interface for the DMS, accessed through https://biorods.rivm.nl. It can be used to monitor processes in the DMS, and to make some manual changes. For example, you can: 

- view all [running pipelines](http://biorods.rivm.nl/jobs/) of one of your projects.
- [search](http://biorods.rivm.nl/collbrowser/search) through all your collections and filter by metadata.
- configure new projects and [pipelines](DMS/pipeline_automation).
- [(re)start analysis pipelines](DMS/pipeline_automation#starting-a-pipeline-manually) with a specific version of your code.
- visually trace the lineage of your dataset back to the point the (raw) data entered iRODS in the [Collection Browser](http://biorods.rivm.nl/collbrowser/)
- upload data with metadata via a simple [online form](../manual_upload_in_idms).
- etc.

### Jobengine
The jobengine is the system that runs analysis pipelines for datasets on the [High-performance Computing Cluster (HPC)](HPC/getting_started). Jobs can be started manually from iDMS, or automatically when a new collection is added to a project. The status of a job/pipeline can be monitored in iDMS as well. 

> If a pipeline fails, the job will show red in the joblist, and the collection will be invalid (as set in the metadata of the collection). The log files (which probably contain the error) are available from the job under `Job Logs` which can be found in the sidebar menu. If the error was not due to a bug, you can choose to manually restart the analysis.

A more technical schema of the jobengine can be found in the [development wiki](https://gitlab.rivm.nl/bioinformatics/DevOps/-/wikis/#schematic-overview-of-the-automatic-pipeline-process).
