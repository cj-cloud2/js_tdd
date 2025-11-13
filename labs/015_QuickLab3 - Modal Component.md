# Lab 015: Modal Component TDD Example Using Node.js, Jest, JSDOM, & React

**Technologies:** Node.js v22.20.0, Jest, jsdom, React

## Domain: Banking Modal Component Walkthrough

Create and test a Modal component with open/close functionality, header, body, and action buttons using Test-Driven Development (TDD). This lab covers advanced Jest features including snapshot testing for UI consistency, setup and teardown hooks for test isolation, and parameterized tests for multiple scenarios.

---

## Jest Features Covered in This Lab

### 1. **Snapshot Testing**
Snapshot testing captures the rendered UI output and compares it against future test runs. This ensures UI consistency and catches unintended visual changes. In our Modal component, we'll use snapshots to verify the modal renders correctly in different states (open/closed).

**Why useful:** Detects accidental UI changes without writing extensive assertions for every element.

### 2. **Setup and Teardown Hooks (beforeEach / afterEach)**
These Jest hooks allow you to run code before and after each test. They're essential for:
- Setting up test fixtures (initial data, DOM elements)
- Cleaning up after tests (clearing state, removing components)
- Ensuring test isolation (each test starts fresh)

**Why useful:** Prevents tests from interfering with each other and reduces code duplication.

### 3. **Parameterized Tests (test.each)**
`test.each()` allows you to run the same test logic with different input values without writing multiple test functions. This is perfect for testing edge cases and multiple scenarios efficiently.

**Why useful:** Tests multiple cases with minimal code duplication and clearer test organization.

---

## Prerequisites

- You must already have Node.js, Jest, and JSDOM installed and working with a valid `jest.config.js`. 
- Jest should use jsdom as its environment and recognize your intended test folders.

### Step 1: Install Dependencies

Run below commands in `<root-folder>` (the same folder where your package.json lives):


- Install Jest and jsdom (should have been done in previous labs.)
```bash
npm install --save-dev jest jsdom
```
- Then Install This
```bash
npm install --save-dev @testing-library/jest-dom
```

- Now React (Should have been done in precious assigments)
```bash
npm install react react-dom
```

Install Babel dependencies for JSX compilation:

```bash
npm install @babel/core @babel/preset-env @babel/preset-react babel-jest --save-dev
```

Install the React Testing Library as a developer dependency:

```bash
npm install --save-dev @testing-library/react
```

### Step 2: Verify/Update Jest Configuration

Ensure your `jest.config.js` in the root folder includes the lab015 path:

```js
module.exports = {
  testEnvironment: "jsdom",
  roots: ["<rootDir>/basic-js-tdd", "<rootDir>/lab003", "<rootDir>/lab015", "<rootDir>/"],
  testMatch: ["**/test/**/*.js", "**/tests/**/*.js", "**/__tests__/**/*.js", "**/?(*.)+(spec|test).js"],
  coverageDirectory: "coverage",
  clearMocks: true,
};
```

### Step 3: Add/Verify Babel Configuration

The **babel.config.js** file should be in the **root folder** of your project, alongside your package.json:

```js
module.exports = {
  presets: ['@babel/preset-env', '@babel/preset-react'],
};
```

---

## Project Structure

Create a new subfolder for this lab:

```
<root-folder>/lab015/
    ├── src/
    │   └── Modal.js
    └── test/
        └── Modal.test.js
```

All production code will go in `src`, all tests go in `test`.

---

## Modal Component Specification

The Modal component is a reusable banking dialog box with the following features:

**Component Properties:**
- **isOpen** (boolean): Controls whether the modal is visible
- **title** (string): Modal header text
- **children** (ReactNode): Modal body content
- **onClose** (function): Callback when close button is clicked
- **onConfirm** (function): Callback when confirm button is clicked
- **confirmText** (string, optional): Text for confirm button (default: "Confirm")

