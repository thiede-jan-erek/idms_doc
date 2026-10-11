When data is stored in iRODS, you can access it from the terminal by mounting the collection or project on the headnodes. This allows you to work on the data, without having to download the data. The lsf nodes will not have this mount automatically, this means the mount is not available on the cluster to perform any computation.
 
You can use the tool **mount_irods** to mount an irods collection or project on the headnodes. 

For this you need to be logged into irods `iinit`


```
(base) [verhager@rivm-biohn-l08p ~]$ mkdir irods
(base) [verhager@rivm-biohn-l08p ~]$ mount_irods -c /rivmZone/projects/vir-ngs -p irods
(base) [verhager@rivm-biohn-l08p ~]$ ll irods
total 0
dr-------- 1 verhager domain users 0 Dec  8 11:45 rivmZone/
(base) [verhager@rivm-biohn-l08p ~]$ ll irods/rivmZone/
total 0
dr-------- 1 verhager domain users 0 Dec  8 11:45 projects/
(base) [verhager@rivm-biohn-l08p ~]$ ll irods/rivmZone/projects/
total 0
dr-------- 1 verhager domain users 0 Apr 21  2021 vir-ngs/
(base) [verhager@rivm-biohn-l08p ~]$ ll irods/rivmZone/projects/vir-ngs/
total 0
dr-------- 1 verhager domain users 0 Feb  6  2020 191213_NB502001_0086_AHVJF7AFXY_0004/
drwx------ 1 verhager domain users 0 Dec 18  2019 191213_NB502001_0086_AHVJF7AFXY_0005/
drwx------ 1 verhager domain users 0 Jan 28  2020 200124_NB502001_0092_AHYHHKAFXY_0005/
drwx------ 1 verhager domain users 0 Jan 28  2020 200124_NB502001_0092_AHYHHKAFXY_0006/
drwx------ 1 verhager domain users 0 Feb  8  2020 200207_NB502001_0096_AHYGV2AFXY_0004/
drwx------ 1 verhager domain users 0 Feb  8  2020 200207_NB502001_0096_AHYGV2AFXY_0005/
drwx------ 1 verhager domain users 0 Mar  7  2020 200305_NB502001_0101_AHY55CAFXY_0002/
drwx------ 1 verhager domain users 0 Mar  7  2020 200305_NB502001_0101_AHY55CAFXY_0004/
drwx------ 1 verhager domain users 0 Mar  7  2020 200305_NB502001_0101_AHY55CAFXY_0005/
drwx------ 1 verhager domain users 0 Mar  7  2020 200305_NB502001_0101_AHY55CAFXY_0006/
drwx------ 1 verhager domain users 0 Mar 18  2020 200313_NB502001_0104_AHYMWCAFXY_0002/
drwx------ 1 verhager domain users 0 Mar 18  2020 200313_NB502001_0104_AHYMWCAFXY_0003/
...
```


```
$ mount_irods --help
usage: mount_irods [-h] [-c COLLECTION] [--config CONFIG] [-a] [-p PATH] [-u]
                   [-r]

Mount irods collection

optional arguments:
  -h, --help            show this help message and exit
  -c COLLECTION, --collection COLLECTION
                        iRODS Collection
  --config CONFIG       Filename to store irodsfs config
  -a, --ask_password    Ask for irods password if irods authentication is not
                        working
  -p PATH, --path PATH  Mountpoint
  -u, --unmount         Unmount currently mounted collection
  -r, --remount         Remount currently mounted collection
```