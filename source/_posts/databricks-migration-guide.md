---
title: Databricks Migration Guide
date: 2020-06-24 22:18:09
tags: [Databricks, migration, scripts]
categories: Databricks
description: When you need to migrate an old Databricks to a new Databricks, all of the files, jobs, clusters, configurations and dependencies are supposed to move. It is time consuming and also easy to omit some parts. I document the detailed migration steps, and also write several scripts to automatically migrate folders, clusters and jobs. In this chapter, I will show you how to migrate Databricks.
---

# Databricks migration steps
When you need to migrate an old Databricks to a new Databricks, all of the files, jobs, clusters, configurations and dependencies are supposed to move. It is time consuming and also easy to omit some parts. I document the detailed migration steps, and also write several scripts to automatically migrate folders, clusters and jobs.  
In this chapter, I will show you how to migrate Databricks.
## 0. Prepare all scripts
Navigate to https://github.com/xinyeah/Azure-Databricks-migration-tutorial, fork this repository, and download all needed scripts.
## 1. Install databricks-cli
```
 pip3 install databricks
```
## 2. Set up authentication for two profiles
Set up authentication for two profiles for old databricks and new databricks. This CLI authentication need to done by a personal access token.
### 2.1 Generate a personal access token.
[Here](https://docs.databricks.com/dev-tools/api/latest/authentication.html) is step by step guide to generate it.
### 2.2 Copy the generated token and store it as a secret in Azure Key Vault.

On the Key Vault properties pages, select **Secrets**.
Click on **Generate/Import**.
On the Create a secret screen choose the following values:
- **Upload options**: Manual.
- **Name**: 
- **Value**: paste the generated token here
- Leave the other values to their defaults. Click **Create**.



### 2.3 Set up profiles

In this case, the profile *primary* is for the old Databricks, and the profile *secondary* is for the new one.
```
databricks configure --profile primary --token
databricks configure --profile secondary --token
```
Every time set up a profile, you need to provide the Databricks host url and the personal access token generated previously.
<img src="https://raw.githubusercontent.com/xinyeah/xinyeah.github.io/master/images/image-20200619153249382.png" alt="image-20200619153249382" style="zoom: 80%;" />



### 2.4 Validate the profile

```
databricks fs ls --absolute --profile primary
databricks fs ls --absolute --profile secondary
```
<img src="https://raw.githubusercontent.com/xinyeah/xinyeah.github.io/master/images/image-20200624141216762.png" alt="image-20200624141216762" style="zoom:50%;" />
Here is the DBFS root locations from [docs](https://docs.microsoft.com/en-us/azure/databricks/data/databricks-file-system)
![image-20200624113619104](https://raw.githubusercontent.com/xinyeah/xinyeah.github.io/master/images/image-20200624113619104.png)

## 3. Migrate Azure Active Directory users
 3.1 Navigate to the old Databricks UI, expand **Account** in the right corner, then click **Admin Console**. You can get a list of users as admin in this Databricks.

<img src="https://raw.githubusercontent.com/xinyeah/xinyeah.github.io/master/images/image-20200619153401452.png" alt="image-20200619153401452" style="zoom: 80%;" />
<img src="https://raw.githubusercontent.com/xinyeah/xinyeah.github.io/master/images/image-20200619153500424.png" alt="image-20200619153500424" style="zoom:80%;" />

3.2 Navigate to the new Databricks portal, click **Add User** under **Users** tag of **Admin Console** to add admins.

<img src="https://raw.githubusercontent.com/xinyeah/xinyeah.github.io/master/images/image-20200624142142524.png" alt="image-20200624142142524" style="zoom: 35%;" />



## 4. Migrate the workspace folders and notebooks

**Solution 1** 
Put the [migrate-folders.py](https://github.com/xinyeah/Azure-Databricks-migration-tutorial/blob/master/step4-migrate-folders.py) in a separate folder (it will export files in this folder), and then run the migrate-folders.py script to migrate folders and notebooks. Libraries are not included using this scripts. It is shown in Step 5 to migrate libraries.
Remember to replace the profile variables in this script to your customized profile names:
```python
EXPORT_PROFILE = "primary"
IMPORT_PROFILE = "secondary"
```
**Solution 2** 
Also, you can do it manually: Export as DBC file and then import.
<img src="https://raw.githubusercontent.com/xinyeah/xinyeah.github.io/master/images/image-20200619155250927.png" alt="image-20200619155250927" style="zoom:50%;" />

## 5. Migrate libraries
There is no external API for libraries, so need to reinstall all libraries into new Databricks manually.
### 5.1 List all libraries in the old Databricks.

<img src="https://raw.githubusercontent.com/xinyeah/xinyeah.github.io/master/images/image-20200619165944999.png" alt="image-20200619165944999" style="zoom:67%;" />

### 5.2 Install all libraries.
Maven libraries:
<img src="https://raw.githubusercontent.com/xinyeah/xinyeah.github.io/master/images/image-20200619165917476.png" alt="image-20200619165917476" style="zoom:67%;" />
PyPI libraries:
<img src="https://raw.githubusercontent.com/xinyeah/xinyeah.github.io/master/images/image-20200619170223832.png" alt="image-20200619170223832" style="zoom:67%;" />

## 6. Migrate the cluster configuration
Run [migrate-cluster.py](https://github.com/xinyeah/Azure-Databricks-migration-tutorial/blob/master/step6-migrate-cluster.py) to migrate all **interactive clusters**. This script will **skip** all *job* source clusters.
Remember to replace the profile variables in this script to your customized profile names:
```python
EXPORT_PROFILE = "primary"
IMPORT_PROFILE = "secondary"
```
## 7. Migrate the jobs configuration
Run [migrate-job.py](https://github.com/xinyeah/Azure-Databricks-migration-tutorial/blob/master/step7-migrate-job.py) to migrate all jobs, **schedule information** will be removed so job doesn't start before proper cutover.
Remember to replace the profile variables in this script to your customized profile names:
```python
EXPORT_PROFILE = "primary"
IMPORT_PROFILE = "secondary"
```
## 8. Migrate Azure Key Vaults secret scopes
There are two types of secret scope: Azure Key Vault-backed and Databricks-backed.
Creating an Azure Key Vault-backed secret scope is supported only in the Azure Databricks UI. You cannot create a scope using the Secrets CLI or API.
### List all secret scopes:
```
databricks secrets list-scopes --profile primary
```
![image-20200619175501231](https://raw.githubusercontent.com/xinyeah/xinyeah.github.io/master/images/image-20200619175501231.png)
### Generate key vault-backed secret scope:
1. Go to `https://<databricks-instance>#secrets/createScope`. This URL is case sensitive; scope in `createScope` must be uppercase.
   <img src="https://raw.githubusercontent.com/xinyeah/xinyeah.github.io/master/images/azure-kv-scope.png" alt="Create scope" style="zoom:50%;" />
2. Enter the name of the secret scope. Secret scope names are case insensitive.
3. These properties are available from the **Properties** tab of an Azure Key Vault in your Azure portal.
   <img src="https://raw.githubusercontent.com/xinyeah/xinyeah.github.io/master/images/azure-kv.png" alt="Azure Key Vault Properties tab" style="zoom:50%;" />
4. Click the **Create** button.



## 9. Migrate Azure blob storage and Azure Data Lake Storage mounts

There is no external API to use, have to manually remount all storage.
### 9.1 List all mount points in old Databricks using `notebook`.
```python
dbutils.fs.mounts()
```
### 9.2 Remount all blob storage following the official [docs](https://docs.databricks.com/data/data-sources/azure/azure-storage.html) using `notebook`.
```python
dbutils.fs.mount(
  source = "wasbs://<container-name>@<storage-account-name>.blob.core.windows.net",
  mount_point = "/mnt/<mount-name>",
  extra_configs = {"<conf-key>":dbutils.secrets.get(scope = "<scope-name>", key = "<key-name>")})
```
where
- `<mount-name>` is a DBFS path representing where the Blob storage container or a folder inside the container (specified in `source`) will be mounted in DBFS.
- `<conf-key>` can be either `fs.azure.account.key.<storage-account-name>.blob.core.windows.net` or `fs.azure.sas.<container-name>.<storage-account-name>.blob.core.windows.net`
- `dbutils.secrets.get(scope = "<scope-name>", key = "<key-name>")` gets the key that has been stored as a [secret](https://docs.databricks.com/security/secrets/secrets.html) in a [secret scope](https://docs.databricks.com/security/secrets/secret-scopes.html).



## 10. Migrate cluster init scripts

Copy all cluster initialization scripts to new Databricks using DBFS CLI.
```bash
// Primary to local
dbfs cp -r dbfs:/databricks/init ./old-ws-init-scripts --profile primary

// Local to Secondary workspace
dbfs cp -r old-ws-init-scripts dbfs:/databricks/init --profile secondary
```


## 11. ADF config

For Databricks jobs scheduled by Azure Data Factory, navigate to Azure Data Factory UI. Create a new Databricks linked service linked to the new Databricks by the personal access key generated in step 2.
<img src="https://raw.githubusercontent.com/xinyeah/xinyeah.github.io/master/images/image-20200624174250964.png" alt="image-20200624174250964" style="zoom: 33%;" />

# Reference
https://docs.microsoft.com/en-us/azure/databricks/administration-guide/cloud-configurations/azure/vnet-inject
https://docs.microsoft.com/en-us/azure/azure-databricks/howto-regional-disaster-recovery#detailed-migration-steps
https://docs.microsoft.com/en-us/azure/databricks/dev-tools/cli/
https://docs.microsoft.com/en-us/azure/databricks/security/secrets/secret-scopes