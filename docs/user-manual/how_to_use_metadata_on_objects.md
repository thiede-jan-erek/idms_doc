## Introduction

In iRODS, objects are files with metadata. This metadata enables you to find files quickly. At the moment it is possible to search through metadata on objects from the iRODS command line tools in the Scientific Workspace. We can use `imeta` and `iquest`. In the near future it will also be possible to search through metadata on objects in the web-interface [NGSweb](https://biorods.rivm.nl).

This documentation provides a use case to demonstrate how to work with metadata on objects, including how to search and retrieve objects using metadata and how to ask interesting "questions" of the data. In this documentation we will focus on the example projects in iRODS: **rvp_bp**, **rvp_spn** and **rvp_nm**. All projects use the [juno-assembly](https://github.com/RIVM-bioinformatics/juno-assembly) pipeline to generate assemblies from raw fastq files and the [juno-typing](https://github.com/RIVM-bioinformatics/juno-typing) pipeline for bacterial typing in the default process group.

Three extra pipelines need to be added to the process group of the projects (Figure 1). One pipeline is for setting metadata, such as sample name and file type, to the raw fastq files ([sample name to metadata](https://gitlab.rivm.nl/bioinformatics/samplename2meta)). Another pipeline is for setting metadata, such as assembly metrics, to the assembled fasta files ([metadata_obj_assembly](https://gitlab.rivm.nl/bioinformatics/metadata_obj_assembly)). The third pipeline is for setting metadata, such as serotypes to an object ([metadata_obj_typing](https://gitlab.rivm.nl/bioinformatics/metadata_obj_typing)).

![image](uploads/a15e0b59a5009d174f454f5461f26245/image.png) _Figure 1: Process group belonging to example project rvp_bp, showing the main Juno assembly and Juno typing pipeline with the additional pipelines to add metadata to objects._

> :bulb: In an ideal world, we would like to add these separate pipelines and their fairly simple code to the Juno assembly and the Juno typing pipelines directly, but that's for later.

Once this metadata has been added to the objects using these pipelines, we can start using the metadata for further processing. In the sections below, we will search though the metadata to find certain objects for further processing. These objects can be "place holder files" that are just used to contain metadata for searching. Or these objects can be files that are results from a certain analysis that contain metadata. After we have found the files of interest, we can download them. Alternatively, it is possible to aggregate all paths to these object of interest and upload this information to iRODS to start a pipelines without downloading anything!

But first we will discuss tools to query iRODS.

## iRODS Tools to Query

### `iquest`

To show all the the _.fastq.gz_ files from a project, e.g. _myco_, we can use the `iquest` tool (`iquest -h` for help). With this tool we can select the attribute fields for collection name `COLL_NAME`, and the object name `DATA_NAME` that has _.fastq.gz_ in the name.

> :bulb: `iquest attrs` can be used to list all attribute fields like `COLL_NAME` and `DATA_NAME`.

```
$ iquest --no-page "%s/%s" "SELECT COLL_NAME, DATA_NAME WHERE COLL_NAME like '/rivmZone/projects/myco%' and DATA_NAME like '%.fastq.gz'"
/rivmZone/projects/myco/131761_0002/NLA002000743_66813-A04_CACTATCAACCATTAACTGA_L001_R1_001_BHWFGYDMXX.filt.fastq.gz
/rivmZone/projects/myco/131761_0002/NLA002000743_66813-A04_CACTATCAACCATTAACTGA_L001_R2_001_BHWFGYDMXX.filt.fastq.gz
/rivmZone/projects/myco/131761_0002/NLA002000766_66813-B01_CGAGGCCAAGCCGGTAACAC_L001_R1_001_BHWFGYDMXX.filt.fastq.gz
/rivmZone/projects/myco/131761_0002/NLA002000766_66813-B01_CGAGGCCAAGCCGGTAACAC_L001_R2_001_BHWFGYDMXX.filt.fastq.gz
/rivmZone/projects/myco/131761_0002/NLA002000768_66813-C01_TTCACGAGACGTAACAATCT_L001_R1_001_BHWFGYDMXX.filt.fastq.gz
...
```

### `imeta`

The `imeta` tool is used to manage and query metadata (`imeta -h` for help)

#### `imeta ls`

To show the metadata on a object, we can use `imeta ls`. With the `-d` option we can query specifically for objects:

```
$ imeta ls -d /rivmZone/projects/rvp_bp/230927_NB552493_0208_AHC5WTAFX5_0058/de_novo_assembly_filtered/D1132300015-01.fasta
AVUs defined for dataObj /rivmZone/projects/rvp_bp/230927_NB552493_0208_AHC5WTAFX5_0058/de_novo_assembly_filtered/D1132300015-01.fasta:
attribute: #_contigs
value: 398
units: 
----
attribute: file_type
value: fasta
units: 
----
attribute: full_species_name
value: Bordetella pertussis
units: 
----
attribute: GC_(%)
value: 67.83
units: 
----
attribute: genus
value: bordetella
units: 
----
attribute: pipeline_name
value: juno_assembly
units: 
...
```

Or in the [webinterface](https://biorods.rivm.nl) look at the metadata window _Collection metadata_ (Figure 2).

![image](uploads/f9566b6bd730dd48105385d277fe0301/image.png) _Figure 2: Metadata on a collection in right lower corner panel of the NGSweb interface. Metadata "Attribute" is the name of the metadata field and the and "Values" is the corresponding value._

#### `imeta qu`

To get all the fasta files from project rvp_bp where the value of contigs are \< 500 we can use `imeta qu` tool (`imeta help qu` for help):

```
$ imeta qu -d project_id = rvp_bp and pipeline_name = juno_assembly \#_contigs n\< 500
collection: /rivmZone/projects/rvp_bp/230927_NB552493_0208_AHC5WTAFX5_0058/de_novo_assembly_filtered
dataObj: D1132300015-01.fasta
----
collection: /rivmZone/projects/rvp_bp/230927_NB552493_0208_AHC5WTAFX5_0058/de_novo_assembly_filtered
dataObj: D1132300016-01.fasta
----
collection: /rivmZone/projects/rvp_bp/231013_NB552493_0213_AHNVJNAFX5_0027/de_novo_assembly_filtered
dataObj: D1132300018-01.fasta
----
collection: /rivmZone/projects/rvp_bp/231025_NB552493_0217_AHV35GAFX5_0025/de_novo_assembly_filtered
dataObj: D1132300023-01.fasta
----
...
```

> :warning: notice the escaped (`\`) characters in the command

## Asking the interesting questions

Below we will discuss three interesting questions we can ask the data with metadata that was put on objects using our example pipelines (Figure 1).

### Question 1. For project _rvp_bp_, I would like to know the vaccine type (vaccine_antigen_mlst) (Juno typing pipeline) from my isolates and download the genome assemblies only if they have less than 500 contigs (Juno assembly pipeline) for quality control.

To answer this, we have to take multiple steps involving two main operations: 1) find the sample name(s) of the files with \< 500 contigs from the Juno assembly metadata, and 2) use the file names to get the Juno typing metadata that has the vaccine type.

**Step 1.** First, we retrieve all FASTA files from the project rvp_bp that were generated using the Juno Assembly pipeline and contain fewer than 500 contigs:

```
$ imeta qu -d project_id = rvp_bp and pipeline_name = juno_assembly and \#_contigs n\< 500
collection: /rivmZone/projects/rvp_bp/230927_NB552493_0208_AHC5WTAFX5_0058/de_novo_assembly_filtered
dataObj: D1132300015-01.fasta
----
collection: /rivmZone/projects/rvp_bp/230927_NB552493_0208_AHC5WTAFX5_0058/de_novo_assembly_filtered
dataObj: D1132300016-01.fasta
----
collection: /rivmZone/projects/rvp_bp/231013_NB552493_0213_AHNVJNAFX5_0027/de_novo_assembly_filtered
dataObj: D1132300018-01.fasta
----
collection: /rivmZone/projects/rvp_bp/231025_NB552493_0217_AHV35GAFX5_0025/de_novo_assembly_filtered
dataObj: D1132300023-01.fasta
----
collection: /rivmZone/projects/rvp_bp/241211_VH01799_174_AAG5NT3M5_0022/de_novo_assembly_filtered
dataObj: D1132401021-01.fasta
...
```

We now get all the objects (fasta files) with \< 500 contigs. The output above is not directly usable. By piping this output to an `awk` command, we can combine the path of the collection, to the object name in that collection. This file path we can use to get the metadata later on:

```
$ imeta qu -d project_id = rvp_bp and pipeline_name = juno_assembly and \#_contigs n\< 500 | awk '{if ($1=="collection:") a=$2; if ($1=="dataObj:") print a "/" $2}'
/rivmZone/projects/rvp_bp/230927_NB552493_0208_AHC5WTAFX5_0058/de_novo_assembly_filtered/D1132300015-01.fasta
/rivmZone/projects/rvp_bp/230927_NB552493_0208_AHC5WTAFX5_0058/de_novo_assembly_filtered/D1132300016-01.fasta
/rivmZone/projects/rvp_bp/231013_NB552493_0213_AHNVJNAFX5_0027/de_novo_assembly_filtered/D1132300018-01.fasta
/rivmZone/projects/rvp_bp/231025_NB552493_0217_AHV35GAFX5_0025/de_novo_assembly_filtered/D1132300023-01.fasta
/rivmZone/projects/rvp_bp/231114_NB552493_0224_AHFY3JBGXV_0025/de_novo_assembly_filtered/D1132300027-01.fasta
/rivmZone/projects/rvp_bp/231114_NB552493_0224_AHFY3JBGXV_0025/de_novo_assembly_filtered/D1132300028-01.fasta
/rivmZone/projects/rvp_bp/241211_VH01799_174_AAG5NT3M5_0022/de_novo_assembly_filtered/D1132401021-01.fasta
...
```

If we want, we can download the fasta file using these file paths with [`iget`](https://gitlab.rivm.nl/bioinformatics/docs/-/wikis/tools/use-iget-to-get-data-from-irods) or `igetwild`.

**Step 2.** Once we have the objects (fasta files) with \< 500 contigs, we want to know the sample names that are related to these files.

To get the `sample_name` metadata from one of the objects we can do:

```
$  imeta ls -d /rivmZone/projects/rvp_bp/241211_VH01799_174_AAG5NT3M5_0022/de_novo_assembly_filtered/D1132401021-01.fasta "sample_name"
AVUs defined for dataObj /rivmZone/projects/rvp_bp/241211_VH01799_174_AAG5NT3M5_0022/de_novo_assembly_filtered/D1132401021-01.fasta:
attribute: sample_name
value: D1132401021-01
units: 
```

**Step 3.** We now have the sample_name, in this case _D1132401021-01_. With this sample name we want to find the object with vaccine type metadata. The metadata attribute _analysis_ with value _vaccine_antigen_mlst_ from the Juno typing pipeline:

```
$ imeta qu -d sample_name = D1132401021-01 and pipeline_name = juno_typing and analysis = vaccine_antigen_mlst
collection: /rivmZone/projects/rvp_bp/241211_VH01799_174_AAG5NT3M5_0021/vaccine_antigen_mlst
dataObj: D1132401021-01.tsv
```

This output now contains the object with the vaccine_antigen_mlst analysis metadata field.

**Step 4.** With this object path, we can get the _vaccine type_ metadata field defined by _VA_ST_:

```
$ imeta ls -d /rivmZone/projects/rvp_bp/241211_VH01799_174_AAG5NT3M5_0021/vaccine_antigen_mlst/D1132401021-01.tsv "VA_ST"
AVUs defined for dataObj /rivmZone/projects/rvp_bp/241211_VH01799_174_AAG5NT3M5_0021/vaccine_antigen_mlst/D1132401021-01.tsv:
attribute: VA_ST
value: 4
units: 
```

To automate this a bit [metadata_objects_scripts repository](https://gitlab.rivm.nl/bioinformatics/metadata_objects_scripts):

```
./find_result1.sh  samples1

#"VA_ST" "sample_name" "object thats containts the metadata from the juno typing pipeline"
9 D1132300031-01 /rivmZone/projects/rvp_bp/231121_NB552493_0227_AHVMFGAFX5_0021/vaccine_antigen_mlst/D1132300031-01.tsv
4 D1132300039-01 /rivmZone/projects/rvp_bp/231122_NB502001_0046_AHWLJNAFX5_0021/vaccine_antigen_mlst/D1132300039-01.tsv
4 D1132300041-01 /rivmZone/projects/rvp_bp/231128_NB552493_0229_AHWJVFAFX5_0012/vaccine_antigen_mlst/D1132300041-01.tsv
9 D1132300046-01 /rivmZone/projects/rvp_bp/231129_NB502001_0047_AHWMF5AFX5_0021/vaccine_antigen_mlst/D1132300046-01.tsv
9 D1132300047-01 /rivmZone/projects/rvp_bp/231129_NB502001_0047_AHWMF5AFX5_0021/vaccine_antigen_mlst/D1132300047-01.tsv
9 D1132300043-01 /rivmZone/projects/rvp_bp/231204_NB552493_0231_AHY3VLAFX5_0016/vaccine_antigen_mlst/D1132300043-01.tsv
4 D1132300045-01 /rivmZone/projects/rvp_bp/231204_NB552493_0231_AHY3VLAFX5_0016/vaccine_antigen_mlst/D1132300045-01.tsv
9 D1132300042-01 /rivmZone/projects/rvp_bp/231208_NB552493_0233_AHWY33AFX5_0019/vaccine_antigen_mlst/D1132300042-01.tsv
4 D1132300049-01 /rivmZone/projects/rvp_bp/231208_NB552493_0233_AHWY33AFX5_0019/vaccine_antigen_mlst/D1132300049-01.tsv
4 D1132300051-01 /rivmZone/projects/rvp_bp/231213_NB502001_0050_AHY5WVAFX5_0019/vaccine_antigen_mlst/D1132300051-01.tsv
9 D1132300057-01 /rivmZone/projects/rvp_bp/231213_NB502001_0050_AHY5WVAFX5_0019/vaccine_antigen_mlst/D1132300057-01.tsv
4 D1132300067-01 /rivmZone/projects/rvp_bp/231228_NB552493_0237_AHG7MNBGXV_0030/vaccine_antigen_mlst/D1132300067-01.tsv
4 D1132300069-01 /rivmZone/projects/rvp_bp/231228_NB552493_0237_AHG7MNBGXV_0030/vaccine_antigen_mlst/D1132300069-01.tsv
```

### Question 2. For project _rvp_spn_ I would like to have all assemblies with serotype 19A, and know what the MLST type is.

To answer this, we can take similar steps involving the two main operations described in question 1.

**Step 1.** First we have to obtain the object that had the metadata _serotype_ with the value _19A_:

```
$ imeta qu -d project_id = rvp_spn and serotype = 19A
collection: /rivmZone/projects/rvp_spn/230213_NB552493_0138_AHC22TAFX5_0011/serotype/224001001401-1
dataObj: pred.tsv
----
collection: /rivmZone/projects/rvp_spn/230213_NB552493_0138_AHC22TAFX5_0011/serotype/224001002901-1
dataObj: pred.tsv
----
...
```

This gives us the collections and objects in the project rvp_spn with serotype set to 19A.

**Step2.** We can use these paths to find the sample name.

> :warning: In this case _sample_ is used as the metadata attribute in the pipeline and not _sample_name_.

```
$ imeta ls -d /rivmZone/projects/rvp_spn/230213_NB552493_0138_AHC22TAFX5_0011/serotype/224001002901-1/pred.tsv sample
AVUs defined for dataObj /rivmZone/projects/rvp_spn/230213_NB552493_0138_AHC22TAFX5_0011/serotype/224001002901-1/pred.tsv:
attribute: sample
value: 224001002901-1
units: 
```

This gives us the sample name _224001002901-1_.

**Step 3.** With the sample name we can query which dataobject has the metadata attribute _analysis_ equal to _mlst_:

```
$ imeta qu -d sample_name = 224001002901-1 and analysis = mlst
collection: /rivmZone/projects/rvp_spn/230213_NB552493_0138_AHC22TAFX5_0011/mlst7/224001002901-1
dataObj: results.txt
```

This gives us the object _results.txt_.

**Step 4.** We can now use the path to this object to get the MLST with the metadata attribute _ST_type_.

```
$ imeta ls -d /rivmZone/projects/rvp_spn/230213_NB552493_0138_AHC22TAFX5_0011/mlst7/224001002901-1/results.txt ST_type
AVUs defined for dataObj /rivmZone/projects/rvp_spn/230213_NB552493_0138_AHC22TAFX5_0011/mlst7/224001002901-1/results.txt:
attribute: ST_type
value: 165
units: 
```

The MLST for this object is _165_.

**Step 5.** Finally, we can retrieve the fasta file of this sample:

```
$ imeta qu -d sample_name = 224001002901-1 and file_type = fasta and pipeline_name = Juno_assembly
collection: /rivmZone/projects/rvp_spn/230213_NB552493_0138_AHC22TAFX5_0010/de_novo_assembly_filtered
dataObj: 224001002901-1.fasta
```

### Question 3. For project `rvp_nm` I would like to have the genome assemblies from serotype Y.

We can follow similar steps to question 1 and 2.

> :warning: Notice the different naming of the metadata attributes _serotype_ vs. _sero_group_.

**Step 1.** Obtain the objects:

```
$ imeta qu -d project_id = rvp_nm and sero_group = Y
collection: /rivmZone/projects/rvp_nm/221212_NB552493_0118_AH37KFAFX5_0012/serotype/220100931101
dataObj: neisseriatyper.tab
----
collection: /rivmZone/projects/rvp_nm/221212_NB552493_0118_AH37KFAFX5_0012/serotype/220101101101
dataObj: neisseriatyper.tab
----
collection: /rivmZone/projects/rvp_nm/240814_VH01799_103_AAFW7C2M5_0017/serotype/14-0001
dataObj: neisseriatyper.tab
----
collection: /rivmZone/projects/rvp_nm/240814_VH01799_103_AAFW7C2M5_0017/serotype/14-0003
dataObj: neisseriatyper.tab
----
...
```

This gives us four objects in two different collections that have the _sero_group_ attribute with the value _Y_.

**Step 2.** Get the sample name for (one of the) objects:

```
$ imeta ls -d /rivmZone/projects/rvp_nm/221212_NB552493_0118_AH37KFAFX5_0012/serotype/220100931101/neisseriatyper.tab sample_name
AVUs defined for dataObj /rivmZone/projects/rvp_nm/221212_NB552493_0118_AH37KFAFX5_0012/serotype/220100931101/neisseriatyper.tab:
attribute: sample_name
value: 220100931101
units: 
```

**Step 3.** Obtain the fasta file belonging to this object.

```
$ imeta qu -d project_id = rvp_nm and sample_name = 220100931101 and file_type = fasta and pipeline_name = Juno_assembly
collection: /rivmZone/projects/rvp_nm/221212_NB552493_0118_AH37KFAFX5_0011/de_novo_assembly_filtered
dataObj: 220100931101.fasta
```

# :brain: Insights and Recommendations

To truly leverage the power of metadata, it should be consistently applied to both objects and collections. The most effective and sustainable approach is to automatically add metadata during pipeline execution, rather than as a separate, post-hoc process. In this use case, metadata was added after the fact, which proved to be more labor-intensive and prone to inconsistencies.

Another key observation is the variability in metadata terminology. We found that different terms were often used for the same type of data, which undermines the ability to search, filter, and integrate metadata efficiently. Establishing and adhering to a set of controlled vocabulary or consensus metadata terms is essential for enabling automation, interoperability, and robust data governance.

Looking forward, integrating metadata standards and automation directly into data processing pipelines will significantly improve scalability, reproducibility, and discoverability of data.

??? note "Active Development"
    Currently, we are actively developing and evaluating how to use metadata terminology consistently within our iRODS system, to support more standardized and searchable metadata annotations.

# :medal: Acknowledgements

We would like to thank Linda Visser kindly for her time and help to provide us with this use case for metadata on objects. Till now she now was manually searching, e.g., specific sequence files in iRODS. The three example questions were based on her queries. Linda mostly worked with the iRODS projects and pipelines described in this documentation.