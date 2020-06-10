# Building Pipeline (yaml) of a .Net Core Web Application with React 

The following code is the yaml file of a building pipeline for a .Net core web application with React as front-end.

### Triger the build pipeline
Pipeline is triggered for all branches.
```yaml
trigger:
  branches:
    include:
    - '*'
```
### Define pool
The pool keyword specifies which pool to use for a job of the pipeline. A pool specification also holds information 
about the job's strategy for running. You can specify a pool at the pipeline, stage, or job level. The pool specified 
at the lowest level of the hierarchy is used to run the job.

##### Syntax:

```yaml
pool:
  name: string  # name of the pool to run this job in
  demands: string | [ string ]  # see the following "Demands" topic
  vmImage: string # name of the VM image you want to use; valid only in the Microsoft-hosted pool
```

##### Example:

```yaml
pool:
  vmImage: 'ubuntu-latest'
```

```yaml
variables:
  buildConfiguration: 'Release'

steps:

- task: NuGetToolInstaller@1
  inputs:
    versionSpec: 
    checkLatest: true

- task: NuGetCommand@2
  displayName: 'Restore Nuget Packages'
  inputs:
    command: 'restore'
    restoreSolution: '**/*.sln'
    feedsToUse: 'select'

- task: DotNetCoreCLI@2
  displayName: 'dotnet build'
  inputs:
    projects: |
     **/*.csproj
    arguments: '--configuration $(BuildConfiguration)'

- task: Npm@1
  displayName: 'npm install'
  inputs:
    command: 'install'
    workingDir: '[PATH-OF-THE-APPLICATION]'

- task: Npm@1
  displayName: 'npm run build'
  inputs:
    command: 'custom'
    workingDir: '[PATH-OF-THE-APPLICATION]'
    customCommand: 'run build'

- task: Npm@1
  displayName: 'npm test'
  inputs:
    command: custom
    workingDir: '[PATH-OF-THE-APPLICATION]'
    verbose: false
    customCommand: test

- task: DotNetCoreCLI@2
  displayName: 'DotNet Test'
  inputs:
    command: test
    workingDirectory: '[PATH-OF-THE-SOLUTION]'
    projects: '$(Parameters.TestProjects)'
    arguments: '--configuration $(BuildConfiguration)'

- task: DotNetCoreCLI@2
  displayName: 'Publish web'
  inputs:
    command: 'publish'
    workingDirectory: '[PATH-OF-THE-SOLUTION]'
    publishWebProjects: false
    projects: |
     **/[APPLICATION.csproj]
    arguments: '--configuration $(BuildConfiguration) --output $(build.artifactstagingdirectory)'

- task: DotNetCoreCLI@2
  displayName: 'Publish DbUp'
  inputs:
    command: 'publish'
    workingDirectory: '[PATH-OF-THE-SOLUTION]'
    publishWebProjects: false
    projects: |
     **/[APPLICATION.csproj]
    arguments: '--configuration $(BuildConfiguration) --output $(build.artifactstagingdirectory)'

- task: PublishBuildArtifacts@1
  displayName: 'Publish Artifact'
  inputs:
    PathtoPublish: '$(Build.ArtifactStagingDirectory)'
    ArtifactName: 'drop'
    publishLocation: 'Container'
```