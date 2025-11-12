# JavaScript TDD Lab 001: Basic Setup with Jest and jsdom

**Difficulty Level:** 2
**Technologies:** JavaScript (Node.js v22.20.0), Jest, jsdom
**Domain:** Basic JS Functions
**Number of Passes:** 1

***

## Purpose

This lab successfully demonstrates TDD setup for JavaScript projects using:

1. **npm**: Package management and script execution
2. **Jest**: Testing framework with built-in assertion library
3. **jsdom**: Browser-like environment for testing
4. **TDD Workflow**: Red-Green-Refactor cycle

***

## Prerequisites

Ensure you have the following installed:

- **Node.js**: v22.20.0
- **npm**: 10.9.3
- **Visual Studio Code**: Latest version

***

## PASS 1: RED-GREEN-REFACTOR (Complete Setup)

### RED Phase - Create Project Structure

#### Step 1: Create Project Directory

Open VSCode and create your project folder structure:

```bash
# Open terminal in VSCode (Ctrl+` or View > Terminal)
# Create main project directory
mkdir tdd-labs-js
cd tdd-labs-js
```

```bash
# Create folder for this lab
mkdir basic-js-tdd
```


#### Step 2: Initialize npm

Initialize npm to create package.json:

```bash
npm init -y
```

This creates a default `package.json` file that looks like this:[^1][^2]

```json
{
  "name": "tdd-labs-js",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```


#### Step 3: Install Jest and jsdom

Install both packages as development dependencies:[^3][^1]

```bash
npm install --save-dev jest jsdom
```

This will update your `package.json` with:

```json
"devDependencies": {
  "jest": "^29.x.x",
  "jsdom": "^24.x.x"
}
```

#### Step 3.1: Install JSdom test environment

Install both packages as development dependencies:[^3][^1]

```bash
npm install --save-dev jest-environment-jsdom
```

This will update your `package.json` with:

```json
"devDependencies": {
  "jest": "^29.x.x",
  "jsdom": "^24.x.x",
  "jest-environment-jsdom": "^30.2.0",
}
```


#### Step 4: Configure npm Test Script

**CRITICAL FIX**: Edit your `package.json` and update the scripts section:[^2][^4][^1]

**File:** `package.json`

```json
{
  "name": "tdd-labs-js",
  "version": "1.0.0",
  "description": "TDD Labs for JavaScript Testing",
  "main": "index.js",
  "scripts": {
    "test": "jest"
  },
  "keywords": ["tdd", "jest", "testing"],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "jest": "^29.7.0",
    "jsdom": "^24.0.0"
  }
}
```

**Key Change**: Replace `"test": "echo \"Error: no test specified\" && exit 1"` with `"test": "jest"`.[^5][^1][^2]

#### Step 5: Create Jest Configuration

Create a Jest configuration file to use jsdom as the test environment:[^6][^3]

**File:** `jest.config.js` (in root directory)

```javascript
// Jest configuration for browser-like testing environment
module.exports = {
  // Use jsdom to simulate browser environment
  testEnvironment: "jsdom",
  
  // Look for test files in these directories
  roots: ["<rootDir>/basic-js-tdd"],
  
  // Test file patterns
  testMatch: ["**/__tests__/**/*.js", "**/?(*.)+(spec|test).js"],
  
  // Coverage directory
  coverageDirectory: "coverage",
  
  // Automatically clear mock calls between tests
  clearMocks: true
};
```

## Additional npm Scripts 

You can add more useful scripts to your `package.json`:[^9][^2]

```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "test:verbose": "jest --verbose"
  }
}
```



#### Step 6: Create Project Folder Structure

Create the folder structure for this lab:

```bash
# Create source and test directories
mkdir -p basic-js-tdd/src
mkdir -p basic-js-tdd/tests
```

Your project structure should now look like this:

```
tdd-labs-js/
  ├── basic-js-tdd/
  │    ├── src/
  │    └── tests/
  ├── jest.config.js
  ├── package.json
  └── node_modules/
