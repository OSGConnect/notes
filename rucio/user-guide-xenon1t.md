# Rucio User Guide for Xenon Collaboration

Xenon1T will have large amount of data stored in individual files across multiple remote storage sites. Managing these files, including their location, relationships between individual files and sets of files, etc., and transfer between storage sites, as well as to and from jobs, requires a data management system. The data management system has to enable the users to identify and operate on any arbitrary set of files. 

For Xenon1T, we have chosen to use the ATLAS data management system Rucio. It is pure python framework that uses a SQL database as the backend to storage the required information about the data and the relationship between individual data sets. For Rucio, files are the smallest operational unit.

# Rucio-specific Terms

Rucio has some specific terms that will how up in the documentation:

## DID

Dataset identifiers (DIDs) are the core objects in Rucio. They may be files, datasets or containers. Every DID has a unique combination of a scope (defined below) and a name. The Rucio clients always display this as `scope:name`. This implies that an identifier, once used, can never be reused to refer to anything else at all, not even if the data it referred to has been deleted from the system.

When a command requires a DID argument it must be the full DID (`scope:name`) or simply the scope plus a wildcard (`scope:*`). Rucio commands asking for a DID and can accept files, datasets or containers. 

## Scope

Scopes in Rucio and are a way of partitioning the filesystem namespace. On the filesystem level, the scope represents the top-level directory inside the Rucio storage area. If the data is stored at `/xenon/rucio/` in the storage site's filesystem and scope is `bar`, the individual file `foo.bar` (DID is `bar:foo.bar`) will be stored at `/xenon/rucio/bar/XX/YY/foo.bar`, where `XX` and `YY` are parts of the `md5` checksum of `foo.bar`. Scopes starting with `user.` or `group.`, are special in Rucio. They will have a file system structure of `/xenon/rucio/user/joe/XX/YY/foo.bar` for a scope of `user.jdoe` and `/xenon/rucio/group/DMAnalysis/XX/YY/foo.bar` for a scope of `group.DMAnalysis`.


Scopes also have certain sets of read, write, and creation permissions depending on the users. By default, normal user accounts will have read access to all scopes and write access only to their own scope. Privileged (or admin) accounts will have write access to multiple scopes, e.g. the account `production` might use scopes such as `mc16`, `data16`, etc. that reference the official datasets of the collaboration. Users have one default scope `user.<username>`, for example for user `jdoe` this is `user.jdoe`. Associated with this scope a user can generaate whatever DID he or she may desire, e.g. user jdoe can create `user.jdoe:mytest` for a dataset or `user.jdoe:foo.root` for a file. Vanilla users cannot create additional scopes. Only users with administrator priviliges can create scopes and associate them with an account. 

## RSE

Rucio Storage Element (RSE) is an abstraction for storage end-points, eg. `LNGS_DAQDISK` might be the RSE where data from the DAQ system are copied for distribution throughout the Rucio network. At NIKHEF, you might have an RSE called `NIKHEF_DATADISK` which receives raw data for processing.  The terms "site" and "RSE" are used interchangeably. The DIDs are stored on RSEs. The RSEs can be tagged with different tags which can be useful when the number of RSEs grows large.

### Deterministic vs. Non-deterministic RSE

The next question is: do you generate the physical path of the file programmatically or not? Rucio has a concept of a deterministic RSE, which means that a simple function applied to the `scope:name` can give the full path on the RSE.  The primary reason for this is to divide the namespace on disk for file system performance. If not the RSE has to be defined as non-deterministic and the full replica paths should be provided to the the `list_replicas` method.

## Dataset

A dataset is a named set of files. It has the format of any other DID, namely `scope:name`. A dataset is not reflected in the filesystem of the storage sites. It only exists inside the database.

## Container

A container a named set of datasets or, recursively, containers. It has the format of any other DID, namely `scope:name`. A container is not reflected in the filesystem of the storage sites. It only exists inside the database.

## (Replication) Rules 

The replication rules (aka rules) define how a DID should be or is replicated among the RSEs. A rule is associated to an account, a DID and an RSE. When a rule is set on a DID, and a particular RSE, and the DID is not present at the site Rucio will initiate (an) FTS transfer(s) to the site from an RSE for the file(s) associated with the particular DID. If the DID is present, the rule will be ignored. If a replication rule is present, the DID cannot be deleted from that RSE. Otherwise, the DID will be deleted in case the RSE does not have sufficient storage available. 

# Getting an Account

To get an account within rucion you need your grid certificate and to have your grid certifcate added to the Xenon VO. For instrustions in how to get a grid certificate and add yourself to the Xenon VO, see [the instructions on the Xenon1t wiki](https://xecluster.lngs.infn.it/dokuwiki/doku.php?id=xenon:xenon1t:sim:grid). Once you have your grid certificate all set up, you need to email the Rucio admin (?????) with the following information:

* Preferred username for the account

* The DN and email for the x509 certificate registered in the xenon.biggrid.nl VO: `$ openssl x509 -in .globus/usercert.pem -noout -subject -email`

