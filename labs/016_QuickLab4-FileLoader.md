# Lab 016: FileLoader Component TDD Example Using Node.js, Jest, JSDOM, & React

**Technologies:** Node.js v22.20.0, Jest, jsdom, React

## Domain: Create Voucher Screen - FileLoader Component Walkthrough

Create and test a FileLoader component for handling file uploads with validation, progress tracking, and success/error states using Test-Driven Development (TDD). This lab covers advanced Jest features including testing asynchronous code with Promises and async/await, manual mocks for file handling, and generating coverage reports.

---

## Jest Features Covered in This Lab

### 1. **Asynchronous Testing with Promises and async/await**
Jest provides robust support for testing asynchronous code. The async/await keywords make asynchronous test code look synchronous and easy to read. You can use `async` to mark a test function and `await` to wait for Promise resolution before proceeding with assertions.

**Why useful:** File uploads, API calls, and other async operations are fundamental in modern web apps. Testing them properly ensures data consistency and proper error handling.

### 2. **Manual Mocks**
Manual mocks allow you to replace real implementations with controlled versions. For FileLoader, we'll mock the file reading API to simulate various scenarios (success, failures, timeout) without touching the actual file system.

**Why useful:** Speeds up tests, provides deterministic results, eliminates dependencies on external systems, and allows testing of edge cases.

### 3. **Coverage Reports**
Jest's coverage reports show which parts of your code are tested. Coverage metrics include line coverage, branch coverage, function coverage, and statement coverage. A coverage report helps identify untested code paths and improve test quality.

**Why useful:** Ensures comprehensive testing, identifies gaps in test coverage, and helps maintain code quality standards.

---

## Business Requirements (Gherkin Format)

```gherkin
Feature: Create Voucher Screen - FileLoader Component

  Scenario: User uploads a valid voucher file
    Given the FileLoader component is rendered
    When the user selects a valid PDF file
    Then the file should be processed successfully
    And the success message should be displayed
    And the file details should be shown

  Scenario: User uploads an invalid file type
    Given the FileLoader component is rendered
    When the user selects a non-PDF file
    Then an error message should be displayed
    And the error text should indicate invalid file type
    And the file should not be processed

  Scenario: User uploads a file larger than the limit
    Given the FileLoader component is rendered
    When the user selects a file larger than 5MB
    Then an error message should be displayed
    And the error text should indicate file size exceeded
    And the file should not be processed

  Scenario: File reading operation takes time
    Given the FileLoader component is rendered
    When the user selects a file
    Then a progress indicator should be displayed
    And the loading state should be set
    Then after the file is read, the progress indicator should disappear

  Scenario: File reading encounters an error
    Given the FileLoader component is rendered
    When the file reading operation fails
    Then an error message should be displayed
    And the error state should be cleared after retry
```

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

- Then install this
```bash
npm install --save-dev @testing-library/jest-dom
```

- Now React (Should have been done in previous assignments)
```bash
npm install react react-dom
```

Install Babel dependencies for JSX compilation:

```bash
npm install @babel/core @babel/preset-env @babel/preset-react babel-jest --save-dev
```

Install the React Testing Library as a developer dependency:

```bash
npm install --save-dev @testing-library/react @testing-library/user-event
#npm install --save-dev @testing-library/react @testing-library/user-event @testing-library/jest-dom

```

### Step 2: Verify/Update Jest Configuration

Ensure your `jest.config.js` in the root folder includes the lab016 path:

```js
module.exports = {
  testEnvironment: "jsdom",
  roots: ["<rootDir>/basic-js-tdd", "<rootDir>/lab003", "<rootDir>/lab015", "<rootDir>/lab016", "<rootDir>/"],
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
<root-folder>/lab016/
    ├── src/
    │   ├── FileLoader.js
    │   └── __mocks__/
    │       └── fileAPI.js
    └── test/
        └── FileLoader.test.js
```

All production code will go in `src`, all tests go in `test`. Manual mocks go in the `__mocks__` folder.

---

## FileLoader Component Specification

The FileLoader component is a reusable file upload handler for the Create Voucher screen with the following features:

**Component Properties:**
- **onFileUpload** (function): Callback when file is successfully uploaded
- **onError** (function): Callback when an error occurs
- **maxFileSize** (number, optional): Maximum file size in bytes (default: 5MB = 5242880)
- **acceptedFormats** (array, optional): Allowed file extensions (default: ['.pdf'])

**Component States:**
- **loading** (boolean): File is being processed
- **error** (string or null): Error message if upload failed
- **success** (boolean): File was successfully uploaded
- **fileData** (object or null): Information about the uploaded file

**Visual Structure:**
- Input field to select file
- Progress indicator (shows while loading)
- Error message container (conditional)
- Success message with file details (conditional)
- Retry button (shown on error)

---

## Requirements

**File Validation:**
- Positive: Accept PDF files within 5MB
- Negative: Reject non-PDF files
- Negative: Reject files larger than 5MB

**Async File Processing:**
- Positive: Display loading state during file reading
- Positive: Call onFileUpload callback with file data after successful processing
- Negative: Handle file reading errors gracefully

**Error Handling:**
- Positive: Display appropriate error messages for different failure scenarios
- Negative: Clear error state on file retry

**User Feedback:**
- Positive: Display file name and size after successful upload
- Negative: Display retry button when error occurs

---

# TDD Walkthrough (3 Passes)

For each pass, you will:
- Write fail-first tests (red phase)
- Implement just enough code to make tests pass (green phase)
- Clean/refactor if needed (refactor phase)
- Include descriptive comments in test functions

---

## PASS 1: File Validation & Component Rendering

### Business Requirements Addressed in This Pass
```gherkin
Scenario: User uploads a valid voucher file (partial - validation only)
  Given the FileLoader component is rendered
  Then file input should be available

Scenario: User uploads an invalid file type
  Given the FileLoader component is rendered
  When the user selects a non-PDF file
  Then an error message should be displayed

Scenario: User uploads a file larger than the limit
  Given the FileLoader component is rendered
  When the user selects a file larger than 5MB
  Then an error message should be displayed
```

### Jest Features Covered in This Pass
- Basic component rendering with React Testing Library
- Testing event handlers (onChange)
- Mock functions with jest.fn()

### RED Phase - Write Failing Tests

**File:** `lab016/test/FileLoader.test.js`

