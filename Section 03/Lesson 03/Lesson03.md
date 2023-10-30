
Update the Pipeline to use Yarn
```
steps:
- task: NodeTool@0
  inputs:
    versionSpec: '18.x'
  displayName: 'Install Node.js'

- script: |
    npm i -g yarn
  displayName: 'Install yarn'
  
- script: |
    yarn install --frozen-lockfile
    yarn build
  displayName: 'yarn install and build'
```

