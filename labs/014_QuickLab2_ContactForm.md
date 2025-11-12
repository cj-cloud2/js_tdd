# Lab 014: Contact Form TDD Example Using Node.js, Jest, JSDOM, & React

**Technologies:** Node.js v22.20.0, Jest, jsdom, React

## Domain: Contact Form with Validation

Create and test a Contact Form component with input validation, error messaging, and form submission using Test-Driven Development (TDD). This lab covers essential Jest features including user interaction simulation with React Testing Library, async testing with async/await, and Jest mock functions for callback verification.

---

## Jest Features Covered in This Lab

### 1. **React Testing Library (RTL) User Interaction Simulation**
RTL provides utilities to simulate user interactions like typing, clicking buttons, and form submissions. This allows you to test your components as users would interact with them, making tests more realistic and reliable.

**Key methods covered:**
- `fireEvent.change()` - Simulates typing into input fields
- `fireEvent.click()` - Simulates button clicks
- `fireEvent.submit()` - Simulates form submission

**Why useful:** Tests user interactions rather than implementation details, leading to more maintainable and meaningful tests.

### 2. **Async Testing with async/await**
Many React operations like state updates and event handlers execute asynchronously. Jest supports async testing using `async/await` syntax and RTL utilities like `waitFor()` to handle these scenarios properly.

**Why useful:** Ensures your tests wait for asynchronous operations to complete before making assertions, preventing false positives/negatives.

### 3. **Jest Mock Functions (jest.fn())**
Mock functions allow you to track function calls, verify they're called with correct arguments, and control their behavior during tests. Essential for testing callback props and side effects.

**Key capabilities:**
- Track how many times a function was called
- Verify the arguments passed to the function
- Check if the function was called at all

**Why useful:** Validates that components correctly invoke callbacks with expected data without needing real implementations.

---

## Business Requirements in Gherkin Format

```gherkin
Feature: Contact Form Validation and Submission
  As a user
  I want to submit a contact form with my details
  So that I can send my message to the support team

  Scenario: Display form fields
    Given the contact form is rendered
    Then I should see a "Name" input field
    And I should see an "Email" input field
    And I should see a "Message" textarea field
    And I should see a "Submit" button

  Scenario: Validation for empty required fields
    Given the contact form is rendered
    When I submit the form without filling any fields
    Then I should see an error message "Name is required"
    And I should see an error message "Email is required"
    And I should see an error message "Message is required"

  Scenario: Validation for invalid email format
    Given the contact form is rendered
    When I enter "John" in the name field
    And I enter "invalid-email" in the email field
    And I enter "Hello" in the message field
    And I submit the form
    Then I should see an error message "Email is invalid"

  Scenario: Successful form submission with valid data
    Given the contact form is rendered
    When I enter "John Doe" in the name field
    And I enter "john@example.com" in the email field
    And I enter "This is my message" in the message field
    And I submit the form
    Then the onSubmit callback should be called with the form data
    And all error messages should be cleared

  Scenario: Error messages clear when valid data is entered
    Given the contact form has validation errors
    When I correct all the fields with valid data
    And I submit the form
    Then all error messages should disappear
    And the onSubmit callback should be called
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

- Then Install This
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
npm install --save-dev @testing-library/react
```

### Step 2: Verify/Update Jest Configuration

Ensure your `jest.config.js` in the root folder includes the lab014 path:

```js
module.exports = {
  testEnvironment: "jsdom",
  roots: ["<rootDir>/basic-js-tdd", "<rootDir>/lab003", "<rootDir>/lab014", "<rootDir>/"],
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
<root-folder>/lab014/
    ├── src/
    │   └── ContactForm.js
    └── test/
        └── ContactForm.test.js
```

All production code will go in `src`, all tests go in `test`.

---

## Contact Form Specification

The ContactForm component is a user input form with validation capabilities:

**Component Properties:**
- **onSubmit** (function): Callback function invoked when form is submitted with valid data
  - Receives form data object: `{ name, email, message }`

