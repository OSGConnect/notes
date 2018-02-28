# Rucio Software

The Rucio software is split into two pieces: client and admin. 

## Rucio Client

The Rucio client (simply `rucio` in the command line) allows users all the necessary tasks to manage their own data. This includes, uploading data, downloading data, creating replication rules, listen data locations, etc.

### Installing Client

The Rucio client is installed using pip and can be installed locally in your home directory or (not recommended) globally in case you have root priviliges. Installing the Rucio client requires the following dependencies (these require root privileges on the host for installation or access to the OSG OASIS cvmfs respository):

* GCC compiler
* pip
* Gfal2 python (gfal2-python rpm)  - This is available from the EPEL repository. 
* CA certificates and updated CRLs  in /etc/grid-security/certificates.
* Libffi devel packages (libffi-devel rpm) - This is in the standard Scientific Linux repositories
* Kerberos devel packages (kerberos-devel) - This is in the standard Scientific Linux repositories
* Python devel packages (python-devel) - This is in the standard Scientific Linux repositories
* Openssl devel packages (openssl-devel) - This is in the standard Scientific Linux repositories
* Flask (python-flask) - This is specific for rucio version >= 1.15.0

After the dependencies are satisfied the user can install the client without root priviliges. NOTE: These instructions are meant for normal user accounts that do not have root priviliges on the machine.

* Login into the system where the Rucio client will be installed
* Execute `pip install --user rucio-clients-xenon1t`
* Check if `~/.local/rucio` is present
* Append the `PATH` to allow access to the rucio binary: `export PATH=~/.local/bin:$PATH`
* Set `RUCIO_HOME`: `export RUCIO_HOME=~/.local/rucio`
* Set `RUCIO_ACCOUNT`: `export RUCIO_ACCOUNT=username`
* Modify your `.bashrc` or equivalent to add the Rucio client to your `PATH`, and set `RUCIO_HOME` and `RUCIO_ACCOUNT` whenever you log in
* Get the Xenon1T Rucio configuration file: 
```
cd ~/.local/rucio/etc
wget http://stash.osgconnect.net/@xenon1t/public/rucio/rucio.cfg
```
* Check if Rucio is setup properly by running `rucio whoami`. It should output something similar to:
```
$ rucio whoami
Enter PEM pass phrase:
status  : ACTIVE
account : sthapa
account_type : USER
created_at : 2016-07-01T17:35:47
suspended_at : None
updated_at : 2016-07-01T17:35:47
deleted_at : None
email   : ssthapa@uchicago.edu
```

### Rucio Client through cvmfs

Details to follow

### Frequently Used Commands

* Change which user account to use:
`rucio -a <Rucio username> <Rucio command>`
* User identity: 
`rucio whoami`
* Create a dataset
`rucio upload --rse MyRSE --scope user.jdoe user.jdoe:MyDatasetName <list of files>`
* Download a DID
`rucio download <DID>`
* Create an empty dataset
`rucio add-dataset user.jdoe:test.Dataset`
* Create an empty container
`rucio add-container user.jdoe:test.Container`
* Add (attach) DIDs to a dataset/container:
`rucio attach <dataset/container DID> <list_of_DIDs>`
* Delete (attach) DIDs from a dataset/container:
`rucio attach <dataset/container DID> <list_of_DIDs>`
* What are the active storage locations?
`rucio list-rses`
* Show all the file/dataset replicas
`rucio list-file-replicas <DID>`
`rucio list-dataset-replicas <pattern>`
* How to list the content of a DID
`rucio list-dids <scope>:*`, e.g. `rucio list-dids ‘user.jdoe:*’` will list all the datasets/containers belonging to a user jdoe in the scope `user.jdoe`
* List files in a dataset
`rucio list-files <dataset>`    
* List account usage
`rucio list-account-usage jdoe`
* Add a replication rule
`rucio add-rule <list of DIDs> <number of copies> <rse>`. Note this print a UUID of the rule to the screen
* Delete a replication rule
`rucio delete-rule rule_id`
* List rules concerning a DID
These can optionally be filtered by account, subscription or RULE_ID: 
`rucio list-rules`
* List the datasets at an RSE
`rucio list-datasets-rse <rse>`
* List usage at an RSE
`rucio list-rse-usage <rse>`
