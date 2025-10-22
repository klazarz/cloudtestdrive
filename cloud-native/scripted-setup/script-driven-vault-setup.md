![Title image](../../../common/images/customer.logo2.png)

# OCI Vault Creation


**IMORTANT** Cloud shell processor architecture and networking

The OCI cloud shell supports the creation of cloud shell instances using both ARM and X64 based architectures, however these scripts are currently setup to download only x64 based executables, to build x64 based containers and create x64 based clusters.

Work is currently underway to make the scripts a more processor architecture independent (or at least to work against both ARM and x64 based processor architectures) however this involves updating the download process, the build process, the cluster setup process and also ensuring that both x64 and ARM based versions of all of the containers that are used as available. This means that the work will take some time to complete.
 
As such currently the scripts will test to see if the cloud shell instance is running on an ARM or x64 architecture, if it detects an ARM based architecture the script will stop. You will then need to switch to an x64 based architecture. Use the Actions menu for the cloud shell (the upper left menu of the cloud shell window) then chose architecture, then chose an x64 based architecture. The cloud shell will restart (the downloads and home directory will remain). Note that not all tenancies support this option (the `Always Free` tenancies do not, but they don't have the resources to run Kubernetes clusters either) and unfortunately for those tenancies this lab is currently now available.

By default the OCI Cloud shell does not have access to the internet, however this is needed for the lab as you will download some scripts to configure your environment, these scripts will also need to access the internet to download docker images and other things. Thus we need to enable the cloud shell public internet access. If you are an admin in your tenancy you can achieve this by clicking the "Network" dropdown on the upper left of the cloud shell pane, then select "Public network". It will take a short time for the cloud shell to switch. If this option is not available, or you cannot pull the git repo (as below) then you will need to set policies enabled to let you access the cloud shell. [Please see the cloud shell networking documentation.](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellintro_topic-Cloud_Shell_Networking.htm)

## Introduction

The OCI Vault service allows you to hold encrypted information, for example secrets or encryption keys and the vault can manage these for you.

In some labs you will be using the vault, for example in the OCI DevOps lab you can use the vault to hold secrets that represent the details of the OCIR repository that will hold the images your build pipleines produce.

### Objectives

Using the OCI Cloud shell to run a script we will :

  - Create a OCI Vault
  
  - Create a master key for the vault (this can then be used for encrypting secrets held in the vault)

**IMPORTANT** You must have setup a compartment, configured your initials and extracted the user info

## Task 1: Downloading the latest scripts

There are a number of scripts that we have created for this lab to make life easier, if you have previously downloaded those recently (in the last few hours) then you can skip this Task and go to task 2. If you downloaded them longer ago then you should ensure you have the latest version, if you haven't downloaded them then you will need to do so.

If you have recently downloaded the scripts then please jump to **Task 2**, and skip the remainder of task 1.

If you are not sure if you have downloaded the latest version then you can check

  - In the OCI Cloud shell type 
  
  ```bash
  <copy>ls $HOME/helidon-kubernetes</copy>
  ```

If you get output like this then you need to download the scripts, follow the process in **Task 1a**

```
ls: cannot access /home/tim_graves/helidon-kubernetes: No such file or directory
```

If you get output similar to this (the file names and count may vary slightly as the lab evolves) then you have downloaded the scripts previously, but you should get the latest versions, please follow the steps in **Task 1b**

```
base-kubernetes          configurations       deploy.sh   monitoring-kubernetes  README.md     servicesClusterIP.yaml  stockmanager-deployment.yaml  undeploy.sh
cloud-native-kubernetes  create-test-data.sh  management  README                 service-mesh  setup                   storefront-deployment.yaml    zipkin-deployment.yaml
```

### Task 1a: Downloading the scripts

We will use git to download the scripts

  1. Open the OCI Cloud Shell

  2. Make sure you are in the top level directory
  
  ```bash
  <copy>cd $HOME</copy>
  ```
  
  3. Clone the repository with all scripts from github into your OCI Cloud Shell environment
  
  ```bash
  <copy>git clone https://github.com/CloudTestDrive/helidon-kubernetes.git</copy>
  ```
  
  ```
  Cloning into 'helidon-kubernetes'...
remote: Enumerating objects: 723, done.
remote: Counting objects: 100% (723/723), done.
remote: Compressing objects: 100% (452/452), done.
remote: Total 723 (delta 423), reused 537 (delta 249), pack-reused 0
Receiving objects: 100% (723/723), 110.23 KiB | 0 bytes/s, done.
Resolving deltas: 100% (423/423), done.
```

Note that the precise details will vary as the lab is updated over time.

Please go to **Task 2** Do not continue with anything else in task 1

### Task 1b: Updating the scripts

We will use git to update the scripts

  1. Open the OCI Cloud shell type
  
  2. Make sure you are in the home directory
  
  ```bash
  <copy>cd $HOME/helidon-kubernetes</copy>
  ```
  
  3. Use git to get the latest updates
  
  ```bash
  <copy>git pull</copy>
  ```

```
remote: Enumerating objects: 21, done.
remote: Counting objects: 100% (21/21), done.
remote: Compressing objects: 100% (8/8), done.
remote: Total 14 (delta 6), reused 14 (delta 6), pack-reused 0
Unpacking objects: 100% (14/14), done.
From https://github.com/CloudTestDrive/helidon-kubernetes
   51c1ba8..f3845ad  master     -> origin/master
Updating 51c1ba8..f3845ad
Fast-forward
 setup/common/compartment-destroy.sh                 | 50 ++++++++++++++++++++++++++++++++++++++++++++++++++
 setup/common/compartment-setup.sh                   | 14 ++++++++++++++
 setup/common/database-destroy.sh                    | 38 ++++++++++++++++++++++++++++++++++++++
 setup/common/database-setup.sh                      | 15 ++++++++++++++-
 setup/common/delete-from-saved-settings.sh          | 12 ++++++++++++
 setup/common/initials-destroy.sh                    |  3 +++
 setup/common/{get-initials.sh => initials-setup.sh} |  2 +-
 setup/common/kubernetes-destroy.sh                  | 57 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 setup/common/kubernetes-setup.sh                    | 24 ++++++++++++++++++------
 setup/common/oke-terraform.tfvars                   |  2 ++
 10 files changed, 209 insertions(+), 8 deletions(-)
 create mode 100644 setup/common/compartment-destroy.sh
 create mode 100644 setup/common/database-destroy.sh
 create mode 100644 setup/common/delete-from-saved-settings.sh
 create mode 100644 setup/common/initials-destroy.sh
 rename setup/common/{get-initials.sh => initials-setup.sh} (89%)
 create mode 100644 setup/common/kubernetes-destroy.sh
```

Note that the output will vary depending on exactly what changes have been made since you first download the scripts or last updated them.

Please continue with **Task 2**

## Task2: Running the vault setup script

<details><summary><b>What does this script actually do ?</b></summary>  

The script first of all checks the `$HOME/hk8sLabsSettings` file to see if you've already setup a vault and master key (or told it about a specific vault and master key to use) if so it will use that and just exit.

It then checks for the required resources to make sure that you have enough to create the vault and master key (there's no point in starting something that won't complete.)

After checking it will create a vault, and once created create a master key. The OCID's of these will be written to the `$HOME/hk8sLabsSettings` file so they can be checked and used later.

---

</details>

  1. If you are not already there open the OCI cloud shell and go to the scripts directory, type
  
  ```bash
  <copy>cd $HOME/helidon-kubernetes/setup/common/vault</copy>
  ```
  
  2. Run the script to create the vault, in the OCI Cloud Shell type
  
  ```bash
  <copy>bash vault-setup.sh</copy>
  ```
  
  ```
  Loading existing settings information
No reuse information for vault
Operating in compartment CTDOKE
Do you want to use tgLabsVault as the name of the vault to create or re-use in CTDOKE?
```

  3. The script will ask you to confirm the name of the vault to use, unless you need to change the name (for example to re-use an existing vault) then enter `y` and press return. The script will then proceed automatically
  
  ```
  OK, going to use tgLabsVault as the vault name
Checking for  vault tgLabsVault in compartment CTDOKE
No vault named tgLabsVault pending deletion
Vault named tgLabsVault doesn't exist, creating it, there may be a short delay
Action completed. Waiting until the resource has entered state: ('ACTIVE',)
Vault being created using OCID ocid1.vault.oc1.uk-london-1.ejrbjv6qaadrw.abshqljrbra25ajo6rhq766vbzo57d5mvh723gd5v4vmrvynsgsrslp3wrua
```

  4. To protect secrets we need a vault key, this will be of type AES, in the same directory run this, it will create a key names `<your initials>AES` or type `AES` with a key length of `32` (this is in bytes, so 256 bits)
  
  ```bash
  <copy>bash vault-key-setup.sh AES AES 32</copy>
  ```
 
  ```
Setting up for Vault master key
Getting vault endpoint for vault OCID ocid1.vault.oc1.uk-london-1.ejrbjv6qaadrw.abshqljrbra25ajo6rhq766vbzo57d5mvh723gd5v4vmrvynsgsrslp3wrua
Checking for existing key named tgKey in endpoint https://ejrbjv6qaadrw-management.kms.uk-london-1.oci.oraclecloud.com in compartment OCID ocid1.compartment.oc1..aaaaaaaaxoituenn4vx75p3wlv5czz35wyb3culzulomli7b7wsjekmrhsvq
No key named tgAES pending deletion
No existing key with name tgAES, creating it
Action completed. Waiting until the resource has entered state: ('ENABLED',)
Vault master key created with OCID ocid1.key.oc1.uk-london-1.ejrbjv6qaadrw.abshqljrksj4rlquue2tflgqzi6fdsv2wny2wlgw4dxbq3ctaltwkq4talra
  ```
  
  It can sometimes take a few mins to create the vault and key, while this is happening there will be no output once the `...Waiting until the resource...` message is displayed.
  
  Of course the precise output will vary for your environment.
  
  Note that if a vault or key already exist with the same name on your environment the script will re-use them, and if they were in a "Pending Deletion" state then you will be given the option to cancel the deletion process and re-use them, the script will prompt you if needed. 

## Acknowledgements

* **Author** - Tim Graves, Cloud Native Solutions Architect, Oracle EMEA Cloud Native Application Development specialists Team
* **Last Updated By** - Tim Graves, May 2023