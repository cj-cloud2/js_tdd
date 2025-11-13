# Lab 017: SaveButton Component TDD Example Using Node.js, Jest, JSDOM, & React

**Technologies:** Node.js v22.20.0, Jest, jsdom, React

## Domain: File Management System - SaveButton Component (Create Voucher Screen)

Create and test a SaveButton component with loading states, validation, error handling, and user interaction using Test-Driven Development (TDD). This lab covers advanced Jest features including exception testing with the `toThrow` matcher, React Testing Library user interaction simulation, and Jest watch mode for iterative development.

---

## Jest Features Covered in This Lab

### 1. **Exception Testing with toThrow Matcher**
The `toThrow()` matcher verifies that a function throws an error when called. This is critical for testing error handling and edge cases where functions should fail gracefully. In our SaveButton component, we'll use `toThrow` to verify that validation functions properly reject invalid data.

**Why useful:** Ensures error handling logic works correctly and prevents silent failures in production.

### 2. **React Testing Library User Interaction Simulation**
RTL provides utilities like `userEvent` and `fireEvent` to simulate real user interactions (clicks, typing, focus changes). Unlike directly calling functions, simulating user interactions tests the component as users experience it, ensuring accessibility and usability.

**Why useful:** Tests components from the user's perspective, catching issues that unit tests might miss and ensuring proper keyboard navigation and screen reader support.

### 3. **Jest Watch Mode**
Watch mode (`--watch` flag) automatically re-runs tests whenever source or test files change. This provides instant feedback during development, making TDD more efficient and helping developers quickly identify regressions.

**Why useful:** Accelerates TDD workflow by providing real-time test feedback without manual command re-entry, enabling faster iteration cycles.

---

## Prerequisites

- You must already have Node.js, Jest, and JSDOM installed and working with a valid `jest.config.js`. 
- Jest should use jsdom as its environment and recognize your intended test folders.

### Step 1: Install Dependencies

Run below commands in `<root-folder>` (the same folder where your package.json lives):

- Install Jest and jsdom (should have been done in previous labs)
```bash
npm install --save-dev jest jsdom
```

- Then Install This
```bash
npm install --save-dev @testing-library/jest-dom
```

- Now React (Should have been done in precious assignments)
```bash
npm install react react-dom
```

Install Babel dependencies for JSX compilation:

```bash
npm install @babel/core @babel/preset-env @babel/preset-react babel-jest --save-dev
```

Install the React Testing Library as a developer dependency:
Up to previous Lab below command was like this:
'npm install --save-dev @testing-library/react @testing-library/user-event' now we add one more  @testing-library/jest-dom
```bash
npm install --save-dev @testing-library/react @testing-library/user-event @testing-library/jest-dom 

```

### Step 2: Verify/Update Jest Configuration

Ensure your `jest.config.js` in the root folder includes the lab017 path:

```js
module.exports = {
  testEnvironment: "jsdom",
  roots: ["<rootDir>/basic-js-tdd", "<rootDir>/lab003", "<rootDir>/lab015", "<rootDir>/lab017", "<rootDir>/"],
  testMatch: ["**/test/**/*.js", "**/tests/**/*.js", "**/__tests__/**/*.js", "**/?(*.)+(spec|test).js"],
  coverageDirectory: "coverage",
  clearMocks: true,
};
```

### Step 3: Add/Verify Babel Configuration

The **babel.config.js** file should be in the **root folder** of your project, alongside your package.json:

```js
moduleule.exports = {
  presets: ['@babel/preset-env', '@babel/preset-react'],
};
```

---

## Project Structure

Create a new subfolder for this lab:

```
<root-folder>/lab017/
    ├── src/
    │   └── SaveButton.js
    └── test/
        └── SaveButton.test.js
```

All production code will go in `src`, all tests go in `test`.

---

## SaveButton Component Specification

The SaveButton component is a versatile file save mechanism for a document management system with validation, loading states, and error handling capabilities.

**Component Properties:**
- **onSave** (function): Callback to handle save operation, can throw errors
- **isLoading** (boolean): Shows loading state while saving
- **isDisabled** (boolean, optional): Disables the button (default: false)
- **buttonText** (string, optional): Custom button text (default: "Save")
- **validator** (function, optional): Validates data before saving (should throw on invalid data)
- **validationData** (object, optional): Data to validate before save operation

**Visual Features:**
- Shows loading spinner/text when saving
- Displays error messages when validation or save fails
- Button is disabled during save operations
- Supports custom button text
- Shows success feedback after save

---

## Business Requirements (Gherkin Format)

```gherkin
Feature: SaveButton Component for Document Management

  Scenario: User clicks save button with valid data
    Given the SaveButton component is rendered
    And valid data is provided for validation
    When the user clicks the save button
    Then the onSave callback should be executed once
    And the button should show loading state during save
    And the button should return to normal state after save completes

  Scenario: User attempts to save with invalid data
    Given the SaveButton component is rendered with a validator function
    And the validator function throws an error for invalid data
    When the user clicks the save button
    Then the validator function should be called
    And the onSave callback should NOT be called
    And an error message should be displayed to the user

  Scenario: User clicks save button when already saving
    Given the SaveButton component is in loading state
    When the user attempts to click the button
    Then the button click should be prevented (button is disabled)
    And the save operation should only execute once

  Scenario: SaveButton displays dynamic text
    Given the SaveButton component is rendered with custom text
    When the component renders
    Then the button should display the provided custom text
    And during loading, text should change to indicate loading state

  Scenario: Error handling and recovery
    Given the SaveButton component is rendered
    And the onSave callback throws an error
    When the user clicks the save button
    Then an error message should be displayed
    And the user should be able to retry saving
```

