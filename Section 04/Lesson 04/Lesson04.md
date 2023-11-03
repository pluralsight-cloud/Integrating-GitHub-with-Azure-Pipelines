```
# Deploy to Azure Kubernetes Service
# Build and push image to Azure Container Registry; Deploy to Azure Kubernetes Service
# https://docs.microsoft.com/azure/devops/pipelines/languages/docker

trigger:
- main

resources:
- repo: self

variables:

  # Container registry service connection established during pipeline creation
  dockerRegistryServiceConnection: '556a43ca-ccac-4645-aa79-8295817104d1'
  imageRepository: 'heliosproof'
  containerRegistry: 'rivethead42acr.azurecr.io'
  dockerfilePath: '**/Dockerfile'
  tag: '$(Build.BuildId)'
  imagePullSecret: 'rivethead42acr2493797d-auth'

  # Agent VM image name
  vmImageName: 'ubuntu-latest'


stages:
- stage: Test
  displayName: Test stage
  jobs:
  - job: Test
    displayName: Test
    pool:
      vmImage: $(vmImageName)
    steps:
    - task: NodeTool@0
    inputs:
        versionSpec: '18.x'
    displayName: 'Install Node.js'

    - script: |
        npm i -g yarn
    displayName: 'Install yarn'
    
    - script: |
        cd code/ && yarn install
    displayName: 'Install packages'

    - script: |
        cd code/ && yarn test
    displayName: 'Test code'

    - task: PublishTestResults@2
    displayName: 'Publish Test Results junit.xml'
    inputs:
        testResultsFormat: 'JUnit'
        testResultsFiles: junit.xml

    - script: |
        cd code/ && rm -rf node_modules
    displayName: 'Remove packages'

- stage: Build
  displayName: Build stage
  jobs:
  - job: Build
    displayName: Build
    pool:
      vmImage: $(vmImageName)
    steps:
    - task: Docker@2
      displayName: Build and push an image to container registry
      inputs:
        command: buildAndPush
        repository: $(imageRepository)
        dockerfile: $(dockerfilePath)
        containerRegistry: $(dockerRegistryServiceConnection)
        tags: |
          $(tag)

    - upload: manifests
      artifact: manifests

- stage: Deploy
  displayName: Deploy stage
  dependsOn: Build

  jobs:
  - deployment: Deploy
    displayName: Deploy
    pool:
      vmImage: $(vmImageName)
    environment: 'rivethead42heliosproof.default'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: KubernetesManifest@0
            displayName: Create imagePullSecret
            inputs:
              action: createSecret
              secretName: $(imagePullSecret)
              dockerRegistryEndpoint: $(dockerRegistryServiceConnection)

          - task: KubernetesManifest@0
            displayName: Deploy to Kubernetes cluster
            inputs:
              action: deploy
              manifests: |
                $(Pipeline.Workspace)/manifests/deployment.yml
                $(Pipeline.Workspace)/manifests/service.yml
              imagePullSecrets: |
                $(imagePullSecret)
              containers: |
                $(containerRegistry)/$(imageRepository):$(tag)

```