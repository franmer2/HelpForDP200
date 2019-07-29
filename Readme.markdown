# Why Darth Vader still finishes in the top 50 most consulted Wikipedia articles?



## Introduction

In this article, we'll see how to deploy a complete data analytics solution in Azure. We will initially build the solution with several Azure services. Then we will set up a transformation pipeline, with parametrized processes, to automate data movements and transformations. Finally, we will see how to deploy our solution in another environment. (French version available [here](https://franmerms.wordpress.com/2019/04/18/pourquoi-darth-vader-termine-toujours-dans-le-top-50-des-articles-wikipedia-les-plus-consultes/))

I will go through the topic of DataOps quickly, and present how the Azure services of our data platform can help implement some of the principles of DataOps.

One of the goals of DataOps is to have a quality value production pipeline, while still being able to inject modifications and continuous innovation without interrupting this pipeline.
In each of the environments (development, tests, ...), you have to be able to test your ideas to implement innovation, create several test branches, and merge them to get them to production as quickly as possible. The illustration below proposes, for a given environment, a sequence of the different stages of production of value while allowing the infusion of ideas and innovations without interrupting this sequence.

![sparkles](pictures/image001.png)

These sequences must be able to be performed in a development environment, but also in other environments such as testing or production. It is therefore necessary to be able to move as easily as possible from one environment to another.

As a result, 7 major steps towards DataOps have been observed:

- Perform tests
- Use a version control system
- Make branches and merge
- Use multiple environments
- Reuse and containerize
- Set up treatments
- Have simple storage

To illustrate some of the principles of DataOps, I decided to recycle an old demo. Because it is a demonstration that I use very regularly with always the same impact, and also because with the age I became lazy ... and that I wanted to automate as much as possible this demonstration ... and, by the way, I significantly reduced the risk of error due to this automation (which for the moment, is a principle of DevOps).