```js
import React from 'react';
import { render, screen, fireEvent } from '@testing-library/react';
import FileLoader from '../src/FileLoader';
import '@testing-library/jest-dom';

// PASS 1: File Validation & Component Rendering
describe('FileLoader Component - PASS 1: Validation & Rendering', () => {
  let mockOnFileUpload;
  let mockOnError;

  // Setup: Create fresh mock functions before each test
  beforeEach(() => {
    mockOnFileUpload = jest.fn();
    mockOnError = jest.fn();
    jest.clearAllMocks();
  });

  // Teardown: Clean up after each test
  afterEach(() => {
    // Additional cleanup if needed
  });

  // In this test, we verify the component renders with file input element
  it('should render file input element', () => {
    render(
      <FileLoader onFileUpload={mockOnFileUpload} onError={mockOnError} />
    );
    const fileInput = screen.getByTestId('file-input');
    expect(fileInput).toBeInTheDocument();
  });

  // In this test, we verify the component renders with upload button
  it('should render upload button', () => {
    render(
      <FileLoader onFileUpload={mockOnFileUpload} onError={mockOnError} />
    );
    const uploadBtn = screen.getByTestId('upload-btn');
    expect(uploadBtn).toBeInTheDocument();
  });

  // In this test, we verify that non-PDF files are rejected immediately
  it('should reject non-PDF files and show error', () => {
    render(
      <FileLoader onFileUpload={mockOnFileUpload} onError={mockOnError} />
    );
    const fileInput = screen.getByTestId('file-input');
    
    // Create a mock file object that is NOT a PDF
    const invalidFile = new File(['content'], 'voucher.txt', { type: 'text/plain' });
    
    // Simulate user selecting the invalid file
    fireEvent.change(fileInput, { target: { files: [invalidFile] } });
    
    // The error callback should be called
    expect(mockOnError).toHaveBeenCalled();
    expect(mockOnError).toHaveBeenCalledWith('Invalid file type. Only PDF files are accepted.');
  });

  // In this test, we verify that files larger than 5MB are rejected
  it('should reject files larger than 5MB and show error', () => {
    render(
      <FileLoader onFileUpload={mockOnFileUpload} onError={mockOnError} />
    );
    const fileInput = screen.getByTestId('file-input');
    
    // Create a mock PDF file that is 6MB (larger than 5MB limit)
    const largeFile = new File(['x'.repeat(6 * 1024 * 1024)], 'large-voucher.pdf', { type: 'application/pdf' });
    
    // Simulate user selecting the large file
    fireEvent.change(fileInput, { target: { files: [largeFile] } });
    
    // The error callback should be called with size error message
    expect(mockOnError).toHaveBeenCalled();
    expect(mockOnError).toHaveBeenCalledWith('File size exceeds 5 MB limit.');
  });

  // In this test, we verify that valid PDF files within size limit are accepted
  it('should accept valid PDF files within size limit', () => {
    render(
      <FileLoader onFileUpload={mockOnFileUpload} onError={mockOnError} />
    );
    const fileInput = screen.getByTestId('file-input');
    
    // Create a mock PDF file that is within size limit
    const validFile = new File(['PDF content'], 'voucher.pdf', { type: 'application/pdf' });
    
    // Simulate user selecting the valid file
    fireEvent.change(fileInput, { target: { files: [validFile] } });
    
    // The error callback should NOT be called for validation
    expect(mockOnError).not.toHaveBeenCalled();
  });

  // In this test, we verify that no file selected shows appropriate state
  it('should display initial state when no file is selected', () => {
    render(
      <FileLoader onFileUpload={mockOnFileUpload} onError={mockOnError} />
    );
    
    // Initially, no error or success message should be visible
    const errorMessage = screen.queryByTestId('error-message');
    const successMessage = screen.queryByTestId('success-message');
    
    expect(errorMessage).not.toBeInTheDocument();
    expect(successMessage).not.toBeInTheDocument();
  });
});
```

**Run tests from the root folder using:** (Tests should fail - RED)

```bash
npm test -- ./lab016/test/FileLoader.test.js
```

Expected output: 6 failing tests - the FileLoader component doesn't exist yet.

### GREEN Phase - Make Tests Pass

**File:** `lab016/src/FileLoader.js`

```js
import React, { useState } from 'react';

// FileLoader component for handling voucher file uploads with validation
// Props:
//   - onFileUpload (function): Callback when file is successfully uploaded
//   - onError (function): Callback when an error occurs
//   - maxFileSize (number, optional): Maximum file size in bytes (default: 5MB)
//   - acceptedFormats (array, optional): Allowed file extensions (default: ['.pdf'])
function FileLoader({ 
  onFileUpload, 
  onError, 
  maxFileSize = 5 * 1024 * 1024, // 5MB default
  acceptedFormats = ['.pdf']
}) {
  const [error, setError] = useState(null);
  const [success, setSuccess] = useState(false);
  const [loading, setLoading] = useState(false);
  const [fileData, setFileData] = useState(null);

  // Validate file before processing
  // Checks file type and file size
  const validateFile = (file) => {
    // Check if file extension is in accepted formats
    const fileExtension = '.' + file.name.split('.').pop().toLowerCase();
    if (!acceptedFormats.includes(fileExtension)) {
      return { valid: false, error: 'Invalid file type. Only PDF files are accepted.' };
    }

    // Check if file size exceeds maximum allowed
    if (file.size > maxFileSize) {
      return { valid: false, error: 'File size exceeds 5 MB limit.' };
    }

    return { valid: true };
  };

  // Handle file selection from input
  // Validates file and triggers appropriate callback
  const handleFileChange = (event) => {
    const files = event.target.files;
    
    if (!files || files.length === 0) {
      return;
    }

    const file = files[0];
    
    // Validate the selected file
    const validation = validateFile(file);
    
    if (!validation.valid) {
      setError(validation.error);
      setSuccess(false);
      onError(validation.error);
      return;
    }

    // File passed validation - clear any previous errors
    setError(null);
    setSuccess(false);
  };

  return (
    <div data-testid="file-loader-container" className="file-loader">
      {/* File Input */}
      <input
        type="file"
        data-testid="file-input"
        onChange={handleFileChange}
        className="file-input"
        accept=".pdf"
      />

      {/* Upload Button */}
      <button data-testid="upload-btn" className="upload-btn">
        Upload
      </button>

      {/* Error Message Container */}
      {error && (
        <div data-testid="error-message" className="error-container">
          <p className="error-text">{error}</p>
        </div>
      )}

      {/* Success Message Container */}
      {success && fileData && (
        <div data-testid="success-message" className="success-container">
          <p className="success-text">File uploaded successfully!</p>
          <p className="file-details">File: {fileData.name} ({fileData.size} bytes)</p>
        </div>
      )}
    </div>
  );
}

export default FileLoader;
```

**Run tests from the root folder using:** (Tests should pass - GREEN)

```bash
npm test -- ./lab016/test/FileLoader.test.js
```

Expected output: 6 passing tests

### Refactor

- The component structure is clean and focused
- Validation logic is separated into its own function
- State management is clear with individual useState hooks
- No major refactoring needed at this stage

---

## PASS 2: Asynchronous File Processing with Promises & async/await

### Business Requirements Addressed in This Pass
```gherkin
Scenario: User uploads a valid voucher file
  Given the FileLoader component is rendered
  When the user selects a valid PDF file
  Then the file should be processed successfully
  And the success message should be displayed

Scenario: File reading operation takes time
  Given the FileLoader component is rendered
  When the user selects a file
  Then a progress indicator should be displayed
  And the loading state should be set
```

### Jest Features Covered in This Pass
- Testing asynchronous code with async/await
- Mocking modules with jest.mock()
- Waiting for async operations in tests with waitFor()
- Testing Promise resolution and rejection

### RED Phase - Add Failing Tests

**File:** `lab016/test/FileLoader.test.js`

**Important:** Add the following code AFTER the closing brace of the first describe block (PASS 1), but BEFORE the end of the file. This is a new describe block for PASS 2 tests.

