# Data Organization

For Xenon100, there are several ways how the data was organized. On NIKHEF: 

```
/dpm/nikhef.nl/home/xenon.biggrid.nl/
|
+-- archive
|   |
|   \-- data
|       |
|       \-- xenon100
|           |
|           \-- run_XX
|               |
|               \-- xe100_YYMMDD_HHMM
|                   |
|                   \--<files>
```

On the MWT2 Ceph pool:

```
/xenon/
|
+-- run_XX
|   |
|   \-- xe100_YYMMDD_HHMM
|       |
|       \--<files>
```

For Rucio, a combination of using scopes, containers, and datasets replaces the filesystem structure. Scopes are top level directories in a file system and are reflected in the filesystem, while the datasets and containers are similar to sub-directories yet are only reflected in the database. For the scopes `data`, `simulation`, and `user.jdoe`, the MWT2 Ceph pool would create a file system structure of:

```
/xenon/
|
+-- data
|   |
|   \-- XX
|       |
|       \-- YY
|           |
|           \-- foo.bar
|
+-- simulation
|   |
|   \-- ZZ
|       |
|       \-- AA
|           |
|           \-- sim_foo.bar
|
+-- user
|   |
|   \-- jdoe
|       |
|       \-- BB
|           |
|           \-- CC
|               |
|               \-- user_file_foo.bar
```


The subdirectories (ZZ/AA, BB/CC, etc.) for every scope in the above example are “deterministic”, i.e. can be calculated algorithmically.