**Visual Structure:**
- Header with title and close button (X)
- Body container for content
- Footer with "Cancel" and "Confirm" buttons

---

## Requirements

**Modal Visibility:**
- Positive: Modal renders when `isOpen` is true
- Negative: Modal doesn't render when `isOpen` is false

**Modal Actions:**
- Positive: Close button calls `onClose` callback
- Negative: Confirm button calls `onConfirm` callback when clicked

**Modal Content:**
- Positive: Modal displays correct title text
- Negative: Modal displays correct body content (children)

**Snapshot Testing:**
- Modal UI structure remains consistent across renders

**Parameterized Tests:**
- Test multiple button text scenarios (Confirm, Save, Submit, etc.)

---

# TDD Walkthrough (3 Passes)

For each pass, you will:
- Write fail-first tests (red phase)
- Implement just enough code to make tests pass (green phase)
- Clean/refactor if needed (refactor phase)
- Include descriptive comments in test functions

---

## PASS 1: Modal Visibility & Basic Rendering

### RED Phase - Write Failing Tests

**File:** `lab015/test/Modal.test.js`

```js
import React from 'react';
import { render, screen } from '@testing-library/react';
import Modal from '../src/Modal';
import '@testing-library/jest-dom';


// This test suite groups all modal-related tests together
describe('Modal Component - PASS 1: Visibility & Basic Rendering', () => {
  // Setup: This runs BEFORE each individual test in this describe block
  beforeEach(() => {
    // Clear any previous renders or state
    jest.clearAllMocks();
  });

  // Teardown: This runs AFTER each individual test in this describe block
  afterEach(() => {
    // Additional cleanup if needed (DOM elements are automatically cleaned)
  });

  // In this test, we are checking that the modal renders when isOpen is true
  it('should render modal when isOpen is true', () => {
    render(<Modal isOpen={true} title="Test Modal" onClose={() => {}} onConfirm={() => {}} />);
    const modalElement = screen.queryByTestId('modal-container');
    expect(modalElement).toBeInTheDocument();
  });

  // In this test, we verify the modal does NOT render when isOpen is false
  it('should not render modal when isOpen is false', () => {
    render(<Modal isOpen={false} title="Test Modal" onClose={() => {}} onConfirm={() => {}} />);
    const modalElement = screen.queryByTestId('modal-container');
    expect(modalElement).not.toBeInTheDocument();
  });

  // In this test, we check that the modal displays the correct title
  it('should display correct title in modal header', () => {
    render(<Modal isOpen={true} title="Withdraw Funds" onClose={() => {}} onConfirm={() => {}} />);
    const titleElement = screen.getByTestId('modal-title');
    expect(titleElement).toHaveTextContent('Withdraw Funds');
  });

  // In this test, we verify the modal displays its children content
  it('should display children content in modal body', () => {
    render(
      <Modal isOpen={true} title="Confirm" onClose={() => {}} onConfirm={() => {}}>
        <p>Are you sure you want to proceed?</p>
      </Modal>
    );
    expect(screen.getByText('Are you sure you want to proceed?')).toBeInTheDocument();
  });
});
```

**Run tests from the root folder using:** (Tests should fail - RED)

```bash
npm test -- .\lab015\test\Modal.js
```

Expected output: 4 failing tests - the Modal component doesn't exist yet.

### GREEN Phase - Make Tests Pass

**File:** `lab015/src/Modal.js`

