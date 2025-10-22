![Title image](../../../common/images/customer.logo2.png)

# Setting up the OCI environment using the individual scripts

For people who have want to run the script manually per script these instructions explain the process.

**IMORTANT** Cloud shell processor architecture and networking

The OCI cloud shell supports the creation of cloud shell instances using both ARM and X64 based architectures, however these scripts are currently setup to download only x64 based executables, to build x64 based containers and create x64 based clusters.

Work is currently underway to make the scripts a more processor architecture independent (or at least to work against both ARM and x64 based processor architectures) however this involves updating the download process, the build process, the cluster setup process and also ensuring that both x64 and ARM based versions of all of the containers that are used as available. This means that the work will take some time to complete.
 
As such currently the scripts will test to see if the cloud shell instance is running on an ARM or x64 architecture, if it detects an ARM based architecture the script will stop. You will then need to switch to an x64 based architecture. Use the Actions menu for the cloud shell (the upper left menu of the cloud shell window) then chose architecture, then chose an x64 based architecture. The cloud shell will restart (the downloads and home directory will remain). Note that not all tenancies support this option (the `Always Free` tenancies do not, but they don't have the resources to run Kubernetes clusters either) and unfortunately for those tenancies this lab is currently now available.

By default the OCI Cloud shell does not have access to the internet, however this is needed for the lab as you will download some scripts to configure your environment, these scripts will also need to access the internet to download docker images and other things. Thus we need to enable the cloud shell public internet access. If you are an admin in your tenancy you can achieve this by clicking the "Network" dropdown on the upper left of the cloud shell pane, then select "Public network". It will take a short time for the cloud shell to switch. If this option is not available, or you cannot pull the git repo (as below) then you will need to set policies enabled to let you access the cloud shell. [Please see the cloud shell networking documentation.](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellintro_topic-Cloud_Shell_Networking.htm)
You are assumed to have downloaded the helidon-kubernetes script repo, and to have downloaded step.

## Task 1: Recording your initials

For a number of activities in the lab we use your initials to identify instances (database, Kubernetes etc.) We do this to enable multiple user to operate in the same tenancy withouth conflict (This is only for some versions of the lab, in most cases you will be using your own free trial tenancy). 

We need to capture your initials and save them. It is important that when you enter your initials you use lower case only and only the letters a-z, no numbers of special characters

  1. If you are not already there open the OCI cloud shall and go to the scripts directory, type
  
  ```bash
  <copy>cd $HOME/helidon-kubernetes/setup/common</copy>
  ```
  
  2. Run the script to gather and save your initials, when prompted by the script enter your initials and press return, in the example below as my name is Tim Graves I used `tg` as the initials
  
  ```bash
  <copy>bash ./initials-setup.sh</copy>
  ```
  
  ```
  Please can you enter your initials - use lower case a-z only and no spaces, for example if your name is John Smith your initials would be js. This will be used to do things like name the database
tg
OK, using tg as your initials
```

Of course unless your initials are also `tg` you would enter something different !


## Task 2: Configuring your user identity

A number of processes require knowledge of your users identity, this script will locate that and save it away

1. If you are not already there open the OCI cloud shall and go to the scripts directory, type
  
  ```bash
  <copy>cd $HOME/helidon-kubernetes/setup/common</copy>
  ```
  
  2. Run the compartment setup script, it does not require any input
  
  ```bash
  <copy>bash ./user-identity-setup.sh</copy>
  ```
  
  ```
  Loading existing settings
No existing user info, retrieving
Checking for local user
Checking for federated user
You are a federated user, getting information
You are a federated user, your user name is oracleidentitycloudservice/tim.graves@oracle.com, saved details
```


## Task 3: Creating the compartment

In OCI all resources live in compartments, we are going to create a compartment for this lab. If you have already created a compartment in the past when doing other parts of this lab then please re-use the same one when prompted.

The following instructions follow through the prompts one at a time, unless you have specific requirements (usually because you are not running in a free trial account) then just take the defaults.

  1. If you are not already there open the OCI cloud shall and go to the scripts directory, type
  
  ```bash
  <copy>cd $HOME/helidon-kubernetes/setup/common</copy>
  ```
  
  2. Run the compartment setup script
  
  ```bash
  <copy>bash ./compartment-setup.sh</copy>
  ```
  
  ```
  Loading existing settings
No reuse information for compartment
Parent is tenancy root
This script will create a compartment called CTDOKE for you if it doesn't exist, this will be in the Tenancy root. If a compartment with the same name already exists you can re-use change the name to create or re-use a different compartment.
If you want to use somewhere different from Tenancy root as the parent of the compartment you are about to create (or re-use) then enter n, if you want to use Tenancy root for your parent then enter y
Use the Tenancy root (y/n) ?
```

  3. At this prompt unless you want to create the compartment somewhere else please enter `y` and press return. You should chose the tenancy root unless you are in a shared or commercial tenancy and you explicitly understand you need to work somewhere other than the tenancy root. If you really want to create or reuse a compartment somewhere other than the tenancy root enter `n` and follow the instructions that will be displayed.
  
  ```
  We are going to create or if it already exists reuse use a compartment called CTDOKE in Tenancy root, if you want you can change the compartment name from CTDOKE - this is not recommended and you will need to remember to use a different name in the lab.
 Do you want to use CTDOKE as the compartment name (y/n) ? 
 ```
 
  4. You are being asked if you want to use `CTDOKE` as the name of the compartment to use (or if it already exists re-use). If you have chosen a different compartment as the parent that will be displayed instead of `Tennancy root`. Unless you explicitly know you need to use a different compartment then please enter `y` here. If you really want a different name for your compartment (perhaps because you are re-using one with a different name, or `CTDOKE` is in use for something else) then type `n` and follow the prompts to enter a different name.
  
  ```
  OK, going to use CTDOKE as the compartment name
Compartment CTDOKE, doesn't already exist in the Tenancy root, creating it
Created compartment CTDOKE in the Tenancy root It's OCID is ocid1.compartment.oc1..aaaaabaas5lazl434a7oizjiuife3tjffwucxrbom2zdhyhvh5t66mb75olq
It may take a short while before new compartment has propogated and the web UI reflects this
```
  
  In this case the compartment `CTDOKE` (or whatever name you entered if you chose to override it) did not exist in the tenancy root, so the compartment was created, If it had existed then the script would have retrieved it's information for re-use.
  
  **Important** It can take a short while for the compartment information to be propagated to all OCI regions and environments.
  
## Task 4: Creating the database

The microservices that form the base content of these labs use a database to store their data, so we need to create a database. The Following script will create the database in the compartment we just created, then download the connection information (the "Wallet") and use that to connect to the database and create the user used by the labs.

  1. If you are not already there open the OCI cloud shall and go to the scripts directory, type
  
  ```bash
  <copy>cd $HOME/helidon-kubernetes/setup/common</copy>
  ```
  
  2. Run the script to create or re-use the database
  
  ```bash
  <copy>bash ./database-setup.sh</copy>
  ```
  
  ```
  Loading existing settings information
No reuse information for database
Operating in compartment CTDOKE
Do you want to use tgdb as the name of the databse to create or re-use in CTDOKE?
```

  3. if you are creating a new database then enter `y` if however you have an existing database in this compartment, perhaps created doing another part of these Kubernetes labs which used a different name then please enter `n` and when prompted enter that name - you will need to know the ADMIN password for that database and will be prompted for it.
  
  ```
  OK, going to use tgdb as the database name
Checking for database tgdb in compartment CTDOKE
Database named tgdb doesn't exist, creating it, there may be a short delay
The generated database admin password is 2005758405_SeCrEt Please ensure that you save this information in case you need it later
```

  4. The database will be created for you unless you chose a name that already exists, The script will then setup a temporary set of access credentials using the wallet, connect to the database using the password it generated (or in the case of a database that you are re-using the password you provided) and setup the labs user in the database.
  
  If you are reusing a database and had already setup the labs user then you may get error messages that the user conflicts with the existing one.
  
  
  ```
  Downloaded Wallet.zip file
Preparing temporary database connection details
Getting wallet contents for temporaty processing
Archive:  Wallet.zip
  inflating: README                  
  inflating: cwallet.sso             
  inflating: tnsnames.ora            
  inflating: truststore.jks          
  inflating: ojdbc.properties        
  inflating: sqlnet.ora              
  inflating: ewallet.p12             
  inflating: keystore.jks            
updating temporary sqlnet.ora
Connecting to database to create labs user

SQL*Plus: Release 19.0.0.0.0 - Production on Fri Nov 19 21:22:24 2021
Version 19.13.0.0.0

Copyright (c) 1982, 2021, Oracle.  All rights reserved.


Connected to:
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.13.0.1.0


User created.


Grant succeeded.


Grant succeeded.


Grant succeeded.


Grant succeeded.

Disconnected from Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.13.0.1.0
Deleting temporary database connection info
The generated admin password is 2005758405_SeCrEt Please ensure that you save this information in case you need it later

```
  
  5. **IMPORTANT** you are **strongly** recommended to save the generated database password (`2005758405_SeCrEt` in this case) in case you need to administer the database later. If there is an existing `$HOME/Wallet.zip` then it will be saved before downloading the new wallet.
  
## Acknowledgements

* **Author** - Tim Graves, Cloud Native Solutions Architect, Oracle EMEA Cloud Native Application Development specialists Team
* **Author** - Jan Leemans, Director Business Development, EMEA Divisional Technology
* **Last Updated By** - Tim Graves, May 2023