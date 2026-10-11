- Every collection has a unique ID: metadata field `sys::dataset_id`
- Collections created by a pipeline from another parent collection have the ID of the parent collection: metadata field `sys::pipeline::input_collection_id`

To determine the input collection:

example:


1. What is the input of collection `/rivmZone/projects/bsr_amr/20230330_0939_X3_FAV75126_1f360831_0005`:

__with imeta__
```
$ imeta ls -C /rivmZone/projects/bsr_amr/20230330_0939_X3_FAV75126_1f360831_0005 sys::pipeline::input_collection_id
AVUs defined for collection /rivmZone/projects/bsr_amr/20230330_0939_X3_FAV75126_1f360831_0005:
attribute: sys::pipeline::input_collection_id
value: 6f5297b6-488f-46e1-84ab-f66965960317
units:
```
__or with iquest__
```
$ iquest %s "SELECT META_COLL_ATTR_VALUE where COLL_NAME = '/rivmZone/projects/bsr_amr/20230330_0939_X3_FAV75126_1f360831_0005' and META_COLL_ATTR_NAME = 'sys::pipeline::input_collection_id'";
6f5297b6-488f-46e1-84ab-f66965960317
```

2. Which collection has id `6f5297b6-488f-46e1-84ab-f66965960317`:
```
$ iquest "%s" "select COLL_NAME where META_COLL_ATTR_NAME = 'sys::dataset_id' and META_COLL_ATTR_VALUE = '6f5297b6-488f-46e1-84ab-f66965960317'"
/rivmZone/projects/bsr_amr/20230330_0939_X3_FAV75126_1f360831_0004
```

3. What is the input of collection `/rivmZone/projects/bsr_amr/20230330_0939_X3_FAV75126_1f360831_0004`:
```
$ imeta ls -C /rivmZone/projects/bsr_amr/20230330_0939_X3_FAV75126_1f360831_0004 sys::pipeline::input_collection_id
AVUs defined for collection /rivmZone/projects/bsr_amr/20230330_0939_X3_FAV75126_1f360831_0004:
attribute: sys::pipeline::input_collection_id
value: e7bd68b8-6f5d-4693-80f9-437e72f0b619
units:
```

4. Which collection has id `e7bd68b8-6f5d-4693-80f9-437e72f0b619`:
```
$ iquest "%s" "select COLL_NAME where META_COLL_ATTR_NAME = 'sys::dataset_id' and META_COLL_ATTR_VALUE = 'e7bd68b8-6f5d-4693-80f9-437e72f0b619'"
/rivmZone/projects/ngslab/output/20230330_0939_X3_FAV75126_1f360831_0001
```

So:

| # | collection |
| ---- | ---- |
| 1st| `/rivmZone/projects/ngslab/output/20230330_0939_X3_FAV75126_1f360831_0001` |
| 2nd| `/rivmZone/projects/bsr_amr/20230330_0939_X3_FAV75126_1f360831_0004` |
| 3th| `/rivmZone/projects/bsr_amr/20230330_0939_X3_FAV75126_1f360831_0005` |


![image](../../!assets/images/image.png)