```yaml
    # ASP.NET Core
# Build and test ASP.NET Core projects targeting .NET Core.
# Add steps that run tests, create a NuGet package, deploy, and more:
# https://docs.microsoft.com/azure/devops/pipelines/languages/dotnet-core

trigger:
  branches:
    include:
    - '*'

pool:
  vmImage: 'ubuntu-latest'

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