# Building Pipeline (yaml) of a .Net Core Web Application with React

The following code is the yaml file of a building pipeline for a .Net core web application with React as front-end.

## Triger the build pipeline

Pipeline is triggered for all branches.

```yaml
trigger:
  branches:
    include:
    - '*'
```

## Define pool

The pool keyword specifies which pool to use for a job of the pipeline. A pool specification also holds information about the job's strategy for running. You can specify a pool at the pipeline, stage, or job level. The pool specified at the lowest level of the hierarchy is used to run the job.

### Syntax

```yaml
pool:
  name: string  # name of the pool to run this job in
  demands: string | [ string ]  # see the following "Demands" topic
  vmImage: string # name of the VM image you want to use; valid only in the Microsoft-hosted pool
```

### Example

```yaml
pool:
  vmImage: 'ubuntu-latest'
```

## Define variables

Variables give you a convenient way to get key bits of data into various parts of the pipeline. The most common use of variables is to define a value that you can then use in your pipeline. All variables are stored as strings and are mutable. The value of a variable can change from run to run or job to job of your pipeline.

The following example is to define a `BuildConfiguration` variable, and set it to 'Release'.

```yaml
variables:
  buildConfiguration: 'Release'
```

## Steps

```yaml
steps:
```

The following are the steps of the pipeline.

1. Restore NuGet packages
2. Build solution
3. Install npm packages for React
4. Build React project
5. Run React tests
6. Run DotNet tests
7. Publish web
8. Update data migration
9. Publish artifact

### Restore NuGet packages

There are two ways of restoring Nuget packages, either by using NuGet Command or using DotNet command. There is one saying that NuGet command shouldn't be used in .Net Core pipelines, because DotNet Build will impicitly run restore. However, in one of my other project I encountered an issue that package restore occasionally failed in DotNet Build step, so I personally prefer to have a separate step for restoring the packages with either ways.

#### Example of NuGet command

```yaml
- task: NuGetCommand@2
  displayName: 'Restore Nuget Packages'
  inputs:
    command: 'restore'
    restoreSolution: '**/*.sln'
    feedsToUse: 'select'
```

#### Example of DotNet command

```yaml
- taks: DotNetCoreCLI@2
  displayName: 'dotnet restore'
  inputs:
    command: 'restore'
    restoreSolution: '**/*.sln'
    feedsToUse: 'select'
```

### Build solution

```yaml
- task: DotNetCoreCLI@2
  displayName: 'dotnet build'
  inputs:
    projects: |
     **/*.csproj
    arguments: '--configuration $(BuildConfiguration)'
```

### Install npm packages for React

```yaml
- task: Npm@1
  displayName: 'npm install'
  inputs:
    command: 'install'
    workingDir: '[PATH-OF-THE-APPLICATION]'
```

### Build React project

```yaml
- task: Npm@1
  displayName: 'npm run build'
  inputs:
    command: 'custom'
    workingDir: '[PATH-OF-THE-APPLICATION]'
    customCommand: 'run build'
```

### Run React tests

```yaml
- task: Npm@1
  displayName: 'npm test'
  inputs:
    command: custom
    workingDir: '[PATH-OF-THE-APPLICATION]'
    verbose: false
    customCommand: test
```

### Run DotNet tests

```yaml
- task: DotNetCoreCLI@2
  displayName: 'DotNet Test'
  inputs:
    command: test
    workingDirectory: '[PATH-OF-THE-SOLUTION]'
    projects: '$(Parameters.TestProjects)'
    arguments: '--configuration $(BuildConfiguration)'
```

### Publish web

This task will publish both the web api and the React app (they're in the same project in Visual Studio).

```yaml
- task: DotNetCoreCLI@2
  displayName: 'Publish web'
  inputs:
    command: 'publish'
    workingDirectory: '[PATH-OF-THE-SOLUTION]'
    publishWebProjects: false
    projects: |
     **/[APPLICATION.csproj]
    arguments: '--configuration $(BuildConfiguration) --output $(build.artifactstagingdirectory)'
```

### Update data migration

```yaml
- task: DotNetCoreCLI@2
  displayName: 'Publish DbUp'
  inputs:
    command: 'publish'
    workingDirectory: '[PATH-OF-THE-SOLUTION]'
    publishWebProjects: false
    projects: |
     **/[APPLICATION.csproj]
    arguments: '--configuration $(BuildConfiguration) --output $(build.artifactstagingdirectory)'
```

### Publish artifact

```yaml
- task: PublishBuildArtifacts@1
  displayName: 'Publish Artifact'
  inputs:
    PathtoPublish: '$(Build.ArtifactStagingDirectory)'
    ArtifactName: 'drop'
    publishLocation: 'Container'
```