```js
// PASS 2: Asynchronous File Processing
// Import waitFor for testing async operations
import { waitFor } from '@testing-library/react';

// Create a manual mock for the fileAPI module BEFORE the test suite
jest.mock('../src/fileAPI', () => ({
  readFileAsync: jest.fn(),
}));

// Import the mocked module
import { readFileAsync } from '../src/fileAPI';

describe('FileLoader Component - PASS 2: Async Processing & Loading State', () => {
  let mockOnFileUpload;
  let mockOnError;

  // Setup: Create fresh mock functions and clear all mocks before each test
  beforeEach(() => {
    mockOnFileUpload = jest.fn();
    mockOnError = jest.fn();
    jest.clearAllMocks();
    // Reset all mock implementations
    readFileAsync.mockClear();
  });

  // Teardown: Clean up after each test
  afterEach(() => {
    // Additional cleanup if needed
  });

  // In this test, we verify that loading state is shown while file is being processed
  // This test uses async/await to handle the asynchronous file reading
  it('should display loading state while file is being processed', async () => {
    // Mock the file reading to take some time (500ms delay)
    readFileAsync.mockImplementation(
      () => new Promise(resolve => setTimeout(() => resolve({ name: 'test.pdf', size: 1024 }), 500))
    );

    const { rerender } = render(
      <FileLoader onFileUpload={mockOnFileUpload} onError={mockOnError} />
    );
    
    const fileInput = screen.getByTestId('file-input');
    const validFile = new File(['PDF content'], 'voucher.pdf', { type: 'application/pdf' });
    
    // Simulate file selection and upload
    fireEvent.change(fileInput, { target: { files: [validFile] } });
    const uploadBtn = screen.getByTestId('upload-btn');
    fireEvent.click(uploadBtn);
    
    // At this point, the loading indicator should be visible
    // We'll check for this in the component by looking for a loading message
    await waitFor(() => {
      rerender(
        <FileLoader onFileUpload={mockOnFileUpload} onError={mockOnError} />
      );
    });
  });

  // In this test, we verify that file reading succeeds and calls onFileUpload callback
  // Uses async/await to properly handle Promise resolution
  it('should call onFileUpload callback when file is successfully read', async () => {
    // Mock successful file reading
    const mockFileData = { name: 'voucher.pdf', size: 2048, content: 'PDF data here' };
    readFileAsync.mockResolvedValue(mockFileData);

    render(
      <FileLoader onFileUpload={mockOnFileUpload} onError={mockOnError} />
    );
    
    const fileInput = screen.getByTestId('file-input');
    const validFile = new File(['PDF content'], 'voucher.pdf', { type: 'application/pdf' });
    
    // Simulate file selection
    fireEvent.change(fileInput, { target: { files: [validFile] } });
    const uploadBtn = screen.getByTestId('upload-btn');
    
    // Click upload button to trigger async file processing
    fireEvent.click(uploadBtn);
    
    // Wait for the onFileUpload callback to be called
    // The async operation should complete and call the callback
    await waitFor(() => {
      expect(mockOnFileUpload).toHaveBeenCalled();
    });
  });

  // In this test, we verify that file data is displayed after successful upload
  // Tests that async operation results are properly rendered
  it('should display file details after successful upload', async () => {
    const mockFileData = { name: 'voucher.pdf', size: 3072 };
    readFileAsync.mockResolvedValue(mockFileData);

    render(
      <FileLoader onFileUpload={mockOnFileUpload} onError={mockOnError} />
    );
    
    const fileInput = screen.getByTestId('file-input');
    const validFile = new File(['PDF content'], 'voucher.pdf', { type: 'application/pdf' });
    
    fireEvent.change(fileInput, { target: { files: [validFile] } });
    const uploadBtn = screen.getByTestId('upload-btn');
    fireEvent.click(uploadBtn);
    
    // Wait for success message to appear
    await waitFor(() => {
      const successMessage = screen.queryByTestId('success-message');
      expect(successMessage).toBeInTheDocument();
    });
  });

  // In this test, we verify that loading spinner exists in component
  it('should show progress indicator when loading', async () => {
    readFileAsync.mockImplementation(
      () => new Promise(resolve => setTimeout(() => resolve({ name: 'test.pdf', size: 1024 }), 1000))
    );

    render(
      <FileLoader onFileUpload={mockOnFileUpload} onError={mockOnError} />
    );
    
    const fileInput = screen.getByTestId('file-input');
    const validFile = new File(['PDF content'], 'voucher.pdf', { type: 'application/pdf' });
    
    fireEvent.change(fileInput, { target: { files: [validFile] } });
    const uploadBtn = screen.getByTestId('upload-btn');
    fireEvent.click(uploadBtn);
    
    // Check that loading indicator appears (we'll add this to component)
    const loadingIndicator = screen.queryByTestId('loading-indicator');
    // Should be visible during loading
    if (loadingIndicator) {
      expect(loadingIndicator).toBeInTheDocument();
    }
  });
});
```

**Run tests from the root folder using:** (Tests should fail - RED)

```bash
npm test -- ./lab016/test/FileLoader.test.js
```

Expected output: 4 new failing tests (PASS 1 tests should still pass)

### Create the Manual Mock

Before making the tests pass, create the manual mock file that the tests expect.

**File:** `lab016/src/__mocks__/fileAPI.js`

This is a manual mock that we'll use in our tests. It gets automatically picked up by Jest's mock system.

```js
// Manual mock for fileAPI module
// This mock is used in tests to simulate file reading without actual file I/O

module.exports = {
  // Mock function that simulates reading a file asynchronously
  // In tests, we'll use mockResolvedValue() or mockRejectedValue() to control behavior
  readFileAsync: jest.fn(),
};
```

### Create the Real FileAPI Module

Now create the real implementation that will be mocked in tests.

**File:** `lab016/src/fileAPI.js`

```js
// Real file API for handling file reading operations
// This module contains async functions for file processing
// In tests, this entire module will be replaced with the mock version

/**
 * Reads a file asynchronously and returns file data
 * @param {File} file - The file object to read
 * @returns {Promise} - Promise that resolves with file data or rejects with error
 */
export const readFileAsync = async (file) => {
  return new Promise((resolve, reject) => {
    // In a real implementation, this would use FileReader API
    // For now, we'll simulate it
    const reader = new FileReader();
    
    // When file reading succeeds
    reader.onload = () => {
      resolve({
        name: file.name,
        size: file.size,
        content: reader.result,
      });
    };
    
    // When file reading fails
    reader.onerror = () => {
      reject(new Error('Failed to read file'));
    };
    
    // Start reading the file as text
    reader.readAsText(file);
  });
};
```

### GREEN Phase - Make Tests Pass

**File:** `lab016/src/FileLoader.js`

Replace the entire file with the updated version. The key changes are:
- Import the readFileAsync function
- Add file upload logic to the upload button
- Add loading state rendering
- Add success message after file upload