```js
import React from 'react';

// Modal component for displaying dialog boxes in the banking application
// Props: isOpen (bool), title (string), children (ReactNode), onClose (func), onConfirm (func), confirmText (string, optional)
function Modal({ isOpen, title, children, onClose, onConfirm, confirmText = 'Confirm' }) {
  // If modal is not open, render nothing
  if (!isOpen) {
    return null;
  }

  // Render modal structure with header, body, and footer
  return (
    <div data-testid="modal-container" className="modal">
      {/* Modal Header */}
      <div className="modal-header">
        <h2 data-testid="modal-title">{title}</h2>
        <button data-testid="close-btn" onClick={onClose} className="close-btn">
          ×
        </button>
      </div>

      {/* Modal Body */}
      <div data-testid="modal-body" className="modal-body">
        {children}
      </div>

      {/* Modal Footer */}
      <div className="modal-footer">
        <button data-testid="cancel-btn" onClick={onClose} className="cancel-btn">
          Cancel
        </button>
        <button data-testid="confirm-btn" onClick={onConfirm} className="confirm-btn">
          {confirmText}
        </button>
      </div>
    </div>
  );
}

export default Modal;
```

**Run tests from the root folder using:** (Tests should pass - GREEN)

```bash
npm test -- ./lab015/test/Modal.test.js
```

Expected output: 4 passing tests

### Refactor

- The Modal component is clean and focused on rendering logic
- No major refactoring needed at this stage

---

## PASS 2: Modal Actions & Callbacks

### RED Phase - Add Failing Tests

**File:** `lab015/test/Modal.test.js`

```js
//**Important**: Add the following code AFTER the closing brace of the first describe block (after the first PASS 1 tests), 
//but BEFORE the end of the file. This is a new describe block for PASS 2 tests.

describe('Modal Component - PASS 2: Actions & Callbacks', () => {
  let mockOnClose;
  let mockOnConfirm;

  // Setup: Run before EACH test - create fresh mock functions
  beforeEach(() => {
    mockOnClose = jest.fn();
    mockOnConfirm = jest.fn();
  });

  // Teardown: Run after EACH test
  afterEach(() => {
    jest.clearAllMocks();
  });

  // In this test, we verify that clicking the close button triggers the onClose callback
  it('should call onClose callback when close button is clicked', () => {
    const { getByTestId } = render(
      <Modal isOpen={true} title="Test" onClose={mockOnClose} onConfirm={mockOnConfirm} />
    );
    const closeBtn = getByTestId('close-btn');
    closeBtn.click();
    expect(mockOnClose).toHaveBeenCalledTimes(1);
  });

  // In this test, we verify that clicking the cancel button also triggers onClose
  it('should call onClose callback when cancel button is clicked', () => {
    const { getByTestId } = render(
      <Modal isOpen={true} title="Test" onClose={mockOnClose} onConfirm={mockOnConfirm} />
    );
    const cancelBtn = getByTestId('cancel-btn');
    cancelBtn.click();
    expect(mockOnClose).toHaveBeenCalledTimes(1);
  });

  // In this test, we verify that clicking the confirm button triggers the onConfirm callback
  it('should call onConfirm callback when confirm button is clicked', () => {
    const { getByTestId } = render(
      <Modal isOpen={true} title="Test" onClose={mockOnClose} onConfirm={mockOnConfirm} />
    );
    const confirmBtn = getByTestId('confirm-btn');
    confirmBtn.click();
    expect(mockOnConfirm).toHaveBeenCalledTimes(1);
  });
});
```

**Run tests from the root folder using:** (Tests should fail - RED)

```bash
npm test -- ./lab015/test/Modal.test.js
```

Expected output: 3 new failing tests (PASS 1 tests should still pass)

### GREEN Phase - Make Tests Pass

No changes needed to `Modal.js` - the implementation from PASS 1 already handles callbacks correctly!

**Run tests from the root folder using:** (All tests should pass - GREEN)

```bash
npm test -- ./lab015/test/Modal.test.js
```

Expected output: 7 passing tests (4 from PASS 1 + 3 from PASS 2)

### Refactor

- Both describe blocks are well-organized with clear setup/teardown
- Mock functions are properly created and cleared
- Component logic remains clean and focused

---

## PASS 3: Snapshot Testing & Parameterized Tests

### RED Phase - Add Failing Tests