**Form Fields:**
- **Name** (text input): Required field for user's name
- **Email** (text input): Required field with email format validation
- **Message** (textarea): Required field for user's message

**Validation Rules:**
- Name: Cannot be empty
- Email: Cannot be empty and must be valid email format (must contain @ and . in correct order)
- Message: Cannot be empty

**Visual Structure:**
- Three labeled input fields (Name, Email, Message)
- Submit button
- Error messages displayed below each field when validation fails

---

## Requirements

**Form Rendering:**
- Requirement 1: Form displays all required input fields with correct labels
- Requirement 2: Form displays a submit button

**Input Validation:**
- Requirement 3: Empty name field shows error "Name is required"
- Requirement 4: Empty email field shows error "Email is required"
- Requirement 5: Invalid email format shows error "Email is invalid"
- Requirement 6: Empty message field shows error "Message is required"

**Form Submission:**
- Requirement 7: Form with empty fields displays all error messages on submit
- Requirement 8: Form with valid data calls onSubmit callback with correct data object
- Requirement 9: Successful submission clears all error messages

**User Interaction:**
- Requirement 10: Users can type into all input fields
- Requirement 11: Error messages disappear when valid data is entered and form is resubmitted

---

# TDD Walkthrough (3 Passes)

For each pass, you will:
- Write fail-first tests (red phase)
- Implement just enough code to make tests pass (green phase)
- Clean/refactor if needed (refactor phase)
- Include descriptive comments in test functions

---

## PASS 1: Form Rendering & Basic Structure

**Pass 1 Covers:**
- **Requirements:** Requirement 1, Requirement 2
- **Jest Features:** Basic rendering tests, RTL queries (getByLabelText, getByRole)
- **Gherkin Scenarios:** Display form fields

### RED Phase - Write Failing Tests

**File:** `lab014/test/ContactForm.test.js`

```js
import React from 'react';
import { render, screen, fireEvent } from '@testing-library/react';
import ContactForm from '../src/ContactForm';
import '@testing-library/jest-dom';

// This test suite groups all contact form tests together for PASS 1
describe('ContactForm Component - PASS 1: Form Rendering & Basic Structure', () => {
  
  // In this test, we verify that the name input field is rendered with correct label
  it('should render name input field with label', () => {
    render(<ContactForm onSubmit={() => {}} />);
    const nameInput = screen.getByLabelText(/name/i);
    expect(nameInput).toBeInTheDocument();
    expect(nameInput).toHaveAttribute('type', 'text');
  });

  // In this test, we verify that the email input field is rendered with correct label
  it('should render email input field with label', () => {
    render(<ContactForm onSubmit={() => {}} />);
    const emailInput = screen.getByLabelText(/email/i);
    expect(emailInput).toBeInTheDocument();
    expect(emailInput).toHaveAttribute('type', 'text');
  });

  // In this test, we verify that the message textarea is rendered with correct label
  it('should render message textarea with label', () => {
    render(<ContactForm onSubmit={() => {}} />);
    const messageInput = screen.getByLabelText(/message/i);
    expect(messageInput).toBeInTheDocument();
    expect(messageInput.tagName).toBe('TEXTAREA');
  });

  // In this test, we verify that the submit button is rendered
  it('should render submit button', () => {
    render(<ContactForm onSubmit={() => {}} />);
    const submitButton = screen.getByRole('button', { name: /submit/i });
    expect(submitButton).toBeInTheDocument();
    expect(submitButton).toHaveAttribute('type', 'submit');
  });
});
```

**Run tests from the root folder using:** (Tests should fail - RED)

```bash
npm test -- ./lab014/test/ContactForm.test.js
```

Expected output: 4 failing tests - the ContactForm component doesn't exist yet.

### GREEN Phase - Make Tests Pass

**File:** `lab014/src/ContactForm.js`