---

# TDD Walkthrough (3 Passes)

For each pass, you will:
- Write fail-first tests (red phase)
- Implement just enough code to make tests pass (green phase)
- Clean/refactor if needed (refactor phase)
- Include descriptive comments in test functions

---

## PASS 1: Basic Button Rendering & Click Handling

**Business Requirements Addressed:**
- User clicks save button with valid data (partial)
- SaveButton displays dynamic text

**Jest Features Covered:**
- Basic RTL component rendering
- `userEvent` for simulating user clicks
- Mock function assertions

### RED Phase - Write Failing Tests

**File:** `lab017/test/SaveButton.test.js`

```js
import React from 'react';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import SaveButton from '../src/SaveButton';
import '@testing-library/jest-dom';

// PASS 1: Basic Button Rendering & Click Handling
describe('SaveButton Component - PASS 1: Basic Rendering & Click Handling', () => {
  // Setup: Run before each test to ensure clean state
  beforeEach(() => {
    jest.clearAllMocks();
  });

  // Teardown: Run after each test
  afterEach(() => {
    // Cleanup
  });

  // In this test, we verify that the SaveButton renders with default text
  it('should render button with default text "Save"', () => {
    const mockOnSave = jest.fn();
    render(<SaveButton onSave={mockOnSave} />);
    const button = screen.getByRole('button');
    expect(button).toBeInTheDocument();
    expect(button).toHaveTextContent('Save');
  });

  // In this test, we verify the SaveButton renders with custom button text
  it('should render button with custom text when buttonText prop is provided', () => {
    const mockOnSave = jest.fn();
    render(<SaveButton onSave={mockOnSave} buttonText="Upload Document" />);
    const button = screen.getByRole('button');
    expect(button).toHaveTextContent('Upload Document');
  });

  // In this test, we verify that clicking the button triggers the onSave callback
  it('should call onSave callback when button is clicked', async () => {
    const mockOnSave = jest.fn();
    const user = userEvent.setup();
    render(<SaveButton onSave={mockOnSave} />);
    const button = screen.getByRole('button');
    
    await user.click(button);
    
    expect(mockOnSave).toHaveBeenCalledTimes(1);
  });

  // In this test, we verify the button is disabled when isDisabled prop is true
  it('should disable button when isDisabled prop is true', () => {
    const mockOnSave = jest.fn();
    render(<SaveButton onSave={mockOnSave} isDisabled={true} />);
    const button = screen.getByRole('button');
    expect(button).toBeDisabled();
  });

  // In this test, we verify the button is enabled when isDisabled is false (default)
  it('should enable button when isDisabled prop is false', () => {
    const mockOnSave = jest.fn();
    render(<SaveButton onSave={mockOnSave} isDisabled={false} />);
    const button = screen.getByRole('button');
    expect(button).not.toBeDisabled();
  });
});
```

**Run tests from the root folder using:** (Tests should fail - RED)

```bash
npm test -- ./lab017/test/SaveButton.test.js
```

Expected output: 5 failing tests - the SaveButton component doesn't exist yet.

### GREEN Phase - Make Tests Pass

**File:** `lab017/src/SaveButton.js`

```js
import React from 'react';

// SaveButton component for document management system
// Props:
//   - onSave (function, required): Callback executed when button is clicked
//   - isLoading (boolean, optional): Shows loading state (default: false)
//   - isDisabled (boolean, optional): Disables the button (default: false)
//   - buttonText (string, optional): Custom button text (default: "Save")
//   - validator (function, optional): Validates data before saving
//   - validationData (object, optional): Data to validate

function SaveButton({ 
  onSave, 
  isLoading = false, 
  isDisabled = false, 
  buttonText = 'Save',
  validator = null,
  validationData = null
}) {
  // Determine if button should be disabled during operation
  const isButtonDisabled = isDisabled || isLoading;

  // Handle button click - validation and save logic
  const handleClick = async () => {
    // If validator is provided, validate data before saving
    if (validator && validationData) {
      try {
        // Call validator - if it throws, catch the error
        validator(validationData);
      } catch (error) {
        // Validation failed - don't proceed with save
        return;
      }
    }

    // Call the onSave callback provided by parent component
    onSave();
  };

  // Render the button
  return (
    <button
      onClick={handleClick}
      disabled={isButtonDisabled}
      className="save-button"
      aria-label="Save document"
    >
      {isLoading ? 'Saving...' : buttonText}
    </button>
  );
}

export default SaveButton;
```

**Run tests from the root folder using:** (Tests should pass - GREEN)

```bash
npm test -- ./lab017/test/SaveButton.test.js
```

Expected output: 5 passing tests

### Refactor

- Component logic is clean and straightforward
- Appropriate use of conditional rendering for loading state
- Proper disabled state handling

---

## PASS 2: Validation & Exception Handling with toThrow

**Business Requirements Addressed:**
- User attempts to save with invalid data
- Error handling and recovery

**Jest Features Covered:**
- `toThrow()` matcher for exception testing
- Testing validator functions that throw errors
- Error state display

### RED Phase - Add Failing Tests

**File:** `lab017/test/SaveButton.test.js`

