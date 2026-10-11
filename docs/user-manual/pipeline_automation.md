# Pipeline automation with iRODS + HPC

## Introduction

Pipelines (or any data-driven code) can be executed automatically when
new input data arrives in [iRODS][iRODS]. This is especially useful 
for common analysis steps or projects that very regularly collect new
data.

Benefits of triggering a pipeline automatically are:

* No time wasted, the pipeline starts as soon as data arrives.
* Less prone to human errors (e.g. errors that happen when one has to
  execute a similar, but slightly different command every week).
* Automatic import of results into iRODS, potentially triggering other
  dependent pipelines.
* Free provenance for pipeline results (i.e. information on what code
  was applied to what input data to generate these results).

A very simple example pipeline can be found [here](https://gitlab.rivm.nl/bioinformatics/test-pipeline-snakemake/). It uses a snakemake workflow to count the number of words in all files in the input folder.


## API

A pipeline can be configured to run automatically in iDMS when it has:

* A reachable Git repository (e.g. on https://gitlab.rivm.nl or 
  https://github.com/rivm-syso/). It is reachable by setting the repo to Public or providing it with a security Token.
* A `run_pipeline.sh` script in the top-level directory of that
  repository that calls runs your workflow/pipeline.

The system always passes three arguments to the `run_pipeline.sh` script. However, it is up to the script implementation whether and how these parameters are used.

The provided arguments are:

1. A path to a local directory with input data. This directory contains the data that needs to be processed by the pipeline.
2. Local empty directory for output data. Everything that is put in this directory will be imported back into the irods system as result of the pipeline.
3. Local directory with context data:
  * Two files, `object_metadata.json` and `dataset_metadata.json` with metadata of the input files and input collection
  * A `runsheet.json` file with run parameters

Most pipelines will at least use the first and second parameter for input and output of the pipeline respectively. The context files in the directory given by the third parameter can be used if your pipeline needs access to the runsheet details or the input collection metadata. 

Check [this page](/DMS/context_files) for an example of the formats of the files in the context directory and when you would use these files.

An example of the script with two arguments:

```bash
#!/usr/bin/env bash
echo "My input is in directory: ${1}"
echo "My output will be in directory: ${2}"
echo "Input collection ID: ${irods_input_sys__dataset_id}"
bsub -q bio "du -d 0 '${1}' > '${2}/du.txt'"
```
If you use Snakemake for your pipeline, we recommend looking at the page [Decorating your Snakemake pipeline](HPC/decorating_snakemake). Here an example run_pipeline.sh can be found with an explanation on how to effectively run a Snakemake pipeline on the HPC. If your pipeline is destined to become one that runs in iRODS, we advise to not use the `--conda-prefix` command as explained in [Working with Conda and Singularity environments](HPC/decorating_snakemake#working-with-conda-and-singularity-environments). Although this flag can be very useful, it can cause issues when a pipeline is run in parallel (different from Snakemake running rules in parellel), since conda cannot resolve environments in parallel. This is also the case for pipelines in [Nextflow](https://github.com/nextflow-io/nextflow/issues/1819). For singularity this is not an issue and `--singularity-prefix` can be used. Consequently, another option would be to [Containerize a Conda environment](tools/conda2singularity).

After execution of `run_pipeline.sh`:

* The return code of `run_pipeline.sh` is used to indicate the pipeline
  is finished successfully (return code `0`) or whether it has failed
  (return code nonzero) and consequentially the resulting output data 
  is *valid*.
* The data in the output directory (2nd parameter to the script) is
  imported into iRODS and a reference is put in the metadata pointing
  to the collection with input data.

An example of metadata on output of a pipeline:

| key  |                                 value |
|------|-----|
|`sys::dataset_id` |                     `5debc1e3-a36d-4657-b410-e229e8d3c8c4` |
|`sys::data::type` |                    `valid` |
|`sys::pipeline::githash` |	            `005c8a5db68622d6020498748691cae5ccddb757` |
|`sys::pipeline::gitrepo` |             `https://deploy-token:p@gitlab.rivm.nl/u/pipeline.git` |
|`sys::pipeline::input_collection_id` |  `7a3e988a-2384-4431-99ed-ba2fa148fb70` |


## Using a pipeline in a project

To use a pipeline in a project, you must first define the pipeline as a process, and then set it in a processgroup for the project.

### Creating a process

Go to the [processes page](http://biorods.rivm.nl/projects/?page=processes) of iDMS, and click the ´+ Add new process´ button in the Actions part of the sidebar menu. In the pop-up, fill in the name of your pipeline and click ´Submit´.

Now select the new process in the list of processes. In the Managers tab, you should see that you are now a manager of this process. If desired, you can add some extra managers to the process, so they can also change the settings of the pipeline.

In the Settings tab, you should now set a description of the pipeline, and the location of your pipeline on either gitlab or github. For example, the example pipeline we mentioned earlier can be found at https://gitlab.rivm.nl/bioinformatics/test-pipeline-snakemake.git . If your pipeline requires authentication, it is possible to pull the git repo with a token in this url.

### Setting a (default) processgroup

To set a (default) pipeline for your project, navigate to your project on the [projects page](http://biorods.rivm.nl/projects/) of iDMS. Then click on the ´Processgroups´ tab. There, you can create processgroups. A processgroup is a number of processes, run in a set order. The processgroup called ***default*** is run whenever a new collection is made in the project. 

> Because you can connect multiple processes in one processgroup, it is not necessary to put the whole pipeline in one git repo. You can simply use the output of one process as the input of the next!

To modify the default processgroup, you can click on boxes in the flow diagram to modify the settings of the corresponding process. For example, here you can set the version of your script that you want to use by selecting a git tag (also see next section about git tags for your pipeline).

Or you can add to the diagram by adding a process: fill in a name (optional), select the process you want to add and click the `Add process` button.

### Processes and Process-dependencies
Processes can be linked together linearly, where the output of one process becomes the input of the next process. In this case, the processes form a pipeline in which data flows sequentially from one step to the next.

In contrast, a process dependency means that a process still runs on its own input data, but it also depends on the completion or output of another process. The dependency is therefore not only about passing data directly from one process to another, but about requiring another process to finish before execution can proceed.

Process dependencies are only required in one of these cases:

- A process uses output of another process in addition to its _main_ input data
- The process that another process depends on has the property `Modifies input collection` enabled.

For instance, suppose process X depends on process Y, and both processes operate on NEW DATA. 

- In a new process group, you add both processes X and Y. 
- Select process X and, under `Process-Dependencies`, click `Add dependency` and select process Y.
- If process Y modifies the the data that process X uses, the `Modifies input collection` should be enabled in process Y. 

### Updating your pipeline to a new version
[When working with Git](https://gitlab.rivm.nl/bioinformatics/docs/-/wikis/software_engineering/git), it is important to keep track of changes and stable versions of your code. To manage stable versions, you typically work with [branches](https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell). Once your code has been tested, you can merge the branch into main (or master). After this merge, you can create a [Git tag](https://git-scm.com/book/en/v2/Git-Basics-Tagging) to mark the version and link it to your process (group) in iDMS. The following procedure outlines how to do this from start to finish using GitLab interface and iDMS:

1. Push your branch to [GitLab](gitlab.rivm.nl).
2. In the GitLab interface, find your branch in the remote repository and merge it into main (or master). A merge can be coupled to a merge request if working together with other people, of consider creating a merge request when working with others.
3. In GitLab, go to `Code` > `Tags` in the left-hand menu.
4. Click `New tag` and choose the tag name. (e.g., if current tag was v1.3, the new name would be v1.4).
5. From the drop down menu `Create from` select `main` (or `master`).

:bulb: You can also perform steps 1 through 5 via the command line. Please refer to the git help pages on [branching and merging](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging), [pushing commits](https://git-scm.com/docs/git-push), and [tagging](https://git-scm.com/book/en/v2/Git-Basics-Tagging).

You can now choose to change the tag for the process, affecting all projects related to that process, or change the tag for the process in a particular project only. 

#### For all projects related to a process
The first route to change the tag is to select the _process_ in iDMS:
 
6. After creating the tag, go to the process (group): `Processes` > `<process_name>`. 
7. Select the `Usage` tab.
8. Use the checkboxes to select which process (group) you want to update. You can only select the ones connected to projects you have rights to.
9. In the `New GIT TAG` dropdown, select the newly created the tag. 
10. Click `Update selected` to finalize.

#### For the process in one particular project only
Alternatively, the second route is to change the tag in the _processgroup configuration_ of a _project_. This allows you to only change the tag for the process linked to that project. If you have access to the project you can follow these steps instead:

6. After creating the tag, go to a project which has the process (group) coupled:`Projects` > `<project_name>`. 
7. Select the `Processgroups` tab. 
8. Select the processgroup from the dropdown. 
9. Click the process icon (arrow box). 
10. Below a menu will appear with `Actions`. Under `Actions` you can select the tag from `Git Tag` dropdown.
11. Finalize with `Submit`.

:warning:  If you follow this route, you will have to change the tag in every project that needs this new tag. If you want to change the tag for the process that will impact all projects, follow the first route. 

### Starting a pipeline manually

To manually start a pipeline, find the collection you want to use as input for the pipeline in iDMS. You can do this via the search function, or directly in the 'Collection browser'. Alternatively, if you know the iRODS path of the collection, you can directly go there by appending the path to the following url `http://biorods.rivm.nl/collbrowser?path=`, for example: `http://biorods.rivm.nl/collbrowser?path=/rivmZone_test_01/projects/testproject/testcollection`.

When you have found the collection, go to the pipeline tab in the top right panel. There, you have the option to either choose a Processgroup or a Single process and then start it on the current collection.


### Custom conditions for pipeline triggering

Currently this is manual configuration. Email one of [us](home) for possibilities.


<!-- References -->
[iRODS]: DMS/iRODS