```js
import React from 'react';

// ContactForm component for collecting user contact information
// Props: onSubmit (function) - callback invoked with form data when form is submitted with valid data
function ContactForm({ onSubmit }) {
  
  return (
    <form data-testid="contact-form">
      {/* Name Input Field */}
      <div className="form-group">
        <label htmlFor="name">Name:</label>
        <input 
          type="text" 
          id="name" 
          name="name"
          data-testid="name-input"
        />
      </div>

      {/* Email Input Field */}
      <div className="form-group">
        <label htmlFor="email">Email:</label>
        <input 
          type="text" 
          id="email" 
          name="email"
          data-testid="email-input"
        />
      </div>

      {/* Message Textarea Field */}
      <div className="form-group">
        <label htmlFor="message">Message:</label>
        <textarea 
          id="message" 
          name="message"
          data-testid="message-input"
          rows="4"
        />
      </div>

      {/* Submit Button */}
      <button type="submit" data-testid="submit-button">
        Submit
      </button>
    </form>
  );
}

export default ContactForm;
```

**Run tests from the root folder using:** (Tests should pass - GREEN)

```bash
npm test -- ./lab014/test/ContactForm.test.js
```

Expected output: 4 passing tests

### Refactor

- The ContactForm component has a clear structure with semantic HTML
- Each form field is properly labeled for accessibility
- No major refactoring needed at this stage

---

## PASS 2: User Input Simulation & Form Validation

**Pass 2 Covers:**
- **Requirements:** Requirement 3, Requirement 4, Requirement 5, Requirement 6, Requirement 7, Requirement 10
- **Jest Features:** fireEvent for user interactions, mock functions (jest.fn()), testing validation logic
- **Gherkin Scenarios:** Validation for empty required fields, Validation for invalid email format

### RED Phase - Add Failing Tests

**File:** `lab014/test/ContactForm.test.js`

**Important**: Add the following code AFTER the closing brace of the first describe block (after PASS 1 tests), but BEFORE the end of the file. This is a new describe block for PASS 2 tests.

```js
describe('ContactForm Component - PASS 2: User Input & Validation', () => {
  let mockOnSubmit;

  // Setup: Run before EACH test - create fresh mock function
  beforeEach(() => {
    mockOnSubmit = jest.fn();
  });

  // Teardown: Run after EACH test
  afterEach(() => {
    jest.clearAllMocks();
  });

  // In this test, we verify users can type into the name input field
  it('should allow typing into name input', () => {
    render(<ContactForm onSubmit={mockOnSubmit} />);
    const nameInput = screen.getByLabelText(/name/i);
    
    // Simulate user typing into the name field
    fireEvent.change(nameInput, { target: { value: 'John Doe' } });
    
    expect(nameInput.value).toBe('John Doe');
  });

  // In this test, we verify users can type into the email input field
  it('should allow typing into email input', () => {
    render(<ContactForm onSubmit={mockOnSubmit} />);
    const emailInput = screen.getByLabelText(/email/i);
    
    // Simulate user typing into the email field
    fireEvent.change(emailInput, { target: { value: 'john@example.com' } });
    
    expect(emailInput.value).toBe('john@example.com');
  });

  // In this test, we verify users can type into the message textarea
  it('should allow typing into message textarea', () => {
    render(<ContactForm onSubmit={mockOnSubmit} />);
    const messageInput = screen.getByLabelText(/message/i);
    
    // Simulate user typing into the message field
    fireEvent.change(messageInput, { target: { value: 'This is my message' } });
    
    expect(messageInput.value).toBe('This is my message');
  });

  // In this test, we verify that submitting empty form shows all error messages
  it('should show error messages when form is submitted with empty fields', () => {
    render(<ContactForm onSubmit={mockOnSubmit} />);
    const submitButton = screen.getByRole('button', { name: /submit/i });
    
    // Submit form without filling any fields
    fireEvent.click(submitButton);
    
    // Check that all error messages are displayed
    expect(screen.getByText(/name is required/i)).toBeInTheDocument();
    expect(screen.getByText(/email is required/i)).toBeInTheDocument();
    expect(screen.getByText(/message is required/i)).toBeInTheDocument();
    
    // Verify onSubmit was NOT called because form has errors
    expect(mockOnSubmit).not.toHaveBeenCalled();
  });

  // In this test, we verify that invalid email format shows specific error
  it('should show error message for invalid email format', () => {
    render(<ContactForm onSubmit={mockOnSubmit} />);
    
    const nameInput = screen.getByLabelText(/name/i);
    const emailInput = screen.getByLabelText(/email/i);
    const messageInput = screen.getByLabelText(/message/i);
    const submitButton = screen.getByRole('button', { name: /submit/i });
    
    // Fill form with invalid email
    fireEvent.change(nameInput, { target: { value: 'John Doe' } });
    fireEvent.change(emailInput, { target: { value: 'invalid-email' } });
    fireEvent.change(messageInput, { target: { value: 'Hello' } });
    
    // Submit form
    fireEvent.click(submitButton);
    
    // Check that email validation error is displayed
    expect(screen.getByText(/email is invalid/i)).toBeInTheDocument();
    
    // Verify onSubmit was NOT called
    expect(mockOnSubmit).not.toHaveBeenCalled();
  });

  // In this test, we verify that only the empty field shows error when others are filled
  it('should show specific error messages for empty fields only', () => {
    render(<ContactForm onSubmit={mockOnSubmit} />);
    
    const nameInput = screen.getByLabelText(/name/i);
    const submitButton = screen.getByRole('button', { name: /submit/i });
    
    // Fill only name field
    fireEvent.change(nameInput, { target: { value: 'John Doe' } });
    
    // Submit form
    fireEvent.click(submitButton);
    
    // Name error should NOT appear
    expect(screen.queryByText(/name is required/i)).not.toBeInTheDocument();
    
    // Email and Message errors should appear
    expect(screen.getByText(/email is required/i)).toBeInTheDocument();
    expect(screen.getByText(/message is required/i)).toBeInTheDocument();
  });
});
```