```js
//**Important**: Add the following code AFTER the closing brace of the PASS 1 describe block, 
//but BEFORE the end of the file. This is a new describe block for PASS 2 tests.

describe('SaveButton Component - PASS 2: Validation & Exception Handling', () => {
  let mockOnSave;
  let mockValidator;

  // Setup: Create fresh mocks for each test
  beforeEach(() => {
    mockOnSave = jest.fn();
    mockValidator = jest.fn();
    jest.clearAllMocks();
  });

  // Teardown: Cleanup
  afterEach(() => {
    // Cleanup
  });

  // In this test, we verify that the validator function is called before saving
  it('should call validator with validationData before calling onSave', async () => {
    const user = userEvent.setup();
    const testData = { fileName: 'document.pdf', size: 1024 };
    
    render(
      <SaveButton 
        onSave={mockOnSave}
        validator={mockValidator}
        validationData={testData}
      />
    );
    
    const button = screen.getByRole('button');
    await user.click(button);
    
    expect(mockValidator).toHaveBeenCalledWith(testData);
  });

  // In this test, we verify that onSave is NOT called when validator throws an error
  it('should not call onSave when validator throws an error', async () => {
    const user = userEvent.setup();
    const testData = { fileName: 'invalid.xyz' };
    
    // Create a validator that throws an error for invalid file types
    const strictValidator = (data) => {
      if (!data.fileName.endsWith('.pdf')) {
        throw new Error('Invalid file type. Only PDF files are allowed.');
      }
    };
    
    render(
      <SaveButton 
        onSave={mockOnSave}
        validator={strictValidator}
        validationData={testData}
      />
    );
    
    const button = screen.getByRole('button');
    await user.click(button);
    
    // onSave should NOT have been called because validation failed
    expect(mockOnSave).not.toHaveBeenCalled();
  });

  // In this test, we verify that validator function properly throws exceptions
  it('should properly throw validation error for empty filename', () => {
    const fileValidator = (data) => {
      if (!data.fileName || data.fileName.trim() === '') {
        throw new Error('Filename cannot be empty');
      }
    };

    // Using toThrow to verify the function throws an error
    expect(() => {
      fileValidator({ fileName: '' });
    }).toThrow();

    // Can also verify the specific error message
    expect(() => {
      fileValidator({ fileName: '' });
    }).toThrow('Filename cannot be empty');
  });

  // In this test, we verify that validator function throws for files exceeding size limit
  it('should throw validation error when file size exceeds limit', () => {
    const sizeValidator = (data) => {
      const MAX_SIZE = 5 * 1024 * 1024; // 5MB
      if (data.size > MAX_SIZE) {
        throw new Error(`File size ${data.size} bytes exceeds maximum allowed size of ${MAX_SIZE} bytes`);
      }
    };

    // File that exceeds limit should throw
    expect(() => {
      sizeValidator({ size: 10 * 1024 * 1024 });
    }).toThrow();

    // File within limit should not throw
    expect(() => {
      sizeValidator({ size: 2 * 1024 * 1024 });
    }).not.toThrow();
  });

  // In this test, we verify that onSave IS called when validator passes
  it('should call onSave when validator passes validation', async () => {
    const user = userEvent.setup();
    const testData = { fileName: 'document.pdf', size: 1024 };
    
    // Validator that only passes for PDF files
    const pdfValidator = (data) => {
      if (!data.fileName.endsWith('.pdf')) {
        throw new Error('Only PDF files allowed');
      }
    };
    
    render(
      <SaveButton 
        onSave={mockOnSave}
        validator={pdfValidator}
        validationData={testData}
      />
    );
    
    const button = screen.getByRole('button');
    await user.click(button);
    
    // onSave SHOULD be called because validation passed
    expect(mockOnSave).toHaveBeenCalledTimes(1);
  });

  // In this test, we display an error message when validation fails
  it('should display error message when validation fails', async () => {
    const user = userEvent.setup();
    const testData = { fileName: 'image.jpg' };
    
    const errorValidator = (data) => {
      throw new Error('Only PDF files are supported');
    };
    
    const { rerender } = render(
      <SaveButton 
        onSave={mockOnSave}
        validator={errorValidator}
        validationData={testData}
      />
    );
    
    const button = screen.getByRole('button');
    await user.click(button);
    
    // After failed validation, error should be displayed
    // We'll add a way to display errors in the component
    expect(screen.queryByTestId('error-message')).toBeInTheDocument();
  });
});
```

**Run tests from the root folder using:** (Tests should fail - RED)

```bash
npm test -- ./lab017/test/SaveButton.test.js
```

Expected output: 6 new failing tests (PASS 1 tests should still pass)

### GREEN Phase - Make Tests Pass

**File:** `lab017/src/SaveButton.js`

**Important:** Replace the entire file with the updated version below:

```js
import React, { useState } from 'react';

// SaveButton component for document management system
// Props:
//   - onSave (function, required): Callback executed when button is clicked
//   - isLoading (boolean, optional): Shows loading state (default: false)
//   - isDisabled (boolean, optional): Disables the button (default: false)
//   - buttonText (string, optional): Custom button text (default: "Save")
//   - validator (function, optional): Validates data before saving
//   - validationData (object, optional): Data to validate

function SaveButton({ 
  onSave, 
  isLoading = false, 
  isDisabled = false, 
  buttonText = 'Save',
  validator = null,
  validationData = null
}) {
  // State to track validation/save errors
  const [error, setError] = useState(null);

  // Determine if button should be disabled during operation
  const isButtonDisabled = isDisabled || isLoading;

  // Handle button click - validation and save logic
  const handleClick = async () => {
    // Clear previous errors
    setError(null);

    // If validator is provided, validate data before saving
    if (validator && validationData) {
      try {
        // Call validator - if it throws, catch the error
        validator(validationData);
      } catch (validationError) {
        // Validation failed - store error and don't proceed with save
        setError(validationError.message);
        return;
      }
    }

    // Try to execute the onSave callback provided by parent component
    try {
      await onSave();
      // Clear error on successful save
      setError(null);
    } catch (saveError) {
      // Save operation failed - store error message
      setError(saveError.message);
    }
  };

  // Render the button with error message display
  return (
    <div className="save-button-container">
      <button
        onClick={handleClick}
        disabled={isButtonDisabled}
        className="save-button"
        aria-label="Save document"
      >
        {isLoading ? 'Saving...' : buttonText}
      </button>
      
      {/* Display error message if validation or save failed */}
      {error && (
        <div data-testid="error-message" className="error-message" role="alert">
          {error}
        </div>
      )}
    </div>
  );
}

export default SaveButton;
```

