# Rucio Admin Guide for Xenon Collaboration

NOTE: This is only for users with privileged accounts. Others cannot execute these command

The Rucio admin tools (`rucio-admin` in the commandline) is the collection of commands to configure and manage the Rucio instance. 

## Installing Rucio Admin

The admin tools have the same dependencies as the client. After the dependencies have been installed you can install the client through:

* Login into the system where the Rucio client will be installed
* Execute `pip install --user rucio`
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

## Frequently Used Commands

* Creating scope
```
rucio-admin scope add --help
usage: rucio-admin scope add [-h] --account ACCOUNT --scope SCOPE
optional arguments:
  -h, --help        show this help message and exit
  --account ACCOUNT  Account name
  --scope SCOPE     Scope name
```
* Creating an account
    * Adding account
      `rucio-admin account add <username>`
    * Map account to an identity:
        ```
        rucio-admin identity add -h
        usage: rucio-admin identity add [-h] --account ACCOUNT --type
                                        {X509,GSS,USERPASS} --id IDENTITY --email  EMAIL
        optional arguments:
          -h, --help            show this help message and exit


          --account ACCOUNT     Account name
          --type {X509,GSS,USERPASS}
                                Authentication type [X509|GSS|USERPASS]
          --id IDENTITY         Identity
          --email EMAIL         Email address associated with the identity
        ```
        Example: 
        ```
        rucio-admin -a root identity add --account <username> --type X509 --id "/DC=org/DC=opensciencegrid/O=Open Science Grid/OU=People/CN=Suchandra Thapa" --email <email>
        ```
    * Create user scope
        ```
        rucio-admin scope add --help
        usage: rucio-admin scope add [-h] --account ACCOUNT --scope SCOPE
        optional arguments:
          -h, --help        show this help message and exit
          --account ACCOUNT  Account name
          --scope SCOPE     Scope name
        ```
    * Create quota for account on RSEs
        `rucio-admin -a root account set-limits <username> <rse> <quota_in_bytes>`
* Registering an RSE
    * To add the RSE
        `rucio-admin rse add <rse>`
    * Set attributes for the RSE
        ```
        rucio-admin rse set-attribute --help
        usage: rucio-admin rse set-attribute [-h] --rse RSE --key KEY --value VALUE
        ```
    * Configure gridftp for an existing endpoint. From python command line:
        ```
        from rucio.core.rse import add_protocol
        proto = {'hostname': 'wipp-se.weizmann.ac.il', 
                 'scheme': 'srm', 
                 'port': 8444, 
                 'prefix': '/xenon/rucio',
                 'impl': 'rucio.rse.protocols.gfalv2.Default', 
                 'extended_attributes': None, 
                 'domains': {'lan': {'read': 1, 
                                     'write': 1, 
                                     'delete': 1},
                 'wan': {'read': 1, 
                         'write': 1, 
                         'delete': 1}}}
        add_protocol(<rse>, parameter=proto)
        ```
        For SRM endpoints:
        ```
        proto = {'hostname': 'wipp-se.weizmann.ac.il', 'scheme': 'srm', 'port': 8444, 'prefix': '/xenon/rucio', 'impl': 'rucio.rse.protocols.gfalv2.Default', 'extended_attributes': {'space_token': 'xenon', 'web_service_path': '/srm/managerv2?SFN='}, 'domains': {'lan': {'read': 1, 'write': 1, 'delete': 1}, 'wan': {'read': 1, 'write': 1, 'delete': 1}}}
        ```
        Note: SRM requires to have `extended_attributes` `space_token` and `web_service_path` set. It can be an empty string if necessary.
    * Set distances between the RSEs
        ```
        from rucio.core.rse import list_rses
        from rucio.core.distance import add_distance
        rses = list_rses()
        for src in rses:
            for dst in rses:
                try:
                    add_distance(src_rse_id=src['id'], dest_rse_id=dst['id'], ranking=1, agis_distance=1)
                except:
                    continue
        ```
    * Set FTS server:
        ```
        from rucio.core.rse import add_rse_attribute
        add_rse_attribute('CCIN2P3_USERDISK', 'fts', 'https://fts.usatlas.bnl.gov:8446')```
    * Changing RSE quota: `rucio-admin -a root account set-limits <account> <rse> <quota_in_bytes>`
    ```
    from rucio.core.rse import add_protocol
    from rucio.core.rse import add_rse
    from rucio.core.rse import add_rse_attribute
    from rucio.core.distance import add_distance
    from rucio.core.rse import list_rses

    ucosg_srm = {'hostname': 'ceph-se.osgconnect.net',
                'scheme': 'srm',
                'port': 8443,
                'prefix': '/cephfs/srm/xenon',
                'impl': 'rucio.rse.protocols.gfal.Default',
                'extended_attributes': {u'space_token': u'XENON',
                                        u'web_service_path': u'/srm/v2/server?SFN='},
                'domains': {"lan": {"read": 1,
                                    "write": 1,
                                    "delete": 1},
                            "wan": {"read": 1,
                                    "write": 1,
                                    "delete": 1}}}
    add_rse('UC_OSG_USERDISK')
    add_rse_attribute('UC_OSG_USERDISK', 'fts', 'https://fts.usatlas.bnl.gov:8446')
    add_rse_attribute('UC_OSG_USERDISK', 'fts_testing', 'https://fts.usatlas.bnl.gov:8446')
    add_rse_attribute('UC_OSG_USERDISK', 'istape', False)
    add_protocol('UC_OSG_USERDISK', parameter=ucosg_srm)

    rses = list_rses()
    for src in rses:
        for dst in rses:
            add_distance(src_rse_id=src['id'], dest_rse_id=dst['id'], ranking=1, agis_distance=1)
    ```

