### Jobengine

The jobengine is a python process, responsible for configuring, starting and monitoring automatic pipeline processes, processing data in iRODS collections, as well as managing the generated result data. 

Processes can be triggered automatically, when new data arrives, or on user request.

The jobengine works by processing so-called **runsheets** in various states (see [runsheet states](#runsheet-states)). Normally runsheets consist of metadata attached to a pipeline output collection. However, a initial runsheet can also be a yaml-file uploaded to a specific collection. The jobengine will read this yaml file and convert it to a collection with metadata.

The jobengine can either process a single pipeline, or a combination of pipelines that have to be executed in a specific order, called a **process group**. Process group configuration is stored in the [BioRest](dms/projectdb) database.

#### Process or processgroup creation

Processes or process groups can be created in two different ways:

* By uploading a runsheet yaml file to the collection /\<zone>/system/runsheet/incoming. In this way, only a single process can be created, not a process group.
* By setting the metadata attribute **processrequest** on a collection. The jobengine will scan for this attribute. It then checks for the attributes **processgroupID** and **processID**.
  * If **processgroupID** is present, it will read the processgroup configuration from BioREST, and create all required output collection runsheets, including dependencies.
  * If **processgroupID** is not present, but **processID** is, it will read the process configuration from BioREST, and use it to create a runsheet file, in the collection /\<zone>/system/runsheet/incoming.

The runsheet yaml file has an option to specify a list of dataobjects as input, instead of a single input collection. The dataobjects can come from multiple collections.

The following table summarizes the various ways to trigger the jobengine.

Trigger|Creates process|Creates processgroup|Input from multiple collections|
-|-|-|-
runsheet file|YES|NO|YES
processrequest and processID|YES|NO|NO
processrequest and processgroupID|YES|YES|NO


#### Runsheet processing
After running the process and processgroup creation, the jobengine will process the runsheet yaml files, and all runsheet collections that have a state other than **done**.

#### The runsheet file

The automatic run of a pipeline can be started by a yaml file called a runsheet. This file contains all necessary parameters to initiate a pipeline run.

When processing the runsheet, its content will be attached as metadata for ease of handling. The metadata attribute names use the prefix ```sys::runsheet::```

The runsheet file has some additional flexibility over a predefined process:
* The process to be executed can be specified as a git repository of a process ID
* The input data for the pipeline can be an iRODS collection or a list of dataobjects
* You can add extra data to the runsheet file, that you can use in the pipeline process

The table below describes all parameters that can be in the runsheet.
The columns repo, processID, input_collection and input_list show wether parameters are required (R), valid (V), or forbidden (F) in that specific runsheet type

Parameter|Type|repo|processID|input_list|input_collection|Description
---|---|---|---|---|---|---
projectID|string|-|-|R|V|Project where the output data will be. If ommitted, the input_collection project will be used
processID|string|F|R|-|-|The process ID from the process database
repo|string|R|F|-|-|GIT repository where the pipeline code resides
tag|string|R|R|-|-|Tag or branch name within the git repository
input_collection|string|-|-|F|R|Pipeline input collection
lsf_queue|string|V|V|V|V|Name of the LSF queue (`bio-acc`|`bio-prio`|etc.) to run the pipeline on. If not set, it uses the default queue of the current iRODS environment.
distribution|boolean|V|V|V|V|Determines if irods reads the distribution.yml file to distribute pipeline output
modify_in_place|boolean|-|-|F|V|Determines if the pipeline can modify the input collection, or the input collection metadata
omit_bringonline|boolean|V|V|V|V||Determines whether input data will be retrieved from the archive
omit_staging|boolean|V|V|V|V|Determines whether input data will be downloaded to the input directory
requesting_user|string|V|V|V|V|The user name of the user that started the processgroup. (Only used when runsheet owner is rods)
use_projectaccount|boolean|V|V|V|V|Whether to use the project service account to run the pipeline. (Only used when runsheet owner is rods)
reference_data|dict|V|V|V|V|Contains a dictionary with reference data specifications. See the [Reference data section](/dms/jobengine#reference-data))


Example runsheet file, specifiying a git repo:

```
pipeline:
  type: input_collection
  input_collection: /rivmZone/projects/ngslab/nextseq/190902_NB502001_0051_AHTFCVAFXY
  tag: v2.0
  repo: https://gitlab+deploy-token-2:kRHhZm7FbnBYbzV6pQMJ@gitlab.rivm.nl/wierinve/bcl2fastq.git  
  modify_in_place: 'false'
  projectID: my_project
  restartable: 'false'
  reference_data:
    cellranger-ref:
      reference_set: cellranger_human
      reference_version: 17
```

Example runsheet file, specifiying a process ID:

```
pipeline:
  type: input_collection
  input_collection: /rivmZone/projects/ngslab/nextseq/190902_NB502001_0051_AHTFCVAFXY
  tag: v2.0
  projectID: my_project
  processID: bcl2fastq
  modify_in_place: 'false'
  repo: https://gitlab+deploy-token-2:kRHhZm7FbnBYbzV6pQMJ@gitlab.rivm.nl/wierinve/bcl2fastq.git
  restartable: 'false'
  reference_data:
    cellranger-ref:
      reference_set: cellranger_human
      reference_version: 17
```

#### Reference data

Reference data to be used can be specified in a reference_data dict in the runsheet. The structure is:
```
  reference_data:
    <label>:
      reference_set: <set name|set id>
      reference_version: <number|"latest"|name|UUID>

```

If a processID is specified in the runsheet:

* The `label` has to match a reference data label specified in the process definition
* The `reference_set` attribute is ignored, since the reference set from the process definition is used. Only the version can be overruled.

If a git repo is specified in the runsheet:

* The `label` is arbitrary
* The `reference_set` must be a valid reference set from the reference data database
* The `reference_version` must be a valid reference set version from the reference data database

#### RUnsheet metadata

The processing rules add additional metadata to the runsheet:

Attribute|Type|Description
---|---|---
sys::run::environment|string|The environment string for the pipeline
sys::run::input_dir|string|The directory where input files are located
sys::run::output_collection|string|Output collection
sys::run::output_dir|string|Output directory
sys::run::pipeline_dir|string|Directory where the pipeline code is cloned
sys::run::projectID|string|The projectID from the input collection
?? sys::run::processID|string|The processID from the input collection
sys::run::service_account|string|account that starts the pipeline

The runsheet has to be placed in the collection ``/<ZONE>/system/runsheet/incoming``. Uploading a runsheet file to this collection immediately triggers a rename operation, to make sure the runsheet file name is unique. The rule __bioRunner_Incoming__ will perform the following actions:

* Create an output collection for the pipeline
* Create a code directory, clone the git repo, and put the __wrapper.sh__ wrapper in the code directory.
* Export the metadata and input and output collection as environment variables. These variables have the following format
```
irods_meta_<attr>=<value>
irods_input_collection=<value>
irods_output_collection=<value>
```

* Move the runsheet to the collection ``/<ZONE>/system/runsheet/queued``

Runsheets in the collection ../../queued are ready to be executed. The rule __bioRunsheet_Queued__ does:

* Start the pipeline through __wrapper.sh__
* Store the PID for the wrapper process as metadata on the runsheet.
* Move the runsheet to ``/<ZONE>/system/runsheet/active``

The rule __bioRunsheet_Active__ :

* Checks if the wrapper process is still active
* If the wrapper has finished successfully:
  * Register files in the output collection
  * Process ``metadata.yml``
  *

TODO


To automatically trigger the execution of a pipeline upon arrival of new data, modifying the metadata attribute __complete__ to __true__ triggers the generation of a runsheet file. Rules for this process are in the rule file __bioRunsheet.re__


#### Output distribution
A pipeline can force the output data to be distributed to other projects/processes, by creating a file ```distribute.yml``` or ```distribute.json```
This file will only be processed if ```sys::runsheet::distribution``` is set to ```true``` for the current process. 
The format of the distribute file is documented [here](/dms/jobengine#distribution)

#### Further information

* We also have a [guide](/guides/create-pipeline.md) for configuring pipelines.
* See [this page](/guides/Development-and-testing/Jobengine) for some pointers on testing and developing the jobengine.

-------------------------


## Runsheet types

Various runsheet types are possible:

|type|Description
|-|-|
|input_collection|Process one input collection
|input_list|Create input from input object list|

### Input collection description

TODO

### Input list description

When using the **input_list** runsheet type, the runsheet provides a list of dataobjects to copy to the input directory for the pipeline. This list should be provided under the *dataobjects* key.

*example*
```
pipeline:
  dataobjects:
    - /rivmZone_test_01/projects/yers/220607_NB552493_0076_AHMG5MAFX3_0012/1072200059_S85_R1_001.fastq.gz
    - /rivmZone_test_01/projects/yers/220607_NB552493_0076_AHMG5MAFX3_0012/1072200059_S85_R2_001.fastq.gz
    - /rivmZone_test_01/projects/svstec/220607_NB552493_0076_AHMG5MAFX3_0011/1072200060_S86_R2_001.fastq.gz  
  ...
```

The process to run can be specified by either supplying a *processID* and a *tag*, or a *repo* and a *tag*.

*example 1*
```
pipeline:
  processID: Apollo mapping
  tag: v0.7
  ...
```

*example 2*
```
pipeline:
  repo: https://github.com/RIVM-bioinformatics/apollo-mapping
  tag: v0.7
  ...
```

The *projectID* is the project where the output is stored, this field is non-optional.

*example*
```
pipeline:
  projectID: vir-ngs
  ...
```

The *lsf_queue* is the LSF queue that will be used, this field is non-optional.

*example*
```
pipeline:
  lsf_queue: bio-acc
  ...
```

An optional property *full_path* specifies how input files are stored in the input directory. When full path is set to **true**, the collection path of the input objects will be created as directories in the input directory. When set to **false** the files will be placed directly under the input directory. 

All pipeline settings are stored under a key **pipeline**. You can also store other data in this sample sheet. This allows for combining for example a sample sheet with pipeline information. In this case, instead of supplying all dataobjects, it is also possible to give a *yamlpath* to other objects in the runsheet. Look at the example to see how this works.

### Runsheet Examples

1) Basic input_list example with processID
```
pipeline:
  processID: Nanopore demultiplex 
  tag: master
  projectID: vir_ngs
  lsf_queue: bio-acc
  name: 'Schiphol-specific Analysis 2021-01-03 - 2021-06-28'
  type: input_list
  full_path: false
  dataobjects:
    - /rivmZone_test_01/projects/yers/220607_NB552493_0076_AHMG5MAFX3_0012/1072200059_S85_R1_001.fastq.gz
    - /rivmZone_test_01/projects/yers/220607_NB552493_0076_AHMG5MAFX3_0012/1072200059_S85_R2_001.fastq.gz
    - /rivmZone_test_01/projects/svstec/220607_NB552493_0076_AHMG5MAFX3_0011/1122200158_S88_R1_001.fastq.gz
    - /rivmZone_test_01/projects/svstec/220607_NB552493_0076_AHMG5MAFX3_0011/1122200158_S88_R2_001.fastq.gz
    - /rivmZone_test_01/projects/svlismon/220607_NB552493_0076_AHMG5MAFX3_0009/1142200038_S94_R1_001.fastq.gz
    - /rivmZone_test_01/projects/svlismon/220607_NB552493_0076_AHMG5MAFX3_0009/1142200038_S94_R2_001.fastq.gz

```

2) Basic input_list example with git repo
```
pipeline:
  repo: https://gitlab.rivm.nl//landmanf/nanopore_demultiplex.git
  tag: master
  projectID: TEST123
  lsf_queue: bio-acc
  name: 'Schiphol-specific Analysis 2021-01-03 - 2021-06-28'
  type: input_list
  full_path: false
  dataobjects:
    - /rivmZone_test_01/projects/yers/220607_NB552493_0076_AHMG5MAFX3_0012/1072200059_S85_R1_001.fastq.gz
    - /rivmZone_test_01/projects/yers/220607_NB552493_0076_AHMG5MAFX3_0012/1072200059_S85_R2_001.fastq.gz
    - /rivmZone_test_01/projects/svstec/220607_NB552493_0076_AHMG5MAFX3_0011/1122200158_S88_R1_001.fastq.gz
    - /rivmZone_test_01/projects/svstec/220607_NB552493_0076_AHMG5MAFX3_0011/1122200158_S88_R2_001.fastq.gz
    - /rivmZone_test_01/projects/svlismon/220607_NB552493_0076_AHMG5MAFX3_0009/1142200038_S94_R1_001.fastq.gz
    - /rivmZone_test_01/projects/svlismon/220607_NB552493_0076_AHMG5MAFX3_0009/1142200038_S94_R2_001.fastq.gz

```

3) Combined sample sheet/runsheet

The runsheet/sample sheet will be stored in the output directory, so it can be used by the pipeline as sample sheet. The jobengine will only use the **pipeline** part.

The **dataobject_path** is a [yamlpath](https://pypi.org/project/yamlpath/), that refers to other keys in the runsheet.

```
collection: 'Schiphol-specific Analysis 2021-01-03 - 2021-06-28'

samples:
  21-068-1:
    status: 'yes'
    reference: /mnt/db/nrs/MN908947.3.fasta
    annotation: /mnt/db/nrs/MN908947.3.gff
    primerset: /mnt/db/nrs/primers/artic_primers_v3_low_pool.bed
    reads:
      R1: /rivmZone_test_01/projects/yers/220607_NB552493_0076_AHMG5MAFX3_0012/1072200059_S85_R1_001.fastq.gz
      R2: /rivmZone_test_01/projects/yers/220607_NB552493_0076_AHMG5MAFX3_0012/1072200059_S85_R2_001.fastq.gz
    source:
      sample_date: '2021-01-03'
      location: Schiphol
      comment: 'test run'
  21-068-2:
    status: 'yes'
    reference: /mnt/db/nrs/MN908947.3.fasta
    annotation: /mnt/db/nrs/MN908947.3.gff
    primerset: /mnt/db/nrs/primers/artic_primers_v4_1.bed
    reads:
      R1: /rivmZone_test_01/projects/svstec/220607_NB552493_0076_AHMG5MAFX3_0011/1122200158_S88_R1_001.fastq.gz
      R2: /rivmZone_test_01/projects/svstec/220607_NB552493_0076_AHMG5MAFX3_0011/1122200158_S88_R2_001.fastq.gz
    source:
      sample_date: '2021-01-03'
      location: Schiphol
      comment: 're-run'
  21-19175-2:
    status: 'yes'
    reference: /mnt/db/nrs/MN908947.3.fasta
    annotation: /mnt/db/nrs/MN908947.3.gff
    primerset: /mnt/db/nrs/primers/artic_primers_v3_low_pool.bed
    reads:
      R1: /rivmZone_test_01/projects/svlismon/220607_NB552493_0076_AHMG5MAFX3_0009/1142200038_S94_R1_001.fastq.gz
      R2: /rivmZone_test_01/projects/svlismon/220607_NB552493_0076_AHMG5MAFX3_0009/1142200038_S94_R2_001.fastq.gz
    source:
      sample_date: '2021-06-28'
      location: Schiphol
      comment: 'high primer-dimer in collection'

pipeline:
  repo: https://gitlab.rivm.nl//landmanf/nanopore_demultiplex.git
  tag: master
  projectID: TEST123
  lsf_queue: bio-acc
  name: 'Schiphol-specific Analysis 2021-01-03 - 2021-06-28'
  type: input_list
  full_path: false
  dataobject_path: samples.*.reads.*
```

## Pipeline execution account

The account that runs a pipeline depends on a number of conditions:

* If a user submits a runsheet, it is always run as the user who submitted it
* If **rods** submits a runsheet, executing user will depend:
  * If the property **use_projectaccount** is set to **true** in the runsheet, the pipeline will run as the project service account
  * If the property **use_projectaccount** is set to **false**, the pipeline will run as:
    * The value of **requesting_user**, if not equal to **rods**.
    * The project service account if **requesting_user** is equal to **rods**

If a user starts a pipeline or processgroup through the web interface, the resulting runsheet is submitted by **rods**, with the property **use_projectaccount** set to true. Thus the pipeline will run as the project service account.

## Parameter reference

### input_collection

Parameter|description|required|default|remarks
-|-|-|-|-
type|input_collection|no|
input_collection|iRODS collection where the pipeline input is|yes
repo|git repo url where the pipeline code is located|yes|
tag|git reference to check out from **repo**|yes|
projectID|project where output collection is created|no|NA|If not supplied, projectID from input_collection will be used
lsf_queue|value of LSB_DEFAULTQUEUE for this pipeline|no|system default queue|
modify_in_place|Determines if next pipeline will run on input or output collection. Also changes the way **metadata.yml** and **distribute.yml** are handled.|no|false|remove this?
next_processID|processID to start when this process finishes succesfully|no|none|can go away
next_projectID|projectID where to next_processiD is in|no|none|can go away
omit_staging|Do not stage data to compute resource|no|false|
restartable|If pipeline fails, watch changes in input coll, and restart when a change is made|no|false|can go away
service_account|not used|no|NA|can go away
description|process description|no||
distribution|If **true**, input or output of this pipeline will be distributed by processing **distribute.yml**|no|false
modify_in_place|Determines if next pipeline will run on input or output collection. Also changes the way **metadata.yml** and **distribute.yml** are handled.|no|false|remove this?
restartable|If pipellien fails, watch changes in input coll, and restart when a change is made|no|false|can go away
omit_staging|Do not stage data to compute resource|no|false|
~~processID~~|name of the process to start|yes|NA|probably not required any more
~~next_processID~~|processID to start when this process finishes succesfully|no|none|can go away
~~next_projectID~~|projectID where to next_processiD is in|no|none|can go away
~~service_account~~|not used|no|NA|can go away

### input_list
Parameter|description|required|default|remarks
-|-|-|-|-
type|input_list|no|input_collection|
name|job name and hint for output collection name|yes|
dataobjects|list of files to use as input|dataobjects or dataobject_path|
dataobject_path|yamlpath that refers to other parts of the runsheet|dataobjects or dataobject_path|
full_path|if **true**, copy input files with the full irods path|no|false 
projectID|project where output collection is created|yes|
repo|git repo url where the pipeline code is located|repo or processID|
processID|processID from the process database, one of **repo** or **processID** is required|repo or processID|
tag|git reference to check out from **repo**|yes|
lsf_queue|value of LSB_DEFAULTQUEUE for this pipeline|yes||
description|process description|no||
requesting_user|User that requests the pipeline run|no|runsheet owner|Can only be set by rods
use_projectaccount|Indicates the pipeline should run as the project svc account|no|false|Can only be set by rods
distribution|If **true**, input or output of this pipeline will be distributed by processing **distribute.yml**|no|false
omit_staging|Do not stage data to compute resource|no|false|? Not sure if this makes sense for this type

## Jobengine states

The pipeline jobs will go through a number of states, with well-defined state transitions. State is stored in **sys::runsheet::state**. The different states are described below. 

### incoming

The **incoming** state only applies to runsheet yaml files. The following actions will be executed in **incoming**

* Read the runsheet file
* Verify if the format is valid
* Check if all required attributes are present, and if no invalid combination of attributes is used
* Create an output collection for the process
* Copy all runsheet attributes to the runsheet collection as metadata with prefix: **sys::runsheet::**

**Troubleshooting**:
If an error occurs, the runsheet will be discarded. This can only be seen in the jobengine log, since no runsheet collection has been created yet. This makes troubleshooting for the end user a bit tedious. The failed runsheet will be moved to `/<zone>/system/runsheet/finished`

If no error occurs, the next state will be **choose**

### depends

All processes in a processgroup start in this state. Processes that are not in a processgroup will never be in this state.

The jobengine will check if dependant pipelines/jobs have finished succesfully before moving a job to `choose`. If a job that this job depends on finishes with an error, this job will move to state `notrun`

### choose 

In this state, the jobengine will decide on what cluster to run the job. For now, there is only one configured cluster. The cluster configuration is read from the constant data structure SITES_AND_CLUSTERS, and stored on the runsheet. 

**Troubleshooting**: If the cluster configuration is not found, this will be logged as an error. Other errors are unlikely.

Next state is **prepare**

### prepare

In this state, a work directory will be created, and the pipeline code will be downloaded. The **stage** attribute will be set for all required input collections.

**Troubleshooting**: Normally all errors should be logged in the runsheet log. If a job remains in the **prepare** state, check the **jobengine.log** for errors (grep for the runsheet collection).

Next state is **stage**

### stage

A detailed explanation of the data staging process and the involved modules can be found [here](#data-staging-process)

The job will remain in **stage** untill all input collections (including reference data) have been replicated to an accessible resource.
Detailed mechanism:

* The **jobengine** (in state prepare) sets the **sys::archive::stage** or the **sys::archive::online** attribute to the site where the job should run. It also adds a timestamp to the collection, with name **sys::pipeline::stage_time**
* The **irods_stage.py** script will scan for the **sys::archive::stage** and **sys::archive::online** attribute, and set a corresponsing **sys::archive::desired_state**.
* Setting **sys::archive::desired_state** will trigger a `collection_tiering` job on the delay queue
* The `collection_tiering` job will copy objects to the requested resources. After finishing, it sets a timestamp to **sys::archive::last_run**
* The jobengine (in the **stage** state) will check if **sys::archive::last_run** > **sys::pipeline::stage_time**, and if the collection is actually online in the requested site. If all input collections pass this test, it will move the runsheet to state **spacecheck**

```mermaid
sequenceDiagram
box cronjobs
participant jobengine.py
participant irods_stage.py
end
box collections
participant runsheet
end
box irods rules
participant schedule_tiering_desired_state
participant collection_tiering
end

jobengine.py->>runsheet: SET sys::pipeline::stage_time
jobengine.py->>runsheet: SET sys::archive::stage/online
irods_stage.py->>runsheet: READ sys::archive::stage/online
irods_stage.py->>runsheet: SET sys::archive::desired_state
runsheet->>schedule_tiering_desired_state: trigger tiering
schedule_tiering_desired_state->>delay queue: start collection_tiering
delay queue->>collection_tiering: Run collection tiering
collection_tiering->>runsheet: SET sys::archive::last_run
jobengine.py->>runsheet: READ sys::archive::last_run
```

**Troubleshooting**
If jobs remain in **stage** for a long time:

* Run the **iqstat** command to see the delay queue.
  * If there are many jobs, staging might take longer. 
  * Verify if there is a `collection_tiering` job for the input collection(s) of the pipeline. If there is, staging is still in progress
* Check the logfile `/var/lib/irods/log/irods_rules_log` on the ICAT server. Filter for the input collection name. 
  * Sometimes you see multiple data COPY lines for the same dataobject. This might indicate a problem with the dataobject.
    * Check for [locked dataobjects](guides/troubleshoot/irods#locked-dataobject)
    * Check for [checksum issues](guides/troubleshoot/irods#checksum-issues)
  * You might also see checksum errors on TRIM actions. Check [this procedure](guides/troubleshoot/irods#checksum-issues)
* If there is no `collection_tiering` job, and the pipeline is still stuck in `stage`, this might indicate a python error. These are logged in `/var/log/irods/irodsServer.log`. Grep for `python_exception`:
```
{"log_category":"rule_engine","log_level":"error","log_message":"caught python exception","python_exception":"Traceback (most recent call last):\n  File \"/etc/irods/irods_rules/irods_rules/rodsutils.py\", line 203, in pep_wrapper\n    result = func(*funcargs)\n  File \"/etc/irods/irods_rules/irods_rules/policy.py\", line 300, in collection_tiering_locked\n    repeat_collection_tiering(callback, collection, ticketid)\n  File \"/etc/irods/irods_rules/irods_rules/tiering.py\", line 222, in repeat_collection_tiering\n    ret_val = apply_collection_tiering(callback, collection, ticket)\n  File \"/etc/irods/irods_rules/irods_rules/tiering.py\", line 484, in apply_collection_tiering\n    if not equal_states and (iDS - iS).empty():\nTypeError: unsupported operand type(s) for -: 'iState' and 'iState'\n","request_api_name":"EXEC_RULE_EXPRESSION_AN","request_api_number":1206,"request_api_version":"d","request_client_user":"rods","request_host":"127.0.0.1","request_proxy_user":"rods","request_release_version":"rods4.3.2","rule_engine_plugin":"python","server_host":"rivm-bioir-l04p.rivm.ssc-campus.nl","server_pid":3025936,"server_timestamp":"2025-09-16T11:51:05.852Z","server_type":"agent","server_zone":"rivmZone"}
```
The exact python error is almost always clearly logged here.


### spacecheck

This stage is meant to check if there is enough space on the working directory. Errors in this stage are unlikely, and the check result is logged in the runsheet log file.

The free space will be calculated using the following formula:
```
estimated free space = current free space - (download_jobs + 1) * JOBENGINE_JOB_RESERVED_INPUT_SIZE - (queued_or_active_jobs + 1) * JOBENGINE_JOB_RESERVED_OUTPUT_SIZE
```
To prevent having to calculate collection sizes for all running jobs, a reservation is made for all download jobs. (See [download stage](#download)). For all active jobs, space is reserved for the output data. This is generally much less, so the reservation is lower. This means that a job could remain in `spacecheck` if there are many download jobs running. Check the runsheet log to verify this.

## download

The jobengine downloads the input data to the working directory. Issues in this stage are uncommon. If downloading fails, try the job again. If it still fails, this might indicate a problem with a dataobject in the collection. Try downloading the collection manually.

## queued

In the `queued` state, the jobengine checks the project concurrency limit, and the process concurrency limit:

* The `project concurrency limit` (MAX_CONCURRENT_JOBS_PER_PROJECT) prevents too many jobs running for the same project.
* The `process concurrency limit` prevents starting too many instances of the same process (regardless of the project it runs in). This can be set (for example) if the process uses specific external resources.

If the limits are exceeded, this is logged in the runsheet log.

If the limits are not exceeded, the pipeline is started. All issues here are logged in the runsheet log.

## active

In this stage, the jobengine checks if the pipeline is still running. If it is not, it moves to the `finishing` state.

## finishing

The jobengine starts intoRods to import the pipeline output data. This import tended to fail sometimes in the past, so it can be tried multiple times. For this purpose, a post counter attribute is stored on the runsheet collection. The intoRods process is started in the background.

## postprocessing

Wait for the intoRods import process to finish. If it fails, go back to the `finishing` state if  maximum retry count is not exceeded. 
If the pipeline output is imported sucessfully:

* store some metadata on the output, indicating the status.
* if the pipeline is a distribute pipeline, start the [data distribution](#distribution)
* Cleanup the work directory
* Release locks on collections
* Remove temporary runsheet metadata

State changes to `notify`

## notrun

This state can only be reached from `depends`, if a dependant job fails. It will set the job result to NOTRUN and move to `notify`

## notify

This state will create user notifications about the pipeline result. User subscriptions to notifications are stored in BioREST.
In case of processgroups, notifications are only sent if the complete processgroup is finished.

## done 
This is the final runsheet state. Nothing happens



-----------------------
The jobengine states and transitions are depicted below


```mermaid
flowchart TD

  INCOMING[INCOMING]
  click INCOMING "/bioinformatics/DevOps/-/wikis/dms/jobengine#incoming" "Incoming"
  DEPENDS[DEPENDS]
  click DEPENDS "/bioinformatics/DevOps/-/wikis/dms/jobengine#depends" "Depends"

  CHOOSE[CHOOSE]
  click CHOOSE "/bioinformatics/DevOps/-/wikis/dms/jobengine#choose" "Choose a cluster"
  PREPARE[PREPARE]
  click PREPARE "/bioinformatics/DevOps/-/wikis/dms/jobengine#prepare" "Prepare"
  STAGE[STAGE]
  click STAGE "/bioinformatics/DevOps/-/wikis/dms/jobengine#stage" "Stage: download input data"

  QUEUED[QUEUED]
  click QUEUED "/bioinformatics/DevOps/-/wikis/dms/jobengine#queued" "Queued"
  ACTIVE[ACTIVE]
  click ACTIVE "/bioinformatics/DevOps/-/wikis/dms/jobengine#active" "Active"
  FINISHING[FINISHING]
  click FINISHING "/bioinformatics/DevOps/-/wikis/dms/jobengine#finishing" "Finishing"
  POSTPROCESSING[POSTPROCESSING]
  click POSTPROCESSING "/bioinformatics/DevOps/-/wikis/dms/jobengine#postprocessing" "Postprocessing"
  ERROR{ERROR}
  click ERROR "/bioinformatics/DevOps/-/wikis/dms/jobengine#error" "Error"
  NOTRUN[NOTRUN]
  click NOTRUN "/bioinformatics/DevOps/-/wikis/dms/jobengine#notrun" "Not run"
  NOTIFY[NOTIFY]
  click NOTIFY "/bioinformatics/DevOps/-/wikis/dms/jobengine#notify" "Notify"
  

  subgraph DONE[DONE]
  FINISHED_OK[OK]
  click FINISHED_OK "/bioinformatics/DevOps/-/wikis/dms/jobengine#done" "Done"
  FINISHED_ERROR{ERROR}
  click FINISHED_ERROR "/bioinformatics/DevOps/-/wikis/dms/jobengine#error" "Error"
  FINISHED_NOTRUN[NOTRUN]
  click FINISHED_NOTRUN "/bioinformatics/DevOps/-/wikis/dms/jobengine#notrun" "Not run"
  end

    INCOMING --> CHOOSE
    INCOMING ----> ERROR

    DEPENDS --> CHOOSE
    DEPENDS ----> ERROR
    DEPENDS ----> NOTRUN

    CHOOSE ----> ERROR
    CHOOSE --> PREPARE

    PREPARE --> STAGE
    PREPARE ----> ERROR
    PREPARE --> QUEUED

    STAGE --> QUEUED
    STAGE --> STAGE

    QUEUED --> QUEUED
    QUEUED ----> ERROR
    QUEUED --> ACTIVE

    ACTIVE --> ACTIVE
    ACTIVE ----> ERROR
    ACTIVE --> FINISHING

    FINISHING --> FINISHING
    FINISHING --> POSTPROCESSING

    POSTPROCESSING --> POSTPROCESSING
    POSTPROCESSING --> ERROR
    POSTPROCESSING --> FINISHING
    POSTPROCESSING ----> NOTIFY
    ERROR --> NOTIFY
    NOTRUN --> NOTIFY
    NOTIFY --> DONE

    classDef redNode fill:#f22
    class ERROR redNode
    class FINISHED_ERROR redNode
```




## Cluster integration

The jobengine can support different cluster types and allows for extending this support to other clusters by creating a `cluster integration module`

This module should define a class that inherits from `cluster_base`. In this class, various member functions can overwrite base functionality. The module should also import `factory` from `cluster_base.py` and register itself using the line
```
factory.register_builder('<name>', <class>.factory)
```
where name is replaced by a cluster name, and class by the newly defined cluster class.

### Cluster integration configuration

Specific instances of a cluster type have to be defined in the `clusters.yml` configuration file. A typical cluster config file could look like:
```
sites_and_clusters:
  RIVM:
    primary: true  
    clusters:
      RIVM-LSF-PRD:
        type: lsf
        module: lsf_2
        cost: 100
        hosts:    
          head_node:
            name: rivm-biohn-l02p.rivm.ssc-campus.nl  
            account: svc-sscc-irods
        workdir: '/mnt/irodsscratch/pipelines/prod'
        default_queue: 'bio-prio'
        queues:
          - bio
          - bio-prio
          - bio-gpu-v100
      RIVM-LSF-ACC:
        type: lsf
        module: lsf_2
        cost: 100
        hosts:    
          head_node:
            name: rivm-biohn-l03a.rivm.ssc-campus.nl  
            account: svc-sscc-irods
        workdir: '/mnt/scratch_dir/svc-sscc-irods/pipelines'
        default_queue: 'bio-prio'
        queues:
          - bio-acc
          - bio-prio-acc
      HNP1:
        type: host
        module: single_host
        cost: 100
        hosts:
          head_node:
            name: rivm-biohn-l02p.rivm.ssc-campus.nl
            account: svc-sscc-irods
        workdir: '/mnt/scratch_dir/svc-sscc-irods/pipelines'
      HNA1:
        type: host
        module: single_host
        cost: 100
        hosts:
          head_node:
            name: rivm-biohn-l03a.rivm.ssc-campus.nl
            account: svc-sscc-irods
        workdir: '/mnt/scratch_dir/svc-sscc-irods/pipelines'
```
At toplevel, a `site` is defined, in this case `RIVM`. All clusters in a site should be able to access online storage resources that have the same site property set.
Multiple clusters can be defined. Some of the configuration parameters should always be present, but you can also add specific ones for specific cluster types. Below is a list of parameters that have to be present:
Parameter|Description
-|-
module|Name of the cluster integration module
workdir|Directory where input data and pipeline code is staged for the pipeline run
hosts|connect information for hosts that are being used. This could be a single head node. The `cluster_base` class refers to a *head_node* host, but this could be overwritten.

All other parameters are only used by specific cluster integration modules.

### Cluster integration module functions

The following `cluster_base` member functions can be overwritten to implement specific cluster functionality:

Function|Called from state|Parameters|Description
-|-|-|-
set_params|PREPARE|runsheet|Can be used to add cluster settinsg to the runsheet
download_irods|STAGE|runsheet, local_run_dir|Should download input data to the local run directory
prepare_pipeline|QUEUED|runsheet, local_run_dir|Can be used to generate jobscripts 
startup_command|QUEUED|runsheet|Should return a CMDResult containing the command to start the piipeline
get_starttime|ACTIVE|runsheet, local_run_dir|Should return the pipeline start time
upload_irods|ACTIVE|runsheet, local_run_dir, collection, rel_path, resource|Upload data to iRODS, used for syncing log files during the pipeline run
job_active|ACTIVE|runsheet|Check if pipeline job is still running
kill_jobs|ACTIVE|runsheet|Kill all pipeline jobs
get_result|FINISHING|runsheet, local_run_dir|Return the result (exit code) of the pipeline run
import_result|FINSIHING|runsheet, local_run_dir|Upload the pipelien output to iRODS
cleanup|POSTPROCESSING|runsheet, local_run_dir|Remove run directory and other temporary structures

Check the `cluster_base.py` module for implementation details.

```mermaid
graph TD

  INCOMING
  DEPENDS

  click INCOMING "/my-linked-page" "This is a link"

  subgraph RUNNING[Running]
  CHOOSE
  PREPARE
  STAGE
  QUEUED
  ACTIVE
  FINISHING
  POSTPROCESSING
  NOTIFY
  end

  subgraph CLUSTER[Cluster]
  test_cluster
  set_params
  download_irods
  prepare_pipeline
  startup_command
  get_starttime
  upload_irods
  job_active
  kill_job
  get_result
  import_result
  cleanup
  end

  test_cluster -.- set_params -.- download_irods -.- startup_command -.- prepare_pipeline
  prepare_pipeline -.- get_starttime -.- kill_job -.- job_active
  job_active -.- upload_irods -.- get_result -.- import_result -.- cleanup

  subgraph FINISHED[Finished]
  DONE
  end

    INCOMING --> CHOOSE
    DEPENDS --> CHOOSE
    DEPENDS --> NOTRUN
    CHOOSE --> PREPARE
    CHOOSE --> test_cluster
    PREPARE --> STAGE
    PREPARE --> QUEUED
    PREPARE --> set_params
    STAGE --> QUEUED
    STAGE --> download_irods
    QUEUED --> ACTIVE
    QUEUED --> prepare_pipeline
    QUEUED --> startup_command
    ACTIVE --> FINISHING
    ACTIVE --> get_starttime
    ACTIVE --> upload_irods
    ACTIVE --> job_active
    ACTIVE --> kill_job
    FINISHING --> POSTPROCESSING
    FINISHING --> get_result
    FINISHING --> import_result
    POSTPROCESSING --> FINISHING
    POSTPROCESSING --> NOTIFY
    POSTPROCESSING --> cleanup
    NOTRUN --> NOTIFY
    NOTIFY --> DONE


```


## Data staging process

In the staging phase of the jobengine process the input collection(s) is/are replicated to the required irods resource. Unlike normal data management operations, this is not controlled by the `irods_dm.py` process, but by the `irods_stage.py` process. This is actually a lightweight version of `irods_dm.py` that only evaluates collections that are required by active pipelines.

For controlling the staging process, three metadata attributes are used. They are:
ATTR NAME|attr in use|Value|Description
-|-|-|-
ATTR_ARCHIVE_STAGE|sys::archive::stage|\<site>|Is set to the site where a pipeline is running, if the input data should be on a resource in site \<site> with ATTR_RESOURCE_STAGE set to true. This is the case when the process property *Download data from iRODS to input directory* is set.
ATTR_ARCHIVE_ONLINE|sys::archive::online|\<site>|Is set to the site where a pipeline is running, if the input data should be on a resource in site \<site> with ATTR_RESOURCE_ONLINE set to true. This is the case when the process property *Ensure data is online before starting process* is set.
ATTR_PIPELINE_USEDBY|sys::pipeline::used_by|Runsheet name|Is set on a collection when a pipeline uses the data. Can be set multiple times. It allows the cleanup function in `irods_dm.py` to evaluate if there are any ATTR_ARCHIVE_STAGE or ATTR_ARCHIVE_ONLINE avus on a collection that can be removed.


## Flowchart of relevant staging processes
```mermaid
flowchart TD
subgraph Dataset
stage("ATTR_ARCHIVE_STAGE") ~~~
online("ATTR_ARCHIVE_ONLINE") ~~~
used_by("ATTR_PIPELINE_USEDBY")
end

style Dataset fill:#81d5e6,stroke:#333,stroke-width:2px

subgraph jobengine
START1("START") ---> 
prepare -. set to runsheet .-> used_by
stage1("stage") -. set to site .-> stage & online
stage1 --> run(".. run pipeline ..")
run --> finishing -. clear .-> used_by
prepare --> stage1
end

subgraph irods_dm
START("START") --> 
query2["query irods"] -. query .->  stage & used_by & online
query2 --> check_active{"Are these active?"}
check_active -- Y --> END
check_active -- N --> cleanup
cleanup -. remove .-> stage & online
cleanup -. remove .-> used_by 
cleanup --> END("END")
end

subgraph irods_stage
start("START")
query["query irods"] -. query .-> stage
stage_data["Set desired_state"]
start --> query --> stage_data
stage_data --> end2("END")
end
```

* jobengine:
  * sets ATTR_PIPELINE_USEDBY to the runsheet name
  * depending on the process property *download input data to run directory* sets either `ATTR_ARCHIVE_STAGE` or `ATTR_ARCHIVE_ONLINE` to the staging SITE
---

* irods_stage:
  * reads staging SITES
  * sets appropriate desired_states


* jobengine:
  * removes ATTR_PIPELINE_USEDBY after pipeline is finished


* irods_dm
 
  This process is reponsible for cleanup of STAGE, ONLINE and USED_BY attributes
  * searches for ATTR_ARCHIVE_STAGE
    * check if ATTR_PIPELINE_USEDBY is an active runsheet
    * optionally removes ATTR_ARCHIVE_STAGE
    * optionally removes ATTR_ARCHIVE_ONLINE
    * optionally removes ATTR_PIPELINE_USEDBY

## Distribution

A pipeline can distribute data to new collections in other projects. For this to work:

* The pipeline needs to have the __Distribution pipeline__ property enabled
* The pipeline output directory has to have a file with name `distribute.yaml`, `distribute.yml` or `distribute.json`.

Two versions of the distribute format are supported:

### Distribute version 1
If no version property is specified in the distribute file, it is assumed to be version 1.

This format allows simple moves of top level subcollections in the output or input colletion of a pipeline. Whether the output or the input is being processed depends on the process property __Modifies input collection, no output__
```
---
folders:
  outfolder1:
      path: outfolder1
      next_projectID: project2
      next_processID: proj2_start
      next_processgroup: group2
  outfolder2:
      path: outfolder2
      next_projectID: project5
      next_processgroup: proj5_start
  outfolder3:
      path: outfolder3
      next_projectID: project9
      next_processID: proj9_start
```

For each entry, the data in path __path__ will be moved to a new collection in the project indicated in __next_projectID__.
The processID or the processgroupID are then stored on this new collection as metadata. Also, the `complete` attribute is set, so a pipeline will be started, if this is configured for this project.

### Distribute version 2

This format allows for more fine grained distribution of subcollections or dataobjects, copying of dataobjects, potentially to multiple projects, and setting metadata to collections or dataobjects in the distributed project collections.

Example:
```
---
version: 2

objects:
# COPY <src>/data1/* to <dest>/data1/*
-   cmd: copy
    projectID: test44
    source: data1
    dest: .
# COPY <src>/data2/subdir2/* to <dest>/subdir2/*
-   cmd: copy
    projectID: test44
    source: data2/subdir2
    dest: .
# COPY <src>/data3/subdir3/* to <dest>/output3/*
-   cmd: copy
    projectID: test44
    source: data3/subdir3/
    dest: output3
# COPY <src>/data4/subdir4/* to <dest>/output4/*
-   cmd: copy
    projectID: test44
    source: data4/subdir4/
    dest: output4
# COPY <src>/data5/subdir5/* to <dest>/output5/*
-   cmd: copy
    projectID: test44
    source: data5/subdir5/
    dest: output5/
# COPY <src>/data6/subfile6 to <dest>/output6/outfile6
-   cmd: copy
    projectID: test44
    source: data6/subfile6
    dest: output6/outfile6
# COPY <src>/data7/subfile7 to <dest>/output7/subfile7
-   cmd: copy
    projectID: test44
    source: data7/subfile7
    dest: output7
# COPY <src>/data8/subfile8 to <dest>/output8/subfile8
-   cmd: copy
    projectID: test44
    source: data8/subfile8
    dest: output8/
       
# move (1)
-   cmd: move
    projectID: test44
    source: data11
    dest: .
# move (2)
-   cmd: move
    projectID: test44
    source: data12/subdir12
    dest: output12
# move (3)
-   cmd: move
    projectID: test44
    source: data13/subdir13/
    dest: output13
# move (4)
-   cmd: move
    projectID: test44
    source: data14/subdir14/
    dest: output14
# move (5)
-   cmd: move
    projectID: test44
    source: data15/subdir15/
    dest: output15/
# move (6)
-   cmd: move
    projectID: test44
    source: data16/subfile16
    dest: output16/outfile16
# move (7)
-   cmd: move
    projectID: test44
    source: data17/subfile17
    dest: output17
# move (8)
-   cmd: move
    projectID: test44
    source: data18/subfile18
    dest: output18/

# Store metadata on the root collection in project test44
metadata:
    - projectID: test44
      path: .
      type: collection
      avus:
        - attr: processgroupID
          value: default
        - attr: sys::data::type
          value: fastq

```

Note the explicit version property in this format.

Notice the '/' behind directory names to copy/move only the content content, and not copy/move the whole dicretcory (creating a subdirectory).
