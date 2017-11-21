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

### Removing DIDs that won't die

First check /var/log/rucio/undertaker.log for errors such as the following:

```
2017-10-30 09:14:57,184	11215	ERROR	Undertaker(1): Got database error Database exception.
Details: (raised as a result of Query-invoked autoflush; consider using a session.no_autoflush block if this flush is occurring prematurely) (_mysql_exceptions.IntegrityError) (1062, "Duplicate entry 'x1t_SR000-x1t_SR000_161108_0719_tpc-x1t_SR000_161108_0719_tpc-ra' for key 'PRIMARY'") [SQL: u'INSERT INTO contents_history (scope, name, child_scope, child_name, did_type, child_type, bytes, adler32, md5, guid, events, rule_evaluation, did_created_at, deleted_at, updated_at, created_at) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)'] [parameters: ('x1t_SR000', 'x1t_SR000_161108_0719_tpc', 'x1t_SR000_161108_0719_tpc', 'raw', 'C', 'D', None, None, None, None, None, None, datetime.datetime(2017, 2, 23, 6, 6, 24), datetime.datetime(2017, 10, 30, 14, 14, 57, 180006), datetime.datetime(2017, 10, 19, 19, 16, 15), datetime.datetime(2017, 10, 19, 19, 15, 47))].
```

For each DID generating errors, manually delete the offending record in contents_history.

```
MariaDB [rucio]> delete from contents_history where scope='x1t_SR000' and name='x1t_SR000_161108_0719_tpc' and child_scope='x1t_SR000_161108_0719_tpc';
Query OK, 1 row affected (0.01 sec)
```

Otherwise, verify there are no associated rules for the DID and that the DID is not properly removed by the undertaker daemon.
```
[jlstephen@rucio ~]$ rucio ls x1t_SR001_170608_1321_tpc:acquisition_monitor_data.pickles
+------------------------------------------------------------+--------------+
| SCOPE:NAME                                                 | [DID TYPE]   |
|------------------------------------------------------------+--------------|
| x1t_SR001_170608_1321_tpc:acquisition_monitor_data.pickles | FILE         |
+------------------------------------------------------------+--------------+

[jlstephen@rucio ~]$ rucio list-rules x1t_SR001_170608_1321_tpc:acquisition_monitor_data.pickles
ID    ACCOUNT    SCOPE:NAME    STATE[OK/REPL/STUCK]    RSE_EXPRESSION    COPIES    EXPIRES (UTC)
----  ---------  ------------  ----------------------  ----------------  --------  ---------------

[jlstephen@rucio ~]$ rucio set-metadata --did x1t_SR001_170608_1321_tpc:acquisition_monitor_data.pickles --key lifetime --value 0

[jlstephen@rucio ~]$ tail /var/log/rucio/undertaker.log
...
2017-11-20 16:24:23,374 11176   INFO    Undertaker(1): Receive 1 dids to delete
2017-11-20 16:24:23,374 11176   INFO    Removing did x1t_SR001_170608_1321_tpc:acquisition_monitor_data.pickles (FILE)
2017-11-20 16:24:23,382 11176   INFO    Undertaker(1): Delete 1 dids
```
And yet the file still exists in the catalogue
```
[jlstephen@rucio ~]$ rucio ls x1t_SR001_170608_1321_tpc:acquisition_monitor_data.pickles
+------------------------------------------------------------+--------------+
| SCOPE:NAME                                                 | [DID TYPE]   |
|------------------------------------------------------------+--------------|
| x1t_SR001_170608_1321_tpc:acquisition_monitor_data.pickles | FILE         |
+------------------------------------------------------------+--------------+
```
After ensuring the DID *really* doesn't exist on disk, remove the DID manually from both the replicas and dids tables.
```
MariaDB [rucio]> delete from replicas where scope = 'x1t_SR001_170608_1321_tpc' and name = 'acquisition_monitor_data.pickles';
Query OK, 1 row affected (0.01 sec)

MariaDB [rucio]> delete from dids where scope = 'x1t_SR001_170608_1321_tpc' and name = 'acquisition_monitor_data.pickles';
Query OK, 1 row affected (0.01 sec)
```

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

The various rucio daemons are managed via supervisord.

```
$ cat /etc/supervisord.d/rucio.ini 

[unix_http_server]
file=/tmp/supervisor.sock   ; (the path to the socket file)

[supervisord]
logfile=/var/log/rucio/supervisord.log ; (main log file;default $CWD/supervisord.log)
logfile_maxbytes=50MB        ; (max main logfile bytes b4 rotation;default 50MB)
logfile_backups=10           ; (num of main logfile rotation backups;default 10)
loglevel=info                ; (log level;default info; others: debug,warn,trace)
pidfile=/tmp/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
nodaemon=false               ; (start in foreground if true;default false)
minfds=1024                  ; (min. avail startup file descriptors;default 1024)
minprocs=200                 ; (min. avail process descriptors;default 200)

[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=unix:///tmp/supervisor.sock ; use a unix:// URL  for a unix socket

[program:rucio-conveyor-transfer-submitter]
command=/bin/rucio-conveyor-transfer-submitter  --activities 'User Subscriptions'
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
command=/bin/rucio-reaper --total-workers 10 --greedy
stdout_logfile=/var/log/rucio/reaper.log
environment=GLOBUS_THREAD_MODEL=pthread,X509_USER_PROXY=/opt/rucio/etc/web/x509up,X509_USER_KEY=/opt/rucio/etc/web/x509up,X509_USER_CERT=/opt/rucio/etc/web/x509up

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

[program:rucio-judge-cleaner]
command=/bin/rucio-judge-cleaner
stdout_logfile=/var/log/rucio/judge-cleaner.log

[program:rucio-judge-repairer]
command=/bin/rucio-judge-repairer
stdout_logfile=/var/log/rucio/judge-repairer.log

[program:rucio-conveyor-stager]
command=/bin/rucio-conveyor-stager
stdout_logfile=/var/log/rucio/conveyor-stager.log

[program:rucio-conveyor-receiver]
command=/bin/rucio-conveyor-receiver
stdout_logfile=/var/log/rucio/conveyor-receiver.log

[program:rucio-hermes]
command=/bin/rucio-hermes
stdout_logfile=/var/log/rucio/hermes.log

[program:rucio-kronos]
command=/bin/rucio-kronos
stdout_logfile=/var/log/rucio/kronos.log
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

