# Setup an Azure Web Job

## Create a console app

Console app can be used as a web job. There are a couple things need to be set in the app.

### Install Azure.WebJobs

There are a few NuGet packages need to be installed to have the web job to work properly:

1. MicroSoft.Azure.WebJobs
2. MicroSoft.Azure.WebJobs.Extensions
3. MicroSoft.Azure.WebJobs.Logging.ApplicationInsights

The third is only needed when you wish to do logging in Application Insights.

### Host Builder

In `Program.cs`, create a `HostBuilder`.

```c#
var builder = new HostBuilder()
    .ConfigureHostConfiguration(configHost =>
    {
        // Host configuration, e.g. add environment variables, etc.
        configHost.AddEnvironmentVariables(prefix: "ASPNETCORE_");
    })
    // Here is to configure the web job
    .ConfigureWebJobs(webJobsBuilder =>
    {
        // Must add Azure storage core services for .Net Core 3
        webJobsBuilder.AddAzureStorageCoreServices();
    })
    .ConfigureAppConfiguration((builderContext, config) =>
    {
        // Load appsettings.json
        var environment = builderContext.HostingEnvironment;
        config.AddJsonFile("appsettings.json", optional: false, reloadOnChange: true);
        config.AddJsonFile($"appsettings.{environment.EnvironmentName}.json",
            optional: true,
            reloadOnChange: true);

        // Configure user secret if needed
        if (environment.IsDevelopment())
        {
            config.AddUserSecrets<SecretClass>();
        }
    })
    .ConfigureLogging((builderContext, loggingBuilder) => 
    {
        // Configure loggings
        var config = builderContext.Configuration.GetSection("Logging");
        loggingBuilder.AddConfiguration(config);
        loggingBuilder.AddApplicationInsightsWebJobs();
        loggingBuilder.AddDebug();
        loggingBuilder.AddConsole();
    })
    .ConfigureServices((builderContext, services) =>
    {
        // Configure services, e.g. connection string, IOptions, dependency injection, etc.
        services.Configure<SecretClass>(
            builderContext.Configuration.GetSection(nameof(SecretClass)));

        services.AddDbContext<DbContext>(options =>
            options.UseSqlServer(builderContext.Configuration.GetConnectionString("Database")));
    });
```
