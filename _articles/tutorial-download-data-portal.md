---
title: Download Data Programmatically from Portals
layout: article
excerpt: Use R, Python or the command line to download data.
category: in-practice
---

Portals allow you to explore data stored in [Synapse](https://sagebionetworks.org/tools_resources/synapse-platform/), a technology platform that allows researchers to aggregate, organize, analyze and share scientific data, code and insights. Synapse is designed to integrate seamlessly with your analytical workflow. Therefore, options to download data are available in the [R client]({{ site.baseurl }}{% link _articles/tutorial-download-data-portal.md %}#r), [command line client]({{ site.baseurl }}{% link _articles/tutorial-download-data-portal.md %}#command-line) and [Python client]({{ site.baseurl }}{% link _articles/tutorial-download-data-portal.md %}#python).

All entities in Synapse are automatically assigned a globally unique identifier used for reference with the format `syn12345678`. Often abbreviated to “synID”, the ID of an object never changes, even if the name does. You will use a synID to locate the files you wish to download.

## Find Files using Explore

Search the available files via Explore Data or Explore Files in the navigation bar. The Explore section presents several ways to select data files of interest. The top of the page displays pie charts that summarize the number of files based on file annotations of interest, including *Assay* and *Tissue*, among others. Selection of one of these chart segments will filter the table below to subset the set of files. Alternatively, access the filters using the facet selection boxes to the left of the table. For this example, you will [download the *processed* data and *metadata* from the *MC-CAA* study in the Alzheimer's Disease (AD) Knowledge Portal](https://adknowledgeportal.synapse.org/Explore/Data?QueryWrapper0=%7B%22sql%22%3A%22SELECT%20*%20FROM%20syn11346063%22%2C%22limit%22%3A25%2C%22offset%22%3A0%2C%22selectedFacets%22%3A%5B%7B%22concreteType%22%3A%22org.sagebionetworks.repo.model.table.FacetColumnValuesRequest%22%2C%22columnName%22%3A%22study%22%2C%22facetValues%22%3A%5B%22MC-CAA%22%5D%7D%2C%7B%22concreteType%22%3A%22org.sagebionetworks.repo.model.table.FacetColumnValuesRequest%22%2C%22columnName%22%3A%22dataSubtype%22%2C%22facetValues%22%3A%5B%22processed%22%2C%22metadata%22%5D%7D%5D%7D).

## Download Files

### Command Line

The Synapse command line client can be used to download all data and file annotations with a single command.

The command line client is installed with the Synapse Python client, therefore [Python 3](https://www.python.org/downloads/) is required to [install the Synapse command line client](http://python-docs.synapse.org/build/html/index.html#installation). [Login](https://python-docs.synapse.org/build/html/CommandLineClient.html#login) to Synapse. In almost all cases, your Synapse [API key]({{ site.baseurl }}{% link _articles/user_profiles.md %}#api-key) is more secure than your password and is recommended to be used to login.

```
synapse login
```

From Explore Data in the portal, select the **Download Options** icon and **Programmatic Options** to visualize the command to download the data subset.

<img style="width: 25%;" src="/assets/images/programmatic-options-viz.png" alt="alt text">

The command [`synapse get`](https://python-docs.synapse.org/build/html/CommandLineClient.html#get) with the `-q` argument downloads files from the entirety of the portal data that meet the specified condition. In this example, all *processed* and *metadata* files from the *MC-CAA* study will be downloaded. Execute the following command from the directory where you would like to store the files.

```
synapse get -q "SELECT * FROM syn11346063 WHERE ( ( "study" = 'MC-CAA' ) AND ( "dataSubtype" = 'processed' OR "dataSubtype" = 'metadata' ) )" 
```

Also in your working directory, you will find a *SYNAPSE_TABLE_QUERY_###.csv* file that lists the annotations associated with each downloaded file. Here, you will find helpful experimental details relevant to how the data was processed. Additionally, you will find important details about the file itself including the file version number.

### R

In order to download data programmatically with R, you need a list of synIDs that correspond to the files. For downloading a large set of files, we recommend using the Synapse Python client. The Python client has been optimized for multi-threaded download and will provide you with faster download speeds.

Once you have identified the files you want to download from Explore Data, **Export Table** from Download Options. The table includes annotations associated with each downloaded file.

<img style="width: 25%;" src="/assets/images/export-table-viz.png" alt="alt text">

You may choose to download the file as a *.csv* or *.tsv*. Files are named *Job-####* (where # is a long set of numbers). Move this file to your working directory to proceed with the following steps. 

[Install the Synapse R client](https://r-docs.synapse.org/#installation) `synapser` to download data from Synapse. [Login](https://r-docs.synapse.org/articles/manageSynapseCredentials.html#manage-synapse-credentials) to Synapse with your [API key]({{ site.baseurl }}{% link _articles/user_profiles.md %}#api-key).

```
library(synapser)
synLogin("my_username", "api_key")
```
Read the exported table into R replacing Job-#### with the complete filename of the downloaded table. Create a directory to store files and download data using `synGet`. If `downloadLocation` is not specified, the files are downloaded to a hidden directory called `~/.synapseCache`. 
```
exported_table <- read.csv("Job-####.csv")
dir.create("files")
lapply(exported_table$id, synGet, downloadLocation = "./files")
```

The annotations in `exported_table` include experimental details relevant to how the data was processed.

### Python

In order to download data programatically, you need a list of synIDs that correspond to the files.

Once you have identified the files you want to download from Explore Data, **Export Table** from Download Options. The table includes annotations associated with each downloaded file.

<img style="width: 25%;" src="/assets/images/export-table-viz.png" alt="alt text">

You may choose to download the file as a *.csv* or *.tsv*. Files are named *Job-####*, where # includes a long set of numbers. Move this file to your working directory to proceed with the following steps. 

[Install the Synapse Python client](http://python-docs.synapse.org/build/html/index.html#installation) `synapseclient` to download data from Synapse, the `pandas` library to read a csv file and the `os` module to make a directory. [Login](http://python-docs.synapse.org/build/html/index.html#connecting-to-synapse) to Synapse with your [API key]({{ site.baseurl }}{% link _articles/user_profiles.md %}#api-key).
```
import synapseclient, pandas, os
syn = synapseclient.Synapse()

syn.login('my_username', 'api_key')
```
Read the exported table into R replacing Job-#### with the complete filename of the downloaded table. Create a directory to store files and download data using `syn.get`. If `downloadLocation` is not specified, the files are downloaded to a hidden directory called `~/.synapseCache`.

```
exported_table = pandas.read_csv("Job-####.csv")
os.mkdir("files")
[syn.get(x, downloadLocation = "./files") for x in exported_table.id]
```

The annotations in `exported_table` include experimental details relevant to how the data was processed.