```


***

### GREEN Phase - Write First Test (TDD: Red)

Following TDD principles, we write the test BEFORE the implementation.[^7][^8]

**File:** `basic-js-tdd/tests/sum.test.js`

```javascript
// Import the sum function that we will create
// This will initially cause an error - that's expected in TDD!
const sum = require('../src/sum');

// Test suite for sum function
describe('Sum Function', () => {
  
  // Test case 1: Basic addition
  test('adds 1 + 2 to equal 3', () => {
    // Arrange: Set up test data
    const a = 1;
    const b = 2;
    const expected = 3;
    
    // Act: Call the function
    const result = sum(a, b);
    
    // Assert: Verify the result
    expect(result).toBe(expected);
  });
  
  // Test case 2: Adding negative numbers
  test('adds -1 + -1 to equal -2', () => {
    expect(sum(-1, -1)).toBe(-2);
  });
  
  // Test case 3: Adding zero
  test('adds 5 + 0 to equal 5', () => {
    expect(sum(5, 0)).toBe(5);
  });
});
```

**Expected Result**: Running `npm test` will fail because the `sum.js` file doesn't exist yet. This is the **RED** phase of TDD.[^8][^7]

***

### GREEN Phase - Make Test Pass (TDD: Green)

Now create the implementation to make the test pass:[^7][^8]

**File:** `basic-js-tdd/src/sum.js`

```javascript
// Simple addition function
// Takes two numbers and returns their sum
function sum(a, b) {
  return a + b;
}

// Export the function so it can be tested
module.exports = sum;
```

**Run the test:**

```bash
npm test
```

**Expected Result**: All tests pass ✅[^1][^8][^7]

You should see output similar to:

```
PASS  basic-js-tdd/tests/sum.test.js
  Sum Function
    ✓ adds 1 + 2 to equal 3 (2 ms)
    ✓ adds -1 + -1 to equal -2 (1 ms)
    ✓ adds 5 + 0 to equal 5 (1 ms)

Test Suites: 1 passed, 1 total
Tests:       3 passed, 3 total
```


***

### REFACTOR Phase - Improve Code Quality

Let's refactor the code to use modern JavaScript syntax:[^8][^7]

**File:** `basic-js-tdd/src/sum.js` (Refactored)

```javascript
// Refactored to use arrow function and JSDoc comments

/**
 * Adds two numbers together
 * @param {number} a - First number
 * @param {number} b - Second number
 * @returns {number} Sum of a and b
 */
const sum = (a, b) => a + b;

// Export the function
module.exports = sum;
```

**Run tests again to ensure refactoring didn't break anything:**

```bash
npm test
```

**Expected Result**: All tests still pass ✅[^7][^8]

***


## Additional npm Scripts 

You can add more useful scripts to your `package.json`:[^9][^2]

```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "test:verbose": "jest --verbose"
  }
}
```

**Usage examples:**

- `npm test` - Run all tests once
- `npm run test:watch` - Run tests in watch mode (re-runs on file changes)
- `npm run test:coverage` - Generate code coverage report
- `npm run test:verbose` - Show detailed test output

***

## Project Structure for Future Labs

This project is designed to accommodate future testing labs. Here's how to organize:

```
tdd-labs-js/                    # Main project (current)
  ├── basic-js-tdd/             # Lab 001 (current lab)
  │    ├── src/
  │    │    └── sum.js
  │    └── tests/
  │         └── sum.test.js
  ├── advanced-js-tdd/          # Lab 002 (future)
  ├── dom-manipulation-tdd/     # Lab 003 (future)
  ├── jest.config.js
  ├── package.json
  └── node_modules/

react-js-tdd/                   # Future React project
  ├── component-testing/
  ├── jest.config.js
  └── package.json

vue-js-tdd/                     # Future Vue project
  ├── component-testing/
  ├── jest.config.js
  └── package.json

angular-js-tdd/                 # Future Angular project
  ├── component-testing/
  ├── jest.config.js
  └── package.json
```


***