```js
import React, { useState } from 'react';
import { readFileAsync } from './fileAPI';

// FileLoader component for handling voucher file uploads with validation
// Props:
//   - onFileUpload (function): Callback when file is successfully uploaded
//   - onError (function): Callback when an error occurs
//   - maxFileSize (number, optional): Maximum file size in bytes (default: 5MB)
//   - acceptedFormats (array, optional): Allowed file extensions (default: ['.pdf'])
function FileLoader({ 
  onFileUpload, 
  onError, 
  maxFileSize = 5 * 1024 * 1024, // 5MB default
  acceptedFormats = ['.pdf']
}) {
  const [error, setError] = useState(null);
  const [success, setSuccess] = useState(false);
  const [loading, setLoading] = useState(false);
  const [fileData, setFileData] = useState(null);
  const [selectedFile, setSelectedFile] = useState(null);

  // Validate file before processing
  // Checks file type and file size
  const validateFile = (file) => {
    // Check if file extension is in accepted formats
    const fileExtension = '.' + file.name.split('.').pop().toLowerCase();
    if (!acceptedFormats.includes(fileExtension)) {
      return { valid: false, error: 'Invalid file type. Only PDF files are accepted.' };
    }

    // Check if file size exceeds maximum allowed
    if (file.size > maxFileSize) {
      return { valid: false, error: 'File size exceeds 5 MB limit.' };
    }

    return { valid: true };
  };

  // Handle file selection from input
  // Validates file and triggers appropriate callback
  const handleFileChange = (event) => {
    const files = event.target.files;
    
    if (!files || files.length === 0) {
      return;
    }

    const file = files[0];
    
    // Validate the selected file
    const validation = validateFile(file);
    
    if (!validation.valid) {
      setError(validation.error);
      setSuccess(false);
      onError(validation.error);
      setSelectedFile(null);
      return;
    }

    // File passed validation - clear any previous errors
    setError(null);
    setSuccess(false);
    setSelectedFile(file); // Store file for upload
  };

  // Handle file upload - reads file asynchronously
  // IMPORTANT: Insert this entire function AFTER the handleFileChange function above
  // (after line: setSelectedFile(file); // Store file for upload)
  const handleFileUpload = async () => {
    if (!selectedFile) {
      onError('No file selected');
      return;
    }

    try {
      // Set loading state to show progress indicator
      setLoading(true);
      setError(null);

      // Call the async file reading function
      const result = await readFileAsync(selectedFile);

      // File successfully read - update state with file data
      setFileData(result);
      setSuccess(true);
      setLoading(false);

      // Call the onFileUpload callback with the file data
      onFileUpload(result);
    } catch (err) {
      // Error occurred during file reading
      setLoading(false);
      setError(err.message || 'Failed to process file');
      setSuccess(false);
      onError(err.message || 'Failed to process file');
    }
  };

  return (
    <div data-testid="file-loader-container" className="file-loader">
      {/* File Input */}
      <input
        type="file"
        data-testid="file-input"
        onChange={handleFileChange}
        className="file-input"
        accept=".pdf"
      />

      {/* Upload Button */}
      <button 
        data-testid="upload-btn" 
        onClick={handleFileUpload}
        disabled={loading}
        className="upload-btn"
      >
        {loading ? 'Uploading...' : 'Upload'}
      </button>

      {/* Loading Indicator */}
      {loading && (
        <div data-testid="loading-indicator" className="loading-container">
          <div className="spinner"></div>
          <p className="loading-text">Processing file...</p>
        </div>
      )}

      {/* Error Message Container */}
      {error && (
        <div data-testid="error-message" className="error-container">
          <p className="error-text">{error}</p>
          <button 
            data-testid="retry-btn"
            onClick={() => setError(null)}
            className="retry-btn"
          >
            Retry
          </button>
        </div>
      )}

      {/* Success Message Container */}
      {success && fileData && (
        <div data-testid="success-message" className="success-container">
          <p className="success-text">File uploaded successfully!</p>
          <p className="file-details">File: {fileData.name} ({fileData.size} bytes)</p>
        </div>
      )}
    </div>
  );
}

export default FileLoader;
```

**Run tests from the root folder using:** (Tests should pass - GREEN)

```bash
npm test -- ./lab016/test/FileLoader.test.js
```

Expected output: 10 passing tests (6 from PASS 1 + 4 from PASS 2)

### Refactor

- The component now properly handles async operations
- Error handling is comprehensive with try/catch
- Loading state is managed correctly
- Retry functionality added for better UX
- The handleFileUpload function is well-documented

---

## PASS 3: Error Handling, Edge Cases, & Coverage Reports

### Business Requirements Addressed in This Pass
```gherkin
Scenario: File reading encounters an error
  Given the FileLoader component is rendered
  When the file reading operation fails
  Then an error message should be displayed
  And the error state should be cleared after retry

Scenario: Multiple file uploads in sequence
  When the user uploads multiple files in succession
  Then each upload should be tracked independently
```

### Jest Features Covered in This Pass
- Testing Promise rejection with mockRejectedValue()
- Parameterized tests with test.each() for multiple scenarios
- Generating coverage reports with --coverage flag
- Testing edge cases and error scenarios

### RED Phase - Add Failing Tests

**File:** `lab016/test/FileLoader.test.js`

**Important:** Add the following code AFTER the closing brace of the PASS 2 describe block, but BEFORE the end of the file. This is a new describe block for PASS 3 tests.

