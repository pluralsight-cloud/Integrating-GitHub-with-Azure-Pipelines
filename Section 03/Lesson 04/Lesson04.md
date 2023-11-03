Install Jest packages:
```
yarn add --dev jest @testing-library/react @testing-library/jest-dom @testing-library/user-event jest-environment-jsdom @babel/preset-env @babel/preset-react jest-junit
```

Create jest.config.js:
```
const nextJest = require('next/jest');
 
const createJestConfig = nextJest({
  // Provide the path to your Next.js app to load next.config.js and .env files in your test environment
  dir: './',
})
 
// Add any custom config to be passed to Jest
/** @type {import('jest').Config} */
const config = {
  // Add more setup options before each test is run
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
  testEnvironment: 'jest-environment-jsdom',
}
 
// createJestConfig is exported this way to ensure that next/jest can load the Next.js config which is async
module.exports = createJestConfig(config)
```

Create jest.setup.js:
```
import '@testing-library/jest-dom'
```

Edit package.json:
```
"test": "jest --ci --reporters=default --reporters=jest-junit"
```

Add after devDependencies:
```
 "jest-junit": {
    "output": "output/coverage/junit/junit.xml",
    "usePathForSuiteName": "true"
  }
```

Create __tests__/Home.test.jsx:
```
import { render, screen } from '@testing-library/react'
import Home from '@/pages/index'

describe('Home', () => {
   it('should have About text', () => {
        render(<Home />);

        const getElm = screen.getByText('About');

        expect(getElm).toBeInTheDocument();
    });

    it('should containe the text "Lorem ipsum"', () => {
        render(<Home />);

        const getElm = screen.getByText(/Welcome to Alaph Corp/i);

        expect(getElm).toBeInTheDocument();
    });
});
```

Create __tests__/About.test.jsx:
```
import { render, screen } from '@testing-library/react'
import About from '@/pages/about'

describe('About', () => {

    it('should containe the text "About"', () => {
        render(<About />);

        const getElm = screen.getByText(/About/i);

        expect(getElm).toBeInTheDocument();
    });

    it('should containe the text "Back to the homse page"', () => {
        render(<About />);

        const getElm = screen.getByText(/Back to the home page/i);

        expect(getElm).toBeInTheDocument();
    });

    it('should containe the text "Lorem ipsum"', () => {
        render(<About />);

        const getElm = screen.getByText(/Lorem ipsum/i);

        expect(getElm).toBeInTheDocument();
    });
});
```

Update Pipeline steps to include tests:
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
    yarn install
  displayName: 'Install packages'

- script: |
    yarn test
  displayName: 'Test code'

- task: PublishTestResults@2
  displayName: 'Publish Test Results junit.xml'
  inputs:
    testResultsFormat: 'JUnit'
    testResultsFiles: junit.xml

- script: |
    rm -rf node_modules
  displayName: 'Remove packages'

- script: |
    yarn install --production --frozen-lockfile
    yarn build
  displayName: 'yarn install and build'
```