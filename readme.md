# Advanced Tutorial: General Predictive Maintenance
![](images/042116_1633_PredictiveM1.png "Predictive Maintenance")

* Documentation site for Microsoft internal dog food.
* Documentation site for external private preview customers.

## Introduction

Understanding fleet maintenance requirements can have a large impact on business safety and profitability. 

An initial approach is to rely on **corrective maintenance**, where parts are replaced as they fail. Corrective maintenance ensures parts are used completely (not wasting component life), but incurs expense in both downtime and unscheduled maintenance requirements (off hours, or inconvenient locations).

An alternative is a **preventative maintenance** schedule. Here a business may track or test component failures and determine a safe lifespan in which to replace that component before failure. For safety critical machinery, this approach can insure no catastrophic failures. The down side is components are replaced frequently, many with remaining life left. 

The goal of **predictive maintenance** is to optimize the balance between corrective and preventative maintenance. This approach only replaces those components when they are close to failure. The savings in this case come from both extending component lifespans (compared to preventive maintenance), and reducing unscheduled maintenance (over corrective maintenance) and improving safety associated component failure.

The goal of this scenario is to guide a data scientist through the implementation and operationalization of the predictive maintenance solution using *Azure Machine Learning Workbench*. 

## Use Case Overview

A major problem faced by businesses in asset-heavy industries is the significant costs that are associated with delays to mechanical problems. Most businesses are interested in predicting when these problems arise in order to proactively prevent them before they occur. This reduces the costs by reducing downtime and, in some cases, increasing safety. Refer to the [playbook for predictive maintenance](https://docs.microsoft.com/en-us/azure/machine-learning/cortana-analytics-playbook-predictive-maintenance) for a detailed explanation of common use cases and the modeling approach for predictive maintenance.

This scenario leverages the ideas from the playbook by providing the steps to implement a predictive model for a scenario, which is based on a synthesis of multiple real-world business problems by bringing together common data elements observed among many predictive maintenance use cases..

The business problem for this simulated data is to predict issues caused by component failures. The business question therefore is “*What is the probability that a machine goes down due to failure of a component*?” This problem is formatted as a multiclass classification problem (multiple components per machine) and a machine learning algorithm is used to create the predictive model. The model is trained on historical data collected from machines. In this scenario, the user goes through the various steps of implementing such a model within the Azure Machine Learning Workbench environment.

## Prerequisites

* An [Azure account](https://azure.microsoft.com/en-us/free/) (free trials are available).
* An installed copy of [Azure Machine Learning Workbench](./overview-what-is-azure-ml) following the [quick start installation guide](./quick-start-installation) to install the program and create a workspace.
* Intermediate results for use across Jupyter notebooks in this scenario is stored in an Azure Blob Storage container. Instructions for setting up an Azure Storage account are at this [link](https://docs.microsoft.com/en-us/azure/storage/common/storage-create-storage-account#create-a-storage-account). 
* For [operationalization](https://github.com/Azure/Machine-Learning-Operationalization) of the model, it is best if the user runs a [Docker engine](https://www.docker.com/) installed and running locally. If not, you can use the cluster option but be aware that running an [Azure Container Service (ACS)](https://azure.microsoft.com/en-us/services/container-service/) can often be expensive.
* This scenario assumes that the user is running Azure ML Workbench on a Windows 10 machine with Docker engine locally installed. 
* The scenario was built and tested on a Windows 10 machine with the following specification: Intel Core i7-4600U CPU @ 2.10 GHz, 8-GB RAM, 64-bit OS, x64-based processor with Docker Version 17.06.0-ce-win19 (12801). 
* Model operationalization was done using this version of Azure ML CLI: azure-cli-ml==0.1.0a22

 
## Let's Begin

Launch the *Azure Machine Learning Workbench* App, sign-in, and create a new blank project by selecting the `+` option near the `Projects` menu on the top left pane. Call it "PredictiveMaintenance" by entering this in the `Project name` column and then select the `Create` button at the bottom of the pane. 

Next from the [GitHub repo](https://github.com/Azure/MachineLearningSamples-PredictiveMaintenance) download the [Jupyter notebooks](https://github.com/Azure/MachineLearningSamples-PredictiveMaintenance/tree/master/Code) to run within *Azure Machine Learning Workbench* App. These files need to be saved in the same folder called "PredictiveMaintenance", so that they can be run from within the app.  

To learn more about running these notebooks within the *Azure Machine Learning Workbench* App refer to this [link](https://github.com/Azure/ViennaDocs/blob/master/Documentation/UsingJupyter.md). From the `File` menu on the top left menu, select either the `Open Command Prompt` or `Open PowerShell`. Then run these commands to open a new window `http://localhost:8888/tree` where the code can be accessed within the `Code` folder. 

`az ml experiment prepare --target docker --run-configuration docker`

`pip install notebook`

`az ml notebook start`

Ensure that docker is running before you run the notebooks in order. When you open the notebooks, set the kernel to "PredictiveMaintenance docker"

## Task 1: Prepare your data

The simulated data consists of five comma-separated values (.csv) files. 

* [Machines](https://pdmmodelingguide.blob.core.windows.net/pdmdata/machines.csv): Features differentiating each machine. For example, age and model.
* [Error](https://pdmmodelingguide.blob.core.windows.net/pdmdata/errors.csv): The error log contains non-breaking errors thrown while the machine is still operational. These errors are not considered as failures, though they may be predictive of a future failure event. The error date-time are rounded to the closest hour since the telemetry data is collected at an hourly rate.
* [Maint](https://pdmmodelingguide.blob.core.windows.net/pdmdata/maint.csv): The maintenance log contains both scheduled and unscheduled maintenance records. Scheduled maintenance corresponds with regular inspection of components, unscheduled maintenance may arise from mechanical failure or other performance degradation. The maintenance date-time are rounded to the closest hour since the telemetry data is collected at an hourly rate.
* [Telemetry](https://pdmmodelingguide.blob.core.windows.net/pdmdata/telemetry.csv): The telemetry time-series data consists of voltage, rotation, pressure, and vibration sensor measurements collected from each machine in real time. The data is averaged over an hour and stored in the telemetry logs
* [Failures](https://pdmmodelingguide.blob.core.windows.net/pdmdata/failures.csv): Failures correspond to component replacements within the maintenance log. Each record contains the Machine ID, component type, and replacement date and time. These records are used to create the machine learning labels that the model is trying to predict.

See the [Data Ingestion](https://github.com/Azure/MachineLearningSamples-PredictiveMaintenance/blob/master/Code/data_ingestion.ipynb) Jupyter Notebook scenario in the Code section to download the raw data sets from the GitHub repository and create the PySpark data sets for this analysis.

The Data Ingestion Jupyter Notebook task in the `Code/1_data_ingestion.ipnyb` loads the five input data sets into PySpark format for this analysis and does some preliminary data visualization. The raw data is stored in an Azure Blob storage container on your subscription for use in the feature engineering task.

## Task 2: Feature Engineering

See the Feature Engineering Jupyter Notebook task in `Code/2_feature_engineering.ipnyb`, that takes PySpark data sets and creates the time series features used in the modeling step for this analysis. This feature engineering data set is also stored in your Azure Blob storage container for use in the model building and evaluation task.

## Task 3: Model Building & Evaluation

See the Model Building Jupyter Notebook task in `Code/3_model_building.ipnyb` that takes PySpark feature engineering data set from blob storage and split into two namely a train and a test dataset based on a date-time stamp. Then two models namely a Random Forest Classifier and Decision Tree Classifier are built on the training datasets. It then compares these models to determine a "best" solution for predict component failures. The resulting model is serialized and stored in your Azure Blob storage container for use in the operationalization task.

## Task 4: Operationalization

See the Model Building Jupyter Notebook task in `Code/4_operationalization.ipnyb` that takes one of the best models and builds the init() and run() functions need to deploy and operationalize these models. These functions are first tested locally and three files are saved locally pdmrfull.model, pdmscore.py,  service_schema.json in preparation for operationalization. 

### Task 4.1: Use the CLI to deploy and manage your web service

Use the following commands to set up an environment and account to run the web service.
 
* Create the environment (you need to do this once per environment e.g. dev or prod)
`az ml env setup -c -n <yourclustername> --location <e.g. eastus2>`

* Create a Model Management account (one time setup)
`az ml account modelmanagement create --location <e.g. eastus2> -n <your-new-acctname> -g <yourresourcegroupname> --sku-instances 1 --sku-name S1`

* Set the Model Management account
`az ml account modelmanagement set -n <youracctname> -g <yourresourcegroupname>`

* Set the environment. The cluster name is the name used in step 1 above. The resource group name was the output of the same process and would be in the command window when the setup process is completed.
`az ml env set -n <yourclustername> -g <yourresourcegroupname>`

Once this setup is complete, switch to a bash shell, and run the following commands to deploy your service and run it.
Enter the path where the notebook and other files are saved. Your actual path may be different from this example.
`cd ~/notebooks/azureml/realtime/`

This assumes that you saved your model, schema file and .py file locally.
`az ml service create realtime -f pdmscore.py -r  spark-py -m pdmrfull.model -s service_schema.json -n pdmservice --cpu 0.1`

This command will return the sample run command with sample data. You can get the Service Id from the output of the create command above.

`az ml service show realtime -i <yourserviceid>
Call the web service to get a prediction
az ml service run realtime -i <yourserviceid> -d "{\"input_df\": [{\"machineID\":114, \"vo
lt_rollingmean_3\":163.375732902, \"rotate_rollingmean_3\":333.149484586, \"pressure_rollingmean_3\":100.183951698, \"vibration_rollingmean_3\":44.0958812638, \"volt_rollingme
an_24\":164.114723991, \"rotate_rollingmean_24\":277.191815232, \"pressure_rollingmean_24\":97.6289110707, \"vibration_rollingmean_24\":50.8853505161, \"volt_rollingstd_3\":21
.0049565219, \"rotate_rollingstd_3\":67.5287259378, \"pressure_rollingstd_3\":12.9361526861, \"vibration_rollingstd_3\":4.61359760918, \"volt_rollingstd_24\":15.5377738062, \"
rotate_rollingstd_24\":67.6519885441, \"pressure_rollingstd_24\":10.528274633, \"vibration_rollingstd_24\":6.94129487555, \"error1sum_rollingmean_24\":0.0, \"error2sum_rolling
mean_24\":0.0, \"error3sum_rollingmean_24\":0.0, \"error4sum_rollingmean_24\":0.0, \"error5sum_rollingmean_24\":0.0, \"comp1sum\":489.0, \"comp2sum\":549.0, \"comp3sum\":549.0
, \"comp4sum\":564.0, \"age\":180}]}"`

This should return the predicted output label is as follows:
`"0.0"` 


## Conclusion

This scenario gives the reader an overview of how to build an end to end predictive maintenance solution using PySpark within the Jupyter notebook environment in *Azure Machine Learning Workbench*. The scenario also guides the reader on how the best model can be easily operationalized and deployed using *Azure Machine Learning Model Management* environment for use in a production environment for making realtime failure predictions. Then the reader can edit relevant parts of the scenario to fit their business needs.  

## References

This use case has been previously developed on multiple platforms:
 
* [Predictive Maintenance Solution Template](https://docs.microsoft.com/en-us/azure/machine-learning/cortana-analytics-playbook-predictive-maintenance)
* [Predictive Maintenance Modeling Guide](https://gallery.cortanaintelligence.com/Collection/Predictive-Maintenance-Modelling-Guide-1)
* [Predictive Maintenance Modeling Guide using SQL R Services](https://gallery.cortanaintelligence.com/Tutorial/Predictive-Maintenance-Modeling-Guide-using-SQL-R-Services-1)
* [Predictive Maintenance Modeling Guide Python Notebook](https://gallery.cortanaintelligence.com/Notebook/Predictive-Maintenance-Modelling-Guide-Python-Notebook-1)
* [Predictive Maintenance using PySpark](https://gallery.cortanaintelligence.com/Tutorial/Predictive-Maintenance-using-PySpark)

## How to get help and send feedback
We are eager to hear your experience as you go through this example scenario. If you have any feedback or need help, contact us through the [forum](https://social.msdn.microsoft.com/forums/azure/en-US/home?forum=machinelearning).

# Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us the rights to use your contribution. For details, visit https://cla.microsoft.com.

When you submit a pull request, a CLA-bot automatically determines whether you need to provide a CLA and decorate the PR appropriately. You only need to follow the instructions provided by the bot across all Microsoft repos to use our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
More information is available at [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.