**File:** `lab015/test/Modal.test.js`

```js
//**Important**: Add the following code AFTER the closing brace of the PASS 2 describe block, but BEFORE the end of the file. This is a new describe block for PASS 3 tests.

describe('Modal Component - PASS 3: Snapshots & Parameterized Tests', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  afterEach(() => {
    // Cleanup
  });

  // In this test, we capture a snapshot of the modal's rendered structure when open
  // Snapshots help detect unintended UI changes across different test runs
  it('should match snapshot when modal is open', () => {
    const { container } = render(
      <Modal isOpen={true} title="Banking Transaction" onClose={() => {}} onConfirm={() => {}}>
        <p>Confirm transaction details</p>
      </Modal>
    );
    expect(container.firstChild).toMatchSnapshot();
  });

  // In this test, we verify the modal renders as null (no output) when closed
  // This is a snapshot of the closed state
  it('should match snapshot when modal is closed', () => {
    const { container } = render(
      <Modal isOpen={false} title="Banking Transaction" onClose={() => {}} onConfirm={() => {}} />
    );
    expect(container.firstChild).toMatchSnapshot();
  });

  // Parameterized test using test.each - tests multiple confirm button text values
  // This single test function runs 4 times with different data
  // Each row represents: [confirmText, expectedButtonText]
  describe('Confirm button text variations', () => {
    test.each([
      ['Confirm', 'Confirm'],
      ['Save', 'Save'],
      ['Submit', 'Submit'],
      ['Proceed', 'Proceed'],
    ])('should display "%s" on confirm button when confirmText prop is "%s"', (confirmText, expected) => {
      const { getByTestId } = render(
        <Modal 
          isOpen={true} 
          title="Test" 
          confirmText={confirmText}
          onClose={() => {}} 
          onConfirm={() => {}} 
        />
      );
      const confirmBtn = getByTestId('confirm-btn');
      expect(confirmBtn).toHaveTextContent(expected);
    });
  });

  // Parameterized test for modal titles
  describe('Modal title variations', () => {
    test.each([
      ['Withdraw Funds', 'Banking'],
      ['Transfer Money', 'Banking'],
      ['Update Profile', 'Account'],
      ['Delete Account', 'Account'],
    ])('should display "%s" as modal title', (titleText) => {
      const { getByTestId } = render(
        <Modal 
          isOpen={true} 
          title={titleText}
          onClose={() => {}} 
          onConfirm={() => {}} 
        />
      );
      const titleElement = getByTestId('modal-title');
      expect(titleElement).toHaveTextContent(titleText);
    });
  });
});
```

**Run tests from the root folder using:** (Tests should fail - RED)

```bash
npm test -- ./lab015/test/Modal.test.js
```

Expected output: 2 snapshot tests + 4 parameterized tests + 4 more title tests = 10 new failing tests

### GREEN Phase - Make Tests Pass

The Modal component from PASS 1 already handles all these requirements correctly. The snapshots will be created on first run, and parameterized tests will pass with the existing implementation.

**Run tests from the root folder using:** (All tests should pass - GREEN)

```bash
npm test -- ./lab015/test/Modal.test.js
```

When prompted about snapshots, press `u` to update/create them.

Expected output: 17 passing tests total
- 4 from PASS 1
- 3 from PASS 2
- 10 from PASS 3 (2 snapshots + 8 parameterized)

### Refactor

- Organize parameterized tests into logical groups with nested describe blocks
- Snapshots are automatically created in `lab015/test/__snapshots__/Modal.test.js`
- Consider adding CSS classes for styling (optional)

---

## Complete Test File Reference

**File:** `lab015/test/Modal.test.js` (Final Version)