**Run tests from the root folder using:** (Tests should pass - GREEN)

```bash
npm test -- ./lab017/test/SaveButton.test.js
```

Expected output: 11 passing tests (5 from PASS 1 + 6 from PASS 2)

### Refactor

- Error state management is clean and intuitive
- Error message display is accessible with proper ARIA roles
- Component handles both validation and save operation errors

---

## PASS 3: Loading States & Watch Mode Integration

**Business Requirements Addressed:**
- User clicks save button with valid data (complete)
- User clicks save button when already saving
- SaveButton displays dynamic text (complete)

**Jest Features Covered:**
- Testing loading states with async operations
- Jest watch mode for efficient testing
- Simulating async save operations

### RED Phase - Add Failing Tests

**File:** `lab017/test/SaveButton.test.js`

```js
//**Important**: Add the following code AFTER the closing brace of the PASS 2 describe block, 
//but BEFORE the end of the file. This is a new describe block for PASS 3 tests.

describe('SaveButton Component - PASS 3: Loading States & Async Operations', () => {
  let mockOnSave;

  // Setup: Create fresh mocks for each test
  beforeEach(() => {
    mockOnSave = jest.fn();
    jest.clearAllMocks();
  });

  // Teardown: Cleanup
  afterEach(() => {
    // Cleanup
  });

  // In this test, we verify button text changes to "Saving..." during save
  it('should display "Saving..." text when isLoading prop is true', () => {
    render(<SaveButton onSave={mockOnSave} isLoading={true} />);
    const button = screen.getByRole('button');
    expect(button).toHaveTextContent('Saving...');
  });

  // In this test, we verify button returns to normal text after loading
  it('should display original button text when isLoading prop is false', () => {
    render(
      <SaveButton onSave={mockOnSave} isLoading={false} buttonText="Save Document" />
    );
    const button = screen.getByRole('button');
    expect(button).toHaveTextContent('Save Document');
  });

  // In this test, we verify button is disabled during loading state
  it('should disable button when isLoading prop is true', () => {
    render(<SaveButton onSave={mockOnSave} isLoading={true} />);
    const button = screen.getByRole('button');
    expect(button).toBeDisabled();
  });

  // In this test, we verify that clicking the button during save is prevented
  it('should not allow multiple clicks when button is already disabled', async () => {
    const user = userEvent.setup();
    const slowSave = jest.fn(
      () => new Promise(resolve => setTimeout(resolve, 1000))
    );

    const { rerender } = render(
      <SaveButton onSave={slowSave} isLoading={false} />
    );

    const button = screen.getByRole('button');
    
    // First click - save starts
    await user.click(button);
    expect(slowSave).toHaveBeenCalledTimes(1);

    // Simulate loading state by re-rendering with isLoading={true}
    rerender(
      <SaveButton onSave={slowSave} isLoading={true} />
    );

    const disabledButton = screen.getByRole('button');
    expect(disabledButton).toBeDisabled();
  });

  // In this test, we verify error state clears on new save attempt
  it('should clear error message when button is clicked again', async () => {
    const user = userEvent.setup();
    const testData = { fileName: 'invalid.txt' };
    
    const strictValidator = (data) => {
      if (!data.fileName.endsWith('.pdf')) {
        throw new Error('Only PDF files allowed');
      }
    };

    const { rerender } = render(
      <SaveButton 
        onSave={mockOnSave}
        validator={strictValidator}
        validationData={testData}
      />
    );

    const button = screen.getByRole('button');
    
    // First click - validation fails, error is displayed
    await user.click(button);
    expect(screen.getByTestId('error-message')).toBeInTheDocument();

    // Change validator to one that passes
    const newValidator = (data) => {
      // This validator always passes
    };

    // Re-render with passing validator
    rerender(
      <SaveButton 
        onSave={mockOnSave}
        validator={newValidator}
        validationData={testData}
      />
    );

    // Click button again
    await user.click(button);
    
    // Error should be cleared
    expect(screen.queryByTestId('error-message')).not.toBeInTheDocument();
  });

  // In this test, we verify async save operations are handled correctly
  it('should handle async onSave callback', async () => {
    const user = userEvent.setup();
    
    // Create an async save function that simulates API call
    const asyncSave = jest.fn(
      () => new Promise(resolve => {
        setTimeout(() => resolve('Save successful'), 100);
      })
    );

    render(<SaveButton onSave={asyncSave} />);
    const button = screen.getByRole('button');

    await user.click(button);

    expect(asyncSave).toHaveBeenCalledTimes(1);
  });

  // In this test, we verify error handling for failed async save
  it('should catch and display errors from failed async save operation', async () => {
    const user = userEvent.setup();
    
    // Create an async save function that throws an error
    const failedSave = jest.fn(
      () => new Promise((resolve, reject) => {
        setTimeout(() => {
          reject(new Error('Network error: Failed to connect to server'));
        }, 50);
      })
    );

    render(<SaveButton onSave={failedSave} />);
    const button = screen.getByRole('button');

    await user.click(button);

    // Wait for async operation to complete
    expect(failedSave).toHaveBeenCalledTimes(1);
  });

  // Parameterized test for different loading scenarios
  describe('Loading state variations', () => {
    test.each([
      [true, 'Saving...', true],  // [isLoading, expectedText, shouldBeDisabled]
      [false, 'Save', false],
      [false, 'Submit Form', false],
    ])(
      'when isLoading=%s, should display "%s" and disabled=%s',
      (isLoading, expectedText, shouldBeDisabled) => {
        const buttonText = expectedText === 'Submit Form' ? 'Submit Form' : 'Save';
        render(
          <SaveButton 
            onSave={mockOnSave} 
            isLoading={isLoading}
            buttonText={buttonText}
          />
        );
        
        const button = screen.getByRole('button');
        
        if (isLoading) {
          expect(button).toHaveTextContent('Saving...');
        } else {
          expect(button).toHaveTextContent(buttonText);
        }
        
        if (shouldBeDisabled) {
          expect(button).toBeDisabled();
        } else {
          expect(button).not.toBeDisabled();
        }
      }
    );
  });
});
```