**Run tests from the root folder using:** (Tests should fail - RED)

```bash
npm test -- ./lab014/test/ContactForm.test.js
```

Expected output: 6 new failing tests (PASS 1 tests should still pass)

### GREEN Phase - Make Tests Pass

**File:** `lab014/src/ContactForm.js`

**Important**: Replace the entire content of ContactForm.js with the following code:

```js
import React, { useState } from 'react';

// ContactForm component for collecting user contact information with validation
// Props: onSubmit (function) - callback invoked with form data when form is valid
function ContactForm({ onSubmit }) {
  // State to store form field values
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    message: ''
  });

  // State to store validation error messages
  const [errors, setErrors] = useState({
    name: '',
    email: '',
    message: ''
  });

  // Handle input changes - updates formData state when user types
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prevData => ({
      ...prevData,
      [name]: value
    }));
  };

  // Validate email format - checks if email has @ and . in proper order
  const isValidEmail = (email) => {
    const trimmedEmail = email.trim();
    // Must have @ symbol
    if (!trimmedEmail.includes('@')) {
      return false;
    }
    // Must have . after @
    const atIndex = trimmedEmail.indexOf('@');
    const dotIndex = trimmedEmail.lastIndexOf('.');
    if (dotIndex <= atIndex + 1) {
      return false;
    }
    // Must have characters before @
    if (atIndex === 0) {
      return false;
    }
    // Must have characters after @
    if (atIndex === trimmedEmail.length - 1) {
      return false;
    }
    // Must have characters after .
    if (dotIndex === trimmedEmail.length - 1) {
      return false;
    }
    return true;
  };

  // Validate individual field and return error message (empty string if valid)
  const validateField = (fieldName, value) => {
    switch (fieldName) {
      case 'name':
        return value.trim() ? '' : 'Name is required';
      
      case 'email':
        if (!value.trim()) {
          return 'Email is required';
        }
        return isValidEmail(value) ? '' : 'Email is invalid';
      
      case 'message':
        return value.trim() ? '' : 'Message is required';
      
      default:
        return '';
    }
  };

  // Validate all form fields and return true if form is valid
  const validateForm = () => {
    const newErrors = {
      name: validateField('name', formData.name),
      email: validateField('email', formData.email),
      message: validateField('message', formData.message)
    };

    // Update errors state
    setErrors(newErrors);

    // Return true if no errors exist (all error messages are empty strings)
    return Object.values(newErrors).every(error => error === '');
  };

  // Handle form submission
  const handleSubmit = (e) => {
    e.preventDefault();

    // Validate form before submitting
    if (validateForm()) {
      // Call onSubmit callback with form data if validation passes
      onSubmit(formData);
    }
  };

  return (
    <form data-testid="contact-form" onSubmit={handleSubmit}>
      {/* Name Input Field */}
      <div className="form-group">
        <label htmlFor="name">Name:</label>
        <input 
          type="text" 
          id="name" 
          name="name"
          value={formData.name}
          onChange={handleChange}
          data-testid="name-input"
        />
        {/* Display error message if name validation fails */}
        {errors.name && (
          <div className="error-message" data-testid="name-error">
            {errors.name}
          </div>
        )}
      </div>

      {/* Email Input Field */}
      <div className="form-group">
        <label htmlFor="email">Email:</label>
        <input 
          type="text"
          id="email" 
          name="email"
          value={formData.email}
          onChange={handleChange}
          data-testid="email-input"
        />
        {/* Display error message if email validation fails */}
        {errors.email && (
          <div className="error-message" data-testid="email-error">
            {errors.email}
          </div>
        )}
      </div>

      {/* Message Textarea Field */}
      <div className="form-group">
        <label htmlFor="message">Message:</label>
        <textarea 
          id="message" 
          name="message"
          value={formData.message}
          onChange={handleChange}
          data-testid="message-input"
          rows="4"
        />
        {/* Display error message if message validation fails */}
        {errors.message && (
          <div className="error-message" data-testid="message-error">
            {errors.message}
          </div>
        )}
      </div>

      {/* Submit Button */}
      <button type="submit" data-testid="submit-button">
        Submit
      </button>
    </form>
  );
}

export default ContactForm;
```

