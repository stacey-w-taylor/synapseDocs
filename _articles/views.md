---
title: "Views"
layout: article
excerpt: Use project, file and submission views to query across multiple projects, folders and evaluation queues.
category: managing-data
---

A View is a type of Synapse [Table]({{ site.baseurl }}{% link _articles/tables.md %}) that queries across metadata ([Annotations]({{ site.baseurl }}{% link _articles/annotation_and_query.md %})) for particular items (projects, files or submissions) with a particular "scope". A `File View` lists all `Files` or `Tables` within one or more `Folders` or `Projects`. A `Project View` lists all `Projects` you've added to the view. A `Submission View` lists all `Submissions` within one or multiple `Evaluation Queues`. Views can:

* Allow `Projects`, `Files`, `Submissions`, and `Tables` to be easily searched and queried
* Allow view/editing metadata attributes in bulk
* Provide a way of isolating or linking data based on similarities
* Provide the ability to link `Projects`, `Files`, `Submissions`, and `Tables` together by their annotations

## Create a File View

To create a File View, select the Project in which you would like to create the View. The Project you choose does not have to contain the files you are including in your view. In order to create the File View, navigate to the **Tables** tab and select "Add File View" in the **Tables Tools** menu. You will select the files of interest by defining the scope, which is the Project(s) and Folders that contain your files. File Views can also contain Tables or Folders; you can choose which kinds of items you would like to include during this process.

{% include note.html content= "The scope of a File View can have a maximum of 20,000 folders or sub-folders." %}

