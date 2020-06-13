# Setup an Azure Web Job

## Create a console app

Console app can be used as a web job. There are a couple things need to be set in the app.

### Install Azure.WebJobs

There are a few NuGet packages need to be installed to have the web job to work properly:

1. MicroSoft.Azure.WebJobs
2. MicroSoft.Azure.WebJobs.Extensions
3. MicroSoft.Azure.WebJobs.Logging.ApplicationInsights

The third is only needed when you wish to do logging in Application Insights.