```js
describe('FileLoader Component - PASS 3: Error Handling & Coverage', () => {
  let mockOnFileUpload;
  let mockOnError;

  // Setup: Create fresh mock functions and clear all mocks before each test
  beforeEach(() => {
    mockOnFileUpload = jest.fn();
    mockOnError = jest.fn();
    jest.clearAllMocks();
    readFileAsync.mockClear();
  });

  // Teardown: Clean up after each test
  afterEach(() => {
    // Additional cleanup if needed
  });

  // In this test, we verify that errors during file reading are handled gracefully
  // We simulate the readFileAsync function rejecting with an error
  it('should handle file reading errors gracefully', async () => {
    // Mock the file reading to reject with an error
    readFileAsync.mockRejectedValue(new Error('File read failed'));

    render(
      <FileLoader onFileUpload={mockOnFileUpload} onError={mockOnError} />
    );
    
    const fileInput = screen.getByTestId('file-input');
    const validFile = new File(['PDF content'], 'voucher.pdf', { type: 'application/pdf' });
    
    fireEvent.change(fileInput, { target: { files: [validFile] } });
    const uploadBtn = screen.getByTestId('upload-btn');
    fireEvent.click(uploadBtn);
    
    // Wait for error message to appear
    await waitFor(() => {
      const errorMessage = screen.queryByTestId('error-message');
      expect(errorMessage).toBeInTheDocument();
    });

    // Verify onError callback was called
    expect(mockOnError).toHaveBeenCalled();
  });

  // In this test, we verify that retry button clears error state
  // After clearing error, user should be able to retry upload
  it('should clear error state when retry button is clicked', async () => {
    readFileAsync.mockRejectedValue(new Error('Network error'));

    render(
      <FileLoader onFileUpload={mockOnFileUpload} onError={mockOnError} />
    );
    
    const fileInput = screen.getByTestId('file-input');
    const validFile = new File(['PDF content'], 'voucher.pdf', { type: 'application/pdf' });
    
    fireEvent.change(fileInput, { target: { files: [validFile] } });
    const uploadBtn = screen.getByTestId('upload-btn');
    fireEvent.click(uploadBtn);
    
    // Wait for error message to appear
    await waitFor(() => {
      const errorMessage = screen.queryByTestId('error-message');
      expect(errorMessage).toBeInTheDocument();
    });

    // Click retry button to clear error
    const retryBtn = screen.getByTestId('retry-btn');
    fireEvent.click(retryBtn);
    
    // Error message should now be gone
    await waitFor(() => {
      const errorMessage = screen.queryByTestId('error-message');
      expect(errorMessage).not.toBeInTheDocument();
    });
  });

  // In this test, we verify that no file selected triggers error
  it('should show error when upload is clicked without selecting a file', async () => {
    render(
      <FileLoader onFileUpload={mockOnFileUpload} onError={mockOnError} />
    );
    
    const uploadBtn = screen.getByTestId('upload-btn');
    
    // Click upload without selecting a file
    fireEvent.click(uploadBtn);
    
    // Error callback should be called
    expect(mockOnError).toHaveBeenCalledWith('No file selected');
  });

  // Parameterized test for different error scenarios
  // Tests multiple error conditions with minimal code duplication
  describe('Different error scenarios', () => {
    test.each([
      ['Corrupted file', new Error('Corrupted file'), 'Corrupted file'],
      ['Permission denied', new Error('Permission denied'), 'Permission denied'],
      ['Timeout', new Error('Operation timeout'), 'Operation timeout'],
      ['Unknown error', new Error('Unknown error occurred'), 'Unknown error occurred'],
    ])('should handle error: %s', async (scenarioName, errorThrown, expectedMessage) => {
      // Mock readFileAsync to reject with specific error
      readFileAsync.mockRejectedValue(errorThrown);

      render(
        <FileLoader onFileUpload={mockOnFileUpload} onError={mockOnError} />
      );
      
      const fileInput = screen.getByTestId('file-input');
      const validFile = new File(['PDF content'], 'voucher.pdf', { type: 'application/pdf' });
      
      fireEvent.change(fileInput, { target: { files: [validFile] } });
      const uploadBtn = screen.getByTestId('upload-btn');
      fireEvent.click(uploadBtn);
      
      // Wait for error and verify it was called with expected message
      await waitFor(() => {
        expect(mockOnError).toHaveBeenCalledWith(expectedMessage);
      });
    });
  });

  // Parameterized test for different file sizes
  // Ensures validation works for various file sizes
  describe('File size validation', () => {
    test.each([
      ['1 byte', new File(['x'], 'tiny.pdf', { type: 'application/pdf' }), true],
      ['1 MB', new File(['x'.repeat(1024 * 1024)], 'small.pdf', { type: 'application/pdf' }), true],
      ['4.9 MB', new File(['x'.repeat(Math.floor(4.9 * 1024 * 1024))], 'large.pdf', { type: 'application/pdf' }), true],
      ['5 MB exactly', new File(['x'.repeat(5 * 1024 * 1024)], 'exact.pdf', { type: 'application/pdf' }), true],
      ['5.1 MB', new File(['x'.repeat(Math.floor(5.1 * 1024 * 1024))], 'toobig.pdf', { type: 'application/pdf' }), false],
    ])('should %s file (%s) correctly', async (sizeDesc, file, shouldPass) => {
      render(
        <FileLoader onFileUpload={mockOnFileUpload} onError={mockOnError} />
      );
      
      const fileInput = screen.getByTestId('file-input');
      fireEvent.change(fileInput, { target: { files: [file] } });
      
      if (shouldPass) {
        // Should NOT call error callback for valid files
        expect(mockOnError).not.toHaveBeenCalled();
      } else {
        // SHOULD call error callback for files exceeding limit
        expect(mockOnError).toHaveBeenCalledWith('File size exceeds 5 MB limit.');
      }
    });
  });

  // In this test, we verify that loading state is cleared after error
  it('should clear loading state when error occurs', async () => {
    readFileAsync.mockRejectedValue(new Error('Failed'));

    const { rerender } = render(
      <FileLoader onFileUpload={mockOnFileUpload} onError={mockOnError} />
    );
    
    const fileInput = screen.getByTestId('file-input');
    const validFile = new File(['PDF content'], 'voucher.pdf', { type: 'application/pdf' });
    
    fireEvent.change(fileInput, { target: { files: [validFile] } });
    const uploadBtn = screen.getByTestId('upload-btn');
    fireEvent.click(uploadBtn);
    
    // Wait for loading to complete and error to appear
    await waitFor(() => {
      const loadingIndicator = screen.queryByTestId('loading-indicator');
      expect(loadingIndicator).not.toBeInTheDocument();
    });
  });

  // In this test, we verify successful upload clears previous errors
  it('should clear previous errors on successful upload', async () => {
    // First mock a failure
    readFileAsync.mockRejectedValueOnce(new Error('First attempt failed'));

    const { rerender } = render(
      <FileLoader onFileUpload={mockOnFileUpload} onError={mockOnError} />
    );
    
    let fileInput = screen.getByTestId('file-input');
    let validFile = new File(['PDF content'], 'voucher.pdf', { type: 'application/pdf' });
    
    fireEvent.change(fileInput, { target: { files: [validFile] } });
    let uploadBtn = screen.getByTestId('upload-btn');
    fireEvent.click(uploadBtn);
    
    // Wait for error to appear
    await waitFor(() => {
      const errorMessage = screen.queryByTestId('error-message');
      expect(errorMessage).toBeInTheDocument();
    });

    // Now mock a success for second attempt
    readFileAsync.mockResolvedValueOnce({ name: 'voucher.pdf', size: 1024 });
    
    // Clear error by clicking retry
    const retryBtn = screen.getByTestId('retry-btn');
    fireEvent.click(retryBtn);
    
    // Try upload again
    uploadBtn = screen.getByTestId('upload-btn');
    fireEvent.click(uploadBtn);
    
    // Wait for success message
    await waitFor(() => {
      const successMessage = screen.queryByTestId('success-message');
      expect(successMessage).toBeInTheDocument();
    });

    // Error message should be gone
    const errorMessage = screen.queryByTestId('error-message');
    expect(errorMessage).not.toBeInTheDocument();
  });
});
```

**Run tests from the root folder using:** (Tests should fail - RED)

```bash
npm test -- ./lab016/test/FileLoader.test.js
```

Expected output: Multiple new failing tests (PASS 1 and PASS 2 tests should still pass)

### GREEN Phase - Make Tests Pass

The FileLoader component from PASS 2 already handles most of these requirements. However, we need to ensure all error cases are properly managed. The current implementation should make all tests pass.

**File:** `lab016/src/FileLoader.js` - No changes needed from PASS 2

**Run tests from the root folder using:** (All tests should pass - GREEN)

```bash
npm test -- ./lab016/test/FileLoader.test.js
```

Expected output: All tests passing

### Refactor

- Code is clean and well-organized
- Error handling is comprehensive
- Loading states are properly managed

### Generate Coverage Report

Now let's generate a coverage report to see how much of the code is tested.

**Run coverage report:**

```bash
npm test -- ./lab016/test/FileLoader.test.js --coverage
```

This will generate a coverage report showing:
- **Line Coverage:** Which lines of code were executed during tests
- **Branch Coverage:** Which conditional branches were executed
- **Function Coverage:** Which functions were called
- **Statement Coverage:** Which statements were executed

The coverage report will also be available as HTML in `coverage/lcov-report/index.html`

You can open it in a browser to see visual coverage information:

```bash
# On Windows
start coverage/lcov-report/index.html

# On Mac
open coverage/lcov-report/index.html

# On Linux
xdg-open coverage/lcov-report/index.html
```

---

## Complete Test File Reference

**File:** `lab016/test/FileLoader.test.js` (Final Version)

