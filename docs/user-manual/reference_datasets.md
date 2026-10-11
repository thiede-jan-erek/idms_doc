
#Reference Datasets

<!--
| Chapter | Title |
|----|----|
| **1.** | [Introduction](#introduction)|
| **2.** | [Using reference datasets in a pipeline](#using-reference-datasets-in-a-pipeline) |
| **3.** | [How to create a new reference dataset](#how-to-create-a-new-reference-dataset) |
-->

## Introduction
Unlike collections (datasets) that are created by pipelines and belong to a specific project, reference datasets are for general use by pipelines of _any_ project.

You might be familiar with the **deprecated** storage of reference datasets under `/mnt/db/...`.
Keep in mind that any dataset stored outside of a Data Management System such as iDMS cannot be used to reliably reproduce pipeline results! Therefore, we strongly recommend to put all reference data into the iDMS to be inline with the **FAIR** principles.

In the iDMS, reference datasets are **automatically updated** in regular intervals and each change will lead to a new version. All previous versions are kept in the iDMS and can still be used in pipeline processes if this is desired.
The iDMS will keep provenance information of each generated collection and will link it to the specific version of a reference dataset that was used during processing.

The content of a reference dataset can be both _data_ (e.g., kraken2, BLAST, dbSNP) or _algorithms/tool workflows_ with executable code (e.g., a singularity container). The content is public within RIVM and readable by everyone. If you need a solution for sensitive data, please contact us.

In the Collection Browser in [iDMS](https://biorods.rivm.nl/) you can find all available reference datasets and their versions under: `/rivmZone/reference_dbs/`

Alternatively, you can use "Reference Datasets" in the left hand menu (Fig. 1). 

![image](uploads/d46d488b6929d681d80dfca1b97ca0b7/image.png)
*Figure 1.* 

If in the menu to the left does not yet show this option, in your user settings (Fig. 2.a.) you can activate the option with "Show reference data configuration" (Fig. 2.b.):

![refdata_user_options_menu](uploads/f4268d87aa67d56d388457984f280405/refdata_user_options_menu.png)
*Figure 2.a.*

![refdata_user_options](uploads/f642a6f3c0d80275e5c30cd6486284d8/refdata_user_options.png)
*Figure 2.b.* 

A list of the available reference datasets will now be shown, as described in the next section.

### List of available reference datasets

The list within Reference Datasets (Fig. 3) displays the following: 
* the name of a reference dataset,
* how long ago the dataset was updated (i.e., when the last version was created),
* the number of available versions
* the current state of the dataset: 
    - gray = deactivated
    - red = errors during last update 
    - yellow = suspiciously long since last update 
    - white = all fine

![refdata_dataset_list](uploads/ec2a400436da59d7ee161aefeb1db2c7/refdata_dataset_list.png)
*Figure 3.* 

By selecting one reference dataset from this list you open the details page, which contains several tabs:

The **settings** tab (Fig. 4) shows a description of the dataset along with several settings. To change any settings you need to be a privileged user. A complete description of the available settings can be found in the chapter [How to create a new reference dataset](#how-to-create-a-new-reference-dataset).

![refdata_dataset_details](uploads/9ab811c05130467a3582071e5431b43f/refdata_dataset_details.png)
*Figure 4.* 


The **contacts** tab (Fig. 5) allows you to subscribe to notifications. Depending on the selected options you will receive an email each time a new version becomes available, an error occurred during the update process, or a setting was changed:

![refdata_contacts](uploads/bc947b42e01a9eec8e5262d73cd5c6b0/refdata_contacts.png)
*Figure 5.* 

The **versions** tab (Fig. 6) lists all available versions for this dataset. Each version is a collection with a consecutive number as the name and a unique identifier, the _Dataset id_, as meta data.
Additionally, it is possible to assign a human readable name to a version for easy identification. This could be a version tag, a date, or some other marker. In Fig 6. version 1 got a human readable name assigned ("113.4").

If it turns out a particular version is defective, either because of a mistake on the source side or a technical problem during downloading, the administrators can mark a version as *unusable* (red). This version of this reference data will no longer show up in the selection lists for processes. A defective version will be kept in the iDMS to keep complete provenance information for related collections.  In Fig 6. version 0 was marked as defective, which is indicated by the red background.                                                                                        

![refdata_versions](uploads/49e9db9f68856da59c6938acbf9f8acc/refdata_versions.png)
*Figure 6.* 


## Using reference datasets in a pipeline

To use an _existing_ reference dataset in your pipeline processes, you can follow these steps:

1. For any _process_ you can define which version of a dataset in the iDMS needs to be used (see the section [Process Definition](#process-definition)).
2. Optionally, you can overwrite these process settings in a _process group_ (e.g., different projects might use the same process but different versions of reference datasets)(see the section [Process Group Definition](#process-group-definition)).
3. The iDMS will make the required versions available and generates an `input.yml` in the working directory of the _pipeline_. You will can parse the content of this generated file to find the physical location of the datasets (see the section [Pipeline Code](#pipeline-code)).

These steps will be described in detail below.


### Process Definition

First you have to define which version of a reference dataset should be presented to a pipeline process.
To do this you need to access the _process_ definition and select the *reference data: from the left menu click `Processes`, followed by the required process, and then the tab `ReferenceData`.

Here you can add as many reference dataset placeholders as you need.

1. For each placeholder you first define a symbolic name (identifier) under `Label`. This identifier will later be used in the generated `input.yml` to point to a dataset and version.
2. Then you select one of the existing datasets under `ReferenceDatasets`. 
3. Lastly,  under `Version` you can either select one specific version of the selected dataset, or choose 'latest (dynamic)' for the most recent version.


Fig. 7 shows the process "SC2LFV Clean". When running this process, two reference datasets are used.
* under the symbolic name "Kraken2Standard" it will get the "lates (dynamic)" version of the reference dataset "kraken2_blm_standard".
* under the symbolic name "SingularityContainers" we could select any of the available versions of the reference dataset "nrs_containers_import_irods" .

![refdata_process_detail_resized](uploads/f3994a1e293d940efc01f154cdc0debe/refdata_process_detail_resized.png) *Figure 7*

When the pipeline code of this process SC2LFV Clean is executed, it can read the file `inputs.yml` that is created that will contain for each of these symbolic names the path to the location of the dataset version to use. In the section [Pipeline code](#pipeline-code) we will show and explain this file.

It is possible to select the same reference dataset several times, e.g., if you want to compare the latest version against a another version.


### Process Group Definition (optional)

An alternative (and optional) way to add reference data to a process is through the process group definition. The versions selected in the process definition (as described above) are used by default. However, different projects might want to use different versions of a dataset. For this reason, it is possible to overwrite the selected versions in the _Process Group_ Definition: from the left hand menu click `Projects`, then the project with the process linked that you want to adjust, followed by the tab `Processgroups`.

Fig. 8 shows for the project "ssa" the `Processgroup` "default" and the process "SC2LFV Typing". In the bottom right section "Reference Data" we can see that this process requires 3 reference datasets under the symbolic names:
* SigularityContainers
* Freyja
* Pangolin

The default defined for the process in each case is "Latest (dynamic)". The selection drop-down menu's allow you to overwrite this default and select a specific version to use _in this processgroup_.

![refdata_processgroup](uploads/682419289c552d0f6b63f96e86c435e8/refdata_processgroup.png) *Figure 8*


### Pipeline code

Now you are halfway there to define in the iDMS which reference datasets should be available for a pipeline. You also need to change your pipeline code to actually use the version the iDMS made available. 

During runtime of a process, the working directory will contain a file generated automatically by iDMS: `inputs.yaml`. This file contains a list of symbolic names for the datasets and for each of those a specific dataset_id and a path on the filesystem. Exmaple 1 shows the general structure of this file with nested `key: value` pairs specific to `.yaml` files. Note the first line `reference_data_versions`.

```
reference_data_versions:
  <identifier>:
    dataset_id: <hash>
    path: <path_to_version>
  <identifier>:
    dataset_id: <hash>
    path: <path_to_version>
  ...
``` 
Example 1

Example 2 illustrates how the file appears in the SC2LFV Typing process. According to the process definition, the file lists the symbolic names SingularityContainers, Freyja, and Pangolin. As defined in the Processgroup, iDMS provides the ‘latest (dynamic)’ version of each dataset, which in this case are versions 5, 7, and 21 respectivly. iDMS is responsible for resolving the correct dataset versions, making them available on the filesystem, and generating the input.yaml file. If the same pipeline is executed at a later time, a new input.yaml will be generated with potentially updated dataset versions.

```
reference_data_versions:
  SingularityContainers:
    dataset_id: c458afdb-e99e-48fa-8a98-f484909797f9
    path: /mnt/db/refdata/reference_dbs/nrs_containers_import_irods/5
  Freyja:
    dataset_id: ac4e8723-3034-4046-9318-223300247b96
    path: /mnt/db/refdata/reference_dbs/freyja_data/7
  Pangolin:
    dataset_id: ce0e43d7-d745-4e6f-abb0-f1ec3bc437bb
    path: /mnt/db/refdata/reference_dbs/pangolin_data/21
``` 
Example 2

````markdown
In your code, you can refer to the `path` key for each of the datasets. For instance in a Snakemake pipeline, for the first dataset this would look something like:
```
config["reference_data_versions"]["SingularityContainers"]["path"]
```
````

## How to create a new reference dataset

Datasets that contains publicly available data, user data, or singularity-containers that should be accessible to all iDMS users, and subject to future changes/updates, qualify as a Reference Dataset.

The source of the data can be a public FTP server (e.g., NCBI dbSNP), a public website (e.g., Zenodo), a Git-Repository, or an internal share (e.g., `/mnt/scratch_dir/...` )

Similar to when you create a normal pipeline-processes, you need to create a public gitlab-repository to put your custom script that downloads the latest dataset from the source. This script is an executable bash script that is used as an entry point. Additionally, the gitab-repository should contain all necessary conda environment (.yaml) files to run the script (if needed).

Unlike a standard processes, this process is executed in regular intervals that can be defined when creating the dataset in iDMS (we will come back to that later). The downloaded files are automatically compared with the latest version in the iDMS. A new dataset version will be created only if differences have been found between the version in iDMS and the source.

You can find a simple example download script in this [github repo](https://gitlab.rivm.nl/bioinformatics/refdata_download_test), with the structure:

```
/env/refdata_download_test.yml
/src/some_script.py
/download.sh
/inspect.sh
/config.yml
```
The `env` directory contains the conda environment yaml files needed to run the download script `download.sh` (if needed). The `inspect.sh` script pre-checks if a dataset needs to be downloaded, i.e., inspect if there is a new version. The `config.yml`

### The download script

The download script is always called with at least one argument for the working directory:
1. the `WORK_DIR`

The working directory is set by the DMS-process and will typically refer to a newly created directory. The script should create a `DATASET_DIR` sub-directory named `dataset`.
Any file stored within the `DATASET_DIR` subdirectory `dataset` will become part of the version/dataset uploaded to the DMS.
Logfiles and temporary files that are not supposed to become part of the version/dataset can be stored directly within the `WORK_DIR`. This is likely data not used for analysis. 

```shell
#!/bin/bash
#set -x
SCRIPT_DIR=$(dirname "$(readlink -f "${BASH_SOURCE[0]}")")

# check if needed environment exists, if not create it!
ENV_NAME="refdata_download_test"
if mamba info --envs | grep -q "^${ENV_NAME}\b"; then
   echo "updating environment"
   mamba env update -q -f ${SCRIPT_DIR}/env/${ENV_NAME}.yml
else
   echo "creating new environment"
   mamba env create -q -f ${SCRIPT_DIR}/env/${ENV_NAME}.yml
fi
echo "activating environment"
source activate ${ENV_NAME}


if [ -z "$1" ]; then
    echo "The script needs the target directory as argument, e.g. \"new\". Target-Directory will be emptied!"
    exit 1
fi

WORK_DIR=$1
DATASET_DIR="${WORK_DIR}/dataset"


rm -rf "$WORK_DIR" || true
mkdir -p "$DATASET_DIR"

echo "starting the download script"

python ${SCRIPT_DIR}/src/download_test.py --config config.yml --output ${DATASET_DIR} --logging ${WORK_DIR} 
```

If you want to use the same script for several different reference datasets, you can either use a configuration-file `config.yml` or add additional parameters to the script that can then be used in the web-interface of iDMS to the synchronization command (e.g. [NCBI reference genome download](https://gitlab.rivm.nl/bioinformatics/refdata_download_ncbi/-/blob/main/download_ncbi_organism.sh?ref_type=heads).

:warning: Be sure the download doesn't contain files with dynamic content that does not relate to the dataset itself. For instance, sometimes an additional metadata file is generated to log the amount of user downloads. This file is updated every day. Because the file is updated, it will cause the download script to run everyday as well, while the actual reference data is not new. In turn this creates a new version of the dataset in iDMS every time. This unneeded updating similarly can happen with datasets that are not updated, but their timestamps are.

:warning: Make the script executable with `chmod +x download.sh`

### Pre-check script

Downloading and/or creating datasets can take several hours. If the source also provides a fingerprint/hash of the contained files, like a checksum, it is possible to avoid unnecessary downloads (also see the warning in the section above).

In these cases, you can additionally implement a pre-check script.

The pre-check script is always called with 3 arguments:
1. FLAG_DIR
2. WORK_DIR
3. VERSION_DIR

The script needs to indicate in the FLAG_DIR if differences, or changes, have been found or not. The script does this by creating a file, called a flagfile, in the FLAG_DIR directory:
* `precheck_finished__no_changes.txt` if changes have not been found
* `precheck_finished__found_changes.txt` if changes have been found

The WORK_DIR is the directory of the (potentially) new version. Again, this would also be the place where the log files are stored.

The VERSION_DIR references to the directory containing the last (older) version saved in the iDMS.

Your script needs to compare the hashes (e.g., [checksum](https://www.techtarget.com/searchsecurity/definition/checksum)) of the files contained in VERSION_DIR with the hashes on the source and will then create the appropriate flagfile. 


[coming soon] Here is an example pre-check script  `inspect.sh`.

### Web interface

After creating the download script you need to define a Reference Dataset Definition in the web interface. Only privileged users are able to do this.
After clicking on "Add new reference dataset" and setting a name, the new dataset is created.
By default this dataset is marked as "disabled". After filling in the form the dataset can be enabled with the button.

![refdata_new](uploads/1cb782a07fd65723f73f7e7efb0695c6/refdata_new.png) *Figure 9*

On the settings tab fill the following fields:

* Description (on the edit-tab)
  
  A short description of the source, of the content of each version and how other users are supposed to use it. You can use markdown for simple formatting.

* git-repository

  The gitlab repository of your download script. The visibility needs to be "public" and the git url needs to be given in the https-version, otherwise the DMS process will not be able to download the content.

* tag

  Drop-down menu offering you all tagged and all HEAD-commits of branches (e.g., "main", "v0.3").

* synchronization frequency

  This value defines how often the pre-check and download script is executed. 

* precheck command

  Here you define the shell script to execute for prechecking. This command can contain additional arguments and will be extended by 3 positional arguments as explained above. Example: `./check_bbc.sh --news-europe --economy` 


* synchronize command 

   Here you define the shell script to execute for downloading. The command can contain additional arguments and will be extended by 1 positional argument as explained above. Example `./download_blastdb.sh -nt`

* execution environment

  Here you can select between different environments the scripts will be executed in. Currently available are `local` for execution on the headnode and `standard` for execution on the cluster. If your script has unusual needs not met on the default queue we can provide additional environments as needed.

After defining all setting you need to activate the reference dataset!

The initial version will be created during the nightly update cycle. 

![refdata_new_settings](uploads/ab21de0de7794fe4715bd839ff9537d2/refdata_new_settings.png) *Figure 10*