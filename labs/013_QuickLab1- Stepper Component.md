# Lab 013: Stepper Component TDD Example Using Node.js, Jest, JSDOM, & React

**Technologies:** Node.js v22.20.0, Jest, jsdom, React

## Domain: Stepper Component Walkthrough

Create and test a Stepper (Counter) component with increment/decrement functionality, configurable step values, and DOM state management using Test-Driven Development (TDD). This lab covers essential Jest features including basic matchers for assertions, React Testing Library (RTL) rendering techniques, and component state testing through user interactions.

---

## Jest Features Covered in This Lab

### 1. **Basic Jest Matchers for Assertions**
Jest matchers are functions that verify whether code behaves as expected. They form the foundation of unit testing by comparing actual values against expected values. Common matchers include:
- `expect(value).toBe()` - Tests exact equality (uses === comparison)
- `expect(value).toEqual()` - Tests deep equality for objects/arrays
- `expect(element).toBeInTheDocument()` - Verifies DOM elements exist
- `expect(element).toHaveTextContent()` - Checks element contains specific text
- `expect(mockFn).toHaveBeenCalled()` - Verifies functions were invoked

**Why useful:** Matchers provide clear, readable assertions that make tests self-documenting and easy to understand.

### 2. **React Testing Library (RTL) Rendering**
React Testing Library provides utilities to render React components in a test environment that simulates browser behavior. Key functions include:
- `render()` - Mounts a React component and returns utilities to query it
- `screen.getByTestId()` - Finds elements by data-testid attribute
- `screen.getByText()` - Finds elements by text content
- `screen.queryByTestId()` - Safely queries elements (returns null if not found, doesn't throw errors)

**Why useful:** RTL encourages testing components the way users interact with them (through the DOM) rather than testing implementation details.

### 3. **Testing Component State Through User Interactions with `act()`**
Instead of directly manipulating component state, we simulate user actions (like button clicks) and verify the DOM updates accordingly. The `act()` function from React Testing Library wraps state-updating operations to ensure React processes them synchronously during tests:
- Simulate button clicks with `.click()`
- Wrap interactions with `act()` from React Testing Library
- Verify DOM text content updates after interactions
- Test that component props affect rendered output

**Why useful:** Tests become more maintainable and reflect how actual users interact with components. The `act()` wrapper ensures state updates are processed before assertions run.

---

## Prerequisites

- You must already have Node.js, Jest, and JSDOM installed and working with a valid `jest.config.js`
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

- Now React (should have been done in previous assignments)
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

Ensure your `jest.config.js` in the root folder includes the lab013 path:

```js
module.exports = {
  testEnvironment: "jsdom",
  roots: ["<rootDir>/basic-js-tdd", "<rootDir>/lab003", "<rootDir>/lab013", "<rootDir>/"],
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
<root-folder>/lab013/
    ├── src/
    │   └── Stepper.js
    └── test/
        └── Stepper.test.js
```

All production code will go in `src`, all tests go in `test`.

---

## Stepper Component Specification

The Stepper component is a reusable counter widget for applications that need to increment/decrement numeric values. It features:

**Component Properties:**
- **initialValue** (number, optional): Starting value for the counter (default: 0)
- **step** (number, optional): Amount to increment/decrement per click (default: 1, can be set via data-step attribute)
- **onValueChange** (function, optional): Callback when value changes

**Visual Structure:**
- Display area showing current counter value
- Decrement button (labeled "-")
- Increment button (labeled "+")
- Support for custom step values via data attribute

---

## Business Requirements (Gherkin Format)

```gherkin
Feature: Stepper Component Functionality

  Scenario: Display initial counter value
    Given a Stepper component is rendered
    When no initial value is provided
    Then the counter should display 0

  Scenario: Increment counter with default step
    Given a Stepper component is displayed with value 0
    When the increment button is clicked
    Then the counter value should increase by 1
    And the display should show the new value

  Scenario: Decrement counter with default step
    Given a Stepper component is displayed with value 5
    When the decrement button is clicked
    Then the counter value should decrease by 1
    And the display should show the new value

  Scenario: Support custom step value
    Given a Stepper component with step value of 5
    When the increment button is clicked
    Then the counter should increase by 5
    And the decrement button should decrease by 5

  Scenario: Trigger callback on value change
    Given a Stepper component with a change callback
    When the increment button is clicked
    Then the callback should be called with the new value

  Scenario: Initial value configuration
    Given a Stepper component with initialValue set to 10
    Then the counter should display 10
```

---

# TDD Walkthrough (3 Passes)

For each pass, you will:
- Write fail-first tests (RED phase)
- Implement just enough code to make tests pass (GREEN phase)
- Clean/refactor if needed (REFACTOR phase)
- Include descriptive comments in test functions

---

## PASS 1: Initial Display & Basic Rendering

### Business Requirements Addressed in PASS 1
```gherkin
Scenario: Display initial counter value
  Given a Stepper component is rendered
  When no initial value is provided
  Then the counter should display 0

Scenario: Initial value configuration
  Given a Stepper component with initialValue set to 10
  Then the counter should display 10
```

### Jest Features Covered in PASS 1
- **Basic Jest Matchers**: `toBe()`, `toBeInTheDocument()`, `toHaveTextContent()`
- **React Testing Library Rendering**: `render()`, `screen.getByTestId()`, `screen.queryByTestId()`

### RED Phase - Write Failing Tests

**File:** `lab013/test/Stepper.test.js`

```js
import React from 'react';
import { render, screen, act } from '@testing-library/react';
import Stepper from '../src/Stepper';
import '@testing-library/jest-dom';

// PASS 1: Initial Display & Basic Rendering
describe('Stepper Component - PASS 1: Initial Display & Basic Rendering', () => {
  // Setup: This runs BEFORE each individual test in this describe block
  beforeEach(() => {
    // Clear any previous renders or state
    jest.clearAllMocks();
  });

  // Teardown: This runs AFTER each individual test in this describe block
  afterEach(() => {
    // Additional cleanup if needed (DOM elements are automatically cleaned)
  });

  // In this test, we verify that the counter displays 0 when no initial value is provided
  it('should render counter with initial value of 0 by default', () => {
    render(<Stepper />);
    const counterDisplay = screen.getByTestId('counter-display');
    expect(counterDisplay).toHaveTextContent('0');
  });

  // In this test, we verify the Stepper component renders in the DOM
  it('should render Stepper component successfully', () => {
    render(<Stepper />);
    const stepperContainer = screen.getByTestId('stepper-container');
    expect(stepperContainer).toBeInTheDocument();
  });

  // In this test, we verify that custom initial values are displayed correctly
  it('should render counter with custom initial value', () => {
    render(<Stepper initialValue={10} />);
    const counterDisplay = screen.getByTestId('counter-display');
    expect(counterDisplay).toHaveTextContent('10');
  });

  // In this test, we verify that increment and decrement buttons are present
  it('should render both increment and decrement buttons', () => {
    render(<Stepper />);
    const incrementBtn = screen.getByTestId('increment-btn');
    const decrementBtn = screen.getByTestId('decrement-btn');
    expect(incrementBtn).toBeInTheDocument();
    expect(decrementBtn).toBeInTheDocument();
  });

  // In this test, we verify button labels are correct
  it('should have correct labels on buttons', () => {
    render(<Stepper />);
    expect(screen.getByTestId('increment-btn')).toHaveTextContent('+');
    expect(screen.getByTestId('decrement-btn')).toHaveTextContent('-');
  });
});
```

**Run tests from the root folder using:** (Tests should fail - RED)

```bash
npm test -- ./lab013/test/Stepper.test.js
```

Expected output: 5 failing tests - the Stepper component doesn't exist yet.

### GREEN Phase - Make Tests Pass

**File:** `lab013/src/Stepper.js`

```js
import React, { useState } from 'react';

// Stepper component for incrementing/decrementing numeric values
// Props:
//   - initialValue (number, optional): Starting value (default: 0)
//   - step (number, optional): Amount to increment/decrement (default: 1)
//   - onValueChange (function, optional): Callback when value changes
function Stepper({ initialValue = 0, step = 1, onValueChange }) {
  // State to track the current counter value
  const [count, setCount] = useState(initialValue);

  // Handler for increment button - increases count by step value
  const handleIncrement = () => {
    const newValue = count + step;
    setCount(newValue);
    // Call the optional callback if provided
    if (onValueChange) {
      onValueChange(newValue);
    }
  };

  // Handler for decrement button - decreases count by step value
  const handleDecrement = () => {
    const newValue = count - step;
    setCount(newValue);
    // Call the optional callback if provided
    if (onValueChange) {
      onValueChange(newValue);
    }
  };

  // Render the stepper component with display and buttons
  return (
    <div data-testid="stepper-container" className="stepper">
      {/* Display current counter value */}
      <div data-testid="counter-display" className="counter-display">
        {count}
      </div>

      {/* Button container for increment and decrement */}
      <div className="button-group">
        {/* Decrement button */}
        <button 
          data-testid="decrement-btn" 
          onClick={handleDecrement}
          className="btn btn-decrement"
        >
          -
        </button>

        {/* Increment button */}
        <button 
          data-testid="increment-btn" 
          onClick={handleIncrement}
          className="btn btn-increment"
        >
          +
        </button>
      </div>
    </div>
  );
}

export default Stepper;
```

**Run tests from the root folder using:** (Tests should pass - GREEN)

```bash
npm test -- ./lab013/test/Stepper.test.js
```

Expected output: 5 passing tests

### Refactor

- The Stepper component is clean and focused on rendering and state management
- Component structure is simple and maintainable
- No major refactoring needed at this stage

---

## PASS 2: User Interactions & State Updates

### Business Requirements Addressed in PASS 2
```gherkin
Scenario: Increment counter with default step
  Given a Stepper component is displayed with value 0
  When the increment button is clicked
  Then the counter value should increase by 1
  And the display should show the new value

Scenario: Decrement counter with default step
  Given a Stepper component is displayed with value 5
  When the decrement button is clicked
  Then the counter value should decrease by 1
  And the display should show the new value
```

### Jest Features Covered in PASS 2
- **User Interaction Testing with `act()`**: Wrapping button clicks and state updates with the `act()` function
- **DOM State Verification**: Checking that DOM updates after user interactions
- **Jest Matchers**: `toHaveTextContent()` for verifying updated values
- **React's `act()` Wrapper**: Ensures state updates are processed before assertions

**⚠️ IMPORTANT NOTE:** When testing React state updates (like button clicks), we MUST wrap the interaction with the `act()` function from React Testing Library. This tells React to process the state update synchronously before the next assertion. This prevents the "An update to Stepper inside a test was not wrapped in act(...)" warning.

### RED Phase - Add Failing Tests

**File:** `lab013/test/Stepper.test.js`

```js
// **Important**: Add the following code AFTER the closing brace of the PASS 1 describe block,
// but BEFORE the end of the file. This is a new describe block for PASS 2 tests.

describe('Stepper Component - PASS 2: User Interactions & State Updates', () => {
  // Setup: Run before EACH test
  beforeEach(() => {
    jest.clearAllMocks();
  });

  // Teardown: Run after EACH test
  afterEach(() => {
    // Cleanup
  });

  // In this test, we verify that clicking the increment button increases the counter by 1
  it('should increment counter by 1 when increment button is clicked', () => {
    render(<Stepper initialValue={0} />);
    
    // Get the button and display elements
    const incrementBtn = screen.getByTestId('increment-btn');
    const counterDisplay = screen.getByTestId('counter-display');

    // Verify initial value is 0
    expect(counterDisplay).toHaveTextContent('0');

    // Simulate user clicking the increment button
    // CRITICAL: Wrap the click in act() to ensure React processes the state update before assertions
    act(() => {
      incrementBtn.click();
    });

    // Verify the display updated to 1
    expect(counterDisplay).toHaveTextContent('1');
  });

  // In this test, we verify that clicking the decrement button decreases the counter by 1
  it('should decrement counter by 1 when decrement button is clicked', () => {
    render(<Stepper initialValue={5} />);
    
    // Get the button and display elements
    const decrementBtn = screen.getByTestId('decrement-btn');
    const counterDisplay = screen.getByTestId('counter-display');

    // Verify initial value is 5
    expect(counterDisplay).toHaveTextContent('5');

    // Simulate user clicking the decrement button
    // CRITICAL: Wrap the click in act() to ensure React processes the state update before assertions
    act(() => {
      decrementBtn.click();
    });

    // Verify the display updated to 4
    expect(counterDisplay).toHaveTextContent('4');
  });

  // In this test, we verify multiple clicks update the counter correctly
  it('should handle multiple increments correctly', () => {
    render(<Stepper initialValue={0} />);
    const incrementBtn = screen.getByTestId('increment-btn');
    const counterDisplay = screen.getByTestId('counter-display');

    // Click increment button 3 times
    // CRITICAL: Wrap each click in act() to ensure React processes each state update
    act(() => {
      incrementBtn.click();
    });
    expect(counterDisplay).toHaveTextContent('1');

    act(() => {
      incrementBtn.click();
    });
    expect(counterDisplay).toHaveTextContent('2');

    act(() => {
      incrementBtn.click();
    });
    expect(counterDisplay).toHaveTextContent('3');
  });

  // In this test, we verify mixed increment and decrement operations
  it('should handle mixed increment and decrement operations', () => {
    render(<Stepper initialValue={10} />);
    const incrementBtn = screen.getByTestId('increment-btn');
    const decrementBtn = screen.getByTestId('decrement-btn');
    const counterDisplay = screen.getByTestId('counter-display');

    // Start at 10, increment to 11
    // CRITICAL: Wrap the click in act() to ensure React processes the state update
    act(() => {
      incrementBtn.click();
    });
    expect(counterDisplay).toHaveTextContent('11');

    // Decrement to 10
    // CRITICAL: Wrap the click in act() to ensure React processes the state update
    act(() => {
      decrementBtn.click();
    });
    expect(counterDisplay).toHaveTextContent('10');

    // Decrement to 9
    // CRITICAL: Wrap the click in act() to ensure React processes the state update
    act(() => {
      decrementBtn.click();
    });
    expect(counterDisplay).toHaveTextContent('9');
  });
});
```

**Run tests from the root folder using:** (Tests should fail - RED)

```bash
npm test -- ./lab013/test/Stepper.test.js
```

Expected output: 4 new failing tests (PASS 1 tests should still pass)

### GREEN Phase - Make Tests Pass

The Stepper component from PASS 1 already handles all these requirements correctly because we implemented proper state management with `useState()` and event handlers in the initial version. The `act()` wrapper in our tests now tells React to process state updates synchronously.

**Run tests from the root folder using:** (All tests should pass - GREEN)

```bash
npm test -- ./lab013/test/Stepper.test.js
```

Expected output: 9 passing tests (5 from PASS 1 + 4 from PASS 2)
- No more "An update to Stepper inside a test was not wrapped in act(...)" warnings

### Refactor

- Both describe blocks are well-organized with clear setup/teardown
- Event handlers are clean and focused
- Component logic remains maintainable
- The `act()` wrapper ensures tests are robust and follow React Testing Library best practices

---

## PASS 3: Custom Step Values & Callbacks

### Business Requirements Addressed in PASS 3
```gherkin
Scenario: Support custom step value
  Given a Stepper component with step value of 5
  When the increment button is clicked
  Then the counter should increase by 5
  And the decrement button should decrease by 5

Scenario: Trigger callback on value change
  Given a Stepper component with a change callback
  When the increment button is clicked
  Then the callback should be called with the new value
```

### Jest Features Covered in PASS 3
- **Mock Functions**: `jest.fn()` for tracking callback invocations
- **Callback Testing**: Verifying functions are called with correct arguments
- **Jest Matcher**: `toHaveBeenCalledWith()` for asserting function call arguments
- **Wrapping Interactions with `act()`**: All user interactions are still wrapped in `act()`

### RED Phase - Add Failing Tests

**File:** `lab013/test/Stepper.test.js`

```js
// **Important**: Add the following code AFTER the closing brace of the PASS 2 describe block,
// but BEFORE the end of the file. This is a new describe block for PASS 3 tests.

describe('Stepper Component - PASS 3: Custom Step Values & Callbacks', () => {
  // Setup: Run before EACH test
  beforeEach(() => {
    jest.clearAllMocks();
  });

  // Teardown: Run after EACH test
  afterEach(() => {
    // Cleanup
  });

  // In this test, we verify that a custom step value works for increments
  it('should increment counter by custom step value', () => {
    render(<Stepper initialValue={0} step={5} />);
    const incrementBtn = screen.getByTestId('increment-btn');
    const counterDisplay = screen.getByTestId('counter-display');

    // Verify initial value is 0
    expect(counterDisplay).toHaveTextContent('0');

    // Click increment - should add 5
    // CRITICAL: Wrap the click in act() to ensure React processes the state update
    act(() => {
      incrementBtn.click();
    });
    expect(counterDisplay).toHaveTextContent('5');

    // Click increment again - should add 5 more (total 10)
    // CRITICAL: Wrap the click in act() to ensure React processes the state update
    act(() => {
      incrementBtn.click();
    });
    expect(counterDisplay).toHaveTextContent('10');
  });

  // In this test, we verify that a custom step value works for decrements
  it('should decrement counter by custom step value', () => {
    render(<Stepper initialValue={20} step={3} />);
    const decrementBtn = screen.getByTestId('decrement-btn');
    const counterDisplay = screen.getByTestId('counter-display');

    // Verify initial value is 20
    expect(counterDisplay).toHaveTextContent('20');

    // Click decrement - should subtract 3
    // CRITICAL: Wrap the click in act() to ensure React processes the state update
    act(() => {
      decrementBtn.click();
    });
    expect(counterDisplay).toHaveTextContent('17');

    // Click decrement again - should subtract 3 more (total 14)
    // CRITICAL: Wrap the click in act() to ensure React processes the state update
    act(() => {
      decrementBtn.click();
    });
    expect(counterDisplay).toHaveTextContent('14');
  });

  // In this test, we verify that the onValueChange callback is called when incrementing
  it('should call onValueChange callback when increment button is clicked', () => {
    // Create a mock function to track callback invocations
    const mockOnValueChange = jest.fn();
    render(<Stepper initialValue={0} onValueChange={mockOnValueChange} />);
    
    const incrementBtn = screen.getByTestId('increment-btn');

    // Click the increment button
    // CRITICAL: Wrap the click in act() to ensure React processes the state update
    act(() => {
      incrementBtn.click();
    });

    // Verify the callback was called once
    expect(mockOnValueChange).toHaveBeenCalledTimes(1);
    // Verify the callback was called with the new value (1)
    expect(mockOnValueChange).toHaveBeenCalledWith(1);
  });

  // In this test, we verify that the onValueChange callback is called when decrementing
  it('should call onValueChange callback when decrement button is clicked', () => {
    // Create a mock function to track callback invocations
    const mockOnValueChange = jest.fn();
    render(<Stepper initialValue={5} onValueChange={mockOnValueChange} />);
    
    const decrementBtn = screen.getByTestId('decrement-btn');

    // Click the decrement button
    // CRITICAL: Wrap the click in act() to ensure React processes the state update
    act(() => {
      decrementBtn.click();
    });

    // Verify the callback was called once
    expect(mockOnValueChange).toHaveBeenCalledTimes(1);
    // Verify the callback was called with the new value (4)
    expect(mockOnValueChange).toHaveBeenCalledWith(4);
  });

  // In this test, we verify that callback receives correct values with custom step
  it('should call onValueChange callback with correct value when using custom step', () => {
    // Create a mock function to track callback invocations
    const mockOnValueChange = jest.fn();
    render(<Stepper initialValue={0} step={5} onValueChange={mockOnValueChange} />);
    
    const incrementBtn = screen.getByTestId('increment-btn');

    // Click the increment button
    // CRITICAL: Wrap the click in act() to ensure React processes the state update
    act(() => {
      incrementBtn.click();
    });

    // Verify the callback was called with the correct new value (0 + 5 = 5)
    expect(mockOnValueChange).toHaveBeenCalledWith(5);
  });

  // In this test, we verify that callback is not called without the prop
  it('should not throw error when onValueChange callback is not provided', () => {
    // This test verifies the component handles missing callback gracefully
    render(<Stepper initialValue={0} />);
    const incrementBtn = screen.getByTestId('increment-btn');

    // Should not throw error when clicking without callback
    // CRITICAL: Wrap the click in act() to ensure React processes the state update
    expect(() => {
      act(() => {
        incrementBtn.click();
      });
    }).not.toThrow();
  });
});
```

**Run tests from the root folder using:** (Tests should fail - RED)

```bash
npm test -- ./lab013/test/Stepper.test.js
```

Expected output: 6 new failing tests (PASS 1 and PASS 2 tests should still pass)

### GREEN Phase - Make Tests Pass

The Stepper component from PASS 1 already handles all these requirements because:
1. We pass `step` prop to handlers
2. We already call `onValueChange` callback with new values
3. We guard the callback with an if statement to prevent errors if not provided

**Run tests from the root folder using:** (All tests should pass - GREEN)

```bash
npm test -- ./lab013/test/Stepper.test.js
```

Expected output: 15 passing tests total
- 5 from PASS 1
- 4 from PASS 2
- 6 from PASS 3

### Refactor

- Component remains clean and well-structured
- All handlers properly manage state and callbacks
- All tests use `act()` wrapper to ensure React processes state updates before assertions
- No major refactoring needed

---

## Complete Test File Reference

**File:** `lab013/test/Stepper.test.js` (Final Version)

```js
import React from 'react';
import { render, screen, act } from '@testing-library/react';
import Stepper from '../src/Stepper';
import '@testing-library/jest-dom';

// PASS 1: Initial Display & Basic Rendering
describe('Stepper Component - PASS 1: Initial Display & Basic Rendering', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  afterEach(() => {
    // Cleanup
  });

  it('should render counter with initial value of 0 by default', () => {
    render(<Stepper />);
    const counterDisplay = screen.getByTestId('counter-display');
    expect(counterDisplay).toHaveTextContent('0');
  });

  it('should render Stepper component successfully', () => {
    render(<Stepper />);
    const stepperContainer = screen.getByTestId('stepper-container');
    expect(stepperContainer).toBeInTheDocument();
  });

  it('should render counter with custom initial value', () => {
    render(<Stepper initialValue={10} />);
    const counterDisplay = screen.getByTestId('counter-display');
    expect(counterDisplay).toHaveTextContent('10');
  });

  it('should render both increment and decrement buttons', () => {
    render(<Stepper />);
    const incrementBtn = screen.getByTestId('increment-btn');
    const decrementBtn = screen.getByTestId('decrement-btn');
    expect(incrementBtn).toBeInTheDocument();
    expect(decrementBtn).toBeInTheDocument();
  });

  it('should have correct labels on buttons', () => {
    render(<Stepper />);
    expect(screen.getByTestId('increment-btn')).toHaveTextContent('+');
    expect(screen.getByTestId('decrement-btn')).toHaveTextContent('-');
  });
});

// PASS 2: User Interactions & State Updates
describe('Stepper Component - PASS 2: User Interactions & State Updates', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  afterEach(() => {
    // Cleanup
  });

  it('should increment counter by 1 when increment button is clicked', () => {
    render(<Stepper initialValue={0} />);
    
    const incrementBtn = screen.getByTestId('increment-btn');
    const counterDisplay = screen.getByTestId('counter-display');

    expect(counterDisplay).toHaveTextContent('0');

    act(() => {
      incrementBtn.click();
    });

    expect(counterDisplay).toHaveTextContent('1');
  });

  it('should decrement counter by 1 when decrement button is clicked', () => {
    render(<Stepper initialValue={5} />);
    
    const decrementBtn = screen.getByTestId('decrement-btn');
    const counterDisplay = screen.getByTestId('counter-display');

    expect(counterDisplay).toHaveTextContent('5');

    act(() => {
      decrementBtn.click();
    });

    expect(counterDisplay).toHaveTextContent('4');
  });

  it('should handle multiple increments correctly', () => {
    render(<Stepper initialValue={0} />);
    const incrementBtn = screen.getByTestId('increment-btn');
    const counterDisplay = screen.getByTestId('counter-display');

    act(() => {
      incrementBtn.click();
    });
    expect(counterDisplay).toHaveTextContent('1');

    act(() => {
      incrementBtn.click();
    });
    expect(counterDisplay).toHaveTextContent('2');

    act(() => {
      incrementBtn.click();
    });
    expect(counterDisplay).toHaveTextContent('3');
  });

  it('should handle mixed increment and decrement operations', () => {
    render(<Stepper initialValue={10} />);
    const incrementBtn = screen.getByTestId('increment-btn');
    const decrementBtn = screen.getByTestId('decrement-btn');
    const counterDisplay = screen.getByTestId('counter-display');

    act(() => {
      incrementBtn.click();
    });
    expect(counterDisplay).toHaveTextContent('11');

    act(() => {
      decrementBtn.click();
    });
    expect(counterDisplay).toHaveTextContent('10');

    act(() => {
      decrementBtn.click();
    });
    expect(counterDisplay).toHaveTextContent('9');
  });
});

// PASS 3: Custom Step Values & Callbacks
describe('Stepper Component - PASS 3: Custom Step Values & Callbacks', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  afterEach(() => {
    // Cleanup
  });

  it('should increment counter by custom step value', () => {
    render(<Stepper initialValue={0} step={5} />);
    const incrementBtn = screen.getByTestId('increment-btn');
    const counterDisplay = screen.getByTestId('counter-display');

    expect(counterDisplay).toHaveTextContent('0');

    act(() => {
      incrementBtn.click();
    });
    expect(counterDisplay).toHaveTextContent('5');

    act(() => {
      incrementBtn.click();
    });
    expect(counterDisplay).toHaveTextContent('10');
  });

  it('should decrement counter by custom step value', () => {
    render(<Stepper initialValue={20} step={3} />);
    const decrementBtn = screen.getByTestId('decrement-btn');
    const counterDisplay = screen.getByTestId('counter-display');

    expect(counterDisplay).toHaveTextContent('20');

    act(() => {
      decrementBtn.click();
    });
    expect(counterDisplay).toHaveTextContent('17');

    act(() => {
      decrementBtn.click();
    });
    expect(counterDisplay).toHaveTextContent('14');
  });

  it('should call onValueChange callback when increment button is clicked', () => {
    const mockOnValueChange = jest.fn();
    render(<Stepper initialValue={0} onValueChange={mockOnValueChange} />);
    
    const incrementBtn = screen.getByTestId('increment-btn');

    act(() => {
      incrementBtn.click();
    });

    expect(mockOnValueChange).toHaveBeenCalledTimes(1);
    expect(mockOnValueChange).toHaveBeenCalledWith(1);
  });

  it('should call onValueChange callback when decrement button is clicked', () => {
    const mockOnValueChange = jest.fn();
    render(<Stepper initialValue={5} onValueChange={mockOnValueChange} />);
    
    const decrementBtn = screen.getByTestId('decrement-btn');

    act(() => {
      decrementBtn.click();
    });

    expect(mockOnValueChange).toHaveBeenCalledTimes(1);
    expect(mockOnValueChange).toHaveBeenCalledWith(4);
  });

  it('should call onValueChange callback with correct value when using custom step', () => {
    const mockOnValueChange = jest.fn();
    render(<Stepper initialValue={0} step={5} onValueChange={mockOnValueChange} />);
    
    const incrementBtn = screen.getByTestId('increment-btn');

    act(() => {
      incrementBtn.click();
    });

    expect(mockOnValueChange).toHaveBeenCalledWith(5);
  });

  it('should not throw error when onValueChange callback is not provided', () => {
    render(<Stepper initialValue={0} />);
    const incrementBtn = screen.getByTestId('increment-btn');

    expect(() => {
      act(() => {
        incrementBtn.click();
      });
    }).not.toThrow();
  });
});
```

---

## Complete Component File Reference

**File:** `lab013/src/Stepper.js` (Final Version - No Changes Needed)

```js
import React, { useState } from 'react';

// Stepper component for incrementing/decrementing numeric values
// Props:
//   - initialValue (number, optional): Starting value (default: 0)
//   - step (number, optional): Amount to increment/decrement (default: 1)
//   - onValueChange (function, optional): Callback when value changes
function Stepper({ initialValue = 0, step = 1, onValueChange }) {
  // State to track the current counter value
  const [count, setCount] = useState(initialValue);

  // Handler for increment button - increases count by step value
  const handleIncrement = () => {
    const newValue = count + step;
    setCount(newValue);
    // Call the optional callback if provided
    if (onValueChange) {
      onValueChange(newValue);
    }
  };

  // Handler for decrement button - decreases count by step value
  const handleDecrement = () => {
    const newValue = count - step;
    setCount(newValue);
    // Call the optional callback if provided
    if (onValueChange) {
      onValueChange(newValue);
    }
  };

  // Render the stepper component with display and buttons
  return (
    <div data-testid="stepper-container" className="stepper">
      {/* Display current counter value */}
      <div data-testid="counter-display" className="counter-display">
        {count}
      </div>

      {/* Button container for increment and decrement */}
      <div className="button-group">
        {/* Decrement button */}
        <button 
          data-testid="decrement-btn" 
          onClick={handleDecrement}
          className="btn btn-decrement"
        >
          -
        </button>

        {/* Increment button */}
        <button 
          data-testid="increment-btn" 
          onClick={handleIncrement}
          className="btn btn-increment"
        >
          +
        </button>
      </div>
    </div>
  );
}

export default Stepper;
```

---

## Understanding the `act()` Function

React's `act()` function is critical for testing React components properly. Here's what you need to know:

### What Does `act()` Do?

The `act()` function wraps interactions that cause state updates and ensures React processes those updates **synchronously** in the test environment. Without it, React may batch updates asynchronously, causing your assertions to run before state updates complete.

### When to Use `act()`

Use `act()` whenever you:
- Simulate user interactions (button clicks, form input)
- Trigger state changes
- Trigger side effects that update the component
- Call functions that modify component behavior

### How to Use `act()`

```js
// ❌ WRONG - Will cause warnings
incrementBtn.click();
expect(counterDisplay).toHaveTextContent('1');

// ✅ CORRECT - Wrap interaction in act()
act(() => {
  incrementBtn.click();
});
expect(counterDisplay).toHaveTextContent('1');
```

### Import `act()` from React Testing Library

```js
import { render, screen, act } from '@testing-library/react';
```

---

## Running All Tests

To run all tests for this lab:

```bash
npm test -- ./lab013/test/Stepper.test.js
```

To run tests in watch mode (re-runs when files change):

```bash
npm test -- ./lab013/test/Stepper.test.js --watch
```

To run tests with coverage report:

```bash
npm test -- ./lab013/test/Stepper.test.js --coverage
```

To run only a specific test:

```bash
npm test -- ./lab013/test/Stepper.test.js -t "should increment counter"
```

---

## Key Takeaways

1. **Basic Jest Matchers** form the foundation of assertions and make tests readable
2. **React Testing Library** provides user-centric testing by querying the DOM like users do
3. **State Testing** is accomplished through user interactions wrapped in `act()` rather than directly manipulating component state
4. **The `act()` Function** ensures React processes state updates synchronously during tests, preventing warnings and flaky tests
5. **Mock Functions** track callback invocations and their arguments
6. **TDD Approach** ensures code is testable and requirements are met before implementation
7. **Test Organization** with describe blocks and clear naming makes tests self-documenting

---

## Optional: Adding Basic Styling

To make the stepper visually complete, create `lab013/src/Stepper.css`:

```css
.stepper {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
  padding: 30px;
  border: 2px solid #007bff;
  border-radius: 8px;
  background-color: #f8f9fa;
  max-width: 300px;
  margin: 20px auto;
  font-family: Arial, sans-serif;
}

.counter-display {
  font-size: 48px;
  font-weight: bold;
  color: #007bff;
  background-color: white;
  border: 2px solid #dee2e6;
  border-radius: 6px;
  padding: 20px 40px;
  min-width: 150px;
  text-align: center;
}

.button-group {
  display: flex;
  gap: 15px;
  justify-content: center;
}

.btn {
  width: 50px;
  height: 50px;
  font-size: 24px;
  font-weight: bold;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
}

.btn-increment {
  background-color: #28a745;
  color: white;
}

.btn-increment:hover {
  background-color: #218838;
  transform: scale(1.05);
}

.btn-increment:active {
  transform: scale(0.95);
}

.btn-decrement {
  background-color: #dc3545;
  color: white;
}

.btn-decrement:hover {
  background-color: #c82333;
  transform: scale(1.05);
}

.btn-decrement:active {
  transform: scale(0.95);
}
```

Then import it in Stepper.js:
```js
import './Stepper.css';
```

---

## Troubleshooting

**Tests fail with "An update to Stepper inside a test was not wrapped in act(...)"?**
- Wrap all button clicks and state-changing interactions with `act()`
- Import `act` from `@testing-library/react`: `import { render, screen, act } from '@testing-library/react'`
- Example: `act(() => { incrementBtn.click(); })`

**Tests fail with "module not found"?**
- Verify file paths are correct relative to test file location
- Ensure babel.config.js is in the root folder

**Component not rendering in tests?**
- Check that all required props have default values
- Verify data-testid attributes match exactly (case-sensitive)

**Mock function not being called?**
- Ensure callback is being invoked in your component code
- Use `mockFn.mock.calls` to debug how many times it was called

**"Cannot find module @testing-library/react"?**
- Run: `npm install --save-dev @testing-library/react @testing-library/jest-dom`

**Tests pass locally but fail in CI/CD?**
- Ensure all dependencies are in package.json (use --save-dev)
- Check Node.js version matches (v22.20.0 or compatible)

---

# Running The Lab

### Build And Browser Run Using Webpack

1. **Install Webpack and necessary dependencies:**

Run these commands at your project root (where `package.json` is):

```
npm install --save-dev webpack webpack-cli webpack-dev-server babel-loader html-webpack-plugin react-refresh @pmmmwh/react-refresh-webpack-plugin
```

2. **Create Webpack config file `webpack.config.js` in the root folder:**

```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const ReactRefreshWebpackPlugin = require('@pmmmwh/react-refresh-webpack-plugin');

const isDevelopment = process.env.NODE_ENV !== 'production';

module.exports = {
  mode: isDevelopment ? 'development' : 'production',
  entry: './lab013/src/index.js', // Entry point for Stepper lab
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
        test: /\.(js|jsx)$/,
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
      template: 'lab013/index.html',
    }),
  ].filter(Boolean),
  resolve: {
    extensions: ['.js', '.jsx'],
  },
};
```

3. **Install CSS loaders (if not already installed):**

```bash
npm install --save-dev style-loader css-loader
```

4. **Create React entry point `lab013/src/index.js`:**

```js
import React, { useState } from 'react';
import { createRoot } from 'react-dom/client';
import Stepper from './Stepper';

function App() {
  const [value1, setValue1] = useState(0);
  const [value2, setValue2] = useState(10);

  return (
    <div style={{ padding: '20px', fontFamily: 'Arial, sans-serif' }}>
      <h1>Stepper Component Demo</h1>
      
      <div style={{ display: 'flex', gap: '40px', flexWrap: 'wrap' }}>
        <div>
          <h2>Default Stepper (step: 1)</h2>
          <Stepper 
            initialValue={value1}
            onValueChange={setValue1}
          />
          <p>Current Value: {value1}</p>
        </div>

        <div>
          <h2>Custom Step Stepper (step: 5)</h2>
          <Stepper 
            initialValue={value2}
            step={5}
            onValueChange={setValue2}
          />
          <p>Current Value: {value2}</p>
        </div>
      </div>
    </div>
  );
}

const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

5. **Create `lab013/index.html`:**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Stepper Component Demo</title>
  <style>
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }
    body {
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      min-height: 100vh;
      display: flex;
      align-items: center;
      justify-content: center;
    }
    #root {
      background: white;
      padding: 40px;
      border-radius: 12px;
      box-shadow: 0 10px 40px rgba(0, 0, 0, 0.3);
    }
  </style>
</head>
<body>
  <div id="root"></div>
</body>
</html>
```

6. **Update `package.json` scripts:**

Add these to your `scripts` section:

```json
"scripts": {
  "test": "jest",
  "dev": "webpack serve --env development",
  "build": "webpack --mode production",
  "start": "webpack serve --mode development"
}
```

7. **Run your development server:**

```bash
npm run dev
```

Your React app should open automatically and display two Stepper components with hot reloading enabled.


---

## Summary of What You Learned

This lab introduced you to **Test-Driven Development (TDD)** fundamentals with Jest and React:

**Testing Concepts:**
- Writing tests BEFORE implementation (RED → GREEN → REFACTOR cycle)
- Using basic Jest matchers (`toBe()`, `toBeInTheDocument()`, `toHaveTextContent()`)
- Rendering React components with React Testing Library
- Simulating user interactions with button clicks wrapped in `act()`
- Testing component callbacks with mock functions
- Understanding React's `act()` function for proper state update handling

**React Concepts:**
- Component props and default values
- React state with `useState` hook
- Event handlers and onClick callbacks
- Conditional rendering and rendering updates

**Best Practices:**
- Keep tests focused on one behavior per test
- Use descriptive test names that explain what is being tested
- Organize tests into logical describe blocks
- Mock external dependencies (like callbacks)
- Always wrap state-changing interactions in `act()`
- Test user interactions rather than implementation details
