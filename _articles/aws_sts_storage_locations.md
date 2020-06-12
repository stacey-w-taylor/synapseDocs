---
title: "AWS Security Token Service Storage Locations"
layout: article
excerpt: Follow these steps to set up STS Storage Locations and use them to access your S3 data directly.
category: managing-data
---

Using AWS Security Token Service (STS), Synapse can securely grant you temporary AWS credentials to access data directly in S3. This can be useful if you want to:

* Transfer data in bulk
* Allow your compute cluster to read S3 objects using the S3 APIs

All of which you can now do with minimal overhead from Synapse. 

There are a few important considerations when determining whether to enable STS on Synapse managed storage compared to external storage with an S3 bucket. With Synapse managed storage, permissions to access are read-only thus data is only accessible to download or compute on directly once STS is enabled. Alternatively, read-only and read-write permissions can be granted on external storage allowing for data to be manipulated directly with the AWS command line interface. Subsequently, connections to Synapse can be updated if data is changed. In some cases, this workflow is preferable.

{% include note.html content="You can only create STS Storage Locations on an empty folder. This is to ensure consistency between the STS Storage Location and the Synapse folder." %}

## Synapse Managed STS Storage Locations

You can create an STS storage location using Synapse storage. Temporary S3 credentials will grant you access to the files and folders scoped to your STS storage location. 

{% include note.html content="For Synapse storage, you can request read-only permissions through STS, but not write permissions. Therefore, you can only upload or modify existing files in Synapse storage through the Synapse website or clients." %}

To set up the STS storage location using Synapse storage, first make sure you have an empty Synapse folder, then run the following code:

##### Python

```python
# Set storage location
import synapseclient
import json
syn = synapseclient.login()
FOLDER = 'syn12345'

destination = {'uploadType':'S3',
               'stsEnabled':True,
               'concreteType':'org.sagebionetworks.repo.model.project.S3StorageLocationSetting'}
destination = syn.restPOST('/storageLocation', body=json.dumps(destination))

project_destination ={'concreteType': 'org.sagebionetworks.repo.model.project.UploadDestinationListSetting',
                      'settingsType': 'upload'}
project_destination['locations'] = [destination['storageLocationId']]
project_destination['projectId'] = FOLDER

project_destination = syn.restPOST('/projectSettings', body = json.dumps(project_destination))
```

##### R

```r
#set storage location
library(synapser)
library(rjson)
synLogin()
folderId <- 'syn12345'

destination <- list(uploadType='S3',
                    stsEnabled=TRUE,
                    concreteType='org.sagebionetworks.repo.model.project.S3StorageLocationSetting')
destination <- synRestPOST('/storageLocation', body=toJSON(destination))

projectDestination <- list(concreteType='org.sagebionetworks.repo.model.project.UploadDestinationListSetting',
                           settingsType='upload')
projectDestination$locations <- list(destination$storageLocationId)
projectDestination$projectId <- folderId

projectDestination <- synRestPOST('/projectSettings', body=toJSON(projectDestination))
```

