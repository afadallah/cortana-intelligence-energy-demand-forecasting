# Energy Demand Forecasting in Cortana Intelligence Suite


## Abstract [YC]


The recently published [Energy Demand Forecasting Solution Template](https://gallery.cortanaintelligence.com/SolutionTemplate/Demand-Forecasting-for-Energy-1) provides one-click deployment of an energy demand forecasting solution in Cortana Intelligence Suite. This [blog](https://blogs.technet.microsoft.com/machinelearning/2016/03/22/solution-template-for-energy-demand-forecasting/) gives a high level overview of the solution template. Advanced analytics solution implementers, i.e. Data Scientists and Data Engineers, usually need deeper understanding of the template components and architecture in order to use, maintain, and improve the solution.

This documentation provides more details of the solution template and step-by-step deployment instructions. Going through this manual deployment process will help implementers gain an inside view on how the solution is built and the function of each component.

## Requirements [PS]

You will need the following accounts and software to create this solution:

1.  Source code and instructions from this GitHub repo

2.  A Microsoft Azure subscription. (<https://azure.microsoft.com/>) **NOTE:** You need to have at least 20 cores quota under your Azure subscription for the on-demand HDInsight cluster. To see your quota, go to *portal.azure.com -> log in -> Subscriptions -> select your subscription -> All settings -> Usage + quotas*.

3.  An Azure Machine Learning Studio account (<http://studio.azureml.net>)

4.  A Microsoft Office 365 subscription for Power BI access.

5.  A network connection

6.  SQL Server Management Studio (<https://msdn.microsoft.com/en-us/library/mt238290.aspx>), Visual Studio (<https://www.visualstudio.com/en-us/visual-studio-homepage-vs.aspx>), or another similar tool to access a SQL server database.

7.  Microsoft Azure Storage Explorer (<http://storageexplorer.com/>)

8.  Power BI Desktop (<https://powerbi.microsoft.com/en-us/desktop>)

It will take about one day to implement this solution if you have all the required software/resources ready to use. The content of the document **Energy Demand Forecasting in Cortana Intelligence Suite_Deployment Guide.docx** is the same as this readme file. Use that documentation if you prefer reading a word document.

## Architecture[YC]
<img src="Figures/energyforecastingdiagram.png"/>

1.	The sample data is streamed by newly deployed **Azure Web Jobs**.

2.	This synthetic data feeds into the **Azure Event Hubs** and **Azure SQL** service as data points or events, that will be used in the rest of the solution flow.

3.	**Azure Stream Analytics** analyze the data to provide near real-time analytics on the input stream from the event hub and directly publish to PowerBI for visualization.

4.	The **Azure Machine Learning** service is used to make forecast on the energy demand of particular region given the inputs received.

5.	**Azure SQL Database** is used to store the prediction results received from the **Azure Machine Learning** service. These results are then consumed in the **Power BI** dashboard.

6. **Azure Data Factory** handles orchestration, and scheduling of the hourly model retraining.

7.	Finally, **Power BI** is used for results visualization, so that users can monitor the energy consumption from a region in real time and use the forecast demand to optimize the power generation or distribution process.

## Setup Steps

There are two paths: Batch path and real-time path.
Explain each path here.

## Batch Path Setup Steps

### 1. Create a new Azure Resource Group [PS]

### 2. Setup Azure SQL Database[PS]

### 3. Setup Azure Web Job/Data Simulator[PS]

### 4. Setup Azure Machine Learning [YC]

This section assumes that you have not set up any workspaces in Azure Machine Learning Studio but that you do have an account at <https://studio.azureml.net/>.

The first thing we need to do is to create the workspace. A workspace is where Azure Machine Learning experiments are created. It is also tied to a storage account for intermediate results during experiment processing. You can invite others to your workspace by adding them as *USERS* to share and collaborate with people.

-   Navigate to ***manage.windowsazure.com*** and login in to your account.

<!-- -->

-   On the left tab click ***MACHINE LEARNING***

-   In the bottom left hand corner click ***NEW***

-   Choose *DATA SERVICES\\MACHINE LEARNING\\QUICK CREATE*

-   For workspace name enter *energytemplate\[UI\]\[N\]*

-   Location South Central US

-   Choose the storage account created earlier. **NOTE:** If the previously created storage account doesn’t show up, use an existing one or create a new one. This storage account is for storing intermediate results and log files of Azure Machine Learning.

-   Click on ***Create an ML Workspace***

Now that we have a workspace to work within, we can copy over the required experiment from the Cortana Intelligence Gallery.

-   Navigate to ***studio.azureml.net*** and log into your account

-   In the upper right hand corner you will see a workspace name. If this does not match the name of the workspace we just created, use the arrow button to expose the workspace selection menu and choose the workspace *energytemplate\[UI\]\[N\]*.

-   Go to [energy forecast sample experiment](https://gallery.cortanaintelligence.com/Details/975ed028d71b490b9268d35094138358) in Cortana Intelligence Gallery.

-   On the experiment page click the ***Open In Studio*** button

-   In the dialog ***Copy experiment from Gallery***, choose appropriate location ***South Central US*** and the workspace we created earlier that you would like to copy it into. Click the **√** button.

-   This process may take a minute and the experiment will open in the requested workspace.

-   Click ***RUN*** at the bottom of the experiment page. This step will take less than a minute to finish and all objects in the graph will have a green check box on them to indicate they have finished running.

-   Click ***DEPLOY WEB SERVICE*** at the bottom of the page to create the Azure Web Service associated with the experiment. When completed, the browser will redirect to the web service home page.

    -   The web service home page can also be found by clicking the ***WEB SERVICES*** button on the left menu of the studio.azureml.net page once logged in.

-   Copy the ***API key*** from the web service home page and add it to the table below as you will need this information later.

-   Click the link ***BATCH EXECUTION*** under the ***API HELP PAGE*** section. On the BATCH EXECUTION help page, copy the ***Request URI*** under the ***Request*** section and add it to the table below as you will need this information later. Copy only the URI part https:… /jobs, ignoring the URI parameters starting with ? .

| **Web Service BES Details** |                           |
|-----------------------------|---------------------------|
| API Key                     | API key from API help page|
| Request URI\*               |                           ||

### 5. Setup Azure Data Factory (ADF) [YC]
We have now created all the necessary components for building data pipelines in Azure Data Factory. Data factory orchestrates all Azure data services to move and process data in pipelines.

Here is an overview of the ADF pipeline.
...

There are 3 main components of ADF: link service, dataset and pipieline. Please check the details here.

#### 1) Create Azure Data Factory


-   Navigate to ***portal.azure.com*** and login in to your account.

-   On the left tab click ***New&gt;Data and Analytics&gt;Data Factory***

-   Name: *energytemplate\[UI\]\[N\]*

-   Resource Group: Choose the resource group created previously ***energytemplate\_resourcegroup***

-   Location: EAST US

-   Click ***Create***

After the data factory is created successfully

-   On the left tab in the portal page (portal.azure.com), click ***Resource groups***

-   Search for the resource group created previously, ***energytemplate\_resourcegroup***

-   Under Resources, click on the data factory we just created, *energytemplate\[UI\]\[N\]*

-   Locate the ***Actions*** panel and click on ***Author and deploy***.

In the ***Author and deploy*** blade, we will create all the components of the data factory. Note that Datasets are dependent on Linked services, and pipelines are dependent on Linked services and Datasets. So we will create Linked services first, then Datasets, and pipelines at last.


#### 2) Create Linked Services
will create 2 Linked services in this solution. The scripts of the Linked services are located in the folder ***Data Factory\\LinkedServices*** of the solution package.

- **AzureSqlLinkedService.** This is the Linked service for the Azure SQL database.

  -   Open the file ***Data Factory\\LinkedServices\\AzureSqlLinkedService.json***. Replace \[Azure SQL server name\], \[Azure SQL database name\], \[SQL login user name\], and \[SQL login password\] with the corresponding information from the [Azure SQL Server and Database](#azure-sql-server-and-database) section.

  -   Go back to ***Author and deploy*** in the data factory on ***portal.azure.com.***

  -   Click ***New data store*** and select ***Azure SQL***.

  -   Overwrite the content in the editor window with the content of the modified AzureSqlLinkedService.json.

  -   Click ***Deploy***.

- **AzureMLEndpoint.** This is the Linked service for the Azure Machine Learning web service.

  -   Open the file ***Data Factory\\LinkedServices\\AzureMLEndpoint.json***. Replace \[Azure Machine Learning web service URI\] and \[Azure Machine Learning web service API key\] with the corresponding information from the [Create Azure Studio ML Workspace and Experiment](#create-azure-studio-ml-workspace-and-experiment) section.

  -   Go back to ***Author and deploy*** in the data factory on ***portal.azure.com.***

  -   Click ***New compute*** and select ***Azure ML***.

  -   Overwrite the content in the editor window with the content of the modified AzureMLEndpoint.json.

  -   Click ***Deploy***.

#### 3) Create Datasets

We will create 11 datasets pointing to Azure SQL tables. We will use the JSON files located at ***Data Factory\\Datasets.*** No modification is needed on the JSON files.

On ***portal.azure.com*** navigate to your data factory and click the ***Author and Deploy*** button.

For each JSON file under ***Data Factory\\Datasets\\SQLDatasets***,

-   At the top of the left tab, click ***New dataset*** and select ***Azure SQL ***

-   Copy the content of the file into the editor.

-   Click ***Deploy***

The excel file ***\\Data Factory\\Datasets\\DatasetDescription.xlsx*** describes the content of each dataset and how it gets updated. Re-visiting this file after creating the data pipelines may help you better understand the datasets.

#### 4) Create Pipelines

We will create 12 pipelines.

![](Figures/ADFPipelineExample.png)



We will use the JSON files located at ***Data Factory\\Pipelines.*** At the bottom of each JSON file, the “start” and “end” fields identify when the pipeline should be active and are in UTC time. We will modify the start and end time of each file to customize the schedule. For more information on scheduling in Data Factory, see [Create Data Factory](https://azure.microsoft.com/en-us/documentation/articles/data-factory-create-pipelines/) and [Scheduling and Execution with Data Factory](https://azure.microsoft.com/en-us/documentation/articles/data-factory-scheduling-and-execution/).

- Hourly aggregation pipeline
- Hourly model retraining and scoring pipeline


### 6. Setup Power BI [YC]
The essential goal of this part is to get the demand forecast of each region and visualize it. Power BI can directly connect to an Azure SQL database as its data source, where the prediction results are stored.

> Note:  1) In this step, the prerequisite is to download and install the free software [Power BI desktop](https://powerbi.microsoft.com/desktop). 2) We recommend you start this process 2-3 hours after you deploy the solution so that you have more data points to visualize.

#### 1) Get the database credentials.

  You can get your database credentials from the previous steps when you setting up the SQL database.

#### 2)	Update the data source of the Power BI file

  -  Make sure you have installed the latest version of [Power BI desktop](https://powerbi.microsoft.com/desktop).

  -	In this Git repository, you can download the **'EnergyDemandForecastSolution.pbix'** file under the folder **'Power BI Template'** and then open it. **Note:** If you see an error massage, please make sure you have installed the latest version of Power BI Desktop.

  - On the top of the file, click **‘Edit Queries’** drop down menu. Then choose **'Data Source Settings'**.
  ![](Figures/PowerBI-7.png)

  - In the pop out window, click **'Change Source'**, then replace the **"Server"** and **"Database"** with	your own server and database names and click **"OK"**. For server
	name, make sure you specify the port 1433 in the end of your server string
	(**YourSolutionName.database.windows.net, 1433**). After you finish editing, close the 'Data Source Settings' window.

  - When you are asked to enter the user name and password, make sure you choose **'Database'** option, then enter the username and password that you choose when you setting up the SQL database.

  - On the top of the screen, you will see a message. Click **'Apply Changes'** and now the dashboard is updated to connect to your database. In the backend, model is scheduled to be refreshed every 1 hour. You can click **'Refresh'** button on the top to get the latest visualization as time moving forward.

#### 3) [Optional] Publish the dashboard to [Power BIonline](http://www.powerbi.com/)
  Note that this step needs a Power BI account (or Office 365 account).
  - Click **"Publish"** on the top pannel. Choose **'My Workspace'** and few seconds later a window appears displaying "Publishing successed". Click the link on the screen to open it in a browser and enter your database credentials by following the instructions. To find detailed instructions, see [Publish from Power BI Desktop](https://support.powerbi.com/knowledgebase/articles/461278-publish-from-power-bi-desktop).

  - Now you can see new items showing under 'Reports' and 'Datasets'. To create a new dashboard: click the **'+'** sign next to the
    **Dashboards** section on the left pane. Enter the name "Energy Demand Forecasting Demo" for this new dashboard.

  - Once you open the report, click   ![Pin](Figures/PowerBI-4.png) to pin all the visualizations to your dashboard. To find detailed instructions, see [Pin a tile to a Power BI dashboard from a report](https://support.powerbi.com/knowledgebase/articles/430323-pin-a-tile-to-a-power-bi-dashboard-from-a-report). Here is an example of the dashboard.

      ![DashboardExample](Figures/PowerBI-11.png)


## Real-time Path Setup Steps

### 1. Setup Azure Blob Storage[PS]

### 2. Setup Azure Event Hub[PS]

### 3. Setup Azure Web Jobs[PS]

### 4. Azure Stream Analytics Jobs [YC]
#### 1) Provision a Stream Analytics job

-	In the Azure portal, click **NEW** > **DATA SERVICES** > **STREAM ANALYTICS** > **QUICK CREATE**.

-	Specify the following values, and then click **CREATE STREAM ANALYTICS JOB**:

	* **JOB NAME**: Enter a job name.

	* **REGION**: Select the region where you want to run the job. Consider placing the job and the event hub in the same region to ensure better performance and to ensure that you will not be paying to transfer data between regions.

	* **STORAGE ACCOUNT**: Choose the Azure storage account that you would like to use to store monitoring data for all Stream Analytics jobs that run within this region. You have the option to choose an existing storage account or create a new one.

3.	Click **STREAM ANALYTICS** in the left pane to list the Stream Analytics jobs.

	The new job will be shown with a status of **CREATED**. Notice that the **START** button on the top of the page is disabled. You must configure the job input, output, and query before you can start the job.

#### 2) Specify job input
-	In your Stream Analytics job, click **INPUTS** at the top of the page, and then click **ADD INPUT**. The dialog box that opens will walk you through several steps to set up your input.

-	Click **DATA STREAM**, and then click the right button.

-	Click **EVENT HUB**, and then click the right button.

-	Type or select the following values on the third page:

	* **INPUT ALIAS**: Enter a friendly name, such as *CallStream*, for this job. Note that you will be using this name in the query later.

	* **EVENT HUB**: If the event hub that you created is in the same subscription as the Stream Analytics job, select the namespace that the event hub is in.

		If your event hub is in a different subscription, select **Use Event Hub from Another Subscription**, and then manually enter information for **SERVICE BUS NAMESPACE**, **EVENT HUB NAME**, **EVENT HUB POLICY NAME**, **EVENT HUB POLICY KEY**, and **EVENT HUB PARTITION COUNT**.

	* **EVENT HUB NAME**: Select the name of the event hub.

	* **EVENT HUB POLICY NAME**: Select the event hub policy that you created earlier in this tutorial.

	* **EVENT HUB CONSUMER GROUP**: Type the name of the consumer group that you created earlier in this tutorial.

- Click the right button.

- Specify the following values:
  * **EVENT SERIALIZER FORMAT**: JSON
  * **ENCODING**: UTF8

- Click the **CHECK** button to add this source and to verify that Stream Analytics can successfully connect to the event hub.


#### 3) Specify job query
Stream Analytics supports a simple, declarative query model that describes transformations for real-time processing. To learn more about the language, see the [Azure Stream Analytics Query Language Reference](https://msdn.microsoft.com/library/dn834998.aspx).

Now we will create the queries for the jobs:

-   Navigate to ***manage.windowsazure.com*** and log in to your account.

<!-- -->

-   On the left tab click ***STREAM ANALYTICS***

-   Click on the one of the jobs that was created in the earlier steps.

-   At the top of the right page click ***QUERY ***

-   In the query box, copy the content of one of the scripts from the package folder ***Stream Analytics Queries***. The query files are named identically to the job name.

-   Click ***Save*** at the bottom of the page


#### 4) Specify job output

  - You can use the instructions in
[Azure Stream Analytics & Power BI: A real-time analytics dashboard for real-time visibility of streaming data](stream-analytics-power-bi-dashboard.md)
to set up the output of your Azure Stream Analytics job as your Power BI dashboard.

  - Locate the stream analytics job in your [Azure management portal](https://portal.azure.com). The name of the job should be: YourSolutionName+"saJob" (i.e. energydemosaJob). Click **'Outputs'** from the panel on the right.

  ![Add PowerBI output on ASA 1](Figures/PowerBI-1.png)

  - On the new window, click **'+Add'** on the top, and then it will show a window asking for information of the output. Under **'Sink'**, choose **'Power BI'**, then click **'Authorize'**. In the pop out window, log in with your Power BI account.

  - Once you successfully authorize your Power BI account, fill in other informtion as follows. Set the **Output Alias** as **'outputPBI'**. Set your **'Dataset Name'** and **'Table Name'** as **'EnergyForecastStreamData'**. Click **'Create'** once you finish.

#### 5) Start job for real-time processing
Because a job input, query, and output have all been specified, we are ready to start the Stream Analytics job for real-time fraud detection.

-	From the job **DASHBOARD**, click **START** ![Start](Figures/PowerBI-2.png) at the top of the page.

-	In the dialog box that opens, click **JOB START TIME**, and then click the **CHECK** button on the bottom of the dialog box. The job status will change to **Starting** and will shortly change to **Running**.

- You will get a confirmation message (e.g. 'Streaming Job started successfully') once everything is set up correctly.

### 5. Setup Real-time Power BI [YC]
#### 1) Login on [Power BI online](http://www.powerbi.com)

-   On the left panel Datasets section in My Workspace, you should be able to see a new dataset showing on the left panel of Power BI. This is the streaming data you pushed from Azure Stream Analytics in the previous step.

-   Make sure the ***Visualizations*** pane is open and is shown on the
    right side of the screen.

#### 2) Create a visulization on PowerBI online
We will use this example to show you how to create the "Demand by Timestamp" tile:

-	Click dataset **EnergyForecastStreamData** on the left panel Datasets section.

-	Click **"Line Chart"** icon.![LineChart](Figures/PowerBI-3.png)

-	Click EnergyForecastStreamData in **Fields** panel.

-	Click **“time”** and make sure it shows under "Axis". Click **“demand”** and make sure it shows under "Values".

-	Click **'Save'** on the top and name the report as “EnergyStreamDataReport”. The report named “EnergyStreamDataReport” will be shown in Reports section in the Navigator pane on left.

-	Click **“Pin Visual”**![](Figures/PowerBI-4.png) icon on top right corner of this line chart, a "Pin to Dashboard" window may show up for you to choose a dashboard. Please select "EnergyStreamDataReport", then click "Pin".

## Validation and Results
need to discuss if we want to keep this