**Run tests from the root folder using:** (Tests should pass - GREEN)

```bash
npm test -- ./lab014/test/ContactForm.test.js
```

Expected output: 10 passing tests (4 from PASS 1 + 6 from PASS 2)

### Refactor

- Form validation logic is clean and separated into dedicated functions
- State management uses React hooks properly
- Error display is conditional and tied to validation state
- Email validation uses clear string checks for transparency
- No major refactoring needed at this stage

---

## PASS 3: Successful Form Submission & Error Clearing

**Pass 3 Covers:**
- **Requirements:** Requirement 8, Requirement 9, Requirement 11
- **Jest Features:** Testing callback invocation with arguments, verifying state changes after async operations
- **Gherkin Scenarios:** Successful form submission with valid data, Error messages clear when valid data is entered

### RED Phase - Add Failing Tests

**File:** `lab014/test/ContactForm.test.js`

**Important**: Add the following code AFTER the closing brace of the PASS 2 describe block, but BEFORE the end of the file. This is a new describe block for PASS 3 tests.

```js
describe('ContactForm Component - PASS 3: Successful Submission & Error Clearing', () => {
  let mockOnSubmit;

  beforeEach(() => {
    mockOnSubmit = jest.fn();
  });

  afterEach(() => {
    jest.clearAllMocks();
  });

  // In this test, we verify that valid form data calls onSubmit with correct data object
  it('should call onSubmit callback with form data when all fields are valid', () => {
    render(<ContactForm onSubmit={mockOnSubmit} />);
    
    const nameInput = screen.getByLabelText(/name/i);
    const emailInput = screen.getByLabelText(/email/i);
    const messageInput = screen.getByLabelText(/message/i);
    const submitButton = screen.getByRole('button', { name: /submit/i });
    
    // Fill form with valid data
    fireEvent.change(nameInput, { target: { value: 'John Doe' } });
    fireEvent.change(emailInput, { target: { value: 'john@example.com' } });
    fireEvent.change(messageInput, { target: { value: 'This is my message' } });
    
    // Submit form
    fireEvent.click(submitButton);
    
    // Verify onSubmit was called exactly once
    expect(mockOnSubmit).toHaveBeenCalledTimes(1);
    
    // Verify onSubmit was called with correct data object
    expect(mockOnSubmit).toHaveBeenCalledWith({
      name: 'John Doe',
      email: 'john@example.com',
      message: 'This is my message'
    });
  });

  // In this test, we verify that error messages disappear when form becomes valid
  it('should clear error messages when valid data is submitted after errors', () => {
    render(<ContactForm onSubmit={mockOnSubmit} />);
    
    const nameInput = screen.getByLabelText(/name/i);
    const emailInput = screen.getByLabelText(/email/i);
    const messageInput = screen.getByLabelText(/message/i);
    const submitButton = screen.getByRole('button', { name: /submit/i });
    
    // First, submit empty form to trigger errors
    fireEvent.click(submitButton);
    
    // Verify errors are displayed
    expect(screen.getByText(/name is required/i)).toBeInTheDocument();
    expect(screen.getByText(/email is required/i)).toBeInTheDocument();
    expect(screen.getByText(/message is required/i)).toBeInTheDocument();
    
    // Now fill form with valid data
    fireEvent.change(nameInput, { target: { value: 'Jane Smith' } });
    fireEvent.change(emailInput, { target: { value: 'jane@example.com' } });
    fireEvent.change(messageInput, { target: { value: 'Valid message here' } });
    
    // Submit form again
    fireEvent.click(submitButton);
    
    // Verify all error messages are cleared
    expect(screen.queryByText(/name is required/i)).not.toBeInTheDocument();
    expect(screen.queryByText(/email is required/i)).not.toBeInTheDocument();
    expect(screen.queryByText(/message is required/i)).not.toBeInTheDocument();
    
    // Verify onSubmit was called
    expect(mockOnSubmit).toHaveBeenCalledTimes(1);
  });

  // In this test, we verify that email format error clears when valid email is entered
  it('should clear email format error when valid email is provided', () => {
    render(<ContactForm onSubmit={mockOnSubmit} />);
    
    const nameInput = screen.getByLabelText(/name/i);
    const emailInput = screen.getByLabelText(/email/i);
    const messageInput = screen.getByLabelText(/message/i);
    const submitButton = screen.getByRole('button', { name: /submit/i });
    
    // Fill form with invalid email
    fireEvent.change(nameInput, { target: { value: 'Test User' } });
    fireEvent.change(emailInput, { target: { value: 'bad-email' } });
    fireEvent.change(messageInput, { target: { value: 'Some message' } });
    
    // Submit to trigger email validation error
    fireEvent.click(submitButton);
    
    // Verify email error is displayed
    expect(screen.getByText(/email is invalid/i)).toBeInTheDocument();
    
    // Now correct the email
    fireEvent.change(emailInput, { target: { value: 'test@example.com' } });
    
    // Submit form again
    fireEvent.click(submitButton);
    
    // Verify email error is cleared
    expect(screen.queryByText(/email is invalid/i)).not.toBeInTheDocument();
    
    // Verify onSubmit was called
    expect(mockOnSubmit).toHaveBeenCalledTimes(1);
  });

  // In this test, we verify that onSubmit is NOT called multiple times on repeated invalid submissions
  it('should not call onSubmit when form remains invalid after multiple submissions', () => {
    render(<ContactForm onSubmit={mockOnSubmit} />);
    
    const submitButton = screen.getByRole('button', { name: /submit/i });
    
    // Submit empty form multiple times
    fireEvent.click(submitButton);
    fireEvent.click(submitButton);
    fireEvent.click(submitButton);
    
    // Verify onSubmit was never called
    expect(mockOnSubmit).not.toHaveBeenCalled();
    
    // Verify errors still display
    expect(screen.getByText(/name is required/i)).toBeInTheDocument();
  });
});
```

