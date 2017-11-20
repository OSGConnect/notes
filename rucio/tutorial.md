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
