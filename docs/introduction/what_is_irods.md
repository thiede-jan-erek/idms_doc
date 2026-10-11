# What is iRODS? And what do we use it for?


iRODS is the **integrated Rule-Oriented Data System** developed by the iRODS Consortium.
It is essentially three things:
- A virtual filesystem,
- A key-value store,
- A rule engine

## A virtual Filesystem


![](../../!assets/images/filesystems.gif){width=675 height=165}

In some filesystems, the logical structure of the filesystem and the actual physical storage are tightly coupled. In the first example the C: and D: drives are separate physical locations: C: might be a fast, expensive device with limited capacity, while D: is a slower, cheaper, and larger storage medium.

Other filesystems avoid this hard connection between physical storage devices and the logical structure, 
In the second example, you can navigate through the filesystem as a unified tree, and individual directories might reside on different storage devices. However, all files within a specific folder are typically stored on the same device.

In iRODS, the logical structure and the physical storage are completely disconnected.
iRODS maintains its own database containing the logical structure and keeps track of the physical location of files. Folders (called "Collections") do not exist in any physical filesystem at all—they live entirely within the database.

Any file (a "Data Object") can reside on any storage device. In fact, a file can even exist on multiple storage devices simultaneously (as replicas). While the logical structure stays identical, files can be moved or replicated between decives in the background without the user noticing.

Of course, maintaining multiple copies requires additional management—for example, if one copy is modified, other copies must be marked as stale. But this kind of data management is handled automatically by the iRODS system.

## A Key-Value Store

Information about a file (metadata) is often just as important as the file's contents. Traditional filesystems offer very limited support for this, typically tracking only basic attributes like file name, size, and creation date. Some file formats contain internal metadata (such as camera settings embedded in an image file).

However, if you want to store arbitrary contextual information regarding a file (e.g., the involved lab technician, chemical solutions used, device settings, lab humidity, etc.), the traditional solution is to store this metadata in a separate database alongside the file's identifier and location.

Since iRODS already includes a database (the Catalog), it can easily be used as a key-value store to manage any kind of metadata.

In iRODS, we can store AVUs (Attribute-Value-Units): triplets consisting of an Attribute (Key), an associated Value, and an optional Unit field, attached to any Collection or Data Object.

This enables iRODS to use an SQL-like query language to find objects based on this rich metadata.

## A Rule Engine

Lastly, iRODS is rule-oriented. Any action in the filesystem—such as reading a file, creating a collection, or setting or changing metadata—can trigger the execution of custom code.

The true power of iRODS lies in combining all three aspects. For example: a user reads a file, which triggers a rule that automatically updates the file's metadata (e.g., tracking the number of accesses in the last month). Other rules can then react to this metadata—files that are accessed frequently are automatically migrated to high-speed storage, while files that haven't been accessed in a while are tiered down to a tape archive.


# How is it used?
In a pure iRODS installation, users interact with the system via the Linux command line. Standard filesystem navigation commands like ls, cd, and mkdir have iRODS equivalents known as i-commands: ils, icd, and imkdir, alongside data transfer and management tools like iput, iget, and imeta. For users already familiar with a Linux environment, these commands are straightforward to pick up.

However, in many cases, end-users do not want to be bothered with the command line. To integrate iRODS smoothly into researchers' workflows, many institutions have built custom user interfaces on top of the core iRODS system.

Depending on the use case, these implementations serve very different purposes:

- Datahub at Maastricht University: Mostly used as an archival tool. At the end of a research project, data can be ingested (uploaded) via a web interface, and descriptive metadata can be added for long-term preservation.

- Yoda at Utrecht University: Built primarily for active collaborative work. A Yoda workspace is created at the start of a project, and data is continuously added as the project progresses. At the end, the project can be securely archived and/or published.

- iDMS at RIVM (Our own solution): Focuses on the automated, nightly ingestion of data (e.g., from sequencers) and the automated execution of custom pipelines—written by scientists—on our HPC infrastructure. The resulting datasets are automatically stored back into iRODS. The outcome is a traceable chain of datasets with clear provenance linking inputs, executed code, and outputs.