```js
import React from 'react';
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import FileLoader from '../src/FileLoader';
import '@testing-library/jest-dom';

// Create a manual mock for the fileAPI module BEFORE the test suite
jest.mock('../src/fileAPI', () => ({
  readFileAsync: jest.fn(),
}));

// Import the mocked module
import { readFileAsync } from '../src/fileAPI';

// PASS 1: File Validation & Component Rendering
describe('FileLoader Component - PASS 1: Validation & Rendering', () => {
  let mockOnFileUpload;
  let mockOnError;

  beforeEach(() => {
    mockOnFileUpload = jest.fn();
    mockOnError = jest.fn();
    jest.clearAllMocks();
  });

  afterEach(() => {
    // Additional cleanup if needed
  });

  it('should render file input element', () => {
    render(
      <FileLoader onFileUpload={mockOnFileUpload} onError={mockOnError} />
    );
    const fileInput = screen.getByTestId('file-input');
    expect(fileInput).toBeInTheDocument();
  });

  it('should render upload button', () => {
    render(
      <FileLoader onFileUpload={mockOnFileUpload} onError={mockOnError} />
    );
    const uploadBtn = screen.getByTestId('upload-btn');
    expect(uploadBtn).toBeInTheDocument();
  });

  it('should reject non-PDF files and show error', () => {
    render(
      <FileLoader onFileUpload={mockOnFileUpload} onError={mockOnError} />
    );
    const fileInput = screen.getByTestId('file-input');
    
    const invalidFile = new File(['content'], 'voucher.txt', { type: 'text/plain' });
    fireEvent.change(fileInput, { target: { files: [invalidFile] } });
    
    expect(mockOnError).toHaveBeenCalled();
    expect(mockOnError).toHaveBeenCalledWith('Invalid file type. Only PDF files are accepted.');
  });

  it('should reject files larger than 5MB and show error', () => {
    render(
      <FileLoader onFileUpload={mockOnFileUpload} onError={mockOnError} />
    );
    const fileInput = screen.getByTestId('file-input');
    
    const largeFile = new File(['x'.repeat(6 * 1024 * 1024)], 'large-voucher.pdf', { type: 'application/pdf' });
    fireEvent.change(fileInput, { target: { files: [largeFile] } });
    
    expect(mockOnError).toHaveBeenCalled();
    expect(mockOnError).toHaveBeenCalledWith('File size exceeds 5 MB limit.');
  });

  it('should accept valid PDF files within size limit', () => {
    render(
      <FileLoader onFileUpload={mockOnFileUpload} onError={mockOnError} />
    );
    const fileInput = screen.getByTestId('file-input');
    
    const validFile = new File(['PDF content'], 'voucher.pdf', { type: 'application/pdf' });
    fireEvent.change(fileInput, { target: { files: [validFile] } });
    
    expect(mockOnError).not.toHaveBeenCalled();
  });

  it('should display initial state when no file is selected', () => {
    render(
      <FileLoader onFileUpload={mockOnFileUpload} onError={mockOnError} />
    );
    
    const errorMessage = screen.queryByTestId('error-message');
    const successMessage = screen.queryByTestId('success-message');
    
    expect(errorMessage).not.toBeInTheDocument();
    expect(successMessage).not.toBeInTheDocument();
  });
});

// PASS 2: Asynchronous File Processing
describe('FileLoader Component - PASS 2: Async Processing & Loading State', () => {
  let mockOnFileUpload;
  let mockOnError;

  beforeEach(() => {
    mockOnFileUpload = jest.fn();
    mockOnError = jest.fn();
    jest.clearAllMocks();
    readFileAsync.mockClear();
  });

  afterEach(() => {
    // Additional cleanup if needed
  });

  it('should call onFileUpload callback when file is successfully read', async () => {
    const mockFileData = { name: 'voucher.pdf', size: 2048, content: 'PDF data here' };
    readFileAsync.mockResolvedValue(mockFileData);

    render(
      <FileLoader onFileUpload={mockOnFileUpload} onError={mockOnError} />
    );
    
    const fileInput = screen.getByTestId('file-input');
    const validFile = new File(['PDF content'], 'voucher.pdf', { type: 'application/pdf' });
    
    fireEvent.change(fileInput, { target: { files: [validFile] } });
    const uploadBtn = screen.getByTestId('upload-btn');
    fireEvent.click(uploadBtn);
    
    await waitFor(() => {
      expect(mockOnFileUpload).toHaveBeenCalled();
    });
  });

  it('should display file details after successful upload', async () => {
    const mockFileData = { name: 'voucher.pdf', size: 3072 };
    readFileAsync.mockResolvedValue(mockFileData);

    render(
      <FileLoader onFileUpload={mockOnFileUpload} onError={mockOnError} />
    );
    
    const fileInput = screen.getByTestId('file-input');
    const validFile = new File(['PDF content'], 'voucher.pdf', { type: 'application/pdf' });
    
    fireEvent.change(fileInput, { target: { files: [validFile] } });
    const uploadBtn = screen.getByTestId('upload-btn');
    fireEvent.click(uploadBtn);
    
    await waitFor(() => {
      const successMessage = screen.queryByTestId('success-message');
      expect(successMessage).toBeInTheDocument();
    });
  });

  it('should show progress indicator when loading', async () => {
    readFileAsync.mockImplementation(
      () => new Promise(resolve => setTimeout(() => resolve({ name: 'test.pdf', size: 1024 }), 500))
    );

    render(
      <FileLoader onFileUpload={mockOnFileUpload} onError={mockOnError} />
    );
    
    const fileInput = screen.getByTestId('file-input');
    const validFile = new File(['PDF content'], 'voucher.pdf', { type: 'application/pdf' });
    
    fireEvent.change(fileInput, { target: { files: [validFile] } });
    const uploadBtn = screen.getByTestId('upload-btn');
    fireEvent.click(uploadBtn);
    
    const loadingIndicator = screen.queryByTestId('loading-indicator');
    if (loadingIndicator) {
      expect(loadingIndicator).toBeInTheDocument();
    }
  });

  it('should disable upload button while loading', async () => {
    readFileAsync.mockImplementation(
      () => new Promise(resolve => setTimeout(() => resolve({ name: 'test.pdf', size: 1024 }), 300))
    );

    render(
      <FileLoader onFileUpload={mockOnFileUpload} onError={mockOnError} />
    );
    
    const fileInput = screen.getByTestId('file-input');
    const validFile = new File(['PDF content'], 'voucher.pdf', { type: 'application/pdf' });
    
    fireEvent.change(fileInput, { target: { files: [validFile] } });
    const uploadBtn = screen.getByTestId('upload-btn');
    fireEvent.click(uploadBtn);
    
    // Button should be disabled during loading
    expect(uploadBtn).toBeDisabled();
  });
});

// PASS 3: Error Handling & Coverage
describe('FileLoader Component - PASS 3: Error Handling & Coverage', () => {
  let mockOnFileUpload;
  let mockOnError;

  beforeEach(() => {
    mockOnFileUpload = jest.fn();
    mockOnError = jest.fn();
    jest.clearAllMocks();
    readFileAsync.mockClear();
  });

  afterEach(() => {
    // Additional cleanup if needed
  });

  it('should handle file reading errors gracefully', async () => {
    readFileAsync.mockRejectedValue(new Error('File read failed'));

    render(
      <FileLoader onFileUpload={mockOnFileUpload} onError={mockOnError} />
    );
    
    const fileInput = screen.getByTestId('file-input');
    const validFile = new File(['PDF content'], 'voucher.pdf', { type: 'application/pdf' });
    
    fireEvent.change(fileInput, { target: { files: [validFile] } });
    const uploadBtn = screen.getByTestId('upload-btn');
    fireEvent.click(uploadBtn);
    
    await waitFor(() => {
      const errorMessage = screen.queryByTestId('error-message');
      expect(errorMessage).toBeInTheDocument();
    });

    expect(mockOnError).toHaveBeenCalled();
  });

  it('should clear error state when retry button is clicked', async () => {
    readFileAsync.mockRejectedValue(new Error('Network error'));

    render(
      <FileLoader onFileUpload={mockOnFileUpload} onError={mockOnError} />
    );
    
    const fileInput = screen.getByTestId('file-input');
    const validFile = new File(['PDF content'], 'voucher.pdf', { type: 'application/pdf' });
    
    fireEvent.change(fileInput, { target: { files: [validFile] } });
    const uploadBtn = screen.getByTestId('upload-btn');
    fireEvent.click(uploadBtn);
    
    await waitFor(() => {
      const errorMessage = screen.queryByTestId('error-message');
      expect(errorMessage).toBeInTheDocument();
    });

    const retryBtn = screen.getByTestId('retry-btn');
    fireEvent.click(retryBtn);
    
    await waitFor(() => {
      const errorMessage = screen.queryByTestId('error-message');
      expect(errorMessage).not.toBeInTheDocument();
    });
  });

  it('should show error when upload is clicked without selecting a file', async () => {
    render(
      <FileLoader onFileUpload={mockOnFileUpload} onError={mockOnError} />
    );
    
    const uploadBtn = screen.getByTestId('upload-btn');
    fireEvent.click(uploadBtn);
    
    expect(mockOnError).toHaveBeenCalledWith('No file selected');
  });

  describe('Different error scenarios', () => {
    test.each([
      ['Corrupted file', new Error('Corrupted file'), 'Corrupted file'],
      ['Permission denied', new Error('Permission denied'), 'Permission denied'],
      ['Timeout', new Error('Operation timeout'), 'Operation timeout'],
      ['Unknown error', new Error('Unknown error occurred'), 'Unknown error occurred'],
    ])('should handle error: %s', async (scenarioName, errorThrown, expectedMessage) => {
      readFileAsync.mockRejectedValue(errorThrown);

      render(
        <FileLoader onFileUpload={mockOnFileUpload} onError={mockOnError} />
      );
      
      const fileInput = screen.getByTestId('file-input');
      const validFile = new File(['PDF content'], 'voucher.pdf', { type: 'application/pdf' });
      
      fireEvent.change(fileInput, { target: { files: [validFile] } });
      const uploadBtn = screen.getByTestId('upload-btn');
      fireEvent.click(uploadBtn);
      
      await waitFor(() => {
        expect(mockOnError).toHaveBeenCalledWith(expectedMessage);
      });
    });
  });

  describe('File size validation', () => {
    test.each([
      ['1 byte', new File(['x'], 'tiny.pdf', { type: 'application/pdf' }), true],
      ['1 MB', new File(['x'.repeat(1024 * 1024)], 'small.pdf', { type: 'application/pdf' }), true],
      ['4.9 MB', new File(['x'.repeat(Math.floor(4.9 * 1024 * 1024))], 'large.pdf', { type: 'application/pdf' }), true],
      ['5 MB exactly', new File(['x'.repeat(5 * 1024 * 1024)], 'exact.pdf', { type: 'application/pdf' }), true],
      ['5.1 MB', new File(['x'.repeat(Math.floor(5.1 * 1024 * 1024))], 'toobig.pdf', { type: 'application/pdf' }), false],
    ])('should handle %s file (%s) correctly', async (sizeDesc, file, shouldPass) => {
      render(
        <FileLoader onFileUpload={mockOnFileUpload} onError={mockOnError} />
      );
      
      const fileInput = screen.getByTestId('file-input');
      fireEvent.change(fileInput, { target: { files: [file] } });
      
      if (shouldPass) {
        expect(mockOnError).not.toHaveBeenCalled();
      } else {
        expect(mockOnError).toHaveBeenCalledWith('File size exceeds 5 MB limit.');
      }
    });
  });

  it('should clear loading state when error occurs', async () => {
    readFileAsync.mockRejectedValue(new Error('Failed'));

    const { rerender } = render(
      <FileLoader onFileUpload={mockOnFileUpload} onError={mockOnError} />
    );
    
    const fileInput = screen.getByTestId('file-input');
    const validFile = new File(['PDF content'], 'voucher.pdf', { type: 'application/pdf' });
    
    fireEvent.change(fileInput, { target: { files: [validFile] } });
    const uploadBtn = screen.getByTestId('upload-btn');
    fireEvent.click(uploadBtn);
    
    await waitFor(() => {
      const loadingIndicator = screen.queryByTestId('loading-indicator');
      expect(loadingIndicator).not.toBeInTheDocument();
    });
  });

  it('should clear previous errors on successful upload', async () => {
    readFileAsync.mockRejectedValueOnce(new Error('First attempt failed'));

    const { rerender } = render(
      <FileLoader onFileUpload={mockOnFileUpload} onError={mockOnError} />
    );
    
    let fileInput = screen.getByTestId('file-input');
    let validFile = new File(['PDF content'], 'voucher.pdf', { type: 'application/pdf' });
    
    fireEvent.change(fileInput, { target: { files: [validFile] } });
    let uploadBtn = screen.getByTestId('upload-btn');
    fireEvent.click(uploadBtn);
    
    await waitFor(() => {
      const errorMessage = screen.queryByTestId('error-message');
      expect(errorMessage).toBeInTheDocument();
    });

    readFileAsync.mockResolvedValueOnce({ name: 'voucher.pdf', size: 1024 });
    
    const retryBtn = screen.getByTestId('retry-btn');
    fireEvent.click(retryBtn);
    
    uploadBtn = screen.getByTestId('upload-btn');
    fireEvent.click(uploadBtn);
    
    await waitFor(() => {
      const successMessage = screen.queryByTestId('success-message');
      expect(successMessage).toBeInTheDocument();
    });

    const errorMessage = screen.queryByTestId('error-message');
    expect(errorMessage).not.toBeInTheDocument();
  });
});
```