* Update RSE protocols: 
    ```
    from rucio.core.rse import update_protocols
    update_protocols(rse="CCIN2P3_USERDISK", scheme="srm", data={'extended_attributes': {u'space_token': '', u'web_service_path': u''}}, hostname="ccsrm02.in2p3.fr", port=8443)
    ```

## Troubleshooting a Rucio instance



## Setting up a Rucio instance

### Dependency Installation - ADMIN ONLY

This is assuming RHEL7 system:

```
rpm -iUvh http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-5.noarch.rpm
rpm -iUvh http://mirror.grid.uchicago.edu/pub/osg/3.3/osg-3.3-el7-release-latest.rpm
yum -y update
yum install osg-pki-tools
reboot
yum install -y python-pip
pip install --upgrade pip
yum -y install vim gcc python-devel krb5-devel
yum install -y memcached
systemctl enable memcached
yum install -y gfal2 gfal2-python
yum install mariadb-devel mariadb-server mariadb MySQL-python
chkconfig mariadb on
service mariadb restart
pip install rucio
```

### Creating SQL Database

```
mysql-->
create database rucio;
grant all privileges on rucio.* to 'rucio'@'localhost' identified by
'rucio';
```

### Creating Rucio Configuration

```
mkdir -p /var/log/rucio/trace
mkdir -p /opt/rucio/etc/web
cp /usr/rucio/tools /opt/rucio
cp /usr/rucio/etc/rucio.cfg.template /opt/rucio/etc/rucio.cfg
vi /opt/rucio/etc/rucio.cfg:
```

`rucio.cfg`:

    ```
    rucio_host      -> <HTTP_accessible_Rucio_hostname, e.g. https://rucio.mwt2.org:443>
    auth_host       -> <HTTP_accessible_Rucio_hostname, e.g. https://rucio.mwt2.org:443>
    carbon_server   -> <Graphite_hostname>
    carbon_port     -> <Graphite_port>
    ssl_key_file    -> <Location of host key, e.g. /etc/grid-security/hostkey.pem>
    ssl_cert_file   -> <Location of host cert, e.g. /etc/grid-security/hostcert.pem>
    voname          -> <VO_URL, e.g. xenon.biggrid.nl>
    username        -> <rucio_username>
    password        -> <pw>
    userpass_identity   -> <rucio_username>
    userpass_pwd   -> <pw>
    userpass_email -> <admin_email>
    X509_identity   -> <X509_host_cert_DN>
    x509_email      -> <X509_Host_cert_admin_email
    brokers         -> <activemq_host>
    port            -> <activemq_port_on_host>
    ftshosts        -> <FTS_server>
    default     -> <MySQL_URI>
    scheme      -> <Comma-separated_list_transfer_protocols>
    ```

Some config files are not in the above rucio installation, need to get from the git repo

```
git clone https://gitlab.cern.ch/rucio01/rucio.git ~/rucio
cp ~/rucio/etc/alembic.ini.template /opt/rucio/etc/alembic.ini
cp -r ~/rucio/lib/rucio/db/sqla /opt/rucio/lib/rucio/db # change line in alembic.ini to point to this dir
```

### Resetting/Setting Database Schema

```
cd /opt/rucio
tools/reset_database.py
```

### Setting up Apache/HTTP Server

