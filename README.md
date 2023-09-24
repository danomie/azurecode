# azurecode
trigger:
- main
- develop
- release/staging

pr:
- main

pool:
  vmImage: 'ubuntu-latest'

stages:
- stage: Build
  displayName: 'Build stage'
  jobs:
  - job: Build
    displayName: 'Build job'
    steps:
    - task: NodeTool@0
      inputs:
        versionSpec: '16.x'
      displayName: 'Install Node.js'

    - script: |
        npm install
        zip -r myExpressApp.zip .
      displayName: 'Install and Build'

    - publish: '$(System.DefaultWorkingDirectory)/myExpressApp.zip'
      artifact: 'myExpressApp'
      displayName: 'Publish Artifact'

- stage: DeployDev
  displayName: 'Deploy to Dev'
  dependsOn: Build
  condition: eq(variables['Build.SourceBranch'], 'refs/heads/develop')
  jobs:
  - job: DeployDev
    displayName: 'Deploy to Dev job'
    steps:
    - task: DownloadBuildArtifacts@1
      inputs:
          buildType: 'current'
          downloadType: 'single'
          artifactName: 'myExpressApp'
          downloadPath: '$(System.DefaultWorkingDirectory)'    
    - task: AzureWebApp@1
      inputs:
           azureSubscription: theITern-subscription (5135fe87-f70d-43dc-a7d5-
           appName: '$(devappname)'
           package: '$(System.DefaultWorkingDirectory)/**/*.zip'
           runtimeStack: 'NODE|16-lts'
           startUpCommand: 'npm start'
      displayName: 'Deploy to Dev Environment'

- stage: DeployStaging
  displayName: 'Deploy to Staging'
  dependsOn: Build
  condition: eq(variables['Build.SourceBranch'], 'refs/heads/release/staging')
  jobs:
  - job: DeployStaging
    displayName: 'Deploy to Staging job'
    steps:
    - task: DownloadBuildArtifacts@1
      inputs:
         buildType: 'current'
         downloadType: 'single'
         artifactName: 'myExpressApp'
         downloadPath: '$(System.DefaultWorkingDirectory)'    
    - task: AzureWebApp@1
      inputs:
        azureSubscription: theITern-subscription (5135fe87-f70d-43dc-a7d5-
        appName: '$(stagingappname)'
        package: '$(System.DefaultWorkingDirectory)/**/*.zip'
        runtimeStack: 'NODE|16-lts'
        startUpCommand: 'npm start'
      displayName: 'Deploy to Staging Environment'

- stage: DeployProd
  displayName: 'Deploy to Production'
  dependsOn: Build
  condition: or(eq(variables['Build.SourceBranch'], 'refs/heads/main'), eq(variables['Build.SourceBranch'], 'refs/heads/master'))
  jobs:
  - job: DeployProd
    displayName: 'Deploy to Production job'
    steps:
    - task: DownloadBuildArtifacts@1
      inputs:
         buildType: 'current'
         downloadType: 'single'
         artifactName: 'myExpressApp'
         downloadPath: '$(System.DefaultWorkingDirectory)'    
    - task: AzureWebApp@1
      inputs:
        azureSubscription: theITern-subscription (5135fe87-f70d-43dc-a7d5-
        appName: '$(prodappname)'
        package: '$(System.DefaultWorkingDirectory)/**/*.zip'
        runtimeStack: 'NODE|16-lts'
        startUpCommand: 'npm start'
      displayName: 'Deploy to Production Environment'
 