```js
import React from 'react';
import { render, screen } from '@testing-library/react';
import Modal from '../src/Modal';
import '@testing-library/jest-dom';

// PASS 1: Visibility & Basic Rendering
describe('Modal Component - PASS 1: Visibility & Basic Rendering', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  afterEach(() => {
    // Cleanup
  });

  it('should render modal when isOpen is true', () => {
    render(<Modal isOpen={true} title="Test Modal" onClose={() => {}} onConfirm={() => {}} />);
    const modalElement = screen.queryByTestId('modal-container');
    expect(modalElement).toBeInTheDocument();
  });

  it('should not render modal when isOpen is false', () => {
    render(<Modal isOpen={false} title="Test Modal" onClose={() => {}} onConfirm={() => {}} />);
    const modalElement = screen.queryByTestId('modal-container');
    expect(modalElement).not.toBeInTheDocument();
  });

  it('should display correct title in modal header', () => {
    render(<Modal isOpen={true} title="Withdraw Funds" onClose={() => {}} onConfirm={() => {}} />);
    const titleElement = screen.getByTestId('modal-title');
    expect(titleElement).toHaveTextContent('Withdraw Funds');
  });

  it('should display children content in modal body', () => {
    render(
      <Modal isOpen={true} title="Confirm" onClose={() => {}} onConfirm={() => {}}>
        <p>Are you sure you want to proceed?</p>
      </Modal>
    );
    expect(screen.getByText('Are you sure you want to proceed?')).toBeInTheDocument();
  });
});

// PASS 2: Actions & Callbacks
describe('Modal Component - PASS 2: Actions & Callbacks', () => {
  let mockOnClose;
  let mockOnConfirm;

  beforeEach(() => {
    mockOnClose = jest.fn();
    mockOnConfirm = jest.fn();
  });

  afterEach(() => {
    jest.clearAllMocks();
  });

  it('should call onClose callback when close button is clicked', () => {
    const { getByTestId } = render(
      <Modal isOpen={true} title="Test" onClose={mockOnClose} onConfirm={mockOnConfirm} />
    );
    const closeBtn = getByTestId('close-btn');
    closeBtn.click();
    expect(mockOnClose).toHaveBeenCalledTimes(1);
  });

  it('should call onClose callback when cancel button is clicked', () => {
    const { getByTestId } = render(
      <Modal isOpen={true} title="Test" onClose={mockOnClose} onConfirm={mockOnConfirm} />
    );
    const cancelBtn = getByTestId('cancel-btn');
    cancelBtn.click();
    expect(mockOnClose).toHaveBeenCalledTimes(1);
  });

  it('should call onConfirm callback when confirm button is clicked', () => {
    const { getByTestId } = render(
      <Modal isOpen={true} title="Test" onClose={mockOnClose} onConfirm={mockOnConfirm} />
    );
    const confirmBtn = getByTestId('confirm-btn');
    confirmBtn.click();
    expect(mockOnConfirm).toHaveBeenCalledTimes(1);
  });
});

// PASS 3: Snapshots & Parameterized Tests
describe('Modal Component - PASS 3: Snapshots & Parameterized Tests', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  afterEach(() => {
    // Cleanup
  });

  it('should match snapshot when modal is open', () => {
    const { container } = render(
      <Modal isOpen={true} title="Banking Transaction" onClose={() => {}} onConfirm={() => {}}>
        <p>Confirm transaction details</p>
      </Modal>
    );
    expect(container.firstChild).toMatchSnapshot();
  });

  it('should match snapshot when modal is closed', () => {
    const { container } = render(
      <Modal isOpen={false} title="Banking Transaction" onClose={() => {}} onConfirm={() => {}} />
    );
    expect(container.firstChild).toMatchSnapshot();
  });

  describe('Confirm button text variations', () => {
    test.each([
      ['Confirm', 'Confirm'],
      ['Save', 'Save'],
      ['Submit', 'Submit'],
      ['Proceed', 'Proceed'],
    ])('should display "%s" on confirm button when confirmText prop is "%s"', (confirmText, expected) => {
      const { getByTestId } = render(
        <Modal 
          isOpen={true} 
          title="Test" 
          confirmText={confirmText}
          onClose={() => {}} 
          onConfirm={() => {}} 
        />
      );
      const confirmBtn = getByTestId('confirm-btn');
      expect(confirmBtn).toHaveTextContent(expected);
    });
  });

  describe('Modal title variations', () => {
    test.each([
      ['Withdraw Funds'],
      ['Transfer Money'],
      ['Update Profile'],
      ['Delete Account'],
    ])('should display "%s" as modal title', (titleText) => {
      const { getByTestId } = render(
        <Modal 
          isOpen={true} 
          title={titleText}
          onClose={() => {}} 
          onConfirm={() => {}} 
        />
      );
      const titleElement = getByTestId('modal-title');
      expect(titleElement).toHaveTextContent(titleText);
    });
  });
});
```

