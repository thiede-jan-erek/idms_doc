1. Determine which data to get from irods
1. Select a place to put the data
1. The iget command

## Determine which data to get from iRODS

Go to [iDMS](http://biorods.rivm.nl), collection browser, your project and select the dataset/collection you want to get.

eg:


![select collection](uploads/eb1b682bd9fc41b13c6641fb0d784fad/image.png)
  

## Select a place to put the data

On the "Wetenschappelijke Werkplek" open a terminal and go to your scratch_dir.

eg:

```
$ cd /mnt/scratch_dir/verhager
```

## The iget command
If not already done, use `iinit` to connect to irods (use your RIVM e-account password).

The iDMS webpage states: "**Collection: /rivmZone/projects/salm/220104_NB502001_0283_AHFFN2AFX3_0006".**

Simply put iget in front of the collection:

```
$ iget -r -v /rivmZone/projects/salm/220104_NB502001_0283_AHFFN2AFX3_0006
```

options:

1. -r: copy recursively (al files and subdirs)
1. -v: verbose (show what you are doing)
1. -h: help for iget.

  
example:
```
$ iget -r -v /rivmZone/projects/salm/220104_NB502001_0283_AHFFN2AFX3_0006
D- ./220104_NB502001_0283_AHFFN2AFX3_0006 :
D- ./220104_NB502001_0283_AHFFN2AFX3_0006/multiqc_data :
   multiqc_data.json               2.635 MB | 0.580 sec | 0 thr |  4.544 MB/s
   multiqc_fastqc.txt              0.010 MB | 0.053 sec | 0 thr |  0.188 MB/s
   multiqc_general_stats.txt       0.004 MB | 0.074 sec | 0 thr |  0.054 MB/s
   multiqc.log                     0.029 MB | 0.060 sec | 0 thr |  0.490 MB/s
   multiqc_qualimap_bamqc_ge       0.000 MB | 0.057 sec | 0 thr |  0.000 MB/s
   multiqc_sources.txt             0.006 MB | 0.032 sec | 0 thr |  0.189 MB/s
   1092100925_S1_R1_001_fast       0.558 MB | 0.072 sec | 0 thr |  7.793 MB/s
   1092100925_S1_R1_001_fast       0.425 MB | 0.132 sec | 0 thr |  3.224 MB/s
   1092100925_S1_R1_001.fast     305.399 MB | 6.805 sec | 16 thr | 44.881 MB/s
   1092100925_S1_R2_001_fast       0.572 MB | 0.115 sec | 0 thr |  4.974 MB/s
   1092100925_S1_R2_001_fast       0.430 MB | 0.056 sec | 0 thr |  7.685 MB/s
   1092100925_S1_R2_001.fast     309.971 MB | 5.919 sec | 16 thr | 52.373 MB/s
   1092100926_S2_R1_001_fast       0.557 MB | 0.067 sec | 0 thr |  8.276 MB/s
   1092100926_S2_R1_001_fast       0.425 MB | 0.086 sec | 0 thr |  4.951 MB/s
   1092100926_S2_R1_001.fast     223.995 MB | 5.029 sec | 16 thr | 44.540 MB/s
   1092100926_S2_R2_001_fast       0.571 MB | 0.075 sec | 0 thr |  7.568 MB/s
   1092100926_S2_R2_001_fast       0.429 MB | 0.112 sec | 0 thr |  3.817 MB/s
   1092100926_S2_R2_001.fast     227.928 MB | 5.256 sec | 16 thr | 43.367 MB/s
   1092100927_S3_R1_001_fast       0.556 MB | 0.061 sec | 0 thr |  9.119 MB/s
   1092100927_S3_R1_001_fast       0.423 MB | 0.060 sec | 0 thr |  7.002 MB/s
   1092100927_S3_R1_001.fast     221.959 MB | 6.674 sec | 16 thr | 33.257 MB/s
   1092100927_S3_R2_001_fast       0.574 MB | 0.060 sec | 0 thr |  9.624 MB/s
   1092100927_S3_R2_001_fast       0.433 MB | 0.067 sec | 0 thr |  6.439 MB/s
   1092100927_S3_R2_001.fast     226.587 MB | 4.009 sec | 16 thr | 56.514 MB/s
   1092100928_S4_R1_001_fast       0.558 MB | 0.059 sec | 0 thr |  9.524 MB/s
   1092100928_S4_R1_001_fast       0.424 MB | 0.174 sec | 0 thr |  2.445 MB/s
   1092100928_S4_R1_001.fast     260.281 MB | 2.967 sec | 16 thr | 87.714 MB/s
   1092100928_S4_R2_001_fast       0.570 MB | 0.085 sec | 0 thr |  6.700 MB/s
   1092100928_S4_R2_001_fast       0.428 MB | 0.087 sec | 0 thr |  4.944 MB/s
   1092100928_S4_R2_001.fast     263.353 MB | 6.796 sec | 16 thr | 38.754 MB/s
   1092100929_S5_R1_001_fast       0.560 MB | 0.093 sec | 0 thr |  6.051 MB/s
...
```