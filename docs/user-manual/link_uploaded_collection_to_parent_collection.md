If you downloaded data from iRODS and performed manual analyses on it, it is important to maintain proper data governance if you want to upload the data back into iRODS. This means you need to link the new dataset (collection) back to its original source collection in iRODs, i.e. the parent. This wiki will help you set up data governance for an uploaded dataset.

The following steps were taken:

- You have [downloaded a collection](../use_iget_to_get_data_from_irods) from iRODS and started some manual pipeline / scripts on it.
- You have uploaded the dataset output to iRODS and added the metadata to it, [using the web interface](../manual_upload_in_idms).

Now you would like to link the manual created collection to the source collection. You can do this via the terminal in the Wetenschappelijke Werkplek, or via the webinterface [:octicons-link-external-24: iDMS](biorods.rivm.nl).


<!--
| Section | Subsection | Subject | 
| ------- | ------- | ------- |
| 1 | | [From the terminal in Wetenschappelijke Werkplek](#1-from-the-terminal-in-the-wetenschappelijke-werkplek)
| | 1 | [One parent collection](#11-one-parent-collection) | 
| | 2 | [Multiple parent collections](#12-multiple-parent-collections) | 
| | 3 | [Check your metadata](#13-check-your-metadata) | 
| 2 | | [From the web-interface with iDMS](#2-from-the-web-interface-with-idms)|
-->

## 1. From the terminal in the Wetenschappelijke werkplek
**1) Find the ID of the parent collection**

In the metadata field `sys::dataset_id` you can find the value of the ID of the collection, either with:

A) the `imeta` command:

```
$ imeta ls -C /rivmZone/projects/nanlon/240830_VH01799_112_AACGVN7HV_0012 sys::dataset_id
AVUs defined for collection /rivmZone/projects/nanlon/240830_VH01799_112_AACGVN7HV_0012:
attribute: sys::dataset_id
value: 3fccd5c3-b8d8-4a6b-868a-6923930e5862
units: 
```

B) the web browser:

![](../../!assets/images/metadata__dataset_id.png)
_Figure 1_

### 1.1. One parent collection
If you have one parent collection, you can follow these steps:

**2) set the parent collection id to the target collection**

Add the `user::pipeline::input_collection_id` to target the collection with the parent ID you found in step 1:

```
$ imeta set -C /rivmZone/projects/nanlon/240830_VH01799_112_AACGVN7HV_0012_Cellranger user::pipeline::input_collection_id 3fccd5c3-b8d8-4a6b-868a-6923930e5862
```

Your collection (/rivmZone/projects/nanlon/240830_VH01799_112_AACGVN7HV_0012_Cellranger) is now connected to the parent collection (/rivmZone/projects/nanlon/240830_VH01799_112_AACGVN7HV_0012).

![](../../!assets/images/connection_parent_collection.png){: .glightbox title="Figure 2"}

### 1.2. Multiple parent collections
If your uploaded collection is comprised of more parent collections, you can add multiple parent collection ids.

**2) add the parent collection id's to the target collection**

:warning: Do not first use the `imeta set` command, but immediately use `imeta add`.

```
$ imeta add -C /rivmZone/projects/nanlon/240830_VH01799_112_AACGVN7HV_0012_Cellranger user::pipeline::input_collection_id 3fccd5c3-b8d8-4a6b-868a-6923930e5862
$ imeta add -C /rivmZone/projects/nanlon/240830_VH01799_112_AACGVN7HV_0012_Cellranger user::pipeline::input_collection_id cf0b6a53-0134-4e22-84eb-726c04d496da
```

:bulb: You can use a file with parent collection id's and use this to automatically add multiple parent collection id's in a bash script.


### 1.3. Check your metadata
You can check the new metadata that you added with:

```
$ imeta ls -C /rivmZone/projects/nanlon/240830_VH01799_112_AACGVN7HV_0012_Cellranger user::pipeline::input_collection_id
```

If you accidentally added the wrong collection, you can use the following command:

```
$imeta rm -C /rivmZone/projects/nanlon/240830_VH01799_112_AACGVN7HV_0012_Cellranger user::pipeline::input_collection_id cf0b6a53-0134-4e22-84eb-726c04d496da

```

## 2. From the web-interface with iDMS
Alternatively, you can navigate to the collection you want to link. From there, go to the `Provenance` tab in the right upper window in iDMS. Click `Add upstream dataset`:

![](../../!assets/images/provenance.svg){width=80%}

In the search bar you can paste/write the collection you want to add as upstream parent collection:

![](../../!assets/images/add_parent.png){width=80%}

After you added the collection, you will see the uploaded collection linked to the parent collection, like in Figure 2 above. 