---

## Complete Component File Reference

**File:** `lab015/src/Modal.js` (Final Version)

```js
import React from 'react';

// Modal component for displaying dialog boxes in the banking application
// Props:
//   - isOpen (bool): Controls visibility of modal
//   - title (string): Header text
//   - children (ReactNode): Body content
//   - onClose (function): Callback when close/cancel is clicked
//   - onConfirm (function): Callback when confirm is clicked
//   - confirmText (string, optional): Custom confirm button text (default: "Confirm")
function Modal({ isOpen, title, children, onClose, onConfirm, confirmText = 'Confirm' }) {
  // If modal is not open, render nothing (null)
  if (!isOpen) {
    return null;
  }

  // Render complete modal structure
  return (
    <div data-testid="modal-container" className="modal">
      {/* Modal Header Section */}
      <div className="modal-header">
        <h2 data-testid="modal-title">{title}</h2>
        <button 
          data-testid="close-btn" 
          onClick={onClose} 
          className="close-btn"
          aria-label="Close modal"
        >
          ×
        </button>
      </div>

      {/* Modal Body Section - Contains children content */}
      <div data-testid="modal-body" className="modal-body">
        {children}
      </div>

      {/* Modal Footer Section - Action buttons */}
      <div className="modal-footer">
        <button 
          data-testid="cancel-btn" 
          onClick={onClose} 
          className="cancel-btn"
        >
          Cancel
        </button>
        <button 
          data-testid="confirm-btn" 
          onClick={onConfirm} 
          className="confirm-btn"
        >
          {confirmText}
        </button>
      </div>
    </div>
  );
}

export default Modal;
```

---

## Running All Tests

To run all tests for this lab:

```bash
npm test -- ./lab015/test/Modal.test.js
```

To run tests in watch mode (re-runs when files change):

```bash
npm test -- ./lab015/test/Modal.test.js --watch
```

To run tests with coverage report:

```bash
npm test -- ./lab015/test/Modal.test.js --coverage
```

To update snapshots (after intentional UI changes):

```bash
npm test -- ./lab015/test/Modal.test.js -u
```

---

## Key Takeaways

1. **Snapshot Testing** captures UI structure and detects unintended changes quickly
2. **Setup/Teardown Hooks** ensure test isolation and prevent state leakage between tests
3. **Parameterized Tests** reduce code duplication and test multiple scenarios efficiently
4. **Mock Functions** track how many times callbacks are called and with what arguments
5. **Test Organization** with nested describe blocks makes tests more maintainable and readable
6. **TDD Approach** ensures code is testable and requirements are met before implementation

---

## Optional: Adding Basic Styling

To make the modal visually complete, create `lab015/src/Modal.css`:

