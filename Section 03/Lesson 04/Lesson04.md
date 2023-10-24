```
npm i -D @testing-library/jest-dom @testing-library/react @testing-library/user-event jest jest-environment-jsdom 
npm i -D eslint-plugin-gest-dom eslint-plugin-testing-library
```

Update Pipeline steps to include tests
```
steps:
- task: NodeTool@0
  inputs:
    versionSpec: '18.x'
  displayName: 'Install Node.js'

- script: |
    npm i
    npm i -D
  displayName: 'Install packages'

- script: |
    npm run test
  displayName: 'Test code'

- script: |
    rm -rf node_modules
  displayName: 'Remove packages'

- script: |
    npm i -g yarn
  displayName: 'Install yarn'

- script: |
    yarn install --frozen-lockfile
    yarn build
  displayName: 'yarn install and build'
```