Find a fastq file with a sample ID.

The preferred way to find samples is with metadata on the data object in iRODS. The metadata on data objects should be written from the pipeline that created the collection or a new pipeline can write the metadata on objects in a collection. In iDMS there are multiple ways for finding collections or metadata, these examples are to get the data in a script. If you have the metadata on the requested data object, it is by far the quickest way.

An example for an iquest to search for a data object with metadata:

```
$ iquest "%s/%s" "select COLL_NAME,DATA_NAME where META_DATA_ATTR_NAME = 'sample_id' and META_DATA_ATTR_VALUE = '22010315'"
/rivmZone/projects/s-cppa/230315_NB502001_0409_AHC2GNAFX5_0008/22010315_Pae_S11_R1_001.fastq.gz
/rivmZone/projects/s-cppa/230315_NB502001_0409_AHC2GNAFX5_0008/22010315_Pae_S11_R2_001.fastq.gz
```

If the pipeline that created the collection didn’t put any metadata on the data objects, you could search through all the filenames in iRODS. This can take a while depending on what is cached in the database.

A example of a iquest query to search through all filenames:

```
$ iquest "%s/%s" "select COLL_NAME,DATA_NAME where META_COLL_ATTR_NAME = 'sys::data::state' and META_COLL_ATTR_VALUE = 'valid' and DATA_NAME like 'D1092200884-01%fastq.gz'"
/rivmZone/projects/salm/221010_NB502001_0367_AH37K5AFX5_0007/D1092200884-01_S30_R1_001.fastq.gz
/rivmZone/projects/salm/221010_NB502001_0367_AH37K5AFX5_0007/D1092200884-01_S30_R2_001.fastq.gz
```

If you have access to the project (in this case salm) you can retrieve the fastq.gz file with:

```
$ iget -v /rivmZone/projects/salm/221010_NB502001_0367_AH37K5AFX5_0007/D1092200884-01_S30_R1_001.fastq.gz
   D1092200884-01_S30_R1_001     271.055 MB | 2.031 sec | 16 thr | 133.465 MB/s
```

Be sure you delete the files after use, one of the ideas of iRODS is that we don't need several copies of files, they are always accessible in iRODS. The best place is the scratch_dir, that's the place to retrieve data, run pipeline and clean it all again.

Another option for retreiving the data could be the [`mount_irods`](tools/mount_irods) command (we have some documentation about the command, see link when you open a terminal). This mount is only available on the head node, not on the lsf compute nodes.

Notes:

- We will provide a extra option to get you data from irods: NFSrods. This will make it possible to see all the data you have access to on the head nodes AND compute nodes.
- If you need help to put metadata on your data objects, please get in contact with us.
- `iquest attrs` to see all attributes you can use with iquest.