**Run tests from the root folder using:** (Tests should pass - GREEN)

```bash
npm test -- ./lab014/test/ContactForm.test.js
```

The implementation from PASS 2 already handles all these requirements correctly!

Expected output: 14 passing tests (4 from PASS 1 + 6 from PASS 2 + 4 from PASS 3)

### Refactor

The refactored validation logic from PASS 2 is already optimized:
- Validation logic is clean and separated into dedicated functions
- State management uses React hooks properly
- Error display is conditional and tied to validation state
- Email validation uses explicit string checks
- Code is maintainable and testable

No additional refactoring needed.

---

## Running All Tests

To run all tests for this lab:

```bash
npm test -- ./lab014/test/ContactForm.test.js
```

To run tests in watch mode (re-runs when files change):

```bash
npm test -- ./lab014/test/ContactForm.test.js --watch
```

To run tests with coverage report:

```bash
npm test -- ./lab014/test/ContactForm.test.js --coverage
```

**Expected output after all three passes:**

```
 PASS  lab014/test/ContactForm.test.js
  ContactForm Component - PASS 1: Form Rendering & Basic Structure
    √ should render name input field with label (52 ms)
    √ should render email input field with label (9 ms)
    √ should render message textarea with label (11 ms)
    √ should render submit button (72 ms)
  ContactForm Component - PASS 2: User Input & Validation
    √ should allow typing into name input (31 ms)
    √ should allow typing into email input (15 ms)
    √ should allow typing into message textarea (13 ms)
    √ should show error messages when form is submitted with empty fields (95 ms)
    √ should show error message for invalid email format (45 ms)
    √ should show specific error messages for empty fields only (52 ms)
  ContactForm Component - PASS 3: Successful Submission & Error Clearing
    √ should call onSubmit callback with form data when all fields are valid
    √ should clear error messages when valid data is submitted after errors
    √ should clear email format error when valid email is provided
    √ should not call onSubmit when form remains invalid after multiple submissions

Test Suites: 1 passed, 1 total
Tests:       14 passed, 14 total
Snapshots:   0 total
Time:        2.5 s
```

