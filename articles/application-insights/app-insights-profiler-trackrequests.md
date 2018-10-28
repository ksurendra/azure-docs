---
title: Write code to track requests with Azure Application Insights | Microsoft Docs
description: Write code to track requests with Application Insights so you can get profiles for your reqeusts
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm

ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.reviewer: cawa
ms.date: 08/06/2018
ms.author: mbullwin

---
# Write Code to track requests with Application Insights

To see profiles for your application on the Performance page, Application Insights needs to be tracking requests for your application. Application Insights can automatically track requests for applications that are built on frameworks that are already instrumented, like ASP.net and ASP.Net Core. But for other applications, like Azure Cloud Service worker roles and Service Fabric stateless APIs, your need to write code to tell Application Insights where your requests begin and end. Once you've written this code, requests telemetry will be sent to Application Insights and you'll see the telemetry on the Performance page and profiles will be collected for those requests. 

Here are the steps you need to take to manually track requests:


  1. Early in the application lifetime, add the following code:  

        ```csharp
        using Microsoft.ApplicationInsights.Extensibility;
        ...
        // Replace with your own Application Insights instrumentation key.
        TelemetryConfiguration.Active.InstrumentationKey = "00000000-0000-0000-0000-000000000000";
        ```
      For more information about this global instrumentation key configuration, see [Use Service Fabric with Application Insights](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started/blob/dev/appinsights/ApplicationInsights.md).  

  1. For any piece of code that you want to instrument, add a `StartOperation<RequestTelemetry>` **USING** statement around it, as shown in the following example:

        ```csharp
        using Microsoft.ApplicationInsights;
        using Microsoft.ApplicationInsights.DataContracts;
        ...
        var client = new TelemetryClient();
        ...
        using (var operation = client.StartOperation<RequestTelemetry>("Insert_Your_Custom_Event_Unique_Name"))
        {
          // ... Code I want to profile.
        }
        ```

        Calling `StartOperation<RequestTelemetry>` within another `StartOperation<RequestTelemetry>` scope isn't supported. You can use `StartOperation<DependencyTelemetry>` in the nested scope instead. For example:  
        
        ```csharp
        using (var getDetailsOperation = client.StartOperation<RequestTelemetry>("GetProductDetails"))
        {
        try
        {
          ProductDetail details = new ProductDetail() { Id = productId };
          getDetailsOperation.Telemetry.Properties["ProductId"] = productId.ToString();
        
          // By using DependencyTelemetry, 'GetProductPrice' is correctly linked as part of the 'GetProductDetails' request.
          using (var getPriceOperation = client.StartOperation<DependencyTelemetry>("GetProductPrice"))
          {
              double price = await _priceDataBase.GetAsync(productId);
              if (IsTooCheap(price))
              {
                  throw new PriceTooLowException(productId);
              }
              details.Price = price;
          }
        
          // Similarly, note how 'GetProductReviews' doesn't establish another RequestTelemetry.
          using (var getReviewsOperation = client.StartOperation<DependencyTelemetry>("GetProductReviews"))
          {
              details.Reviews = await _reviewDataBase.GetAsync(productId);
          }
        
          getDetailsOperation.Telemetry.Success = true;
          return details;
        }
        catch(Exception ex)
        {
          getDetailsOperation.Telemetry.Success = false;
        
          // This exception gets linked to the 'GetProductDetails' request telemetry.
          client.TrackException(ex);
          throw;
        }
        }
        ```
