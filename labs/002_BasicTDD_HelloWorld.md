# Basic TDD Hello World Example
# Technologies

Node.js v22.20.0, Jest, jsdom

## Domain Context

A business needs a customer onboarding web form that collects:

- Customer Name (single-line textbox)
- Customer Type (select dropdown)
- Interests (multi-select with checkboxes)

Validation requirements:

- Name: must not be empty (positive/negative)
- Type: must be a valid option, not the placeholder (positive/negative)
- Interests: at least one must be selected (positive/negative)

The lab is structured using the TDD "Red-Green-Refactor" cycle, using Jest and jsdom to simulate DOM environment and user events for modern JavaScript testing.

***

# Summary

This walk-through demonstrates a clean TDD cycle for all three form input types (textbox, select-one, checkbox-multiselect) using Node.js, Jest, and jsdom. Each test is explained, all requirements are covered with positive and negative cases, and code refactoring is encouraged at each cycle. The structure and commentary precisely match the style of the provided onboarding TDD example, ensuring your students get a coherent, progressive TDD experience in JavaScript.



***

## Prerequisites

- Folder structure (assume inside your project root):

```
lab002/
  ├─ src/
  │   └─ customerForm.js
  └─ test/
      └─ customerForm.test.js
```

- jest.config.js should contain:

```js
module.exports={
  testEnvironment:"jsdom",
  roots:["<rootDir>/basic-js-tdd","<rootDir>/lab003"],
  testMatch:["**/test/**/*.js","**/tests/**/*.js","**/__tests__/**/*.js","**/?(*.)+(spec|test).js"],
  coverageDirectory:"coverage",
  clearMocks:!0
};
```

- All dependencies installed: Jest, jsdom

***

## Testable Requirements

### 1. Textbox - Customer Name

- Positive: Accepts a non-empty name.
- Negative: Fails when name is empty.


### 2. Select-One - Customer Type

- Positive: Accepts a selection other than default.
- Negative: Fails when nothing or placeholder is selected.


### 3. Checkbox (Multi-select) - Interests

- Positive: Accepts at least one selection.
- Negative: Fails when none are selected.

***

# PASS 1: Red-Green-Refactor - Customer Name Textbox

### RED: Write Failing Test

File: `lab002/test/customerForm.test.js`

```js
/**
 * We are testing that the Name input accepts non-empty input (positive),
 * and is invalid if empty (negative). We'll simulate input and check the validation logic.
 */
const { validateName } = require('../src/customerForm');

test('should accept a non-empty customer name', () => {
  // Positive: Valid input
  expect(validateName('John Doe')).toBe(true);
});

test('should reject empty customer name', () => {
  // Negative: Empty input should fail
  expect(validateName('')).toBe(false);
});
```

*Expected: Tests fail as `validateName` doesn't exist yet.*

***

### GREEN: Make It Pass

File: `lab002/src/customerForm.js`

```js
// Add simple name validation logic
function validateName(name) {
  // Valid if name is non-empty string and not just whitespace
  return typeof name === 'string' && name.trim().length > 0;
}

module.exports = { validateName };
```

*Run tests: Both should now pass.*
```cmd
cd <your\path\to\tdd-labs-folder>

npm run test .\lab002\test\customerForm.test.js

```


***

### REFACTOR

No complex logic yet; code is clear. Refactor only for style/clarity if desired.

***

# PASS 2: Red-Green-Refactor - Customer Type Select-One

### RED: Add Failing Test

File: `lab002/test/customerForm.test.js`

```js
/**
 * We are testing that the Name input accepts non-empty input (positive),
 * and is invalid if empty (negative). We'll simulate input and check the validation logic.
 */
const { validateName, validateType } = require('../src/customerForm');

test('should accept a non-empty customer name', () => {
  // Positive: Valid input
  expect(validateName('John Doe')).toBe(true);
});

test('should reject empty customer name', () => {
  // Negative: Empty input should fail
  expect(validateName('')).toBe(false);
});


/**
 * Here we test if a non-placeholder customer type is considered valid (positive),
 * and if the default placeholder selection is rejected (negative).
 */

test('should accept valid customer type selection', () => {
  // Typically, the placeholder value is '', actual options are non-empty
  expect(validateType('regular')).toBe(true);
});

test('should reject placeholder customer type selection', () => {
  expect(validateType('')).toBe(false); // Placeholder/empty
});

```

*Expected: Fails as `validateType` doesn't exist*

***

### GREEN: Make It Pass

File: `lab002/src/customerForm.js`

```js
function validateName(name) {
  return typeof name === 'string' && name.trim().length > 0;
}

function validateType(type) {
  // Placeholder is '', any non-empty value is valid
  return typeof type === 'string' && type.trim().length > 0;
}

module.exports = { validateName, validateType };
```

*Run tests: All should now pass.*

***

### REFACTOR

If desired, abstract duplicate checks for string validation.

***

# PASS 3: Red-Green-Refactor - Interests Checkbox (Multi-select)

### RED: Add Failing Test

File: `lab002/test/customerForm.test.js`

```js
const { validateName, validateType, validateInterests } = require('../src/customerForm');

/**
 * For interests, we check positive: array with at least one selection,
 * negative: empty array or not an array is rejected.
 */

test('should accept at least one selected interest', () => {
  expect(validateInterests(['finance'])).toBe(true);
  expect(validateInterests(['tech', 'fashion'])).toBe(true);
});

test('should reject with no interests selected', () => {
  expect(validateInterests([])).toBe(false);
  expect(validateInterests('')).toBe(false);
});
```

*Expected: Fails, `validateInterests` not implemented.*

***

### GREEN: Make It Pass

File: `lab002/src/customerForm.js`

```js
function validateName(name) {
  return typeof name === 'string' && name.trim().length > 0;
}

function validateType(type) {
  return typeof type === 'string' && type.trim().length > 0;
}

function validateInterests(interests) {
  // Must be an array with at least one element
  return Array.isArray(interests) && interests.length > 0;
}

module.exports = { validateName, validateType, validateInterests };
```

*Run tests: All now pass.*

***

### REFACTOR

Refactor to enhance reusability and reduce duplication.


***

# Final Folder Structure

```
lab002/
  ├─ src/
  │   └─ customerForm.js
  └─ test/
      └─ customerForm.test.js
```