---

## Key Takeaways

1. **User Interaction Simulation**: RTL's `fireEvent` API allows realistic testing of user interactions like typing and clicking
2. **Mock Functions**: Jest mock functions (`jest.fn()`) track callback invocations and verify they receive correct arguments
3. **Form Validation**: Proper validation separates UI logic from business rules, making code testable and maintainable
4. **State Management**: React's `useState` hook manages form data and error states independently
5. **Test Organization**: Grouping tests by feature/pass makes test suites more readable and maintainable
6. **TDD Benefits**: Writing tests first ensures requirements are met and code remains testable throughout development
7. **Email Validation**: Custom validation logic using string manipulation is more transparent than regex for learning purposes

---

## Optional: Adding Basic Styling

To make the form visually appealing, create `lab014/src/ContactForm.css`:

```css
.form-group {
  margin-bottom: 20px;
}

label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
  color: #333;
}

input[type="text"],
textarea {
  width: 100%;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 14px;
  font-family: inherit;
  box-sizing: border-box;
}

input:focus,
textarea:focus {
  outline: none;
  border-color: #007bff;
  box-shadow: 0 0 0 2px rgba(0, 123, 255, 0.1);
}

textarea {
  resize: vertical;
  min-height: 100px;
}

.error-message {
  color: #dc3545;
  font-size: 13px;
  margin-top: 5px;
  font-weight: 500;
}

button[type="submit"] {
  background-color: #007bff;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  font-size: 16px;
  cursor: pointer;
  transition: background-color 0.2s;
}

button[type="submit"]:hover {
  background-color: #0056b3;
}

button[type="submit"]:active {
  background-color: #004085;
}

form[data-testid="contact-form"] {
  max-width: 500px;
  margin: 0 auto;
  padding: 20px;
}
```