```css
.modal {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: white;
  border: 1px solid #ccc;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  min-width: 400px;
  max-width: 600px;
  z-index: 1000;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px;
  border-bottom: 1px solid #eee;
}

.modal-header h2 {
  margin: 0;
  font-size: 20px;
  color: #333;
}

.close-btn {
  background: none;
  border: none;
  font-size: 24px;
  cursor: pointer;
  color: #999;
}

.close-btn:hover {
  color: #333;
}

.modal-body {
  padding: 16px;
  min-height: 100px;
}

.modal-footer {
  display: flex;
  gap: 8px;
  justify-content: flex-end;
  padding: 16px;
  border-top: 1px solid #eee;
}

.cancel-btn, .confirm-btn {
  padding: 8px 16px;
  border: 1px solid #ccc;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  transition: all 0.2s;
}

.cancel-btn {
  background: #f5f5f5;
  color: #333;
}

.cancel-btn:hover {
  background: #eeeeee;
}

.confirm-btn {
  background: #007bff;
  color: white;
  border-color: #007bff;
}

.confirm-btn:hover {
  background: #0056b3;
}
```

Then import it in Modal.js:
```js
import './Modal.css';
```

---

## Troubleshooting

**Snapshots mismatch?**
- Review the changes carefully, then run: `npm test -- ./lab015/test/Modal.test.js -u`

**Tests timeout?**
- Ensure all async operations are properly handled with async/await

**Mock not clearing between tests?**
- Verify `afterEach(() => { jest.clearAllMocks(); })` is included

**Module not found errors?**
- Check that babel.config.js and jest.config.js are in the root folder
- Verify all imports use correct relative paths

---
# Running The Lab

### Build And Browser Run Using Webpack

1. **Install Webpack and necessary dependencies:**

Run these commands at your project root (where `package.json` is):

```
npm install --save-dev webpack webpack-cli webpack-dev-server babel-loader @babel/core @babel/preset-env @babel/preset-react html-webpack-plugin react-refresh @pmmmwh/react-refresh-webpack-plugin

npm install react react-dom
```

2. **Create Babel config `babel.config.js` in root folder if not already:**

```
module.exports = {
presets: ['@babel/preset-env', '@babel/preset-react'],
};
```

3. **Create Webpack config file `webpack.config.js` in the root folder:**

```
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const ReactRefreshWebpackPlugin = require('@pmmmwh/react-refresh-webpack-plugin');

const isDevelopment = process.env.NODE_ENV !== 'production';

module.exports = {
  mode: isDevelopment ? 'development' : 'production',
  entry: './lab015/src/index.js', // Entry point for Modal lab
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js',
  },
  devtool: isDevelopment ? 'eval-source-map' : 'source-map',
  devServer: {
    static: './dist',
    hot: true,
    open: true,
  },
  module: {
    rules: [
      {
        test: /\.js$/,   // process all .js files (with JSX as well)
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            plugins: [isDevelopment && require.resolve('react-refresh/babel')].filter(Boolean),
          },
        },
      },
    ],
  },
  plugins: [
    isDevelopment && new ReactRefreshWebpackPlugin(),
    new HtmlWebpackPlugin({
      template: 'lab015/index.html',
    }),
  ].filter(Boolean),
  resolve: {
    extensions: ['.js', '.jsx'],
  },
};
```

4. **Create React entry point `lab015/src/index.js`:**

No renaming needed, just ensure JSX is allowed via Babel.

```
import React from 'react';
import { createRoot } from 'react-dom/client';
import Modal from './Modal';

const root = createRoot(document.getElementById('root'));
root.render(<Modal isOpen={true} title="Sample Modal" onClose={() => {}} onConfirm={() => {}} />);
```

5. **Create `lab015/index.html` (or use existing):**

```
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Modal Component Demo</title>
</head>

<body>
  <div id="root"></div> <!-- React mounts here -->
</body>

</html>
```

6. **Update `package.json` scripts:**

```
"scripts": {
  "start": "webpack serve --env development",
  "build": "webpack --mode production",
  "test": "jest"
}
```

7. **Run your development server:**

```
npm start
```

Your React app should open automatically and display the Modal component with hot reloading enabled.

8. **Run your unit tests:**

```
npm test
```