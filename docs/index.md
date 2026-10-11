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