Then import it in `lab014/src/ContactForm.js` at the top (after the React import):
```js
import './ContactForm.css';
```

---

## Troubleshooting

**Tests fail with "Cannot find module" errors?**
- Verify babel.config.js and jest.config.js are in the root folder
- Check that all import paths use correct relative paths
- Ensure all node_modules are installed: `npm install`

**fireEvent.change not updating input values?**
- Ensure inputs have `value` prop bound to state
- Verify `onChange` handler is properly updating state
- Make sure imports include `fireEvent`: `import { render, screen, fireEvent } from '@testing-library/react';`

**Mock functions not being called?**
- Check that validation logic returns true for valid forms
- Ensure form submission preventDefault() is called
- Verify onSubmit is only called when validation passes

**Error messages not appearing?**
- Ensure validateForm() resets all error fields
- Check that setErrors() is called with new error state
- Verify error div elements have the correct text content

**Email validation not working?**
- Check that email input has `type="text"` (not `type="email"` which has browser validation)
- Verify isValidEmail() function checks for @ and . symbols
- Test with: `invalid-email` (should fail), `john@example.com` (should pass)

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
  entry: './lab014/src/index.js',
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
      template: 'lab014/index.html',
    }),
  ].filter(Boolean),
  resolve: {
    extensions: ['.js', '.jsx'],
  },
};
```

4. **Install CSS loaders for webpack:**

```bash
npm install --save-dev style-loader css-loader
```

5. **Create React entry point `lab014/src/index.js`:**

```js
import React, { useState } from 'react';
import { createRoot } from 'react-dom/client';
import ContactForm from './ContactForm';

// Demo wrapper to show form submissions
function App() {
  const [submittedData, setSubmittedData] = useState(null);

  const handleSubmit = (formData) => {
    console.log('Form submitted:', formData);
    setSubmittedData(formData);
    alert(`Form submitted successfully!\n\nName: ${formData.name}\nEmail: ${formData.email}\nMessage: ${formData.message}`);
  };

  return (
    <div style={{ padding: '40px', maxWidth: '600px', margin: '0 auto' }}>
      <h1>Contact Form Demo</h1>
      <ContactForm onSubmit={handleSubmit} />
      
      {submittedData && (
        <div style={{ marginTop: '30px', padding: '15px', background: '#e8f5e9', borderRadius: '4px' }}>
          <h3>Last Submission:</h3>
          <p><strong>Name:</strong> {submittedData.name}</p>
          <p><strong>Email:</strong> {submittedData.email}</p>
          <p><strong>Message:</strong> {submittedData.message}</p>
        </div>
      )}
    </div>
  );
}

const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

6. **Create `lab014/index.html`:**

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Contact Form Demo</title>
  <style>
    body {
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen', 'Ubuntu', 'Cantarell', sans-serif;
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      min-height: 100vh;
      margin: 0;
      padding: 20px;
    }
    
    #root {
      background: white;
      border-radius: 8px;
      box-shadow: 0 10px 40px rgba(0,0,0,0.2);
      max-width: 800px;
      margin: 0 auto;
    }
  </style>
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

Your React app should open automatically at http://localhost:8080 and display the Contact Form with hot reloading enabled.

---

## What You've Learned

By completing this lab, you have:

✅ Implemented form validation logic with error messaging
✅ Used React Testing Library to simulate user interactions (typing, clicking)
✅ Created and used Jest mock functions to verify callbacks
✅ Tested asynchronous state updates in React components
✅ Applied TDD methodology across multiple incremental passes
✅ Built a fully functional, validated form component from scratch
✅ Organized tests logically with describe blocks and setup/teardown hooks
✅ Debugged and fixed common testing issues with React components

---

**Congratulations! You've completed Lab 014 - Contact Form TDD Example!**