---
title: "AWS Security Token Service Storage Locations"
layout: article
excerpt: Follow these steps to set up STS Storage Locations and use them to access your S3 data directly.
category: admin-and-settings
---

Using AWS Security Token Service (STS), Synapse can securely grant you temporary AWS credentials with access to data directly in S3. This can be useful if you want to:

* Transfer data in bulk
* Allow your compute cluster to read S3 objects using the S3 APIs

All of which you can now do with minimal overhead from Synapse.

{% include note.html content="You can only create STS Storage Locations on an empty folder. This is to ensure consistency between the STS Storage Location and the Synapse folder." %}

## Synapse-Managed STS Storage Locations

You can create an STS Storage Location using Synapse storage. Synapse will store the files for you, but you can still get temporary S3 credentials, scoped to just the files and folders within your STS Storage Location.

To set up the STS Storage Location using Synapse storage, first make sure you have an empty Synapse folder, then run the following code:

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

You can then upload files to this STS Storage Location using the normal methods, either through [the Synapse website](https://www.synapse.org/) or through the client of your choice.

## Custom STS Storage Locations

You can also create an STS Storage Location using a custom storage location in an external AWS S3 bucket. To set this up, first follow the steps in [Custom Storage Locations](custom_storage_location.md), up until the step "Set S3 Bucket as Upload Location". Instead, make sure you have an empty Synapse folder, then run the following code.

{% include important.html content="You should specify a baseKey in your storage location, which is a folder path in your bucket that all files in the storage location should go into. The baseKey is optional, but if it is not specified, the temporary AWS credentials vended by STS will give users access to the whole bucket." %}

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

Once your STS Storage Location is set up on your Synapse folder, you can add files to Synapse in one of two ways:

1. You can upload files through Synapse, either through [the Synapse website](https://www.synapse.org/) or through the client of your choice.
2. If you plan to upload files directly to your S3 bucket, or if you already have files in your S3 bucket, you can add representations of those files to Synapse programmatically. Follow the steps at [Adding Files in Your S3 Bucket to Synapse](custom_storage_location.md#adding-files-in-your-s3-bucket-to-synapse).

{% include note.html content="We recommend picking one of these methods and sticking to it and avoiding mixing methods." %}

## Obtaining Temporary S3 Credentials

Once your STS Storage Location is set up, you can use Synapse to request temporary AWS credentials with direct access to your data in S3. These temporary credentials are good for up to 12 hours.

To get these temporary credentials, you may use our [REST interface](https://rest-docs.synapse.org/rest/GET/entity/id/sts.html).

Alternatively, you may use the below sample Java code:

```java
StsCredentials stsCredentials = synapseClient.getTemporaryCredentialsForEntity(folderEntityId, StsPermission.read_only);
AWSCredentials awsCredentials = new BasicSessionCredentials(stsCredentials.getAccessKeyId(), stsCredentials.getSecretAccessKey(), stsCredentials.getSessionToken());
AWSCredentialsProvider awsCredentialsProvider = new AWSStaticCredentialsProvider(awsCredentials);
AmazonS3 s3Client = AmazonS3ClientBuilder.standard().withCredentials(awsCredentialsProvider).build();
```

## Migrating Existing Data

### From an Existing S3 Bucket

If you have existing data in an S3 bucket, either standalone data or data from a previous Synapse project, you can use [this sample code](https://github.com/Sage-Bionetworks/Synapse-Repository-Services/blob/develop/client/sample-code/src/main/java/org/sagebionetworks/sample/sts/MigrateS3Bucket.java) to migrate your S3 data to a new Synapse folder with an STS Storage Location.

{% include note.html content="You'll need to create a new folder with an STS-enabled storage location, as per the instructions above." %}

{% include note.html content="You must be the owner of the storage location in order to import existing S3 files into Synapse this way." %}

### From A Project With Multiple Buckets

If your Synapse project uses data from multiple S3 buckets, or if the data is in an S3 bucket you don't own, then you may need to download the data and re-upload it a new Synapse folder with an STS Storage Location. Use [this sample code](https://github.com/Sage-Bionetworks/Synapse-Repository-Services/blob/develop/client/sample-code/src/main/java/org/sagebionetworks/sample/sts/MigrateSynapseProject.java) to migrate the data in your project.

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

[Custom Storage Locations](custom_storage_location.md)
