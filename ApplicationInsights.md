# Enable Opentelemetry with Azure Application Insights
Azure Monitor is an umbrella service that enables monitoring in Azure resources, other cloud and on prem. Application insights under azure monitoring targets application performance monitoring (APM), providing collection of metrics, traces and logs at central place.

Application insights is based on opentelemetry for instrumentation and Log Analytics workspace for storage, analytics and alerting. For further deep dive, access the [documentation](https://learn.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview)

## Prerequisites
1. An existing Azure application insights instance.  Follow the instructions [here](https://learn.microsoft.com/en-us/azure/azure-monitor/app/create-workspace-resource?tabs=portal#create-an-application-insights-resource/) to create a new one
2. An ASPNET.CORE application

## OTEL instrumentation
OTEL supports both Automatic and Manual instrumentaion. Automatic instrumentation involves either running agents alogside the application or adding libraries which automatically collect telemetry without modifying source code.

When using Application services, APIM and other services that allow enabling application insights, you are basically configuring automatic instrumentation for your application, allowing the collection of metrics, logs and traces.

Automatic instrumentation is beneficial where default telemtry data suffices. In cases where you require custom or application unique telemetry, consider using manual instrumentation which provides more flexibility.

### Instrumenting APSNET.CORE application
1. Add the azure opentelemetry NUGET package
```
dotnet add package Azure.Monitor.OpenTelemetry.AspNetCore
```
2. Update your Program.cs file to use the Azure open telemetry package
```
using Azure.Monitor.OpenTelemetry.AspNetCore;

var builder = WebApplication.CreateBuilder(args);

// Add OpenTelemetry and configure it to use Azure Monitor.
builder.Services.AddOpenTelemetry().UseAzureMonitor();

var app = builder.Build();

app.Run();
```
3. Update your launchsettings file to include the application insights instrumentation key. Replace with your connection string which you can find in the Application Insights overview page. 
```
{
  "AzureMonitor": {
      "ConnectionString": "<Your Connection String>"
  }
}
```
4. Run the application and query successful and non existing endpoints to simulate failure.

   ![Successful Request](/screenshots/appinsights/swagger.png)
   
You can use an application of any language available in this [step by step guide](https://learn.microsoft.com/en-us/azure/azure-monitor/app/opentelemetry-enable?tabs=aspnetcore#enable-opentelemetry-with-application-insights)

## Monitoring Data from Application Insights
Visit the applications insights overview and verify activity on the dashboards.

![Application Insights Dashboard](/screenshots/appinsights/appinsights.png)

The transaction search feature under investigate provides tracing telemetry as well

![Application Insights Traces](/screenshots/appinsights/tracing.png)

Application Insights provide more capabilities for interacting with metrics, traces and logs. Dive deep into these features including Live metrics, application map, availaility, performance, query logs and transaction search for traces.
[Interact with Application Insights](https://learn.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview)