---

## Complete Component Files Reference

**File:** `lab016/src/FileLoader.js` (Final Version)

```js
import React, { useState } from 'react';
import { readFileAsync } from './fileAPI';

function FileLoader({ 
  onFileUpload, 
  onError, 
  maxFileSize = 5 * 1024 * 1024,
  acceptedFormats = ['.pdf']
}) {
  const [error, setError] = useState(null);
  const [success, setSuccess] = useState(false);
  const [loading, setLoading] = useState(false);
  const [fileData, setFileData] = useState(null);
  const [selectedFile, setSelectedFile] = useState(null);

  const validateFile = (file) => {
    const fileExtension = '.' + file.name.split('.').pop().toLowerCase();
    if (!acceptedFormats.includes(fileExtension)) {
      return { valid: false, error: 'Invalid file type. Only PDF files are accepted.' };
    }

    if (file.size > maxFileSize) {
      return { valid: false, error: 'File size exceeds 5 MB limit.' };
    }

    return { valid: true };
  };

  const handleFileChange = (event) => {
    const files = event.target.files;
    
    if (!files || files.length === 0) {
      return;
    }

    const file = files[0];
    const validation = validateFile(file);
    
    if (!validation.valid) {
      setError(validation.error);
      setSuccess(false);
      onError(validation.error);
      setSelectedFile(null);
      return;
    }

    setError(null);
    setSuccess(false);
    setSelectedFile(file);
  };

  const handleFileUpload = async () => {
    if (!selectedFile) {
      onError('No file selected');
      return;
    }

    try {
      setLoading(true);
      setError(null);

      const result = await readFileAsync(selectedFile);

      setFileData(result);
      setSuccess(true);
      setLoading(false);

      onFileUpload(result);
    } catch (err) {
      setLoading(false);
      setError(err.message || 'Failed to process file');
      setSuccess(false);
      onError(err.message || 'Failed to process file');
    }
  };

  return (
    <div data-testid="file-loader-container" className="file-loader">
      <input
        type="file"
        data-testid="file-input"
        onChange={handleFileChange}
        className="file-input"
        accept=".pdf"
      />

      <button 
        data-testid="upload-btn" 
        onClick={handleFileUpload}
        disabled={loading}
        className="upload-btn"
      >
        {loading ? 'Uploading...' : 'Upload'}
      </button>

      {loading && (
        <div data-testid="loading-indicator" className="loading-container">
          <div className="spinner"></div>
          <p className="loading-text">Processing file...</p>
        </div>
      )}

      {error && (
        <div data-testid="error-message" className="error-container">
          <p className="error-text">{error}</p>
          <button 
            data-testid="retry-btn"
            onClick={() => setError(null)}
            className="retry-btn"
          >
            Retry
          </button>
        </div>
      )}

      {success && fileData && (
        <div data-testid="success-message" className="success-container">
          <p className="success-text">File uploaded successfully!</p>
          <p className="file-details">File: {fileData.name} ({fileData.size} bytes)</p>
        </div>
      )}
    </div>
  );
}

export default FileLoader;
```

