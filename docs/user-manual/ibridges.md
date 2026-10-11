A guide for Windows users to manage iRODS via the command line and perform bulk data uploads and downloads (or more).

> [!important]
>
> This guide is only for campus-pro-extra users.

# Preparation

In order to install the command line tool (ibridges), we need to install python and pip first.

## Installing Python

1. Go to https://www.python.org/
2. At the top bar, hover over downloads and select windows
3. Find the latest python version (or at least 3.8)
4. Download Windows installer (64-bit)
5. Open the downloaded python installer
6. Check mark add 'python.exe to PATH'
7. Click Install Now

## Check if python and pip is correctly installed

1. Open a powershell terminal
   1. Press Windows key + R
   2. Type powershell
   3. Click on ok
2. Type in the powershell 'python --version'

```powershell
PS C:\Users\example_user> python --version
Python 3.13.13
```

This should give an output of your installed python version. The example used python version 3.13.13.

After installing python, you may or may not have pip installed.

3. Type in powershell 'pip --version'

```powershell
PS C:\Users\example_user> pip --version
pip 26.0.1 from C:\Users\example_user\AppData\Local\Programs\Python\Python313\Lib\site-packages\pip (python 3.13)
```

If you see output like this above. You have pip installed and should follow Install ibridges. if you see redlines with pip is not found, continue here:

4. Type in powershell 'python -m ensurepip --upgrade'

```
PS C:\Users\example_user> python -m ensurepip --upgrade
```

Now when you run 'pip --version' again you should have a similair output.

# Install commandline tool Ibridges

1. Open a powershell terminal
2. Type in powershell 'pip install ibridges'

```powershell
PS C:\Users\example_user> pip install ibridges
```

3. Ensure ibridges is installed

```powershell
PS C:\Users\example_user> ibridges -h
iBridges CLI version 2.1.0

Usage: ibridges [subcommand] [options]

ibridges commands (v2.1.0):
    ls (list, l):
        List a collection on iRODS.
    pwd:
        Show current working collection.
    tree:
        Show collection/directory tree.
    meta-list:
        List the metadata of a data object or collection on iRODS.
    meta-add:
        Add a metadata item to a collection or data object.
    meta-del:
        Delete metadata for one collection or data object.
    mkcoll:
        Create a new collecion with all its parent collections.
    meta-download:
        Download metadata for a data object or collection recursively.
    meta-upload:
        Upload and apply metadata for a data object or collection. See meta-download for how to create an archive.
    download:
        Download a data object or collection from an iRODS server.
    upload:
        Upload a data object or collection to an iRODS server.
    search:
        Search for dataobjects and collections.
    cd:
        Change current working collection for the iRODS server.
    rm (remove, del):
        Remove collection or data object.
    sync:
        Synchronize files/directories between local and remote.
    gui:
        Start the iBridges GUI.
    version:
        Print the version of iBridges.
    chmod:
        Manipulate the permissions of a data object or collection.
    shell:
        Shell for ibridges commands with autocomplete.
    alias:
        Print existing aliases or create new ones.
    init:
        Create a cached password for future use.
    setup:
        Use templates to create an iRODS environment json.


    The iBridges CLI does not implement the complete iBridges API. For example, there
    are no commands to modify the access rights to data.

    Example usage:

        ibridges download "irods:~/test.txt"
        ibridges upload ~/test.txt "irods:/test"
        ibridges init
        ibridges sync ~/directory "irods:~/collection"
        ibridges list irods:~/collection
        ibridges meta-add irods:some_dataobj_or_collection new_key new_value new_units
        ibridges meta-list irods:some_dataobj_or_collection
        ibridges mkcoll irods://~/bli/bla/blubb
        ibridges tree irods:~/collection
        ibridges search --path-pattern "%.txt"
        ibridges search --metadata "key" "value" "units"
        ibridges search --metadata "key" --metadata "key2" "value2"
        ibridges setup uu-its

    Reuse a configuration by an alias:
        ibridges init ~/.irods/irods_environment.json --alias my_irods
        ibridges init my_irods

    Program information:
        -h, --help    - display this help file and exit
```

# Connect ibridges to Irods

In order to use the commandline tool you will need to make connection with Irods first.

1. In 'C:\\Users\\\<Your username\>', create a map called '.irods'
2. Open the program notepad and replace every '\<your username\>' with your own username and save the file as 'irods_environment.json' in .irods folder

```json
{
    "irods_host": "rivm-bioir-l04p.rivm.ssc-campus.nl",
    "irods_port": 1247,
    "irods_user_name": "<your username>",
    "irods_zone_name": "rivmZone",
    "irods_home": "/rivmZone/home/<your username>",
    "irods_cwd": "/rivmZone/home/<your username>",
    "irods_authentication_scheme": "pam_password",
    "irods_client_server_negotiation": "request_server_negotiation",
    "irods_client_server_policy": "CS_NEG_REQUIRE",
    "irods_encryption_algorithm": "AES-256-CBC",
    "irods_encryption_key_size": 32,
    "irods_encryption_num_hash_rounds": 16,
    "irods_encryption_salt_size": 8,
    "irods_ssl_ca_certificate_file": "C:\\Users\\<your username>\\.irods\\irods.crt",
    "irods_ssl_verify_server": "cert"
}
```

3. Open the notepad again and copy the content of the certificate from the wiki into 'irods.crt' 



> [!caution]
>
> Besure to save both files in '.irods' folder that you have created in 'C:\\Users\<Your username\>' with the correct file extention (file from step 2 is .json and file from step 3 is .cert

Now everything should be setup for connecting to Irods.

1. Open powershell
2. Type in 'ibridges init', this will prompt for password, just type in your rivm password. You won't beable see any typed letters.

```powershell
PS C:\Users\example_user\.irods> ibridges init
Your iRODS password:
```

After entering the password you should get a prompt that says 'ibridges init was succesful.'

If this is not the case, please double check if the folders and files are created correctly. You can also contact us for assistant.

# How to use Ibridges

Ibridges is a helpfull tool that minics commandline tool 'icommand' that is available for Irods users in linux enviroment (wetenschapelijke werkplek). For more extensive guide you can find the documentation [here](https://ibridges.readthedocs.io/en/latest/ibridges_cli.html).

> [!important]
>
> Make sure ibridges init is succesfull before trying these commands. If any of the command fails try to ibridges init again.

This is a quick guide how you can use ibridges to perform simple tasks in powershell. For quick help, type 'ibridges --help'.

* List a collection

  ```powershell
  # list your home collection 
  ibridges list  
  # list a different collection in your home 
  ibridges list "irods:~/<collection>"  
  # list a collection on a different path than your home  
  ibridges list "irods:/full/irods/path"
  ```
* Upload data

  ```powershell
  ibridges upload my_file.json "irods:~/some_collection"
  ```
* Download data

  ```powershell
  ibridges download "irods:~/some_collection/some_object" download_dir
  ```
* Synchronise data

  ```powershell
  ibridges sync some_local_directory "irods:~/some_collection"  
  ```