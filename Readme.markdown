This long article will help you to prepare for passing DP 200 exam.
The first part of the article is about all the steps to build an analytic solution from scratch. At the end of this part, you will also get a cool demo to showcase to your customers.

The second part of this article is about some tips and tricks I think it could help to pass the exam. I highly recommend to [“follow the white rabbit”](https://www.youtube.com/watch?v=6IDT3MpSCKI) ![sparkles](pictures/WhiteRabbit.jpg).

# Build an Analytic solution in Azure

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

Then click on the "**Advanced**" tab, then in the "**Data Lake Storage gen2**" section, click on "**Enabled**". This will enable new features related to the new version of our data lake.![sparkles](pictures/WhiteRabbit.jpg)

![sparkles](pictures/image011.jpg)

If necessary you can add tags by clicking on the tab "Tags". Tags are useful for administering and tracking the costs of your Azure services.

When done, click on the "**Review + create**" button

![sparkles](pictures/image012.jpg)

Then on the "**Create**" button

![sparkles](pictures/image013.jpg)

## Creating the SQL Database Database

From the Azure portal, click on "**Create a resource**", "**Database**", then "**SQL Database**"

![sparkles](pictures/image014.jpg)

Fill out the database creation form and click on "**Review + Create**". For this article, I will call the database "**Musicology**", and I will keep the level of Tier to basic. Nothing will prevent us subsequently to change this level of performance according to the needs (principle of elasticity). (This change can also be done programmatically by using SERVICE_OBJECTIVE argument, but we will discuss this point later in the article ![sparkles](pictures/WhiteRabbit.jpg))

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

From the Azure portal, click on "**Azure Active Directory**" and then on "**App registration**".

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

Then click on "**Access Control (IAM)**" (Actually, we are in the process to define access management for Cloud resources. To manage access control, we use what we called **Role-based access control** (RBAC). [This article](https://docs.microsoft.com/en-us/azure/role-based-access-control/overview) will give you a good overview of what RBAC is. ![sparkles](pictures/WhiteRabbit.jpg) )

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

We will now configure Azure Databricks so that it can use the secrets set in Azure Key Vault. For information, the complete documentation is [here](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html). ![sparkle](pictures/WhiteRabbit.jpg)

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

For the continuation of the article, we will download the notebook (DatabricksNotebook_Wikipedia_ADLSGen2_Generic.dbc) which is at the following address: https://1drv.ms/u/s!Am-C-ktMH9lgg9MCqqG5dS8XKFKxDA or https://github.com/franmer2/HelpForDP200/tree/master/resources



**You will then have to modify the notebook to add your information concerning the application ID, the "data lake" as well as your SQL Database.**

From your Azure Databricks workspace, navigate to where you want to import the notebook. Click on the down arrow to bring up the context menu and click on "**Import**".

![sparkles](pictures/image048.jpg)

Check the **"File"** box, enter the path to the notebook and click on the **"Import"** button

![sparkles](pictures/image049.jpg)

If all goes well, the notebook should be in your workspace as shown below

![sparkles](pictures/image050.jpg)

========================================

Later in this article, we will see how to use this notebook with interactive cluster. It could be a good option to test the notebook before to run it in production ![sparkles](pictures/WhiteRabbit.jpg)

========================================

## Creating access tokens
We will create the access tokens for Power BI and Azure Data FactoryAt the top right of your workspace, click on the character icon and then on "**User Settings**" ![sparkle](pictures/WhiteRabbit.jpg)

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

The zone "demo_datasets" will be created manually. The "wikipedia_results" zone will be created automatically by Azure Databricks. In order to interact with Azure Data Lake Gen2, yo can use [Azure Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/). From Azure Storage Explorer, sign in to your Azure account. Find your lake of data, do a right click on it, then create a container "**wikipedia**". ![sparkles](pictures/WhiteRabbit.jpg)

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

Copy the string below into the "**Add Dynamic Content**" field and click on "**Finish**" (you may have an error after the copy/paste, in this case, double check the quotes (')): ![sparkles](pictures/WhiteRabbit.jpg)

`  @concat(dataset().YearDS,'/',dataset().YearDS,'-',dataset().MonthDS,'/pageviews-',dataset().YearDS,dataset().MonthDS,dataset().DayDS,if(less(int(dataset().HourDS),10),'-0','-'),dataset().HourDS,'0000.gz')`

  

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

`
@concat('pageviews-',dataset().YearDS,dataset().MonthDS,dataset().DayDS,if(less(int(dataset().HourDS),10),'-0','-'),dataset().HourDS,'0000.gz')
`



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

As this activity does not exist yet in the list of available activities, we will create it by code. (Edit: the delete activity is now available through the UI, but you can still do it via the code just for training ![sparkles](pictures/WhiteRabbit.jpg))

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

 `
 @activity('Get Metadata1').output.Exists
 `   



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

A good thing is to be able to have the report updated automatically and have a daily refresh to get Wikipedia data.
We can achieve that with 2 steps
•	Schedule daily Azure Data Factory pipeline execution
•	Automatic Power BI dataset daily refresh


## Schedule daily Azure Data Factory pipeline execution ##

Now it could be nice to have our pipeline runs every day to feed our database and get the report up to date. Azure Data Factory has the capability to define triggers to be executed according rules and time you can defined.


Go back to Azure Data Factory and jump into your pipeline.
Click on "**Add trigger**" and "**New/Edit**" ![sparkles](pictures/WhiteRabbit.jpg)

![sparkles](pictures/image300.jpg)

A new pane appears on the right. Click on the little arrow to open the drop-down list and click on "**+New**"

![sparkles](pictures/image301.jpg)

Define the trigger parameters to fire one time a day like shown in the screenshot below. Then click on "**Next**" button:

![sparkles](pictures/image302.jpg)

In the Edit Trigger pane, enter parameters like showns below and click "**Finish**"


|Parameter|Value|
|:--------|:----|
|Year|`@formatDateTime(utcnow(),'yyyy')`|
|Month|`@formatDateTime(utcnow(),'MM')`|
|Day|`@adddays(utcnow(),-1,'dd')`|
|Hours|`[0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23]`|

(The pipeline may fail at the beginning of each month. I need to solve the issue with the first day of each month and the delay between logs availability on Wikipedia site and UTC time)

![sparkles](pictures/image303.jpg)

Now, your trigger is created and can be monitored in the Azure Data Factory "**Monitor**" section ![sparkles](pictures/WhiteRabbit.jpg)

![sparkles](pictures/image304.jpg)

If you need more Analytics capability, you can also use the preview feature "**Azure Data Factory Analytics (Preview when I'm writting this article (July 29,2019))**"

![sparkles](pictures/image305.jpg)

Here a sample of monitoring my pipeline with "**Log Analytics**"

![sparkles](pictures/image306.jpg)

If I click on the error indicator, I can see clearly when and where the error occurs

![sparkles](pictures/image307.jpg)

# Tips and Tricks for the exam #

## Secure your storage layer ##

### Data Lake Gen2 ###

 I highly recommend to have a look to [this article](https://docs.microsoft.com/en-us/azure/storage/common/storage-advanced-threat-protection).   

Concerning our Data Lake Gen2 storage, we will secure it by using the **Advanced Threat Protection** feature.

From the Azure Portal, click on your Data Lake Gen2 resouce and then click on **Advanced security**. Then click on "**Enable Advanced Thread Protection**"

![sparkles](pictures/image330.jpg)

A new window appears and will give you recommendations an security alerts. You can also get all recommendations by clicking on "**View all recommendations in Security Center**" button.

![sparkles](pictures/image331.jpg)


## Secure SQL Database ##
To secure your SQL database, you can use the features listed below: ![sparkles](pictures/WhiteRabbit.jpg)

- Always Encrypted (we will see this feature later in the article)
- Dynamic Data Masking
- Row Level Security
- Advanced Data Security

### Dynamic Data Masking ###

[This article](https://docs.microsoft.com/en-us/sql/relational-databases/security/dynamic-data-masking?view=sql-server-2017) will give you a good overview of what Dynamic Data Masking is.

You can setup dynamic data masking through Azure portal. From your SQL Database resource, click on "**Dynamic Data Masking**"

![sparkles](pictures/image332.jpg)

Click on "**Add mask**" button

![sparkles](pictures/image333.jpg)

Choose a column and define a mask and click on the "**Add**" button

![sparkles](pictures/image334.jpg)

The new field is ready to be masked. To vailidate, click on "**Save**" button.

![sparkles](pictures/image335.jpg)

You can setup dynamic data masking with T-SQL script. More info in [this article](https://docs.microsoft.com/en-us/sql/relational-databases/security/dynamic-data-masking?view=sql-server-2017)



You can also setup dynamic data masking with Powershell or REST API. more details in [this article](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-dynamic-data-masking-get-started
)

### Row Level Security ###

Row-Level Security enables you to use group membership or execution context to control access to rows in a database table.

[This article](https://docs.microsoft.com/en-us/sql/relational-databases/security/row-level-security?view=sql-server-2017
) will give more more details on this feature

You can use this article's examples with the SQL database you have created in this article.

### Advanced Data Security ###

Through the Azure Portal, you can leverage Advanced Data Security capabilities, a **database level**, for:

- Data Discovery and Classification
- Vulnerability Assessment
- Advanced Threat Protection

![sparkles](pictures/image336.jpg)

At the Azure SQL **server** level, you can also define periodic reccuring scans and get the repport through email

![sparkles](pictures/image339.jpg)

Below a screeshot of a mail I received each week:

![sparkles](pictures/image440.jpg)



### Auditing ###
You can activate the auditing feature. You can choose the log destination between:

- Storage account
- Log Analytics
- Event Hub (for real time log analysis scenario, for instance)

![sparkles](pictures/image337.jpg)

If you choose log Analytics, this below a sample of what you can get via Azure Monitor

![sparkles](pictures/image338.jpg)




## Polybase ##

Until know, you have a first vision of what an Analytics solution could be, but it's not the only pattern. To help you with DP 200, I change a little bit the architecture to introduce Azure DQL Data Warehouse with polybase concept. Pay attention to the sentences with the white rabbit ![sparkles](pictures/WhiteRabbit.jpg) and my advice could be "Follow the white rabbit" ;)

![sparkles](pictures/image308.jpg)



[Azure SQL Data Warehouse] (https://docs.microsoft.com/en-us/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is
) (our Massively Parallel Processing (MPP) Enterprise Data Warehouse) has a cool concept called "Polybase". Polybase allows to create external table that just hold the schema but the data stay in it's original storage. In our case, we will create an Azure Data Warehouse service, and create external table only with the wikipedia data schema and point to our datalake (to the result zone).

### Create Azure SQL Data Warehouse ###

From Azure portal, click on the plus sign, "**Databases**" then "**SQL Data Warehouse**"

![sparkles](pictures/image309.jpg)

Fill the form to create the data warehouse. Choose the SQL server you created previously and select the performance level to the minimum, it will be enough for our test.
Click on "**Review + create**" button. 

![sparkles](pictures/image310.jpg)

It should takes few minutes. When done, click on "**Go to resource**" button.

![sparkles](pictures/image311.jpg)

###   Configure Polybase  ### 
Now, we will configure Polybase to access the data stored in the data lake

You can connect to your Datawarehouse with several tools like

- SQL Server management Studio (SSMS)
- Visual Studio
- Azure Data Studio

In our case, we will use SSMS. Below the connection windows. (in the case you can't connect to your server, check the [Azure SQL server firewall settings](https://docs.microsoft.com/en-us/azure/sql-data-warehouse/load-data-wideworldimportersdw#create-a-server-level-firewall-rule)).

In the server name field, enter the name of SQL server on which you deployed your data warehouse. in our case, the server is called "fallinlove2nite.database.windows.net". For authentication, use the one you created during the server creation step above.

![sparkles](pictures/image312.jpg)


===================================================================

You can get the server name from the portal, in the SQL data warehouse resource "**Overview**" screen 

![sparkles](pictures/image313.jpg)

===================================================================

Below the sequence in order to configure Polybase ![sparkles](pictures/WhiteRabbit.jpg) 

- **Optional:** Create a user for loading data
- Create a Master Key
- Create a database scope credential
- Create an external data source
- Create External File Format
- **Optional:** Create schema for external table
- Create external table
- **Optional:** Create statisitics
- All the steps above is well detailed in this [article](https://docs.microsoft.com/en-us/azure/sql-data-warehouse/load-data-wideworldimportersdw) ![sparkles](pictures/WhiteRabbit.jpg) 

### Create a master key ###
If not yet created in your SQL Server, create a new master key

`CREATE MASTER KEY;`

### Create a database scope credential ###

At the date I'm writing this article (July 2019), Polybase can connect to Data Lake Gen2 only with account storage key.

You can use whatever you want for **IDENTITY** in the script below:




`
CREATE DATABASE SCOPED CREDENTIAL ADLSCredential
WITH
    IDENTITY = 'user',
    SECRET = '<azure_storage_account_key>'
;
`

![sparkles](pictures/image315.jpg)



### Create the external data source ###

Now we need to create an external data source to our Data Lake Gen2. Below, generic code to connect to ADLS Gen2

` 
CREATE EXTERNAL DATA SOURCE AzureDataLakeStorage
WITH (
    TYPE = HADOOP,
    LOCATION='abfs[s]://<container>@<AzureDataLake account_name>.dfs.core.windows.net', -- Please note the abfss endpoint for when your account has secure transfer enabled
    CREDENTIAL = ADLSCredential
);`


In our case, T_SQL script look like:


`
CREATE EXTERNAL DATA SOURCE AzureDataLakeStorage
WITH (
    TYPE = HADOOP,
    LOCATION='abfss://wikipedia@thevaultgen2.dfs.core.windows.net', 
    CREDENTIAL = ADLSCredential
);
`

![sparkles](pictures/image316.jpg)

### Configure data format ###

To be able to query our data, we need to specify the external data format

`
CREATE EXTERNAL FILE FORMAT TextFileFormat
WITH
(   FORMAT_TYPE = DELIMITEDTEXT
,    FORMAT_OPTIONS    (   FIELD_TERMINATOR = '|'
                    ,    STRING_DELIMITER = ''
                    ,    DATE_FORMAT         = 'yyyy-MM-dd HH:mm:ss.fff'
                    ,    USE_TYPE_DEFAULT = FALSE
                    )
);
`

in our case

`
CREATE EXTERNAL FILE FORMAT TextFileFormat
WITH
(   FORMAT_TYPE = DELIMITEDTEXT
,    FORMAT_OPTIONS    (   FIELD_TERMINATOR = ',')
);
`

![sparkles](pictures/image317.jpg)

### Create the external table ###

Now we will create the external table to query our data stored in the data lake. In our case this is the T-SQL script:

`
CREATE EXTERNAL TABLE [dbo].[Wikipedia_external] (
    [Article] [nvarchar](500) NULL,
    [Language] [nvarchar](10) NULL,
    [Nb Hit] [int] NULL,
	[Year] [nvarchar](10) NULL,
	[Month] [nvarchar](10) NULL,
	[Day] [nvarchar](10) NULL
) 
WITH
(
    LOCATION='/wikipedia_results/'
,   DATA_SOURCE = AzureDataLakeStorage
,   FILE_FORMAT = TextFileFormat
,   REJECT_TYPE = Percentage
,   REJECT_VALUE = 90
,   REJECT_SAMPLE_VALUE = 200
);
`

![sparkles](pictures/image318.jpg)


Now you can query your external table. below a very simple query to get the result directly from our data lake

`
Select * from [Wikipedia_external]
`

![sparkles](pictures/image319.jpg)

To complete your skills with polybase, I highly recommend reading this [article](https://docs.microsoft.com/en-us/azure/sql-data-warehouse/sql-data-warehouse-load-from-azure-data-lake-store) ![sparkles](pictures/WhiteRabbit.jpg) 


# Azure Data Warehouse Monitoring #

It could be a good idea to monitor your Azure SQL Data Warehouse, especially the cache usage

This is an [article](https://docs.microsoft.com/en-us/azure/sql-data-warehouse/sql-data-warehouse-how-to-monitor-cache) I highly recommend to read ![sparkles](pictures/WhiteRabbit.jpg).

And find a room in your brain to store this table [cache hit and used percentage section](https://docs.microsoft.com/en-us/azure/sql-data-warehouse/sql-data-warehouse-how-to-monitor-cache#cache-hit-and-used-percentage) ![sparkles](pictures/WhiteRabbit.jpg)


# Backup SQL and move it to another destination #

There are several ways to backup SQL data, but 2 of them deserves our attention: Bacpac and Dacpac. Below a quick description ![sparkles](pictures/WhiteRabbit.jpg)

- A BACPAC is a Windows file with a .bacpac extension that encapsulates a database's **schema and data** . The primary use case for a BACPAC is to move a database from one server to another - or to migrate a database from a local server to the cloud - and archiving an existing database in an open format.

- A DACPAC is focused on capturing and deploying schema, including upgrading an existing database. The primary use case for a DACPAC is to deploy a tightly defined schema to development, test, and then to production environments. And also the reverse: capturing production's schema and applying it back to test and development environments. In other words, DACPAC **only contains the schema**, not the data.

# SQL Database ServiceObjective #

Maybe you need to get information about your SQL database edition and ServiceObjective

To get the information, you can use the following T-SQL script in SSMS

`
SELECT DATABASEPROPERTYEX('Musicology','Edition') AS Edition
, DATABASEPROPERTYEX('Musicology','ServiceObjective') AS ServiceObjective
`

![sparkles](pictures/image320.jpg)

And maybe you need to modify the Service Objective via T-SQL, in this case you have to use **SERVICE_OBJECTIVE** argument with the T-SQL script below:  ![sparkles](pictures/WhiteRabbit.jpg)

`
ALTER DATABASE [Musicology] MODIFY(EDITION = 'standard', MAXSIZE = 100 MB, SERVICE_OBJECTIVE = 'S0');
`

(You have to wait few minutes before the changes occur)

If you run again the first T-SQL script, you can see the new edition and Service Objective

![sparkles](pictures/image321.jpg)

# Protect your SQL data with always encrypted #

From [this article](https://docs.microsoft.com/en-us/sql/relational-databases/security/encryption/always-encrypted-database-engine?view=sql-server-2017
), there is a good definition of what always encrypted is:

"Always Encrypted is a feature designed to protect sensitive data, such as credit card numbers or national identification numbers (for example, U.S. social security numbers), stored in Azure SQL Database or SQL Server databases. Always Encrypted allows clients to encrypt sensitive data inside client applications and never reveal the encryption keys to the Database Engine ( SQL Database or SQL Server). As a result, Always Encrypted provides a separation between those who own the data (and can view it) and those who manage the data (but should have no access). By ensuring on-premises database administrators, cloud database operators, or other high-privileged, but unauthorized users, cannot access the encrypted data, Always Encrypted enables customers to confidently store sensitive data outside of their direct control."

During the process to encrypt your data you will have to choose between determinic or randomized encryption. ![sparkles](pictures/WhiteRabbit.jpg)

- **Deterministic** encryption always generates the same encrypted value for any given plain text value. Using deterministic encryption allows point lookups, equality joins, grouping and indexing on encrypted columns. However, it may also allow unauthorized users to guess information about encrypted values by examining patterns in the encrypted column, especially if there is a small set of possible encrypted values, such as True/False, or North/South/East/West region. Deterministic encryption must use a column collation with a binary2 sort order for character columns.
  
- **Randomized** encryption uses a method that encrypts data in a less predictable manner. Randomized encryption is more secure, but prevents searching, grouping, indexing, and joining on encrypted columns.

- Use deterministic encryption for columns that will be used as search or grouping parameters, for example a government ID number. 
- Use randomized encryption, for data such as confidential investigation comments, which are not grouped with other records and **are not used to join tables.**


From SSMS, right click the WikipediaLogs table and select "**Encrypt Columns...**"

![sparkles](pictures/image322.jpg)

The introduction window appears, click on **Next**

![sparkles](pictures/image323.jpg)

Choose the columns you want to encrypt and the encryption type. Click on the **Next** button

![sparkles](pictures/image324.jpg)

According your security policies, you can store your key either in Windows certificate store or in Azure Key Vault. Click on the **Next** button

![sparkles](pictures/image325.jpg)

On the **Run Settings** window, click on the **Next** button

![sparkles](pictures/image326.jpg)

On the **Summary** window, clik on **Finish** button

![sparkles](pictures/image327.jpg)

If all is ok, you should get a result like the one in the screenshot below

![sparkles](pictures/image328.jpg)

Now, if you query your table, you can see that the data in the Request column are encrypted

![sparkles](pictures/image329.jpg)

# SQL Data Sync #

In the case you have to synchronize bi-directionally data across multiple SQL sources (on-premise or in the Cloud), it's a good idea to read [this article](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-sync-data) ![sparkles](pictures/WhiteRabbit.jpg)

# Azure Data Factory and Powershell #

It’s possible to automate pipeline creation with PowerShell. Below the sequence to create an Azure Data Factory pipeline with PowerShell. (order matters) ![sparkles](pictures/WhiteRabbit.jpg)

- **optional**: create a resources group
- Data Factory creation: Set-AzDataFactoryV2
- Linked Services creation: Set-AzDataFactoryV2LinkedService
- Datasets creation: Set-AzDataFactoryV2Dataset
- Pipeline creation: Set-AzDataFactoryV2Pipeline
- Run the pipeline: Invoke-AzDataFactoryV2Pipeline
- Get details about the run: Get-AzDataFactoryV2ActivityRun

More details are availble in [this article](https://docs.microsoft.com/en-us/azure/data-factory/scripts/bulk-copy-powershell?toc=%2fpowershell%2fmodule%2ftoc.json#sample-script
)

# Azure Data Factory and Integration Runtime #

Azure Data Factory relies on Integration Runtime, a compute infrastructure, to provide several data integration capabilities like:

- **Data Flow**: Execute a [Data Flow](https://docs.microsoft.com/en-us/azure/data-factory/concepts-data-flow-overview) in managed Azure compute environment.
- **Data movement**: Copy data across data stores in public network and data stores in private network (on-premises or virtual private network). It provides support for built-in connectors, format conversion, column mapping, and performant and scalable data transfer.
- **Activity dispatch**: Dispatch and monitor transformation activities running on a variety of compute services such as Azure Databricks, Azure HDInsight, Azure Machine Learning, Azure SQL Database, SQL Server, and more.
- **SSIS package execution**: Natively execute SQL Server Integration Services (SSIS) packages in a managed Azure compute environment.

There are several Integration Runtime types ![sparkles](pictures/WhiteRabbit.jpg) 

| IR Type  |      Public network |Private network|
| :---- | :-------- |:--------|
| **Azure**  |Data Flow|
|   |Data movement   ||
||Activity dispatch||
| **Self-hosted** |    Data movement |Data movement|
||Activity dispatch|Activity dispatch|
| **Azure-SSIS**   |SSIS package execution|SSIS package execution|

More information can be found in [this article](https://docs.microsoft.com/en-us/azure/data-factory/concepts-integration-runtime
)

## Azure Databricks and Interactive cluster ##
Previously in this article, we invoked Azure Databrick's notebook via Azure Data Factory. But you can also use the notebook with interactive cluster. ![sparkles](pictures/WhiteRabbit.jpg)

From your Azure Databricks's workspace, on the left, click on "**Clusters**" then on "**Create Cluster**"

![sparkles](pictures/image441.jpg)

Fill the creation cluster form

![sparkles](pictures/image442.jpg)

You have to choose for a cluster mode ![sparkles](pictures/WhiteRabbit.jpg)

- **Standard** clusters are the default and can be used with Python, R, Scala, and SQL. Standard clusters are configured to automatically terminate after 120 minutes.
- **High-concurrency** clusters are tuned to provide the efficient resource utilization, isolation, security, and the best performance for sharing by multiple concurrently active users. High concurrency clusters support only SQL, Python, and R languages. High concurrency clusters are configured to not terminate automatically.

Select the worker type and driver typpe you need

- **Workers** type run the Spark executors and other services required for the proper functioning of the clusters. When you distribute your workload with Spark, all of the distributed processing happens on workers.


- **Driver** type maintains state information of all notebooks attached to the cluster. The driver node is also responsible for maintaining the SparkContext and interpreting all the commands you run from a notebook or a library on the cluster. The driver node also runs the Apache Spark master that coordinates with the Spark executors.
The default value of the driver node type is the same as the worker node type. You can choose a larger driver node type with more memory if you are planning to collect() a lot of data from Spark workers and analyze them in the notebook.

By default the driver node uses the same instance type as the worker node

*Note*: have in mind that Azure Databricks bills you for virtual machines (VMs) provisioned in clusters **AND** Databricks Units (DBUs) based on the VM instance selected. A DBU is a unit of processing capability per hour. More details in [this article](https://azure.microsoft.com/en-us/pricing/details/databricks/)

Few minutes after clicking the create cluster, you should get a new cluster running

![sparkles](pictures/image443.jpg)

On the left, click on "**Recents**" button and select the notebook

![sparkles](pictures/image444.jpg)

You need to attach the notebook to a cluster. Click on "**Detached**" and select a cluster

![sparkles](pictures/image445.jpg)

You can run the whole notebook or cell by cell.

If you want to run all the notebook, click on "**Run All**" button

![sparkles](pictures/image447.jpg)

If you want to run the notebook cell by cell, click on a cell and hit the "**Shift**" + "**Enter**" key or click on the arrow on the top left pf the cell

![sparkles](pictures/image448.jpg)

Before running the notebook, you have to enter parameters. (If widgets are not available, run cmd4 by hiting "**Shift**" + "**Enter**")

![sparkles](pictures/image446.jpg)




**WARNING**: in our case, to test the notebook with interactive cluster, be sure you have files in the *demo_datasets* folder in your data lake, set the parameters according the files in the folder, do it **cell by cell** and start from the cell **cmd4**

![sparkles](pictures/image449.jpg)



# Conclusion #
I know that is a very long article, but I think it will help you to build a good knowledge on our data platform and maximize your chances to pass DP200 exam (and maybe the DP201 ;)). I also recommend having a look on the following topics that I don't cover in this article:

- Azure Cosmos DB (have a special read on how create collections and how to deal with partition)
- A read on HDInsight (Storm, MapReduce and Spark)
- File type (Avro, Parquet, ORC) ![sparles](pictures/WhiteRabbit.jpg)
- ARM Template (you can check the ARM part of my [original article](https://github.com/franmer2/demowikipedia))
- Real Time data (Stream Analytics, Event hub, Edge)
- SQL Data Warehouse [distributed tables](https://docs.microsoft.com/en-us/azure/sql-data-warehouse/sql-data-warehouse-tables-distribute) ![Sparkles](pictures/WhiteRabbit.jpg)


#
[Franck Mercier](https://www.linkedin.com/in/mercierfranck/)