```
yum -y install httpd mod_ssl mod_wsgi gridsite
systemctl enable httpd
rm -f /etc/httpd/conf.d/*
cp ~/rucio/etc/web/httpd-rucio-443-py26-slc6.conf.template /etc/httpd/conf.d/rucio.conf
cp ~/rucio/etc/web/aliases-py27.conf /opt/rucio/etc/
vi /opt/rucio/etc/aliases-py27.conf     # rucio python in /usr/lib/python2.7/site-packages/rucio
service httpd start


$ curl -k -X GET https://rucio.mwt2.org:443/ping
{"version": "1.5.0"}
```

### Daemon Configuration

In `/etc/supervisord.d/rucio.ini` append:

```
[program:rucio-conveyor-transfer-submitter]
command=/bin/rucio-conveyor-transfer-submitter
stdout_logfile=/var/log/rucio/conveyor-transfer-submitter.log


[program:rucio-conveyor-poller]
command=/bin/rucio-conveyor-poller
stdout_logfile=/var/log/rucio/conveyor-poller.log


[program:rucio-conveyor-finisher]
command=/bin/rucio-conveyor-finisher
stdout_logfile=/var/log/rucio/conveyor-finisher.log


[program:rucio-undertaker]
command=/bin/rucio-undertaker
stdout_logfile=/var/log/rucio/undertaker.log


[program:rucio-reaper]
command=/bin/rucio-reaper
stdout_logfile=/var/log/rucio/reaper.log


[program:rucio-necromancer]
command=/bin/rucio-necromancer
stdout_logfile=/var/log/rucio/necromancer.log


[program:rucio-abacus-account]
command=/bin/rucio-abacus-account
stdout_logfile=/var/log/rucio/abacus-account.log


[program:rucio-abacus-rse]
command=/bin/rucio-abacus-rse
stdout_logfile=/var/log/rucio/abacus-rse.log


[program:rucio-transmogrifier]
command=/bin/rucio-transmogrifier
stdout_logfile=/var/log/rucio/transmogrifier.log


[program:rucio-judge-evaluator]
command=/bin/rucio-judge-evaluator
stdout_logfile=/var/log/rucio/judge-evaluator.log


[program:rucio-judge-repairer]
command=/bin/rucio-judge-repairer
stdout_logfile=/var/log/rucio/judge-repairer.log


[program:rucio-conveyor-stager]
command=/bin/rucio-conveyor-stager
stdout_logfile=/var/log/rucio/conveyor-stager.log
```

### Client Configuration

Without a config the configuration looks something like:

```
$ cat /opt/rucio/etc/rucio.cfg
# Copyright European Organization for Nuclear Research (CERN)
#
# Licensed under the Apache License, Version 2.0 (the "License");
# You may not use this file except in compliance with the License.
# You may obtain a copy of the License at http://www.apache.org/licenses/LICENSE-2.0
#
# Authors:
# - Mario Lassnig, <mario.lassnig@cern.ch>, 2012-2014
# - Thomas Beermann, <thomas.beermann@cern.ch>, 2012, 2015-2016
# - Cedric Serfon, <cedric.serfon@cern.ch>, 2013
```

Append to `/opt/rucio/etc/rucio.cfg`:

```
[common]
logdir = /var/log/rucio
loglevel = DEBUG
mailtemplatedir=/opt/rucio/etc/mail_templates


[client]
rucio_host = https://rucio.mwt2.org:443
auth_host = https://rucio.mwt2.org:443
auth_type = x509
ca_cert = ~/.rucio/ca.crt
client_cert = ~/.rucio/usercert.pem
client_key = ~/.rucio/userkey.pem
client_x509_proxy = ~/.rucio/x509up
request_retries = 3
```

### Create Host Certificate

```
mkdir /etc/grid-security
osg-cert-request -p 7737026282 -n "Lincoln Bryant" -e lincolnb@uchicago.edu -H rucio.mwt2.org -v ATLAS
osg-cert-retrieve 7508
mv hostcert.pem /etc/grid-security
mv hostkey.pem /etc/grid-security


[root@rucio ~]# crontab -l
10 0 * * * /bin/voms-proxy-init -cert /etc/grid-security/hostcert.pem -key /etc/grid-security/hostkey.pem -voms xenon.biggrid.nl -hours 48 -out /opt/rucio/etc/web/x509up -q > /dev/null 2>&1
15 */3 * * * /bin/fts-delegation-init -s https://fts.usatlas.bnl.gov:8446 --proxy /opt/rucio/etc/web/x509up -q > /dev/null 2>&1
```

### Install ActiveMQ