Once the Synapse managed STS storage location is set up, you can upload files through the [Synapse](https://www.synapse.org/) or the Synapse client of your choice.

## External STS Storage Locations

You can also create a STS storage location in an external AWS S3 bucket. 

There are benefits of creating connections to Synapse from an external bucket. If you already have data stored in S3, or if you have large amounts of data that you want to transfer with the AWS command line interface, you can avoid uploading data to Synapse managed storage by creating connections directly to the S3 bucket. Enabling an STS storage location in the external bucket allows access to the S3 directly for future computing.

Follow the steps in the [Custom Storage Locations]({{ site.baseurl }}{% link _articles/custom_storage_location.md %}#setting-up-an-external-aws-s3-bucket) article to set read-write or read-only permissions on your external S3 bucket and enable cross-origin resource sharing (CORS). You may use AWS cloudformation for set up. Instead of [setting the S3 bucket as upload location]({{ site.baseurl }}{% link _articles/custom_storage_location.md %}#set-s3-bucket-as-upload-location), complete set up by running the following code with an empty Synapse folder:

{% include important.html content="If a baseKey is not specified, the temporary AWS credentials vended by STS will give users access to the whole bucket. To prevent access to the whole bucket, enter a folder path in your bucket that all files in the storage location should go into as the baseKey. " %}

##### Python

```python
# Set storage location
import synapseclient
import json
syn = synapseclient.login()
FOLDER = 'syn12345'

destination = {'uploadType':'S3',
               'stsEnabled':True,
               'bucket':'nameofyourbucket',
               'baseKey':'nameofyourbasekey',
               'concreteType':'org.sagebionetworks.repo.model.project.ExternalS3StorageLocationSetting'}
destination = syn.restPOST('/storageLocation', body=json.dumps(destination))

project_destination ={'concreteType': 'org.sagebionetworks.repo.model.project.UploadDestinationListSetting',
                      'settingsType': 'upload'}
project_destination['locations'] = [destination['storageLocationId']]
project_destination['projectId'] = FOLDER

project_destination = syn.restPOST('/projectSettings', body = json.dumps(project_destination))
```

##### R

```r
#set storage location
library(synapser)
library(rjson)
synLogin()
folderId <- 'syn12345'

destination <- list(uploadType='S3',
                    stsEnabled=TRUE,
                    bucket='nameofyourbucket',
                    baseKey='nameofyourbasekey',
                    concreteType='org.sagebionetworks.repo.model.project.ExternalS3StorageLocationSetting')
destination <- synRestPOST('/storageLocation', body=toJSON(destination))

projectDestination <- list(concreteType='org.sagebionetworks.repo.model.project.UploadDestinationListSetting',
                           settingsType='upload')
projectDestination$locations <- list(destination$storageLocationId)
projectDestination$projectId <- folderId

projectDestination <- synRestPOST('/projectSettings', body=toJSON(projectDestination))
```

Once your STS storage location is set up on your Synapse folder, you can add files through the [Synapse](https://www.synapse.org/) website or the Synapse client of your choice. If you plan to upload files directly to your S3 bucket, or if you already have files in your S3 bucket, you can add representations of those files to Synapse programmatically. Follow the [steps to add files in your S3 bucket to Synapse]({{ site.baseurl }}{% link _articles/custom_storage_location.md %}#adding-files-in-your-s3-bucket-to-synapse).

{% include note.html content="We recommend picking one of these methods and sticking to it and avoiding mixing methods." %}

## Obtaining Temporary S3 Credentials

Once your STS storage location is set up, you can use Synapse to request temporary AWS credentials to access your data in S3 directly. These temporary credentials are active for 12 hours.

To get temporary credentials, Python and Java code is provided below. The [REST interface](https://rest-docs.synapse.org/rest/GET/entity/id/sts.html) is also available to request temporary credentials.


##### Java

```java
StsCredentials stsCredentials = synapseClient.getTemporaryCredentialsForEntity(folderEntityId, StsPermission.read_only);
AWSCredentials awsCredentials = new BasicSessionCredentials(stsCredentials.getAccessKeyId(), stsCredentials.getSecretAccessKey(), stsCredentials.getSessionToken());
AWSCredentialsProvider awsCredentialsProvider = new AWSStaticCredentialsProvider(awsCredentials);
AmazonS3 s3Client = AmazonS3ClientBuilder.standard().withCredentials(awsCredentialsProvider).build();
```

##### Python

```python
import synapseclients
import boto3

syn = synapseclient.login()
sts_credentials = syn.restGET(f"/entity/{FOLDER}/sts?permission=read_only")
client = boto3.client(
    's3',
    aws_access_key_id=sts_credentials['accessKeyId'],
    aws_secret_access_key=sts_credentials['secretAccessKey'],
    aws_session_token=sts_credentials['sessionToken'],
)

ent = syn.get("syn12345", downloadFile=False)
client.download_file(ent._file_handle['bucketName'],
                                   ent._file_handle['key'],
                                   ent.name)
# ent._file_handle['bucketName'] -- The name of the Synapse bucket to download from.
# ent._file_handle['key'] -- The path of the file in the Synapse bucket
# ent.name -- The path to the file to download to.
## Migrating Existing Data

### From an Existing S3 Bucket

If you have existing data in an S3 bucket, either stand-alone data or data from a previous Synapse project, you can use [this sample code](https://github.com/Sage-Bionetworks/Synapse-Repository-Services/blob/develop/client/sample-code/src/main/java/org/sagebionetworks/sample/sts/MigrateS3Bucket.java) to migrate your S3 data to a new Synapse folder with a STS storage location.

{% include note.html content="You'll need to create a new folder with an STS-enabled storage location, as per the instructions above." %}

{% include note.html content="You must be the owner of the storage location in order to import existing S3 files into Synapse this way." %}

### From A Project With Multiple Buckets

If your Synapse project uses data from multiple S3 buckets, or if the data is in an S3 bucket you don't own, then you may need to download the data and re-upload it a new Synapse folder with a STS storage location. Use [this sample code](https://github.com/Sage-Bionetworks/Synapse-Repository-Services/blob/develop/client/sample-code/src/main/java/org/sagebionetworks/sample/sts/MigrateSynapseProject.java) to migrate the data in your project.

{% include note.html content="You'll need to create a new folder with an STS-enabled storage location, as per the instructions above." %}

{% include note.html content="This method may incur data transfer costs from S3." %}

## Additional Restrictions

* STS Storage Locations can only be added to folders, not projects.
* An STS Storage Location can only be added to or removed from empty folders.
* An STS Storage Location cannot be added alongside other storage locations.
* If a parent folder has an STS Storage Location, child folders cannot override storage locations, nor can they override ACLs.
* A folder with an STS Storage Location can only contain files and other folders.
* A folder with an STS Storage Location cannot contain a file from outside that storage location.
* A file in an STS Storage Location cannot be placed in a folder with a different storage location.
* A file in an STS Storage Location must have a bucket and base key matching that storage location.
* If a parent folder has an STS Storage Location, the child folder cannot be moved to a parent with a different storage location, nor can child folders from other storage locations be moved to the parent folder.
* If an STS Storage Location is defined on a folder, that folder cannot be placed within another folder hierarchy that also defines an STS Storage Location (even if they are the same storage location).

## See Also

[Custom Storage Locations]({{ site.baseurl }}{% link _articles/custom_storage_location.md %}#toc-custom-storage-locations)