This is my [Wikipedia logs analysis](https://franmerms.wordpress.com/2014/12/04/big-data-analyses-des-logs-wikipdia-avec-hdinsight/) demonstration


To build our solution, we will use several services of our data platform

![sparkles](pictures/image002.png)

- Azure Data Lake Gen 2, for storage
- Azure Data Factory, for orchestrating the production pipeline
- Azure Databricks, for the injection of innovations, experimentation and transformations
- Azure SQL Database for the data presentation layer
- Power BI Desktop, for data mining and reporting  

Of course we will add some bricks of our infrastructure as:
- Azure Active Directory
- Azure Key Vault

Before we begin, let's do a quick focus on Azure Data Factory and Azure Databriks

### Azure Data Factory v2
Azure Data Factory v2 is a fully managed cloud service that enables the orchestration of data movement and transformation processes. Completely integrated with Azure, It allows during the data movements, to invoke other services, such as Azure Databricks or Logic App, to transform and enrich your data. Azure Data Factory v2 also integrates with Azure Devops for integration and continuous development. In addition, with the ability to transform pipelines into ARM models, it is very easy to deploy them in other environments.

![sparkles](pictures/image003.png)

### Azure Databricks
Based on Spark, Azure Databricks is a workspace that allows collaboration between the company's stakeholders to bring innovation through continuous experimentation. Being integrated with Azure DevOps, Azure Databricks enters completely into a CI / CD process as illustrated in [Benjamin Leroux's post](https://thedataguy.blog/ci-cd-with-databricks-and-azure-devops/). Azure Databricks allows the collaboration around Notebooks, which support several languages such as Python, Scala, SQL, ..., and allows the execution of these notebooks both manual and automatic, via "jobs", orchestrated by Azure Data Factory

![sparkles](pictures/image004.png)

The notebook used for this example will do the following:
- Read files deposited by Azure Data Factory into the data lake (Azure Data Lake Storage Gen2)
- Process files to extract information
- Create an archive file in the data lake
- Insert the results into a SQL Database table for the re-porting part

The notebook used for this example is available [here](https://1drv.ms/u/s!Am-C-ktMH9lgg9MCqqG5dS8XKFKxDA) or [here](https://github.com/franmer2/demowikipedia)

### Few words about the data lake
In modern data platform architecture solutions, it is important to separate storage space from data processing engines. This is exactly what we will do in this example. For this article, we will therefore develop our data lake in several zones, in order to preserve the state of the data according to the transformations that they will have undergone. Storage space is no longer a problem today, we will create multiple areas in our data lake as shown in the illustration below

![sparkles](pictures/image005.png)

Even if finally we will use only 2 persistent zones in our data lake for this example (and 1 temporary), the idea is to have several zones in order to preserve all the states of our data, of the raw state in the most refined one.

### Demonstration architecture
At the end of this article we will have realized the following architecture. You will notice the separation of the storage of the calculation engine

![sparkles](pictures/image006.png)

### Prerequisites
- [An Azure subscription](https://azure.microsoft.com/en-us/free/search/?&OCID=AID719803_SEM_sHsHEIyy&lnkd=Bing_Azure_Brand&dclid=CKu35_-GyOACFYRNDAodhEkA3A)
- [Azure Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/)
- [Power BI Desktop](https://www.microsoft.com/en-us/download/details.aspx?id=45331)


## Creating the solution
### Creating the resource group

The resource group is a logical grouping of Azure services. For this article, we will group all the services used to build our solution in one resource group. But nothing prevents you from organizing your Azure services otherwise.

From the [Azure portal](https://portal.azure.com/), on the left, click on "**Resource groups**" then on the "**Add**" button.

![sparkles](pictures/image007.jpg)

Complete the creation form and click on "**Review + Create**". For this article, we will name the resource group "**Paisley-Park**".

![sparkles](pictures/image008.jpg)

## Creation of the data lake
In the previously created resource group, we will add a storage service. For this example we will choose the new service "**Data Lake Store Gen 2**".

From the Azure portal, click on "**Create a resource**", "**Storage**" and then "**Storage account**"

![sparkles](pictures/image009.jpg)

Then fill out the creation form. Here, I decide to create my data lake in the "P**aisley-Park**" resource group.For this article, I will name my data lake "thevaultgen2"

![sparkles](pictures/image010.png)

Then click on the "**Advanced**" tab, then in the "**Data Lake Storage gen2**" section, click on "**Enabled**". This will enable new features related to the new version of our data lake.

![sparkles](pictures/image011.jpg)

If necessary you can add tags by clicking on the tab "Tags". Tags are useful for administering and tracking the costs of your Azure services.

When done, click on the "**Review + create**" button

![sparkles](pictures/image012.jpg)

Then on the "**Create**" button

![sparkles](pictures/image013.jpg)

## Creating the SQL Database Database

From the Azure portal, click on "**Create a resource**", "**Database**", then "**SQL Database**"

![sparkles](pictures/image014.jpg)

Fill out the database creation form and click on "**Review + Create**". For this article, I will call the database "**Musicology**", and I will keep the level of Tier to basic. Nothing will prevent us subsequently to change this level of performance according to the needs (principle of elasticity).

![sparkles](pictures/image015.jpg)

On the "**Create + Review**" page, click on the "Create" button

![sparkles](pictures/image016.jpg)

Once the database is created, consider setting up the "firewall" in case you want to use SQL Server Management Studio from a remote desktop.

![sparkles](pictures/image017.jpg)

## Azure Data Factory Service Creation
From the portal, click on "**Create a resource**", "**Integration**" and then "**Data Factory**"

![sparkles](pictures/image018.jpg)

Fill out the creation form and click on the "**Create**" button

![sparkles](pictures/image019.jpg)


## Creation of the Azure Databricks service
From the Azure portal, click "**Create a resource**", "**Analytics**" and "**Azure Databricks**"

![sparkles](pictures/image020.jpg)

Fill out the creation form. We will take the premium tier for integration with Azure Key Vault.This is especially recommended in the [Databricks documentation](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html):

"*Your account must have the Azure Databricks Premium Plan for you to be able to select Creator. This is the recommended approach: grant MANAGE permission to the Creator when you create the secret scope, and then assign more granular access permissions after you have tested the scope. For an example workflow, see Secret Workflow Example*."

For this article, I will name my Azure Databricks "**CrystalBall**" workspace.Click on the "**Create**" button


![sparkles](pictures/image021.jpg)

## Creating an Azure Key Vault

From the Azure portal, click on "**Create a resource**", then in the search bar enter "**Key Vault"**

![sparkles](pictures/image022.jpg)

Choose "**Key Vault**" from the Marketplace and click on the "**Create**" button.

![sparkles](pictures/image023.jpg)

Fill out the creation form and click on the "**Create**" button

![sparkles](pictures/image024.jpg)


If you have placed all your resources in the same resource group, you must obtain a result like the one in the screenshot below:

![sparkles](pictures/image025.png)


##Creation of an application and a service principal

In this section of the article, we will create the security elements that allow Azure Databricks to access our Data Lake gen2. The general procedure can be found [here](https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-create-service-principal-portal).

In our Azure Active Directory, we will create and register an application.

From the Azure portal, click on "**Azure Active Directory**" and then on "**App registration (preview)**".

![sparkles](pictures/image026.jpg)

Click on the "**New registration**" button

![sparkles](pictures/image027.jpg)

Fill out the registration form and click on the "**register**" button

![sparkles](pictures/image028.jpg)

The application is now created. You must have a screen identical to the one below.

**WARNING!!! ** Note the information of your "Application (client) ID" as well as your "Directory (tenant) ID" (which I have hidden in my screenshot). You will need it later.

![sparkles](pictures/image029.jpg)

Click on "**Certificates & Secrets**" and then on "**New Secret Client**". This "secret ID" is also called "authentication-id" or "secret client"

![sparkles](pictures/image030.jpg)

Fill out the creation form and click on the "**Add**" button

![sparkles](pictures/image031.jpg)

Copy the previously generated secret by clicking on the copy icon

![sparkles](pictures/image032.jpg)

Return to your resource group and select your storage account

![sparkles](pictures/image033.jpg)

Then click on "**Access Control (IAM)**"

![sparkles](pictures/image034.jpg)

In the "**Add a role assignment**" box, click on the "**Add**" button

![sparkles](pictures/image035.jpg)

In the "**Add role assignment**" window, select the following items:
- **Role**: Storage Blob Data Contributor
- **Select**: The name of the application created previously. Here, "**FranmerTheVaultGen2**".
  
Then click on the "**Save**" button

![sparkles](pictures/image036.jpg)

## Setting the Azure Key Vault
In order for Azure Databricks to have access to Data Lake Gen2, we will need the following information:
- storage account name
- Application-id
- authentication-id
- tenant-id 

It is possible that some of this information must not appear in clear text in the Azure Databricks's notebooks. **This is where Azure Key Vault comes into action to protect passwords or logins.**
For this example we will store only the different IDs.

Click on your Azure Key Vault:
![sparkles](pictures/image037.jpg)


Click on "**Secrets**" then on the "**Generate / Import**" button

![sparkles](pictures/image038.jpg)

We will create the secrets for our 3 IDs. Below an example for "**application-id**". 

Click on the "**Create**" button.
Repeat the following steps for the other 2 IDs.

![sparkles](pictures/image039.jpg)

After creating your 3 secrets, you must have a result like the screenshot below:

![sparkles](pictures/image040.png)

A little further down in this article, we'll add a secret for the integration between Azure Data Factory and Azure Databricks

## Configuring Azure Databricks with Azure Key Vault

### Integration with Azure Key Vault

We will now configure Azure Databricks so that it can use the secrets set in Azure Key Vault. For information, the complete documentation is [here](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html).

Return to your resource group and click on your Azure Databricks service:

![sparkles](pictures/image041.jpg)


Click on the button "**Launch Workspace**"

![sparkles](pictures/image042.jpg)

Once in your workspace, note the URL of your Azure Databricks:

![sparkles](pictures/image043.jpg)

Then compose a new URL by adding "**#secrets/createScope**" (**Attention** to the capital "S"!)
In my case the URL will be: ("https://eastus2.azuredatabricks.net#secrets/createScope")

The next window will open, this is where we will create references to the secrets present in our Azure Key Vault

![sparkles](pictures/image044.png)

To do this, we will need the following information from our Azure Key Vault:
- DNS Name
- Resource ID

This information can be found in the properties of Azure Key Vault (see screenshot below)

![sparkles](pictures/image045.jpg)

Once the information is found, the creation of a "**Secret Scope**" is as shown below.
After entering all the required information, click on the "**Create**" button

![sparkles](pictures/image046.jpg)

If all goes well, you must have the following message. Click on the "**OK**" button to validate.

![sparkles](pictures/image047.jpg)

## Import Databricks's notebook

For the continuation of the article, we will download the notebook which is at the following address: https://1drv.ms/u/s!Am-C-ktMH9lgg9MCqqG5dS8XKFKxDA or https://github.com/franmer2/demowikipedia/tree/master/resources


**You will then have to modify the notebook to add your information concerning the application ID, the "data lake" as well as your SQL Database.**

From your Azure Databricks workspace, navigate to where you want to import the notebook. Click on the down arrow to bring up the context menu and click on "**Import**".

![sparkles](pictures/image048.jpg)

Check the **"File"** box, enter the path to the notebook and click on the **"Import"** button

![sparkles](pictures/image049.jpg)

If all goes well, the notebook should be in your workspace as shown below

![sparkles](pictures/image050.jpg)

## Creating access tokens
We will create the access tokens for Power BI and Azure Data FactoryAt the top right of your workspace, click on the character icon and then on "**User Settings**"

![sparkles](pictures/image051.jpg)

Click on the "**Generate New Token**" button

![sparkles](pictures/image052.jpg)

Give a description to the Token, then indicate its lifetime. Click on the "**Generate**" button.

![sparkles](pictures/image053.jpg)

**WARNING** !! Copy your Token in a reliable place, because after pressing the "**Done**" button, it will no longer be accessible.

![sparkles](pictures/image054.jpg)

**Repeat** to create a Token for Power BI. You must have a result similar to the screenshot below:

![sparkles](pictures/image055.png)

**Note**: The token for Azure Databricks needs to be added in Azure Key Vault. We will add the secret "**Databricks**"

=============================================================

Below is a screenshot after adding the "Databricks" secret in Azure Key Vault:

![sparkles](pictures/image056.jpg)

=============================================================

## Landscaping of our data lake

We will now develop our data lake to create an area for raw data from Wikipedia, and an area with the result of the analyzes

![sparkles](pictures/image057.png)

The zone "demo_datasets" will be created manually. The "wikipedia_results" zone will be created automatically by Azure Databricks. In order to interact with Azure Data Lake Gen2, it is necessary to use [Azure Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/). From Azure Storage Explorer, sign in to your Azure account.Find your lake of data, do a right click on it, then create a container "**wikipedia**"

![sparkles](pictures/image058.jpg)

You must obtain a result similar to the screenshot below:

![sparkles](pictures/image059.png)

**OPTIONAL** (Azure Data Factory can create the folder automatically if it is not already present). In your "wikipedia" container, create the directory "demo_datasets".

Click on the "**New Folder"** button

![sparkles](pictures/image060.jpg)


You should obtain the result below:

![sparkles](pictures/image061.png)

## Creating the Azure Data Factory pipeline

The pipeline is quite simple for this example and will consist of the sequence of the following activities, while being based on input parameters (which is, by the way, one of the elements of the DataOps):

- Test the presence of the "demo_datasets" folder in the storage account
- Deleting files from the raw area (demo_datasets) of the data lake
- Download log files (depending on user settings)
- Invocation of an Azure Databricks notebook for data transformation and writing of results to another area of the data lake and in SQL Database.

In the end, we will get a pipeline like the one shown below:

![sparkles](pictures/image062.png)


From the Azure portal, connect to your **Azure Data Factory** service

![sparkles](pictures/image063.jpg)

Click on **"Author & Monitor"**

(If you use MSDN subscription and facing infinite white screen, try to open your browser in private mode)

![sparkles](pictures/image064.jpg)

## Creating linked services and datasets

### Creating linked services

We will create 4 **Linked Services** for:
- Azure Data Lake sotrage Gen2
- The Wikipwdia log site
- Azure Key Vault
- Azure Databricks

In this section of the article, we will create 2 **Linked Services**. The other 2 will be created during dataset creation, to illustrate another way to create them. 

### Azure Key Vault Linked Service

We will start by creating a Linked Service to the "Azure Key Vault" service.

From the authoring interface, click on the pencil on the left, then on "**Connections**"

![sparkles](pictures/image065.jpg)

Click on the "**New**" button

![sparkles](pictures/image066.jpg)

Search and select "**Azure Key Vault**". Click on the "**Continue**" button

![sparkles](pictures/image067.jpg)

Enter information about your Azure Key Vault service. Click on "**Test connection**" then on "**Finish**"

![sparkles](pictures/image068.jpg)

=============================================================

**WARNING** !!! You noticed the "Edit Key vault" message just below the "Azure key vault name" field. It is necessary to add the "Service identity application" in the "Access policies" section of the Azure Key Vault service.

![sparkles](pictures/image069.jpg)

In your Azure Key Vault service, click "**Access policies**" and then the "**Add new**" button to search for the service (with the Service identity application number) and give it access rights.The "**GET**" right for the "**Secret permissions**" field is sufficient

![sparkles](pictures/image070_1.jpg)


=============================================================

After creating your linked service, you should get a result similar to the one shown below:

![sparkles](pictures/image072.png)

Click on the "**Publish all**" button

![sparkles](pictures/image073.jpg)

## Azure Databricks Linked Service

We will repeat the operation to create the Linked Service to Azure Databricks

![sparkles](pictures/image074.jpg)

Fill in the login information for Azure Databricks. Use the secret we created earlier in Azure Key Vault

![sparkles](pictures/image075.jpg)

To illustrate another possibility of creating linked services, we will create the other 2 linked services at the same time as the datasets in the next paragraph.

## Creating datasets


We will now create 3 "datasets":

- 1 to retrieve Wikipedia logs (http), also with parameters
- 1 for the data lake
- 1 for the same data lake but with parameters

### Creating the dataset for the Wikipedia logs site

#### Beginning of the creation of the Wikipedia dataset

Click on the "**+**" sign, then on "**Dataset**"

![sparkles](pictures/image076.jpg)

In the "**New Dataset**" section, look for "**http**", then click on the "**http**" icon and then on the "**Finish**" button

![sparkles](pictures/image077.jpg)

In the "**General**" tab, define a name for your dataset.

![sparkles](pictures/image078.jpg)

Then click on "**Parameters**".Click on the "**New**" button each time to add a parameter

![sparkles](pictures/image079.jpg)

Enter the following parameters in "String" format:

- YearDS
- MonthDS
- DayDS
- HourDS

![sparkles](pictures/image080.png)

Click on the "**Connection**" tab

![sparkles](pictures/image081.jpg)

## Creating the Wikipedia Linked Service 

Here we will see how to create a Linked Service from the creation phase of the dataset. In the "**Connection**" section click on "**New**" to create a linked service to the Wikipedia logs site

![sparkles](pictures/image082.jpg)

Give a name to your linked service. In the field, **base URL**, enter the following URL: https://dumps.wikimedia.org/other/pageviews/.

Click on "**Disable**" and select "**Anonymous**" in the "**Authentication type**" fieldTest your connection and click on the "**Finish**" button

![sparkles](pictures/image083.jpg)

Once the linked service is created, you return to creating your dataset.

### Continuig Wikipedia dataset creation

Click in the field "**Relative URL**" then on "**Add dynamic content**"

![sparkles](pictures/image084.jpg)

Copy the string below into the "**Add Dynamic Content**" field and click on "**Finish**":

    @concat(dataset().YearDS,'/',dataset().YearDS,'-',dataset().MonthDS,'/pageviews-',dataset().YearDS,dataset().MonthDS,dataset().DayDS,if(less(int(dataset().HourDS),10),'-0','-'),dataset().HourDS,'0000.gz')

![sparkles](pictures/image085.jpg)

Then check the "**Binary copy**" box

![sparkles](pictures/image086.jpg)

Click on the "**Publish All**" button

![sparkles](pictures/image087.jpg)

## Creating a Data Lake dataset

### Dataset creation for the data lake

Now, let's create a second Dataset for our data lake. Repeat the creation procedure as previously seen, then select "**Azure Data Lake Storage Gen2**"

![sparkles](pictures/image088.jpg)

Give a name to this new dataset.

### Creating a Data Lake linked service

Click on "**Connection**" and then on the "**New**" button to create a linked service to our data lake.

![sparkles](pictures/image089.jpg)

In the "New Linked Service" window, select your storage account from your Azure subscription:

- The "**Tenant**" field should automatically be filled.
- The "**Service Principal ID**" is the one we created at the beginning of the article
- Select "**Azure Key Vault**"
- Select your "**Azure Key Vault**" service
- Indicate the name of the secret that protects the key of your main service (in my example: "**authentication-id**")

Click on the "**Finish**" button

![sparkles](pictures/image090.jpg)

### Continuing Data Lake dataset creation

Now that the linked service to our data lake has been created, we will continue configuring our dataset. In the "**File path**" field, we will enter the path "**Wikipedia / demo_datasets**".

Check the "**Binary copy**" box

![sparkles](pictures/image091.jpg)

## Creating a parametrized Data Lake dataset 

We will repeat the following procedure to create a dataset on the same data lake, but by adding parameters. We will use the same linked service. On the second dataset of the data lake add the following parameters:

- YearDS
- MonthDS
- DayDS
- HourDS
  


![sparkles](pictures/image092.jpg)

Then in the "**Connection**" tab, define the path as shown below by adding the following expression in the "**File**" field:

@concat('pageviews-',dataset().YearDS,dataset().MonthDS,dataset().DayDS,if(less(int(dataset().HourDS),10),'-0','-'),dataset().HourDS,'0000.gz')

![sparkles](pictures/image093.jpg)

At the top of the screen, click on the "**Publish all**" button.

![sparkles](pictures/image094.png)

## Pipeline creation

Click on the "**+**" sign and then on "**Pipeline**"

![sparkles](pictures/image095.jpg)

### Pipeline settings

In order to make our pipeline more agile, we will add parameters, which will allow us, for example, to recover the logs according to the date and times that interest us.

Click in a free space in the Pipeline Editor, then click on "**Parameters**" then on "**New**" (click 4 times on "New" to add 4 parameters)

![sparkles](pictures/image096.jpg)

Add the following parameters:

| Nom   |      Type |
| :---- | --------: |
| Year  |    String |
| Month |    String |
| Day   |    String |
| Hours | **Array** |

In the example, I added some default values for the example (especially in the Array parameter). But these default values are not required.

![sparkles](pictures/image097.jpg)

### Creation of the "Delete" activity

As this activity does not exist yet in the list of available activities, we will create it by code.

Click on the "**Code**" button to display the code editor

![sparkles](pictures/image098.jpg)

Copy and paste in the editor the script below. Remember to change the name of the dataset. (In my example it is "**AzureDataLakeStorageFile1**", the data lake dataset without parameters).

Click on the "**Finish**" button

```javascript
{
    "name": "pipeline1",
    "properties": {
        "activities": [
            {
                "name": "Delete Activity",
                "type": "Delete",
                "policy": {
                    "timeout": "7.00:00:00",
                    "retry": 0,
                    "retryIntervalInSeconds": 30,
                    "secureOutput": false,
                    "secureInput": false
                },
                "typeProperties": {
                    "dataset": {
                        "referenceName": "<Your Dataset Name>",
                        "type": "DatasetReference"
                    },
                    "recursive": true,
                    "enableLogging": false,
                    "maxConcurrentConnections": 1
                }
            }
        ]
    }
}
```

![sparkles](pictures/image099.jpg)

You should see a new activity in your pipeline:

![sparkles](pictures/image100.png)

## "Get Metadata" activity

The screenshot below show how to create the '**Get Metadata**'activity.

Add the "**Exists**" argument. It will be tested by another activity

![sparkles](pictures/image101.jpg)

## "If Condition" activity

Drag and drop the "**If Condition**" activity from the left menu to the pipeline. Add a link between the "**Get Metadata**" activity with the "**If Condition**" activity.


Click on the "**If Condition**" activity. In "**settings**" tab, clic on "**Add dynamic content**"


![sparkles](pictures/image102.jpg)

Add the following content:

    @activity('Get Metadata1').output.Exists

![sparkles](pictures/image103.jpg)


Right click on the "**Delete**" activity and click on "**Copy**"

![sparkles](pictures/image104.jpg)

Click again on the "**If Condition**" activity and then on the "**Activities**" tab.Then click on the "**Add if True Activity**" button

![sparkles](pictures/image105.jpg)

Right click in the central part to copy the activity. Click on "**Paste**"

![sparkles](pictures/image106.jpg)

The "**Delete**" activity is copied to your "**If condition**" activity.

![sparkles](pictures/image107.png)

Click on the "**Pipeline1**" (or another name if you renamed your pipeline) link to return to the top level of your pipeline and remove the "Delete1" activity.


![sparkles](pictures/image108.jpg)

## "ForEach" activity

We will now create a loop to download the files corresponding to the number of hours that we want to recover. This number will be defined via the parameters.

From the list of activities, on the left, drag and drop the activity "**ForEach**" in the editor. Then link the "**If condition**" activity to the "**ForEach**" activity.

![sparkles](pictures/image109.jpg)

Click on the "**ForEach**" activity and then "**Settings**". Click in the "**Items**" field then click on "**Add dynamic content**"

![sparkles](pictures/image110.jpg)

Add the following content:

    @pipeline().parameters.Hours

Then click on the "**Finish**" button

![sparkles](pictures/image111.png)

The "**Hours**" parameter is now available in the "**Item**" section of the "**ForEach**" activity. This loop will now be repeated according to the number of hours that will be defined in the "**Hours**" parameter.

![sparkles](pictures/image112.jpg)

Double click on the activity "**ForEach**", to add activities inside the loopAdd the "**Copy Data**" activity in the "**ForEach**" loop

![sparkles](pictures/image113.jpg)

Click on "**Source**" and select the Dataset "**Logs_Wikipedia**". Pass the settings as shown in the screenshot below.

- **YearDS**  : @pipeline().parameters.Year
- **MonthDS** : @pipeline().parameters.Month
- **DaysDS**  : @pipeline().parameters.Day
- **HourDS**  : @item()



(Concerning the hours, parameter comes actually from the activity "**ForEach**").

![sparkles](pictures/image114.jpg)

Click on "**Sink**" tab. In the "**Sink Dataset**" field, choose the data lake dataset with the parameters. 

Then fill in the parameters with the values as shown in the screenshot below.

![sparkles](pictures/image115.jpg)


Once the changes are complete, click on the "**Publish All**" button

![sparkles](pictures/image116.jpg)

## "Notebook" activity

Now we're going to add Databricks's Notebook Activity as shown below

![sparkles](pictures/image117.jpg)

Once the activity has been added, link it on the output of the "**For Each**" activity

![sparkles](pictures/image118.jpg)

Click on the "**Notebook**" activity, then on "**Azure Databricks**". Choose the linked service created previously and click on "**Test connection**":

![sparkles](pictures/image119.jpg)

Then click on "**Settings**" and then on the "**Browse**" button

![sparkles](pictures/image120.jpg)

Then click on "**Base Parameters**" to create the parameters that will be passed to your Azure Databricks Notebook.

Respect the following names. If you want to change the names of the basic parameters, then you have to edit the notebook:

|Base parameter name|
|:---------------|
|yearWiki|
|monthWiki|
|dayWiki|

![sparkles](pictures/image121.jpg)

Once the settings are complete, click on "Publish All"

![sparkles](pictures/image122.jpg)

Now we will test our pipeline. Click on the "**Debug**" button:

![sparkles](pictures/image123.jpg)

The parameter page opens. Fill in the fields with consistent values and click on the "**Finish**" button:

![sparkles](pictures/image124.jpg)

Execution can be monitored in the "**Output**" part of the pipeline:

![sparkles](pictures/image125.jpg)

On the side of Azure Databricks, the notebook is run via a job

![sparkles](pictures/image126.jpg)

Hopefully on the Azure Data Factory side, your pipeline should show "**Succeeded**" in the "**Status**" column

![sparkles](pictures/image127.jpg)

Power BI

We will now use Power BI Desktop to connect to our SQL Database. At the time of writing this article (17/03/2019), the Power BI connector for Azure Data Lake Gen2 Store is not yet available. From Power BI Desktop, click the "**Get Data**" button, and select "**SQL Server**"

![sparkles](pictures/image128.jpg)


Fill in the information from your database server, which can be found on the Azure portal as shown below:

![sparkles](pictures/image129.jpg)

Choose whether you want to import the data or do use **DirectQuery** mode. Then click on the "**Ok**" button.

![sparkles](pictures/image130.jpg)

Choose how you want to connect to your database. In this example, I chose a connection with a "**Database**" account.

Click on the "**Connect**" button

![sparkles](pictures/image131.jpg)

Select the table "**WikipediaLogs**", then click on the "**Load**" button

![sparkles](pictures/image132.jpg)

The fields will appear on the right side of Power BI Desktop.

![sparkles](pictures/image133.jpg)

From now on, you have all the tools to create a nice report on the result of the analyzes of the Wikipedia logs. Here is an example of a report:

![sparkles](pictures/image134.png)

![sparkles](pictures/image135.jpg)

## Deploying the solution in another environment

We will now see how to deploy our analytical solution automatically through the use of ARM models.
This preparation and deployment will be done following the steps below:
- Creating an Azure Resource Management (ARM) template
- Deploying Azure resources through the ARM model
- Manual actions to add configurations not yet supported by ARM models
- Creating an ARM Model to Deploy the Azure Data Factory Pipeline in the New Environment
- Deployment of the pipeline via the ARM model

At first I planned to do this exercise with the "Template" service of the Azure portal, but by talking with [David Chapdelaine](https://www.linkedin.com/in/dchapdelaine/), TSP Azure, he also recommended me to watch the Azure CLI track, which I also did 😊 . Therefore, I will illustrate these 2 approaches in the rest of this article. ARM models are available here: https://github.com/franmer2/demowikipedia


## Using the "Template" service of the Azure portal

### Creating an Azure Resource Management (ARM) Template

There are still no miracle solutions for creating a functional ARM model from the resource group. However, the Azure portal provides templates to facilitate full ARM model creation to replay the deployment of our solution. Coupled with Visual Studio Code, we have 2 tools to make our life easier when creating ARM models.

If from your resource group you click on "Deployments" you will be able to see the different deployments made within your resource group. By clicking on each of them, you will be able to be inspired to write your model ARM.

![sparkles](pictures/image136.jpg)

Another source of inspiration may be the "**Export Template**" function also available from your resource group. But as indicated by the warning message, not all resources are yet exportable via this function.

![sparkles](pictures/image137.jpg)

To write the ARM model, [Visual Studio Code](https://code.visualstudio.com/) can be an excellent friend especially if we add modules to help write the ARM models. Then just open an empty JSON file and start writing the ARM template.

![sparkles](pictures/image138.jpg)

ARM template samples for this example are available here: https://github.com/franmer2/demowikipedia

### Deploying Azure Resources Using the ARM Model

Once the ARM model is written, we will deploy it via the Azure portal (but it can be done by other methods, with PowerShell for example).

For this article, I deploy my ARM model in the same tenant and the same "Service Principal" that we created at the beginning of the article will be used, but it is possible to do a deployment in another tenant, but it will create a new "Service Principal".

From the Azure portal, click the "**All services**" button on the left. Then in the search field, enter "**Templates**" then click on "**Templates**"

![sparkles](pictures/image139.jpg)

Click on the "**Add**" button, then "**General**" then give a name and a description to your model. Click on the "**Ok**" button


![sparkles](pictures/image140.jpg)

Click on "**ARM Template**". Delete the content already present in the template and copy / paste from the Visual Studio code into the ARM template edit window. Click on the "**Ok**" button.


![sparkles](pictures/image141.jpg)

Click on the "**Add**" button


![sparkles](pictures/image142.jpg)


We will now use this model to deploy our solution.Click on your model then on the button "**Deploy**"

![sparkles](pictures/image143.jpg)

Fill out the deployment form.

======================================================

As a reminder, the "**objectID**" and other information can be found from the Azure portal:

Click on the "**Azure Active Directory**" icon, then "**App registrations (preview)**".Search for your application:

![sparkles](pictures/image144.jpg)

Information are available in "**Overview**"

![sparkles](pictures/image145.jpg)

======================================================

In the last line of the form, to find your **objectID**, you can use the "**Cloud Shell**", then use the command: 

    az ad signed-in-user show --query "objectId"

Then accept the terms and click on the "**Purchase**" button.

![sparkles](pictures/image146.jpg)


After only a few minutes, you will get a new resource group with all the necessary services for your data analytics solution


![sparkles](pictures/image147.png)

## Using the Cloud Shell and Azure CLI

Another way to deploy ARM models is to use Azure CLI. Below is an example of deployment with the following script

    resourceGroup='<Your Resource Group Name>'
    location='canadacentral'
    az group create -n $resourceGroup -l $location
 
    deploymentName='<Your Deployment Name>'
    az group deployment create -g $resourceGroup -n $deploymentName \
    --template-uri 'https://raw.githubusercontent.com/franmer2/demowikipedia/master/resources/1_Wikipedia_General_ARM_template.json' \
    --parameters location=$location \
    databricksWorkspaceName='<Your Azure Databricks Workspace Name>' \
    databricksTier='premium' \
    storageAccountName='<Your Storage Account Name>' \
    datafactoryName='<Your Azure Datafactory Name>' \
    datafactoryLocation='Canada Central' \
    SQLadministratorLogin='<Your SQL Administrator Name>' \
    SQLadministratorLoginPassword='<Your SQL Administrator Password>' \
    SQLserverName='<Your SQL Server Name>' \
    SQLdatabaseName='<Your SQL Database Name>' \
    vaults_Trust_name='<Your Azure Key Vault Name>' \
    objectId='<Your Application Object ID>' \
    application-id_secretValue='<Your Application (client) ID>' \
    authentication-id_secretValue='<Your Application (client) Secret>' \
    objectOwnerObjectId=$(az ad signed-in-user show --query 'objectId' | xargs)

To illustrate the script above, here is a screenshot with an example of used values:

![sparkles](pictures/image148.jpg)


## Manual actions to add configurations not yet supported by ARM models

However, some actions will have to be done manually, which are not yet supported in an ARM deployment.

1. Databricks: generate the access token and define the scope
2. Databricks: add and modify the notebook
3. Azure Key Vault: add the Azure Data Factory account service in Access Policies
4. Azure Key Vault: add the secret for Azure Databricks with the previously created token
5. Storage account: add the main service to your storage account
6. Storage account: create the "wikipedia" container in the storage account

### Azure Databricks Service Configuration
#### Databricks: Generate the access token and define the scope

At the time of writing this article (April 2019), it is not possible to create an Azure Databricks token automatically.

A vote has been opened on this subject:
https://feedback.azure.com/forums/909463-azure-databricks/suggestions/35257819-expose-api-key-during-arm-deployment

We will manually add an access token to the new Azure Databricks workspace. I'm not going to redo the entire procedure, since I've seen it earlier in this article. Below are just a few screenshots of the main steps:

![sparkles](pictures/image149.jpg)

Then click on "**Generate New Token**". Remember to copy the new token.

![sparkles](pictures/image150.jpg)

We will now create a new "**scope**" following the same steps as in the beginning of the article.

Modify the url of your Azure Databricks workspace by adding:
"#secrets/createScope".

 For the example, here is what my URL looks like for my new workspace:
 https://canadacentral.azuredatabricks.net#secrets/createScope
 
 The following window may appear. Then choose the new workspace:

 ![sparkles](pictures/image151.jpg)

 Create your new scope. Name it "**Trust**" if you do not want to change the notebook.

 ![sparkles](pictures/image152.png)

 Azure Key Vault information can be retrieved from the Azure portal as shown below

 ![sparkles](pictures/image153.jpg)

 ### Databricks: adding the notebook
 
 Download the notebook from this link https://1drv.ms/u/s!Am-C-ktMH9lgg9MCqqG5dS8XKFKxDA or from github: https://github.com/franmer2/demowikipedia
 
 Create a folder in your workspace and import the notebook

![sparkles](pictures/image154.jpg)

Then modify the notebook with the information of the new environment, wherever it is mentioned, as shown below

![sparkles](pictures/image190.png)

![sparkles](pictures/image155.jpg)


### Configuring Azure Key Vault
#### Adding the Data Factory account in Access policies

Now add the main service of Azure Data Factory. Click "**Select Principal**," and then search for the name of your Azure Data Factory service. Select the main service. Click on the "**Ok**" button.

Then set the permissions for the secret. The "**Get**" permission will be sufficient in our case. Click on the "**Ok**" button.

![sparkles](pictures/image156.jpg)

Click on the "**Save**" button

![sparkles](pictures/image157.jpg)

### Adding the Databricks token

We will add the previously created Azure Databricks token in our newly deployed Azure Key Vault. From the Azure portal, in your "**Azure Key Vault**" service, click "**Secret**", "**Generate / Import**", then add the "**Databricks**" secret with the token created in the Azure Databricks workspace

![sparkles](pictures/image158.jpg)

### Adding the Service Principal at the storage account level

In your storage account, add your "Service Principal" account with the role "**Storage Blob Data Contributor**"

![sparkles](pictures/image159.jpg)

For the moment (April 2019), it is not possible to create a container in a Data Lake Gen2 type account with the ARM models. It is therefore necessary to do it manually. It can be done for example via "**Azure Storage Explorer**" or the **Azure portal**. Below is an illustration with the Azure portal.

![sparkles](pictures/image160.jpg)

## Creating an ARM Model to Deploy the Azure Data Factory Pipeline in the New Environment

### Export the ARM model of your pipeline

From your original environment, return to your Azure Data Factory with your transformation pipeline. In the top menu bar, click on "**ARM Template**" and then on "**Export ARM Template**".

![sparkles](pictures/image161.jpg)

Then save the package in an easily accessible place


![sparkles](pictures/image162.png)

### Modify your ARM model

With Visual Studio Code, modify your ARM model to facilitate deployment in a new environment. Notably by creating the url of your storage account and Azure Key Vault from parameters and variables:


    "AzureKeyVault_Trust_properties_typeProperties_baseUrl": "[concat('https://',parameters('AzureKeyVault_Name'),'.vault.azure.net/')]",
            "AzureDataLakeStorageGen2_properties_typeProperties_url": "[concat('https://',parameters('AzureDataLakeStorageGen2_Name') ,'.dfs.core.windows.net')]"


The ARM model that I use is available here: https://github.com/franmer2/demowikipedia/blob/master/resources/2_Wikipedia_ADF_ARM_template.json



## Deploy the ARM pipeline creation model
### From the Azure portal with Template service
 
In the same way as for the first ARM model, save your model on the Azure portal in the "**Template**" section

![sparkles](pictures/image163.jpg)

Then deploy it from the portal in your new environment

![sparkles](pictures/image164.jpg)

Fill out the deployment form. "**Factory Name**" is the name of the Azure Data Factory service that you just deployed through the ARM model. Accept the terms and click on the "**Purchase**" button:

![sparkles](pictures/image165.png)

### With Bash and AzureCLI

Use the script below in the Azure Cloud Shell



    resourceGroup='<YOUR RESOURCE GROUP>'
    location='<YOUR LOCATION>'
    ADFdeploymentName='<YOUR DEPLOYMENT NAME>'

    az group deployment create -g $resourceGroup -n $ADFdeploymentName \
    --template-uri 'https://raw.githubusercontent.com/franmer2/demowikipedia/master/resources/2_Wikipedia_ADF_ARM_template.json' \
    --parameters 'https://raw.githubusercontent.com/franmer2/demowikipedia/master/resources/2_Wikipedia_ADF_ARM_template.parameters.json' \
    --parameters AzureDataLakeStorageGen2_properties_typeProperties_servicePrincipalId='<Your Service Principal ID>' \
    factoryName='<YOUR AZURE DATA FACTORY NAME>' \
    AzureKeyVault_Name='<YOUR AZURE KEY VAULT NAME>' \
    AzureDataLakeStorageGen2_Name='<YOUR AZURE STORAGE NAME>'

Below an example with a screenshot

![sparkles](pictures/image166.jpg)

### Deployment result

After a few seconds, your pipeline is deployed in your new environment

![sparkles](pictures/image167.jpg)

### Manual action after pipeline deployment

After deploying the pipeline, click on the "**Notebook**" activity to redefine the path of your notebook in your Azure Databricks workspace.

![sparkles](pictures/image168.jpg)

Then click on the "**Publish All**" button.

Test the pipeline by clicking "**validate**" and enter your parameters. Normally, everything should be fine. In case your get an error with 

![sparkles](pictures/image169.jpg)

In the case your get an error with the Notebook activity, double check if you have enought quota in your Azure subscription. You can also modify the Databricks connection in Azure Data Factory to size down the cluster. (For a reason I can't explain, sometime i have to click on "Access token"and click back to "Azure Key vault" to have values in the drop down lists)

![sparkles](pictures/image191.png)

## Verification in the SQL Database

Log in to your SQL Database located in the new environment. Click on "**Query editor**", then query your database to check the presence of the data in your table after running the Azure Data Factory pipeline:

![sparkles](pictures/image170.jpg)

## Editing the Power BI Report

If you use the Power BI file available on the [GitHub](https://github.com/franmer2/demowikipedia) (resources folder), you need to modify the connection string to retrieve the data directly from your SQL Database.Click on "**Edit Queries**" and then on "**Data Source Settings**":

![sparkles](pictures/image171.jpg)

Then click on the "**Change Source"** button to indicate the connection information of your SQL Database:

![sparkles](pictures/image172.jpg)

Then refresh the report data

![sparkles](pictures/image173.jpg)

## By the way, what is the relationship with the title of the article??!!


This is certainly the question you need to ask yourself at this point in the article (or not 😊)!

I remember that, from the first days I did this demo, the article about Darth Vader was very often in the top 50 most consulted articles. Having now the history of viewing articles, one can see the evolution of the consultation of this article day after day. Which is nicer, but it does not answer the question of the title of this article 😊



![sparkles](pictures/image174.jpg)

## Sources
https://www.tamr.com/from-devops-to-dataops-by-andy-palmer/

Thank you


#
[Franck Mercier](https://www.linkedin.com/in/mercierfranck/)