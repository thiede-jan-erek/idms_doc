# Using data from iRODS directly in your scripts, code or workflows

The following documentation show you how to use iRODS data directly in you code. 

<!--
| Chapter | Title |
|----|----|
| **1.** | [Python](#python)|
| **2.** | [RStudio on Open On Demand](#rstudio-on-open-on-demand) |
| **3.** | [Snakemake](#snakemake) (Coming soon)|
-->

> :warning:  
	Always use `iinit` before running the scripts (or anytime you want to connect to iRODS).


> :bulb:  
	The examples below point to actual test data on iRODS that you can use. There is an example output after the script for you to compare if you want to try it out yourself.

## Python

To use data from iRODs in a python script, you first need to install the [python-irodsclient](https://github.com/irods/python-irodsclient) in (and/or via) conda.

```
mamba create -n python-irodsclient conda-forge::python-irodsclient
```

Once you installed the client successfully, you can use the client in your python script. Below the is an example of a python script to read in some data from a project public_data in iRODS. Don't forget to activate the environment.

```
# load in required packages
import os
import gzip
from irods.session import iRODSSession
from irods.exception import NetworkException

# read in environment file (already exists on Wetenschappelijke Werkplek)
env_file = os.path.expanduser('~/.irods/irods_environment.json')

# the collection of interest in iRODs
path_collection ="/rivmZone/projects/pub_data/testdata/100k_reads_hiseq/TESTX"

# open de iRODS session and retrieve collection
with iRODSSession(irods_env_file=env_file) as session:
		collection = session.collections.get(str(path_collection))

# extract some metadata for information on the collection
meta_data=[]
for meta_object in collection.metadata.items():
	meta_data += [(meta_object.name, meta_object.value)]

for name, value in meta_data:
	print("Collection meta field >> %s: %s" % (name, value))

# go through data objects (e.g. files)
for data_object in collection.data_objects:
	# retrieve file name
	file_name = data_object.name
	# retrieve file path
	file_path = data_object.path
	print("\n--------\nfile: %s\npath: %s" % (file_name,file_path))
	# get the data file from file_path
	obj=session.data_objects.get(file_path)
	
	# some miscellaneous 
	read_count = 0
	base_count = 0
	total_count = 0
	count_dict = {}
	
	# open the file object in reading mode
	with obj.open("r") as remote_file:
		# in case the file is gz, you can add this, otherwise leave it out
		with gzip.open(remote_file, "rt") as gzip_file:
			# do some stuff with the data in your file
			prev_line = None
			for line in gzip_file:
				if prev_line and prev_line.startswith("@HISEQ"):
					line = line.rstrip()
					for char in line:
						total_count += 1
						if char != "N": 
							base_count += 1
						if char not in count_dict:
							count_dict[char] = 1
						else:
							count_dict[char] += 1
					read_count += 1
				prev_line = line
	GC_content = (count_dict["G"] + count_dict["C"]) / \
	(count_dict["G"] + count_dict["C"] + count_dict["A"]+count_dict["T"])
	
	print("In %s reads: %s (%s) known nucelotides (not N)" % (read_count,base_count, round(base_count/total_count, 4)))
	print("Base counts: %s" % (count_dict))
	print("GC-content: %s\n" % GC_content)					
```

Example output for collection TESTX (there is also a TESTY and TESTZ)

```
$(python-irodsclient) deutekoe@rivm-biohn-l07p:~/projects/binf$ python test.py 

Collection meta field >> user::sample_type: benchmark
Collection meta field >> user::cell_type: cancer
Collection meta field >> user::species: Homo sapiens

--------
file: TESTX_H7YRLADXX_S1_L001_R1_001.fastq.gz
path: /rivmZone/projects/pub_data/testdata/100k_reads_hiseq/TESTX/TESTX_H7YRLADXX_S1_L001_R1_001.fastq.gz
In 25000 reads: 2512609 (0.9951) known nucelotides (not N)
Base counts: {'T': 730596, 'C': 540994, 'G': 529948, 'A': 711071, 'N': 12391}
GC-content: 0.4262270810937953


--------
file: TESTX_H7YRLADXX_S1_L001_R2_001.fastq.gz
path: /rivmZone/projects/pub_data/testdata/100k_reads_hiseq/TESTX/TESTX_H7YRLADXX_S1_L001_R2_001.fastq.gz
In 25000 reads: 2500718 (0.9904) known nucelotides (not N)
Base counts: {'N': 24282, 'T': 718010, 'A': 712783, 'G': 536456, 'C': 533469}
GC-content: 0.4278471223064736


--------
file: TESTX_H7YRLADXX_S1_L002_R1_001.fastq.gz
path: /rivmZone/projects/pub_data/testdata/100k_reads_hiseq/TESTX/TESTX_H7YRLADXX_S1_L002_R1_001.fastq.gz
In 25000 reads: 2512508 (0.9951) known nucelotides (not N)
Base counts: {'T': 730243, 'C': 544010, 'A': 708413, 'G': 529842, 'N': 12492}
GC-content: 0.42740242021119934


--------
file: TESTX_H7YRLADXX_S1_L002_R2_001.fastq.gz
path: /rivmZone/projects/pub_data/testdata/100k_reads_hiseq/TESTX/TESTX_H7YRLADXX_S1_L002_R2_001.fastq.gz
In 25000 reads: 2498816 (0.9896) known nucelotides (not N)
Base counts: {'N': 26184, 'A': 710095, 'T': 714763, 'G': 539476, 'C': 534482}
GC-content: 0.4297867470033808
```


## RStudio on Open On Demand

To use data directly in RStudio, you need to use a couple of packaqes. The main package is [ricmd](https://github.com/jspijker/ricmd), an R package for iRODS icommands. It is based on the [python-irodsclient](https://github.com/irods/python-irodsclient) (used in the previous section: [Python](#python)). R needs the package [reticulate](https://rstudio.github.io/reticulate/) to interact with python and the python client.

In Open On Demand (OOD), these packages are already pre-installed for you. If you open [OOD](https://openondemand.rivm.nl/), navigate to Rstudio app and open RStudio with R version > 4.4.2.

You can read the vignette for ricmd by typing in the console: 

```
vignette("ricmd")
```

> :warning:  
	Unlike with the example from Python given in the above section, with R you won't have a direct connection to iRODS. You will learn how to retrieve the data (i.e. download it locally) to work in it. This means the data should be removed after you have finished working with it. We are looking into ways to work with iRODS data directly in R to circumvent this.   

Before you start working with ricmd, you need to establish a connection with iRODS and authenticate yourself as a user. To authenticate to iRODS, you need to go to the Terminal tab in RStudio that is next to the Console tab in the top left corner. In the terminal, type `iinit` and follow the prompt. The password is your RIVM password. 

After initializing with `iinit`, you can follow the example given below to look at the data in the collection of choice. Here we use the collection TESTX.

First check and set your working directory to your scratch_dir:

```
> getwd()
[1] "/home/deutekoe"
> setwd("/mnt/scratch_dir/deutekoe")
```

Then set the right environment (you can ignore the warning) and load in the `ricmd`

```
> Sys.setenv(RETICULATE_PYTHON = "/mnt/miniconda/envs/ood/bin/python")
> reticulate::use_condaenv("ood", "/mnt/miniconda/bin/conda")
Warning message:
Previous request to `use_python("/mnt/miniconda/envs/ood/bin/python3", required = TRUE)` will be ignored. It is superseded by request to `use_python("/mnt/miniconda/envs/ood/bin/python")
> reticulate::repl_python()
```

You will now be in the python interpreter, you will notice by the `>>>` instead of `>` in the console. To exit, type `exit` or `quit`. Load the library:
```
> library(ricmd)
```

Start the iRODS sessions and retrieve the collection to look into the collection to see what files are present.

```
> ri_session(env = path.expand("~/.irods/irods_environment.json"))
> my_collection = "/rivmZone/projects/pub_data/testdata/iris_test"
> ri_setCollection(my_collection)
> ri_getCollection()
[1] "/rivmZone/projects/pub_data/testdata/iris_test"
> ri_listDataObjects()
[1] "iris_text1.txt" "iris_text2.txt"
```

To further work with the data, you can follow the example script below. First you will download a file you want, e.g. in this example the first object in the collection, using `ri_get()`. This function needs a default data directory. The directory must exists, otherwise this function generates an error (remember your working directory you set before, it will be relative to this working directory).

```
> data_dir <- "iris_test_dir"
> dir.create(data_dir)
> ri_setDatadir("iris_test_dir")
> file_object <- ri_listDataObjects()[1]
> ri_get(file_object,collection = my_collection,datadir = ri_setDatadir("iris_test_dir"))
```

The file will now be in your default data directory, in this example "iris_test_dir". You can now use this data in your data analysis scripts:

```
> file_path <- file.path(data_dir,file_object)
> data <- read.table(file_path, header = TRUE, sep = "\t")
> head(data)
  Sepal.Length Sepal.Width Petal.Length Petal.Width
1          5.1         3.5          1.4         0.2
2          4.9         3.0          1.4         0.2
3          4.7         3.2          1.3         0.2
4          4.6         3.1          1.5         0.2
5          5.0         3.6          1.4         0.2
6          5.4         3.9          1.7         0.4
  Species
1  setosa
2  setosa
3  setosa
4  setosa
5  setosa
6  setosa
```