Instructions for creating Views using the clients can be found in the [Python docs](https://python-docs.synapse.org/build/html/Views.html) and in the [R docs](https://r-docs.synapse.org/articles/views.html).

## Create a Project View

To create a Project View, select the Project in which you would like to create the view. You will select the projects of interest by defining the scope as above. The only notable difference between creating a Project View and a File View is that for project views, there is a 1:1 relationship between the projects you select in your scope and the projects that are shown in the view.

## Create a Submission View

To create a Submission View, select the Project in which you would like to create the view. The Project you choose does not have to contain the submissions or the evaluation queues that are included in the view. Navigate to the **Tables** tab and select "Add Submission View" in the **Tables Tools** menu. The submissions that are included in the view are defined by its scope, which is the list of Evaluation Queues containing the submissions. For more information, read about how to use Submission Views with [Evaluation Queues]({{ site.baseurl }}{% link _articles/evaluation_queues.md %}#creating-the-submission-view).

{% include note.html content= "The scope of a Submission View can have a maximum of 20,000 evaluation queues." %}

Instructions for creating Views using the clients can be found in the [Python docs](https://python-docs.synapse.org/build/html/Views.html) and in the [R docs](https://r-docs.synapse.org/articles/views.html).

## Updating the Scope or Content-Type of a View

Views can be edited to change the scope of the view (e.g. the Project or Folder the view is showing) or which types of content is shown. Both of these options are found by navigating to "Show Scope of View" in the **Tools menu**; from there you may View and Edit the scope and the content-type of the view.

Note that it may take a few moments for the updated View to rebuild as it queries across the system.

## Query a View

A view can be queried exactly the same as any other Table in Synapse. Please see [Tables]({{ site.baseurl }}{% link _articles/tables.md %}) for more examples. See the [Using Simple Search]({{ site.baseurl }}{% link _articles/views.md %}#using-simple-search) and [Using Advanced Search]({{ site.baseurl }}{% link _articles/views.md %}#using-advanced-search) sections below.

For example, to query for everything in syn123:

##### Command Line

```console
synapse query 'SELECT * FROM syn123'
```

##### Python

```python
query = syn.tableQuery('SELECT * FROM syn123')
```

##### R

```r
query <- synTableQuery('SELECT * FROM syn123')
```

{% include note.html content= "Currently, a view is updated only after a query is run against it.  If your query results appear to be stale, you will need to run your query again to see the expected updates." %}

## Update Annotations in Bulk

Views can be used to update annotations in bulk. To add new annotations, see the [Annotations]({{ site.baseurl }}{% link _articles/annotation_and_query.md %}#adding-annotations) article. To update other metadata in bulk, such as provenance, see the [Bulk Processing]({{ site.baseurl }}{% link _articles/uploading_in_bulk.md %}) article.

For example, if you would like to use the Python client to update the annotation "dogSays:bark" to "dogSays:woof" in every file in a File View with the synId syn456, you can use:

```python
from synapseclient import Table

foo = syn.tableQuery('select * from syn456')

bar = foo.asDataFrame()

# add in annotation as a column
bar['dogSays'] = 'woof'

# store the fileview with the new annotation in Synapse
fv = syn.store(synapseclient.Table(foo.tableId, bar))
```

### Using Simple Search

Views are in Simple Search mode by default. You can filter out Projects or Files of interest by selecting what characteristics you like using the facet menu on the left. You can toggle between simple and advanced search using the **Show advanced search/Show simple search** link.

<img id="image" src="../assets/images/fileViewFacetedSearch.png">

### Using Advanced Search

In advanced search, you can use a SQL-like query to search for items in that view. In the example below, we're selecting for all files that have a Cell Type of "PSC".

<img id="image" src="../assets/images/fileViewAdvancedSearch.png">

## Version a View
Versioning is an essential component of conducting reproducible research. Views can be versioned to create a snapshot of your data at a specific point in time. This snapshot can then be referenced and shared.  For example, if you are publishing a collection of sequencing files in a File View, you may want to share your work with others for analysis. You can use the File View to check that your annotations are correct, then create your first version when it's ready to share. Later, when you've added more files and annotations, you can create a second version to share for additional analysis. You can refer back to previous versions to see what Files and Annotations were used at each step of your research.

As you refine a View, changes are saved to the synID (e.g. syn456). Unlike Files, which are automatically versioned anytime the file is changed, you decide when to manually create a View version. Creating a new version adds a number to the parent synID (e.g. syn456.2), which can be used to reference or query that version within Synapse. Navigating to the parent synID (syn456) will always display the most recent changes to the View, regardless of whether you have included them in a version. 

###Create a View Version
From a View, click **Entity View Tools**, then select **Create a New View Version** from the dropdown menu.

Add a label and a comment about this version. Any text entered in the label field is appended with the word "Version" in the version history. For example, entering "Sage" as a label appears as "Version Sage" in your version history.

###View or Share Versions
Navigate to **Entity View Tools** and select **Version History** to see a table with version history information.

Click on your desired version number. After the page updates, your selected version is indicated in bold text in the Version History table, and the View is displayed below.

To return to the current version, click **Go to the current version** in the top left of the version history.

To share a version, copy the url at the top of the page to share with other Synapse users, or you can [mint a DOI]({{ site.baseurl }}{% link _articles/doi.md %}) for the selected version. 

###Delete a Version
To remove a version, navigate to **Entity View Tools** and select **Version History**. Click on the "x" on the right hand side of the table to delete a specific version.

Once deleted, a version number cannot be reused. Any url or Wiki links pointing to a deleted version will break. 

## Insert a View into a Wiki

Views can also be placed inside a [Wiki]({{ site.baseurl }}{% link _articles/wikis.md %}) once they have been created. You can embed the entire view or a subset of a query on it.

To insert a file view with a synId of syn456:

In the **Edit Project Wiki** window, select **Table: Query on a Synapse Table/View** under the **Insert** dropdown. To embed the entire file view into the wiki enter "SELECT * FROM syn456" in the resulting pop-up.

To embed a subset of the file view, like the advanced search query in the previous example, enter "SELECT * FROM syn456 WHERE Cell_Type = 'PSC'".

<img id="image" src="../assets/images/subsetFileViewWiki.png">

Save the query and the edits to the Wiki to embed the view.

# See Also

[Annotations and Queries]({{ site.baseurl }}{% link _articles/annotation_and_query.md %}), [Tables]({{ site.baseurl }}{% link _articles/tables.md %}), [Wikis]({{ site.baseurl }}{% link _articles/wikis.md %})