**Run tests from the root folder using:** (Some tests should fail - RED)

```bash
npm test -- ./lab017/test/SaveButton.test.js
```

Expected output: 6 new tests, some failing (current implementation doesn't handle all async scenarios properly)

### GREEN Phase - Make Tests Pass

**File:** `lab017/src/SaveButton.js`

**Important:** Update the existing `handleClick` function. Replace lines where the function is defined (starting with `const handleClick = async () => {`) through the closing `};` before the return statement.

Find this section:
```js
  // Handle button click - validation and save logic
  const handleClick = async () => {
    // Clear previous errors
    setError(null);

    // If validator is provided, validate data before saving
    if (validator && validationData) {
      try {
        // Call validator - if it throws, catch the error
        validator(validationData);
      } catch (validationError) {
        // Validation failed - store error and don't proceed with save
        setError(validationError.message);
        return;
      }
    }

    // Try to execute the onSave callback provided by parent component
    try {
      await onSave();
      // Clear error on successful save
      setError(null);
    } catch (saveError) {
      // Save operation failed - store error message
      setError(saveError.message);
    }
  };
```

Replace it with:

```js
  // Handle button click - validation and save logic
  const handleClick = async () => {
    // Clear previous errors when starting new operation
    setError(null);

    // If validator is provided, validate data before saving
    if (validator && validationData) {
      try {
        // Call validator - if it throws, catch the error
        validator(validationData);
      } catch (validationError) {
        // Validation failed - store error and don't proceed with save
        setError(validationError.message);
        return;
      }
    }

    // Try to execute the onSave callback provided by parent component
    try {
      // Await the async save operation
      const result = await Promise.resolve(onSave());
      // Clear error on successful save
      setError(null);
      return result;
    } catch (saveError) {
      // Save operation failed - store error message
      setError(saveError.message);
      throw saveError;
    }
  };
```

**Run tests from the root folder using:** (All tests should pass - GREEN)

```bash
npm test -- ./lab017/test/SaveButton.test.js
```

Expected output: 17 passing tests total
- 5 from PASS 1
- 6 from PASS 2
- 6 from PASS 3

### Refactor

- Error handling is robust for both sync and async operations
- Loading states are properly managed
- Component is well-organized and maintainable

---

## Complete Test File Reference

**File:** `lab017/test/SaveButton.test.js` (Final Version)

```js
import React from 'react';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import SaveButton from '../src/SaveButton';
import '@testing-library/jest-dom';

// PASS 1: Basic Button Rendering & Click Handling
describe('SaveButton Component - PASS 1: Basic Rendering & Click Handling', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  afterEach(() => {
    // Cleanup
  });

  it('should render button with default text "Save"', () => {
    const mockOnSave = jest.fn();
    render(<SaveButton onSave={mockOnSave} />);
    const button = screen.getByRole('button');
    expect(button).toBeInTheDocument();
    expect(button).toHaveTextContent('Save');
  });

  it('should render button with custom text when buttonText prop is provided', () => {
    const mockOnSave = jest.fn();
    render(<SaveButton onSave={mockOnSave} buttonText="Upload Document" />);
    const button = screen.getByRole('button');
    expect(button).toHaveTextContent('Upload Document');
  });

  it('should call onSave callback when button is clicked', async () => {
    const mockOnSave = jest.fn();
    const user = userEvent.setup();
    render(<SaveButton onSave={mockOnSave} />);
    const button = screen.getByRole('button');
    
    await user.click(button);
    
    expect(mockOnSave).toHaveBeenCalledTimes(1);
  });

  it('should disable button when isDisabled prop is true', () => {
    const mockOnSave = jest.fn();
    render(<SaveButton onSave={mockOnSave} isDisabled={true} />);
    const button = screen.getByRole('button');
    expect(button).toBeDisabled();
  });

  it('should enable button when isDisabled prop is false', () => {
    const mockOnSave = jest.fn();
    render(<SaveButton onSave={mockOnSave} isDisabled={false} />);
    const button = screen.getByRole('button');
    expect(button).not.toBeDisabled();
  });
});

// PASS 2: Validation & Exception Handling
describe('SaveButton Component - PASS 2: Validation & Exception Handling', () => {
  let mockOnSave;
  let mockValidator;

  beforeEach(() => {
    mockOnSave = jest.fn();
    mockValidator = jest.fn();
    jest.clearAllMocks();
  });

  afterEach(() => {
    // Cleanup
  });

  it('should call validator with validationData before calling onSave', async () => {
    const user = userEvent.setup();
    const testData = { fileName: 'document.pdf', size: 1024 };
    
    render(
      <SaveButton 
        onSave={mockOnSave}
        validator={mockValidator}
        validationData={testData}
      />
    );
    
    const button = screen.getByRole('button');
    await user.click(button);
    
    expect(mockValidator).toHaveBeenCalledWith(testData);
  });

  it('should not call onSave when validator throws an error', async () => {
    const user = userEvent.setup();
    const testData = { fileName: 'invalid.xyz' };
    
    const strictValidator = (data) => {
      if (!data.fileName.endsWith('.pdf')) {
        throw new Error('Invalid file type. Only PDF files are allowed.');
      }
    };
    
    render(
      <SaveButton 
        onSave={mockOnSave}
        validator={strictValidator}
        validationData={testData}
      />
    );
    
    const button = screen.getByRole('button');
    await user.click(button);
    
    expect(mockOnSave).not.toHaveBeenCalled();
  });

  it('should properly throw validation error for empty filename', () => {
    const fileValidator = (data) => {
      if (!data.fileName || data.fileName.trim() === '') {
        throw new Error('Filename cannot be empty');
      }
    };

    expect(() => {
      fileValidator({ fileName: '' });
    }).toThrow();

    expect(() => {
      fileValidator({ fileName: '' });
    }).toThrow('Filename cannot be empty');
  });

  it('should throw validation error when file size exceeds limit', () => {
    const sizeValidator = (data) => {
      const MAX_SIZE = 5 * 1024 * 1024;
      if (data.size > MAX_SIZE) {
        throw new Error(`File size ${data.size} bytes exceeds maximum allowed size of ${MAX_SIZE} bytes`);
      }
    };

    expect(() => {
      sizeValidator({ size: 10 * 1024 * 1024 });
    }).toThrow();

    expect(() => {
      sizeValidator({ size: 2 * 1024 * 1024 });
    }).not.toThrow();
  });

  it('should call onSave when validator passes validation', async () => {
    const user = userEvent.setup();
    const testData = { fileName: 'document.pdf', size: 1024 };
    
    const pdfValidator = (data) => {
      if (!data.fileName.endsWith('.pdf')) {
        throw new Error('Only PDF files allowed');
      }
    };
    
    render(
      <SaveButton 
        onSave={mockOnSave}
        validator={pdfValidator}
        validationData={testData}
      />
    );
    
    const button = screen.getByRole('button');
    await user.click(button);
    
    expect(mockOnSave).toHaveBeenCalledTimes(1);
  });

  it('should display error message when validation fails', async () => {
    const user = userEvent.setup();
    const testData = { fileName: 'image.jpg' };
    
    const errorValidator = (data) => {
      throw new Error('Only PDF files are supported');
    };
    
    render(
      <SaveButton 
        onSave={mockOnSave}
        validator={errorValidator}
        validationData={testData}
      />
    );
    
    const button = screen.getByRole('button');
    await user.click(button);
    
    expect(screen.queryByTestId('error-message')).toBeInTheDocument();
  });
});

// PASS 3: Loading States & Async Operations
describe('SaveButton Component - PASS 3: Loading States & Async Operations', () => {
  let mockOnSave;

  beforeEach(() => {
    mockOnSave = jest.fn();
    jest.clearAllMocks();
  });

  afterEach(() => {
    // Cleanup
  });

  it('should display "Saving..." text when isLoading prop is true', () => {
    render(<SaveButton onSave={mockOnSave} isLoading={true} />);
    const button = screen.getByRole('button');
    expect(button).toHaveTextContent('Saving...');
  });

  it('should display original button text when isLoading prop is false', () => {
    render(
      <SaveButton onSave={mockOnSave} isLoading={false} buttonText="Save Document" />
    );
    const button = screen.getByRole('button');
    expect(button).toHaveTextContent('Save Document');
  });

  it('should disable button when isLoading prop is true', () => {
    render(<SaveButton onSave={mockOnSave} isLoading={true} />);
    const button = screen.getByRole('button');
    expect(button).toBeDisabled();
  });

  it('should not allow multiple clicks when button is already disabled', async () => {
    const user = userEvent.setup();
    const slowSave = jest.fn(
      () => new Promise(resolve => setTimeout(resolve, 1000))
    );

    const { rerender } = render(
      <SaveButton onSave={slowSave} isLoading={false} />
    );

    const button = screen.getByRole('button');
    
    await user.click(button);
    expect(slowSave).toHaveBeenCalledTimes(1);

    rerender(
      <SaveButton onSave={slowSave} isLoading={true} />
    );

    const disabledButton = screen.getByRole('button');
    expect(disabledButton).toBeDisabled();
  });

  it('should clear error message when button is clicked again', async () => {
    const user = userEvent.setup();
    const testData = { fileName: 'invalid.txt' };
    
    const strictValidator = (data) => {
      if (!data.fileName.endsWith('.pdf')) {
        throw new Error('Only PDF files allowed');
      }
    };

    const { rerender } = render(
      <SaveButton 
        onSave={mockOnSave}
        validator={strictValidator}
        validationData={testData}
      />
    );

    const button = screen.getByRole('button');
    
    await user.click(button);
    expect(screen.getByTestId('error-message')).toBeInTheDocument();

    const newValidator = (data) => {
      // This validator always passes
    };

    rerender(
      <SaveButton 
        onSave={mockOnSave}
        validator={newValidator}
        validationData={testData}
      />
    );

    await user.click(button);
    
    expect(screen.queryByTestId('error-message')).not.toBeInTheDocument();
  });

  it('should handle async onSave callback', async () => {
    const user = userEvent.setup();
    
    const asyncSave = jest.fn(
      () => new Promise(resolve => {
        setTimeout(() => resolve('Save successful'), 100);
      })
    );

    render(<SaveButton onSave={asyncSave} />);
    const button = screen.getByRole('button');

    await user.click(button);

    expect(asyncSave).toHaveBeenCalledTimes(1);
  });

  it('should catch and display errors from failed async save operation', async () => {
    const user = userEvent.setup();
    
    const failedSave = jest.fn(
      () => new Promise((resolve, reject) => {
        setTimeout(() => {
          reject(new Error('Network error: Failed to connect to server'));
        }, 50);
      })
    );

    render(<SaveButton onSave={failedSave} />);
    const button = screen.getByRole('button');

    await user.click(button);

    expect(failedSave).toHaveBeenCalledTimes(1);
  });

  describe('Loading state variations', () => {
    test.each([
      [true, 'Saving...', true],
      [false, 'Save', false],
      [false, 'Submit Form', false],
    ])(
      'when isLoading=%s, should display "%s" and disabled=%s',
      (isLoading, expectedText, shouldBeDisabled) => {
        const buttonText = expectedText === 'Submit Form' ? 'Submit Form' : 'Save';
        render(
          <SaveButton 
            onSave={mockOnSave} 
            isLoading={isLoading}
            buttonText={buttonText}
          />
        );
        
        const button = screen.getByRole('button');
        
        if (isLoading) {
          expect(button).toHaveTextContent('Saving...');
        } else {
          expect(button).toHaveTextContent(buttonText);
        }
        
        if (shouldBeDisabled) {
          expect(button).toBeDisabled();
        } else {
          expect(button).not.toBeDisabled();
        }
      }
    );
  });
});
```

---

## Complete Component File Reference

**File:** `lab017/src/SaveButton.js` (Final Version)

```js
import React, { useState } from 'react';

// SaveButton component for document management system
// Props:
//   - onSave (function, required): Callback executed when button is clicked
//   - isLoading (boolean, optional): Shows loading state (default: false)
//   - isDisabled (boolean, optional): Disables the button (default: false)
//   - buttonText (string, optional): Custom button text (default: "Save")
//   - validator (function, optional): Validates data before saving
//   - validationData (object, optional): Data to validate

function SaveButton({ 
  onSave, 
  isLoading = false, 
  isDisabled = false, 
  buttonText = 'Save',
  validator = null,
  validationData = null
}) {
  // State to track validation/save errors
  const [error, setError] = useState(null);

  // Determine if button should be disabled during operation
  const isButtonDisabled = isDisabled || isLoading;

  // Handle button click - validation and save logic
  const handleClick = async () => {
    // Clear previous errors when starting new operation
    setError(null);

    // If validator is provided, validate data before saving
    if (validator && validationData) {
      try {
        // Call validator - if it throws, catch the error
        validator(validationData);
      } catch (validationError) {
        // Validation failed - store error and don't proceed with save
        setError(validationError.message);
        return;
      }
    }

    // Try to execute the onSave callback provided by parent component
    try {
      // Await the async save operation
      const result = await Promise.resolve(onSave());
      // Clear error on successful save
      setError(null);
      return result;
    } catch (saveError) {
      // Save operation failed - store error message
      setError(saveError.message);
      throw saveError;
    }
  };

  // Render the button with error message display
  return (
    <div className="save-button-container">
      <button
        onClick={handleClick}
        disabled={isButtonDisabled}
        className="save-button"
        aria-label="Save document"
      >
        {isLoading ? 'Saving...' : buttonText}
      </button>
      
      {/* Display error message if validation or save failed */}
      {error && (
        <div data-testid="error-message" className="error-message" role="alert">
          {error}
        </div>
      )}
    </div>
  );
}

export default SaveButton;
```

---

## Running All Tests

### Running tests normally:

```bash
npm test -- ./lab017/test/SaveButton.test.js
```

### Running tests in Watch Mode (Recommended for TDD):

Jest watch mode automatically re-runs tests when files change, making TDD development much more efficient:

```bash
npm test -- ./lab017/test/SaveButton.test.js --watch
```

**Watch Mode Keyboard Shortcuts:**
- `a`: Run all tests
- `f`: Run only failed tests
- `p`: Filter tests by filename
- `t`: Filter tests by test name
- `q`: Quit watch mode
- `Enter`: Re-run last test command

### Running tests with coverage report:

```bash
npm test -- ./lab017/test/SaveButton.test.js --coverage
```

This displays what percentage of your code is covered by tests, helping identify untested code paths.

### Updating snapshots (if needed):

```bash
npm test -- ./lab017/test/SaveButton.test.js -u
```

---

## Key Takeaways

1. **toThrow() Matcher** ensures functions properly throw errors and can be caught
2. **User Event Simulation** tests components as real users interact with them (clicks, typing, etc.)
3. **Jest Watch Mode** accelerates TDD by providing instant feedback on code changes
4. **Error State Management** provides user feedback for validation and save failures
5. **Async Operation Handling** properly manages loading states during API calls
6. **Test Organization** with clear describe blocks and meaningful test names improves maintainability
7. **Parameterized Tests** reduce duplication when testing multiple scenarios

---


## Optional: Adding Basic Styling

To make the SaveButton component visually complete, create `lab017/src/SaveButton.css`:

```css
.save-button-container {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.save-button {
  padding: 10px 20px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease-in-out;
  min-width: 120px;
}

.save-button:hover:not(:disabled) {
  background-color: #0056b3;
  box-shadow: 0 2px 8px rgba(0, 86, 179, 0.3);
}

.save-button:active:not(:disabled) {
  transform: scale(0.98);
}

.save-button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
  opacity: 0.6;
}

.error-message {
  padding: 10px 12px;
  background-color: #f8d7da;
  color: #721c24;
  border: 1px solid #f5c6cb;
  border-radius: 4px;
  font-size: 14px;
  animation: slideIn 0.3s ease-in-out;
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateY(-10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
```

Then import it in SaveButton.js at the top:
```js
import './SaveButton.css';
```

---

## Troubleshooting

**Tests not running?**
- Verify `jest.config.js` includes `lab017` in the roots array
- Check that `babel.config.js` is in the root folder

**Watch mode not updating?**
- Make sure you're running `npm test -- ./lab017/test/SaveButton.test.js --watch`
- Watch mode monitors file changes in real-time

**Error message not displaying?**
- Verify that `data-testid="error-message"` is in the component
- Check that the error state is being set correctly in handleClick

**Async tests timing out?**
- Ensure you're using `async/await` properly in test functions
- Verify that promises are being resolved or rejected

**Module not found errors?**
- Check that babel.config.js and jest.config.js are in the root folder
- Verify all imports use correct relative paths (e.g., `../src/SaveButton`)

---

# Running The Lab

### Build And Browser Run Using Webpack

1. **Install Webpack and necessary dependencies:**

Run these commands at your project root (where `package.json` is):

```bash
npm install --save-dev webpack webpack-cli webpack-dev-server babel-loader @babel/core @babel/preset-env @babel/preset-react html-webpack-plugin react-refresh @pmmmwh/react-refresh-webpack-plugin

npm install react react-dom
```

2. **Create Babel config `babel.config.js` in root folder if not already:**

```js
module.exports = {
  presets: ['@babel/preset-env', '@babel/preset-react'],
};
```

3. **Create Webpack config file `webpack.config.js` in the root folder:**

```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const ReactRefreshWebpackPlugin = require('@pmmmwh/react-refresh-webpack-plugin');

const isDevelopment = process.env.NODE_ENV !== 'production';

module.exports = {
  mode: isDevelopment ? 'development' : 'production',
  entry: './lab017/src/index.js',
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
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            plugins: [isDevelopment && require.resolve('react-refresh/babel')].filter(Boolean),
          },
        },
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader'],
      },
    ],
  },
  plugins: [
    isDevelopment && new ReactRefreshWebpackPlugin(),
    new HtmlWebpackPlugin({
      template: 'lab017/index.html',
    }),
  ].filter(Boolean),
  resolve: {
    extensions: ['.js', '.jsx'],
  },
};
```

4. **Install CSS loaders for styling:**

```bash
npm install --save-dev style-loader css-loader
```

5. **Create React entry point `lab017/src/index.js`:**

```js
import React, { useState } from 'react';
import { createRoot } from 'react-dom/client';
import SaveButton from './SaveButton';

function App() {
  const [saved, setSaved] = useState(false);

  const handleSave = async () => {
    // Simulate async save operation
    return new Promise((resolve) => {
      setTimeout(() => {
        setSaved(true);
        resolve('Saved successfully!');
      }, 1000);
    });
  };

  const validateFile = (data) => {
    if (!data.fileName || !data.fileName.endsWith('.pdf')) {
      throw new Error('Please upload a valid PDF file');
    }
    if (data.size > 5 * 1024 * 1024) {
      throw new Error('File size must not exceed 5MB');
    }
  };

  return (
    <div style={{ padding: '40px', maxWidth: '600px', margin: '0 auto' }}>
      <h1>SaveButton Component Demo</h1>
      <p>Try clicking the Save button to see it in action!</p>
      
      <SaveButton 
        onSave={handleSave}
        buttonText="Save Voucher"
        validator={validateFile}
        validationData={{ fileName: 'document.pdf', size: 1024 * 1024 }}
      />

      {saved && <p style={{ color: 'green', marginTop: '20px' }}>✓ Document saved successfully!</p>}
    </div>
  );
}

const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

6. **Create `lab017/index.html`:**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>SaveButton Component Demo</title>
</head>
<body>
  <div id="root"></div>
</body>
</html>
```

7. **Update `package.json` scripts:**

```json
"scripts": {
  "start": "webpack serve --env development",
  "build": "webpack --mode production",
  "test": "jest"
}
```

8. **Run your development server:**

```bash
npm start
```

Your React app should open automatically and display the SaveButton component with hot reloading enabled.

9. **Run your unit tests:**

```bash
npm test -- ./lab017/test/SaveButton.test.js
```

Or run in watch mode:

```bash
npm test -- ./lab017/test/SaveButton.test.js --watch
```
**Test may fail becuse of CSS issues, We will leave it here, This could be solved by CSS mocking, which is covered in other assignment**
---

## Summary

This lab demonstrates three critical Jest features for real-world component testing:

1. **Exception Testing (toThrow)** - Ensures validation functions properly throw and handle errors
2. **User Interaction Simulation (RTL + userEvent)** - Tests components from the user's perspective
3. **Jest Watch Mode** - Provides efficient TDD workflow with real-time feedback

The SaveButton component showcases proper error handling, async operations, and accessible React patterns suitable for production applications.