```
yum install java-1.8.0-openjdk
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.77-0.b03.el7_2.x86_64
wget http://apache.osuosl.org/activemq/5.13.2/apache-activemq-5.13.2-bin.tar.gz (plus gpg verification)
tar -xzf apache-activemq-5.13.2-bin.tar.gz
cd apache-activemq-5.13.2/
vi conf/jetty-realm.properties (change admin, user pws)
bin/activemq start


In web interface: (rucio.mwt2.org:8161, default ports)
add queues: Consumer.kronos.rucio.tracer
add topics: transfer.fts_monitoring_queue_state, rucio.tracer, rucio.fax\x
```

<!-- ### Troubleshooting

#### Restart the Rucio VM

VM lives on `uct2-kvm03.mwt2.org`. To restart the machine `virsh shutdown rucio.mwt2.org`. If it is not responding `virsh destroy rucio.mwt2.org`. 

Host cert, host key, ca cert are located in `/opt/rucio/etc/web`

FTS are complaining about delegation

To check the daemons: `/etc/init.d/supervisord status`

Logs are in: `/var/log/rucio`

MariaDB: `service restart MariaDB`

ActiveMQ: `/opt/apache-activemq-5.13.3/bin/activemq start` 

change space for an RSE: /root/rucio/tools/probes/common/check_srm_space
-->


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

# Endpoints


# Tutorial

Note: This tutorial assumes you are on `login.ci-connect.uchicago.edu`

Given that you have your grid certificate, are a memeber of the Xenon VO, and have an account in Rucio, we can now do some simple tasks with rucio. 

Before we can do anything with rucio, you have to get your grid certicate setup on the machine you are using. You should have a PKCS12 (it should look something like `user_certificate_and_key.UXXXX.p12`) file that you got when your grid certificate was approved. Copy the PKCS12 file to your home directory on the server you are using. Now create 

`mkdir $HOME/.globus`

then run

`openssl pkcs12 -in <PKCS12_filename> -clcerts -nokeys -out $HOME/.globus/usercert.pem`

`openssl pkcs12 -in <PKCS12_filename> -nocerts -out $HOME/.globus/userkey.pem`

and finally we need to change the permissions:

`chmod 0600 $HOME/.globus/usercert.pem`

`chmod 0400 $HOME/.globus/userkey.pem`

Now we can generate a VOMs or grid proxy. This is the active part of your grid certificate and will allow you to initiate transfers. 

First we need to setup the enviroment to get the grid tools and rucio. To use the grid tools for RHEL6 and derivatives rucio from the the Xenon OASIS CVMFS repository run:

`source /cvmfs/xenon.opensciencegrid.org/software/rucio-py26/setup_rucio_1_8_3.sh`

Once you are in the desired environment initiate the proxy run: 

`voms-proxy-init -voms xenon.biggrid.nl -valid 720:00`

Now lets try downloading a single file: 

`rucio download user.briedel:test_rucio.dummy`

This should download the file `test_rucio.dummy` into `$PWD/user.briedel`. If we now try to download a dataset with 

`rucio download user.briedel:tutorial`

It will put the files in that dataset into `$PWD/tutorial`. Now to upload a file, take a random file (you can generate a 100 MB test file using `dd if=/dev/urandom of=test_rucio.dummy bs=100M count=1`). Now run 

`rucio -v upload --rse UC_OSG_USERDISK --scope user.<your_Rucio_username> user.<your_Rucio_username>:tutorial_rucio /path/to/test/file` 

to upload a file. If the file has been uploaded successfully, run the command `rucio list-dids user.<your_Rucio_username>:tutorial_rucio` this should print something like:

```
+-------------------------------------------+--------------+
| SCOPE:NAME                                | [DID TYPE]   |
|-------------------------------------------+--------------|
| user.<your_Rucio_username>:tutorial_rucio | DATASET      |
+-------------------------------------------+--------------+
```

Now run `rucio list-file-replicas user.<your_Rucio_username>:tutorial`, this should prompt something like:

``` 
+----------------------------+------------------+------------+-----------+---------------------------------------------------------------------------------------------------------------------+
| SCOPE                      | NAME             | FILESIZE   | ADLER32   | RSE: REPLICA                                                                                                        |
|----------------------------+------------------+------------+-----------+---------------------------------------------------------------------------------------------------------------------|
| user.<your_Rucio_username> | test_rucio.dummy | 33.6 MB    | checksum  | UC_OSG_USERDISK: gsiftp://gridftp.grid.uchicago.edu:2811/cephfs/srm/xenon/rucio/user/briedel/XX/YY/test_rucio.dummy |
+----------------------------+------------------+------------+-----------+---------------------------------------------------------------------------------------------------------------------+
```