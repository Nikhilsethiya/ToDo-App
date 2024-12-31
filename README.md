# A sample todo app in react

Imported a react application from https://github.com/piyushsachdeva/todoapp-docker

Build Pipeline: -
  
trigger:
  branches:
    include:
      - main
pr:
  branches:
    include:
      - main
      
resources:
- repo: self

variables:
  dockerRegistryServiceConnection: '--'
  imageRepository: 'todoappdocker'
  containerRegistry: 'nikhilcontainer.azurecr.io'
  dockerfilePath: '$(Build.SourcesDirectory)/Dockerfile'
  tag: '$(Build.BuildId)'
  vmImageName: 'my-agent'

stages:
- stage: Build
  displayName: Build and push stage
  jobs:
  - job: Build
    displayName: Build
    pool:
      name: 'my-agent'
    steps:
    - task: AzureCLI@2
      inputs:
        azureSubscription: 'Free Trial'
        scriptType: 'bash'
        scriptLocation: 'inlineScript' 
        inlineScript: 'az acr login --name=$(containerRegistry)'
 
    - task: Docker@2
      displayName: Build and push an image to container registry
      inputs:
        command: buildAndPush
        repository: $(imageRepository)
        dockerfile: $(dockerfilePath)
        containerRegistry: $(dockerRegistryServiceConnection)
        tags: |
          $(tag)

- stage: Deploy
  displayName: Deploy Stage
  dependsOn: Build
  condition: succeeded()
  jobs:
  - deployment: Deploy
    environment: Production
    pool:
      name: $(vmImageName)
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureCLI@2
            inputs:
              azureSubscription: 'Free Trial'
              scriptType: 'bash'
              scriptLocation: 'inlineScript'
              inlineScript: |
                az container create \
                --name nikhil-app \
                --resource-group app-rgGroup \
                --image $(containerRegistry)/$(imageRepository):$(tag) \
                --os-type Linux \
                --cpu 1 \
                --memory 1.5 \
                --registry-login-server $(containerRegistry) \
                --registry-username $(username) \
                --registry-password $(password) \
                --dns-name-label nikhil-app

    ![image](https://github.com/user-attachments/assets/c38507c5-3f9e-4fba-b3eb-3e7ab52b3f4b)