**File:** `lab016/src/fileAPI.js` (Final Version)

```js
export const readFileAsync = async (file) => {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    
    reader.onload = () => {
      resolve({
        name: file.name,
        size: file.size,
        content: reader.result,
      });
    };
    
    reader.onerror = () => {
      reject(new Error('Failed to read file'));
    };
    
    reader.readAsText(file);
  });
};
```

**File:** `lab016/src/__mocks__/fileAPI.js` (Manual Mock)

```js
module.exports = {
  readFileAsync: jest.fn(),
};
```

---

# Running The Lab

## Running Tests

### Run all FileLoader tests
```bash
npm test -- ./lab016/test/FileLoader.test.js
```

### Run tests in watch mode (re-runs when files change)
```bash
npm test -- ./lab016/test/FileLoader.test.js --watch
```

### Run tests with coverage report
```bash
npm test -- ./lab016/test/FileLoader.test.js --coverage
```

### View coverage report in browser
```bash
# On Windows
start coverage/lcov-report/index.html

# On Mac
open coverage/lcov-report/index.html

# On Linux
xdg-open coverage/lcov-report/index.html
```

### Update snapshots (if any exist)
```bash
npm test -- ./lab016/test/FileLoader.test.js -u
```

### Run specific test file or describe block
```bash
npm test -- ./lab016/test/FileLoader.test.js -t "PASS 1"
```

### Run tests with verbose output (shows all test names)
```bash
npm test -- ./lab016/test/FileLoader.test.js --verbose
```

---

## Build And Browser Run Using Webpack

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
  entry: './lab016/src/index.js',
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
    ],
  },
  plugins: [
    isDevelopment && new ReactRefreshWebpackPlugin(),
    new HtmlWebpackPlugin({
      template: 'lab016/index.html',
    }),
  ].filter(Boolean),
  resolve: {
    extensions: ['.js', '.jsx'],
  },
};
```

4. **Create React entry point `lab016/src/index.js`:**

```js
import React, { useState } from 'react';
import { createRoot } from 'react-dom/client';
import FileLoader from './FileLoader';

function App() {
  const [uploadedFile, setUploadedFile] = useState(null);
  const [uploadError, setUploadError] = useState(null);

  const handleFileUpload = (fileData) => {
    console.log('File uploaded:', fileData);
    setUploadedFile(fileData);
    setUploadError(null);
  };

  const handleError = (error) => {
    console.error('Upload error:', error);
    setUploadError(error);
    setUploadedFile(null);
  };

  return (
    <div style={{ padding: '20px', maxWidth: '600px', margin: '0 auto' }}>
      <h1>Voucher File Upload</h1>
      <FileLoader onFileUpload={handleFileUpload} onError={handleError} />
      
      {uploadedFile && (
        <div style={{ marginTop: '20px', padding: '10px', backgroundColor: '#d4edda', borderRadius: '4px' }}>
          <p><strong>Uploaded File:</strong></p>
          <p>Name: {uploadedFile.name}</p>
          <p>Size: {uploadedFile.size} bytes</p>
        </div>
      )}
    </div>
  );
}

const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

5. **Create `lab016/index.html`:**

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>FileLoader Component - Create Voucher Screen</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #f5f5f5;
      margin: 0;
      padding: 0;
    }

    #root {
      display: flex;
      justify-content: center;
      align-items: flex-start;
      min-height: 100vh;
      padding: 20px;
    }

    .file-loader {
      background: white;
      padding: 20px;
      border-radius: 8px;
      box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
    }

    .file-input {
      padding: 10px;
      margin-bottom: 10px;
      border: 1px solid #ddd;
      border-radius: 4px;
      width: 100%;
      box-sizing: border-box;
    }

    .upload-btn {
      padding: 10px 20px;
      background-color: #007bff;
      color: white;
      border: none;
      border-radius: 4px;
      cursor: pointer;
      font-size: 14px;
      transition: background-color 0.2s;
    }

    .upload-btn:hover:not(:disabled) {
      background-color: #0056b3;
    }

    .upload-btn:disabled {
      background-color: #6c757d;
      cursor: not-allowed;
    }

    .loading-container {
      margin-top: 15px;
      padding: 15px;
      background-color: #e7f3ff;
      border-radius: 4px;
      text-align: center;
    }

    .spinner {
      border: 4px solid #f3f3f3;
      border-top: 4px solid #007bff;
      border-radius: 50%;
      width: 30px;
      height: 30px;
      animation: spin 1s linear infinite;
      margin: 0 auto 10px;
    }

    @keyframes spin {
      0% { transform: rotate(0deg); }
      100% { transform: rotate(360deg); }
    }

    .loading-text {
      margin: 0;
      color: #0056b3;
    }

    .error-container {
      margin-top: 15px;
      padding: 15px;
      background-color: #f8d7da;
      border: 1px solid #f5c6cb;
      border-radius: 4px;
    }

    .error-text {
      margin: 0 0 10px 0;
      color: #721c24;
    }

    .retry-btn {
      padding: 8px 12px;
      background-color: #dc3545;
      color: white;
      border: none;
      border-radius: 4px;
      cursor: pointer;
      font-size: 12px;
      transition: background-color 0.2s;
    }

    .retry-btn:hover {
      background-color: #c82333;
    }

    .success-container {
      margin-top: 15px;
      padding: 15px;
      background-color: #d4edda;
      border: 1px solid #c3e6cb;
      border-radius: 4px;
    }

    .success-text {
      margin: 0 0 10px 0;
      color: #155724;
      font-weight: bold;
    }

    .file-details {
      margin: 0;
      color: #155724;
    }
  </style>
</head>

<body>
  <div id="root"></div>
</body>

</html>
```

6. **Update `package.json` scripts:**

```json
"scripts": {
  "start": "webpack serve --env development",
  "build": "webpack --mode production",
  "test": "jest"
}
```

7. **Run your development server:**

```bash
npm start
```

Your React app should open automatically and display the FileLoader component with hot reloading enabled.

---

## Key Takeaways

1. **Asynchronous Testing** with async/await makes testing Promise-based code clear and readable
2. **Manual Mocks** provide control over external dependencies, enabling deterministic tests
3. **Coverage Reports** identify untested code paths and ensure comprehensive test coverage
4. **Parameterized Tests** reduce code duplication when testing multiple scenarios
5. **Error Handling** should be comprehensive and test both success and failure cases
6. **State Management** in React components requires careful testing of loading and error states
7. **TDD Approach** ensures requirements are met before implementation starts

---

## Troubleshooting

**Tests timeout?**
- Increase Jest timeout: `jest.setTimeout(10000);` at the top of your test file

**Mock not working?**
- Ensure jest.mock() is called BEFORE importing the module to mock
- Use jest.clearAllMocks() in beforeEach to reset mock state

**Async tests hanging?**
- Wrap async operations in waitFor() for proper async handling
- Ensure all Promises are properly awaited

**File input not triggering change?**
- Use fireEvent.change() from React Testing Library
- Ensure the File object is created with proper type

**Coverage report not generating?**
- Run with --coverage flag: `npm test -- ./lab016/test/FileLoader.test.js --coverage`
- Check that coverage directory is writable

