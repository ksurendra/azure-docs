---
title: Enable Application Insights for the Azure Machine Learning service in production
description: Learn how to set Application Insights for the Azure Machine Learning service for deployment to Azure Kubernetes Service
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: conceptual
ms.reviewer: jmartens
ms.author: marthalc
author: marthalc
ms.date: 10/01/2018
---
# Monitor your Azure Machine Learning models in production with Application Insights

In this article, you learn how to set up Azure Application Insights for your Azure Machine Learning service. Application Insights gives you the opportunity to monitor:
* Request rates, response times, and failure rates.
* Dependency rates, response times, and failure rates.
* Exceptions.

[Learn more about Application Insights](../../application-insights/app-insights-overview.md). 

## Prerequisites
* An Azure subscription. If you don't have one, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.
* An Azure Machine Learning workspace, a local directory that contains your scripts, and the Azure Machine Learning SDK for Python installed. To learn how to get these prerequisites, see [How to configure a development environment](how-to-configure-environment.md).
* A trained machine learning model to be deployed to Azure Kubernetes Service (AKS). If you don't have one, see the [Train image classification model](tutorial-train-models-with-aml.md) tutorial.
* An [AKS cluster](how-to-deploy-to-aks.md).

## Enable and disable in the portal

You can enable and disable Application Insights in the Azure portal.

### Enable

1. In the [Azure portal](https://portal.azure.com), open your workspace.

1. On the **Deployments** tab, select the service where you want to enable Application Insights.

   [![List of services on the Deployments tab](media/how-to-enable-app-insights/Deployments.PNG)](./media/how-to-enable-app-insights/Deployments.PNG#lightbox)

3. Select **Edit**.

   [![Edit button](media/how-to-enable-app-insights/Edit.PNG)](./media/how-to-enable-app-insights/Edit.PNG#lightbox)

4. In **Advanced Settings**, select the **Enable AppInsights diagnostics** check box.

   [![Selected check box for enabling diagnostics](media/how-to-enable-app-insights/AdvancedSettings.png)](./media/how-to-enable-app-insights/AdvancedSettings.png#lightbox)

1. Select **Update** at the bottom of the screen to apply the changes. 

### Disable
1. In the [Azure portal](https://portal.azure.com), open your workspace.
1. Select **Deployments**, select the service, and select **Edit**.

   [![Edit button](media/how-to-enable-app-insights/Edit.PNG)](./media/how-to-enable-app-insights/Edit.PNG#lightbox)

1. In **Advanced Settings**, clear the **Enable AppInsights diagnostics** check box. 

   [![Cleared check box for enabling diagnostics](media/how-to-enable-app-insights/uncheck.png)](./media/how-to-enable-app-insights/uncheck.png#lightbox)

1. Select **Update** at the bottom of the screen to apply the changes. 

## Enable and disable from the SDK

### Update a deployed service
1. Identify the service in your workspace. The value for `ws` is the name of your workspace.

    ```python
    aks_service= Webservice(ws, "my-service-name")
    ```
2. Update your service and enable Application Insights. 

    ```python
    aks_service.update(enable_app_insights=True)
    ```

### Log custom traces in your service
If you want to log custom traces, follow the [standard deployment process for AKS](how-to-deploy-to-aks.md). Then:

1. Update the scoring file by adding print statements.
    
    ```python
    print ("model initialized" + time.strftime("%H:%M:%S"))
    ```

2. Update the AKS configuration.
    
    ```python
    aks_config = AksWebservice.deploy_configuration(enable_app_insights=True)
    ```

3. [Build the image and deploy it](how-to-deploy-to-aks.md).  

### Disable tracking in Python

To disable Application Insights, use the following code:

```python 
## replace <service_name> with the name of the web service
<service_name>.update(enable_app_insights=False)
```
	

## Evaluate data
Your service's data is stored in your Application Insights account, within the same resource group as your Azure Machine Learning service.
To view it:
1. Go to your resource group in the [Azure portal](https://portal.azure.com) and browse to your Application Insights resource. 
2. The **Overview** tab shows you the basic set of metrics for your service.

   [![Overview](media/how-to-enable-app-insights/overview.png)](./media/how-to-enable-app-insights/overview.png#lightbox)

3. To look into your custom traces, select **Analytics**.
4. In the schema section, select **Traces**. Then select **Run** to run your query. Data should appear in a table format and should map to your custom calls in your scoring file. 

   [![Custom traces](media/how-to-enable-app-insights/logs.png)](./media/how-to-enable-app-insights/logs.png#lightbox)

To learn more about how to use Application Insights, see [What is Application Insights?](../../application-insights/app-insights-overview.md).
    

## Example notebook

The [00.Getting Started/13.enable-app-insights-in-production-service.ipynb](https://github.com/Azure/MachineLearningNotebooks/tree/master/01.getting-started/13.enable-app-insights) notebook demonstrates concepts in this article.  Get this notebook:
 
[!INCLUDE [aml-clone-in-azure-notebook](../../../includes/aml-clone-for-examples.md)]

## Next steps
You can also collect data on your models in production. Read the article  [Collect data for models in production](how-to-enable-data-collection.md). 
