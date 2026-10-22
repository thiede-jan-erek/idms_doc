# Welcome to iDMS

Welcome to the official documentation hub for the RIVMs **iRODS Data Management System (iDMS)** suite. 


!!! note "Active Development"
    Currently, we are moving documentation from the internal wiki to this documentation. For now this documentation will still refer to RIVM internal tools, procedures and datasets. 


iDMS is a modular, scalable data management architecture designed to streamline data processing, storage, and web-based interaction across secure institutional infrastructure.

At the RIVM we use the iDMS internally as :octicons-link-external-24: [biorods](https://biorods.rivm.nl/) to automatically process sequencer data

---

## The iDMS Ecosystem

The system is decoupled into several dedicated, :simple-git: version-controlled Python packages (licensed under the **EUPL 1.2**), each of these repositories provides documentation for developers:

*   **:simple-github:[`iDMS_common_constants`](https://github.com/rivm-syso/iDMS_common_constants)**: constants and attribute names shared accross packages
*   **:simple-github:[`iDMS_common_irods`](https://github.com/rivm-syso/iDMS_common_irods)**: Robust wrappers and utilities for iRODS interactions (i.e.: session management, catalog queries, etc.).
*   **:simple-github:[`iDMS_common_filesys`](https://github.com/rivm-syso/idms_common_filesys)**: A filesystem abstractions to allow traversal and manipulation of iRODS, FTP, SFTP, NFS and others
*   **:simple-github:[`intorods`](https://github.com/rivm-syso/intorods)**: A set of tools to identify newly created data, ingest it into irods and automatically start processing pipelines.
*   **:simple-github:[`iDMS_irods_rules`](https://github.com/rivm-syso/iDMS_irods_rules)**: iRODS rules used to facilitate datamanagement
*   **:simple-github:[`iDMS_persistence`](https://github.com/rivm-syso/iDMS_persistence)**: Datbase model and REST-Api to store projects, processes, processgroups and other iDMS management data residing outside of iRODS
*   **:simple-github:[`iDMS_services`](https://github.com/rivm-syso/iDMS_services)**: A collection of processes run via cronjobs at regular intervalls. Most importantly the jobengine/runsheet-processor (executing pipelines on the HPC), datamanagement (deciding on which iRODS resources data should be located), regular downloading of reference datasets, etc.
*   **:simple-github:[`iDMS_web`](https://github.com/rivm-syso/iDMS_web)**: User-facing web application to allow management of projects, pipelines and processes; inspect jobs; search and traverse collections/datasets; etc. 

---

## Contacts


**Still have questions?** Get more answers from the [:octicons-link-external-24: Research and data loket](http://wiki.rivm.nl/inwiki/bin/view/Onderzoeks+en+Dataloket/WebHome) or ask the bioinformaticians in the [:octicons-link-external-24: teams group](https://teams.microsoft.com/l/team/19%3a63632755c7fc4ee6acab6e3e61b7b233%40thread.tacv2/conversations?groupId=cde5a118-ae37-41d8-9951-ea9c96125362&tenantId=cd8466c6-d7ce-410a-be13-33e40185fdab).

Or mail one of us directly: 

- Erwin van Wieringen (erwin.van.wieringen@rivm.nl), 
- Jan-Erek Thiede (jan.erek.thiede@rivm.nl), 
- Robert Verhagen (robert.verhagen@rivm.nl), 
- Eva Deutekom (eva.deutekom@rivm.nl), 
- Remie Janssen (remie.janssen@rivm.nl), 
- Stefan Lobner (stefan.lobner@rivm.nl), 
- Arthur Knopper (arthur.knopper@rivm.nl), 
- Vasilis Lemonias (vasilis.lemonias@rivm.nl), 
- Yuchen Huang (yuchen.huang@rivm.nl)

<!--
## Dependencies


The high-level data flow between the user interface, core services, and storage backends is structured as follows:

```mermaid
graph TD
    User([User / Client]) -->|HTTP / REST| Web[idms_web]
    Web -->|Business Logic| Services[idms_services]
    Services -->|Database Queries| Persistence[idms_persistence]
    Services -->|Data Grid Operations| iRODS[idms_common_irods]
    
    subgraph Core Shared Layer
        Constants[idms_common_constants]
    end
    
    Web -.-> Constants
    Services -.-> Constants
    Persistence -.-> Constants
```
-->