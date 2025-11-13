# Lab 018: UserList Component TDD Example Using Node.js, Jest, JSDOM, & React

**Technologies:** Node.js v22.20.0, Jest, jsdom, React

## Domain: User Management Module - UserList Component

Create and test a UserList component that displays a list of users with filtering, sorting, and mock data management. This lab covers advanced Jest features including mocking external functions and modules, snapshot testing for list rendering consistency, and testing component props and state changes.

---

## Jest Features Covered in This Lab

### 1. **Mocking Functions with jest.fn()**
`jest.fn()` creates mock functions that track how many times they're called, with what arguments, and what they return. This is essential for testing callbacks and verifying component interactions without executing real side effects.

**Why useful:** Isolates components from their dependencies and verifies that callbacks are invoked correctly. Allows you to test error handling paths without real API failures.

### 2. **Mocking Modules with jest.mock()**
Jest allows you to mock entire modules (like API calls or utility functions) so your component tests don't depend on external services or data sources. You can provide fake implementations that return controlled data.

**Why useful:** Makes tests faster, more reliable, and independent of external systems. Prevents accidental API calls during testing and allows testing both success and failure scenarios.

### 3. **Snapshot Testing for Lists**
Snapshot testing captures the rendered output of a component and compares it against future test runs. For lists, snapshots ensure that the structure of rendered items remains consistent and catches unintended changes to list item rendering.

**Why useful:** Quickly detects changes to list rendering without writing assertions for every item. Useful for verifying complex UI hierarchies don't break unexpectedly.

### 4. **Testing Props and State Mutations**
Components receive props and maintain internal state. Testing that components respond correctly to prop changes and update their internal state properly ensures correct behavior under different conditions.

**Why useful:** Ensures components behave predictably with different inputs and that state updates trigger correct re-renders.

---

## Prerequisites

- You must already have Node.js, Jest, and JSDOM installed and working with a valid `jest.config.js`.
- Jest should use jsdom as its environment and recognize your intended test folders.

### Step 1: Install Dependencies

Run the following commands in `<root-folder>` (the same folder where your package.json lives):

**Install Jest and jsdom (should have been done in previous labs):**

```bash
npm install --save-dev jest jsdom
```

**Install Testing Library utilities:**

```bash
npm install --save-dev @testing-library/jest-dom
```

**Install React (should have been done in previous assignments):**

```bash
npm install react react-dom
```

**Install Babel dependencies for JSX compilation:**

```bash
npm install @babel/core @babel/preset-env @babel/preset-react babel-jest --save-dev
```

**Install React Testing Library:**

```bash
npm install --save-dev @testing-library/react
```

### Step 2: Verify/Update Jest Configuration

Ensure your `jest.config.js` in the root folder includes the lab018 path:

```js
module.exports = {
  testEnvironment: "jsdom",
  roots: ["<rootDir>/basic-js-tdd", "<rootDir>/lab003", "<rootDir>/lab015", "<rootDir>/lab018", "<rootDir>/"],
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
<root-folder>/lab018/
    ├── src/
    │   ├── UserList.js
    │   └── userService.js
    └── test/
        └── UserList.test.js
```

All production code will go in `src`, all tests go in `test`.

---

## UserList Component Specification

The UserList component displays a list of users fetched from a service, with features for filtering by name and sorting by name or email.

**Component Properties:**
- **users** (array): Array of user objects to display
- **onUserSelect** (function): Callback when a user is clicked
- **isLoading** (boolean, optional): Shows loading state
- **filterText** (string, optional): Text to filter users by name

**User Object Structure:**
```js
{
  id: number,
  name: string,
  email: string,
  role: string
}
```

**Visual Structure:**
- Filter input field (optional)
- Sort buttons (by name, by email)
- User list with individual user rows
- Each row displays: Name, Email, Role
- Loading indicator when isLoading is true
- Empty state message when no users match filter

---

## Business Requirements (Gherkin Format)

```gherkin
Feature: User List Management
  As a system administrator
  I want to view and manage a list of users
  So that I can effectively oversee user accounts

  Scenario: Display all users in a table format
    Given the UserList component receives a list of users
    When the component renders
    Then all users should be displayed with their name, email, and role

  Scenario: Filter users by name
    Given the UserList component has a filter input
    When I enter text in the filter field
    Then only users whose name contains the filter text should be displayed

  Scenario: Show loading state
    Given the UserList component with isLoading prop set to true
    When the component renders
    Then a loading indicator should be displayed
    And the user list should not be visible

  Scenario: Handle empty user list
    Given the UserList component with an empty users array
    When the component renders
    Then an empty state message should be displayed

  Scenario: Sort users by name
    Given the UserList component has users
    When I click the sort by name button
    Then users should be displayed in alphabetical order by name

  Scenario: Sort users by email
    Given the UserList component has users
    When I click the sort by email button
    Then users should be displayed in alphabetical order by email

  Scenario: Trigger callback on user selection
    Given the UserList component with users displayed
    When I click on a user row
    Then the onUserSelect callback should be called with that user's data
```

---

## Requirements

**User Display:**
- Positive: Component renders list of users with name, email, and role
- Negative: Component shows empty state when no users provided

**Filtering:**
- Positive: Users are filtered when filterText prop changes
- Negative: Filter is case-insensitive

**Sorting:**
- Positive: Users can be sorted by name (ascending)
- Negative: Users can be sorted by email (ascending)

**Loading State:**
- Positive: Loading indicator appears when isLoading is true
- Negative: User list is hidden during loading

**User Selection:**
- Positive: onUserSelect callback is triggered when user clicks on a row
- Negative: Callback receives correct user object

**Snapshot Testing:**
- List rendering structure remains consistent across renders

**Mock Module Testing:**
- userService API calls are mocked and don't make real HTTP requests
- Both success and failure responses are testable

---

# TDD Walkthrough (3 Passes)

For each pass, you will:
- Write fail-first tests (RED phase)
- Implement just enough code to make tests pass (GREEN phase)
- Clean/refactor if needed (REFACTOR phase)
- Include descriptive comments in test functions

---

## PASS 1: Basic Rendering & Filtering

### Business Requirements Addressed in This Pass
```
Feature: User List Management
  Scenario: Display all users in a table format ✓
  Scenario: Filter users by name ✓
  Scenario: Handle empty user list ✓
```

### Jest Features Covered in This Pass
- **Snapshot Testing:** Capturing list structure
- **Testing Props:** Verifying component behavior with different prop values

### RED Phase - Write Failing Tests

**File:** `lab018/test/UserList.test.js`

```js
import React from 'react';
import { render, screen, within } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import UserList from '../src/UserList';
import '@testing-library/jest-dom';

describe('UserList Component - PASS 1: Basic Rendering & Filtering', () => {
  // Setup: Define test data that will be used in multiple tests
  const mockUsers = [
    { id: 1, name: 'Alice Johnson', email: 'alice@example.com', role: 'Admin' },
    { id: 2, name: 'Bob Smith', email: 'bob@example.com', role: 'User' },
    { id: 3, name: 'Charlie Brown', email: 'charlie@example.com', role: 'Manager' },
  ];

  beforeEach(() => {
    jest.clearAllMocks();
  });

  afterEach(() => {
    // Cleanup
  });

  // Test 1: Verify that all users from the props array are rendered in the list
  it('should render all users when users array is provided', () => {
    render(
      <UserList 
        users={mockUsers} 
        onUserSelect={() => {}}
      />
    );
    
    expect(screen.getByText('Alice Johnson')).toBeInTheDocument();
    expect(screen.getByText('Bob Smith')).toBeInTheDocument();
    expect(screen.getByText('Charlie Brown')).toBeInTheDocument();
  });

  // Test 2: Verify that user details (email and role) are displayed for each user
  it('should display user email and role for each user', () => {
    render(
      <UserList 
        users={mockUsers} 
        onUserSelect={() => {}}
      />
    );
    
    expect(screen.getByText('alice@example.com')).toBeInTheDocument();
    expect(screen.getByText('bob@example.com')).toBeInTheDocument();
    expect(screen.getByText('Admin')).toBeInTheDocument();
    expect(screen.getByText('Manager')).toBeInTheDocument();
  });

  // Test 3: Verify that an empty state message appears when no users are provided
  it('should display empty state message when no users provided', () => {
    render(
      <UserList 
        users={[]} 
        onUserSelect={() => {}}
      />
    );
    
    const emptyMessage = screen.getByTestId('empty-state');
    expect(emptyMessage).toBeInTheDocument();
    expect(emptyMessage).toHaveTextContent('No users found');
  });

  // Test 4: Verify that snapshot matches for the initial rendered state with users
  it('should match snapshot when rendering user list', () => {
    const { container } = render(
      <UserList 
        users={mockUsers} 
        onUserSelect={() => {}}
      />
    );
    
    expect(container.firstChild).toMatchSnapshot();
  });

  // Test 5: Verify that users are filtered based on the filterText prop (case-insensitive)
  it('should filter users by name when filterText prop is provided', () => {
    const { rerender } = render(
      <UserList 
        users={mockUsers} 
        onUserSelect={() => {}}
        filterText=""
      />
    );
    
    // Initially all three users should be visible
    expect(screen.getByText('Alice Johnson')).toBeInTheDocument();
    expect(screen.getByText('Bob Smith')).toBeInTheDocument();
    expect(screen.getByText('Charlie Brown')).toBeInTheDocument();
    
    // Now rerender with filter text
    rerender(
      <UserList 
        users={mockUsers} 
        onUserSelect={() => {}}
        filterText="Charlie"
      />
    );
    
    // Only Charlie should be visible
    expect(screen.getByText('Charlie Brown')).toBeInTheDocument();
    expect(screen.queryByText('Alice Johnson')).not.toBeInTheDocument();
    expect(screen.queryByText('Bob Smith')).not.toBeInTheDocument();
  });

  // Test 6: Verify that filter is case-insensitive
  it('should filter users case-insensitively', () => {
    render(
      <UserList 
        users={mockUsers} 
        onUserSelect={() => {}}
        filterText="alice"
      />
    );
    
    expect(screen.getByText('Alice Johnson')).toBeInTheDocument();
    expect(screen.queryByText('Bob Smith')).not.toBeInTheDocument();
  });

  // Test 7: Verify that empty state appears when filter results in no matches
  it('should display empty state when filter returns no matches', () => {
    render(
      <UserList 
        users={mockUsers} 
        onUserSelect={() => {}}
        filterText="NonExistent"
      />
    );
    
    expect(screen.getByTestId('empty-state')).toBeInTheDocument();
  });
});
```

**Important Note:** Add the following import for user interaction:
```bash
npm install --save-dev @testing-library/user-event
```

**Run tests from the root folder:** (Tests should fail - RED)

```bash
npm test -- ./lab018/test/UserList.test.js
```

Expected output: 7 failing tests - the UserList component doesn't exist yet.

### GREEN Phase - Make Tests Pass

**File:** `lab018/src/UserList.js`

```js
import React, { useMemo } from 'react';

// UserList component for displaying and managing users in the system
// Props:
//   - users (array): Array of user objects to display
//   - onUserSelect (function): Callback when a user row is clicked
//   - isLoading (boolean, optional): Shows loading state when true
//   - filterText (string, optional): Text to filter users by name
function UserList({ users = [], onUserSelect, isLoading = false, filterText = '' }) {
  // Memoize the filtered and sorted users to avoid unnecessary recalculations
  // This hook optimizes performance by only recalculating when inputs change
  const filteredUsers = useMemo(() => {
    if (!Array.isArray(users)) {
      return [];
    }

    // Filter users by name if filterText is provided
    // toLowerCase() makes the filter case-insensitive
    let result = users.filter(user =>
      user.name.toLowerCase().includes(filterText.toLowerCase())
    );

    return result;
  }, [users, filterText]);

  // Show loading indicator if isLoading prop is true
  if (isLoading) {
    return <div data-testid="loading-indicator">Loading users...</div>;
  }

  // Show empty state message if no users match the filter
  if (filteredUsers.length === 0) {
    return <div data-testid="empty-state">No users found</div>;
  }

  // Render the user list
  return (
    <div data-testid="user-list-container" className="user-list">
      <table data-testid="user-list-table" className="user-table">
        <thead>
          <tr>
            <th>Name</th>
            <th>Email</th>
            <th>Role</th>
          </tr>
        </thead>
        <tbody>
          {filteredUsers.map((user) => (
            <tr
              key={user.id}
              data-testid={`user-row-${user.id}`}
              onClick={() => onUserSelect(user)}
              className="user-row"
              style={{ cursor: 'pointer' }}
            >
              <td data-testid={`user-name-${user.id}`}>{user.name}</td>
              <td data-testid={`user-email-${user.id}`}>{user.email}</td>
              <td data-testid={`user-role-${user.id}`}>{user.role}</td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}

export default UserList;
```

**Run tests from the root folder:** (Tests should pass - GREEN)

```bash
npm test -- ./lab018/test/UserList.test.js
```

Expected output: 7 passing tests

When prompted to create snapshots, press `u` to create them.

### Refactor

- The component uses `useMemo` to optimize filtering performance
- Table structure is semantic and accessible
- Props are well-documented with JSDoc comments
- Default values prevent runtime errors with missing props

---

## PASS 2: Loading States & Mocking Functions

### Business Requirements Addressed in This Pass
```
Feature: User List Management
  Scenario: Show loading state ✓
  Scenario: Trigger callback on user selection ✓
```

### Jest Features Covered in This Pass
- **Mocking Functions with jest.fn():** Testing callbacks and verifying their invocations
- **Testing State Changes:** Verifying component behavior with different prop values

### RED Phase - Add Failing Tests

**File:** `lab018/test/UserList.test.js`

**Important:** Add the following code AFTER the closing brace of the first describe block (after PASS 1 tests), but BEFORE the end of the file. This is a new describe block for PASS 2 tests.

```js
describe('UserList Component - PASS 2: Loading States & Callback Functions', () => {
  // Setup: Test data
  const mockUsers = [
    { id: 1, name: 'Diana Prince', email: 'diana@example.com', role: 'Admin' },
    { id: 2, name: 'Edward Norton', email: 'edward@example.com', role: 'User' },
  ];

  let mockOnUserSelect;

  beforeEach(() => {
    // Create a fresh mock function before each test
    // jest.fn() creates a function that tracks how it's called
    mockOnUserSelect = jest.fn();
    jest.clearAllMocks();
  });

  afterEach(() => {
    jest.clearAllMocks();
  });

  // Test 1: Verify loading indicator appears when isLoading is true
  it('should display loading indicator when isLoading is true', () => {
    render(
      <UserList
        users={mockUsers}
        onUserSelect={mockOnUserSelect}
        isLoading={true}
      />
    );

    const loadingIndicator = screen.getByTestId('loading-indicator');
    expect(loadingIndicator).toBeInTheDocument();
    // Verify that the user list is NOT visible during loading
    expect(screen.queryByTestId('user-list-table')).not.toBeInTheDocument();
  });

  // Test 2: Verify user list is visible when isLoading is false
  it('should display user list when isLoading is false', () => {
    render(
      <UserList
        users={mockUsers}
        onUserSelect={mockOnUserSelect}
        isLoading={false}
      />
    );

    expect(screen.getByTestId('user-list-table')).toBeInTheDocument();
    expect(screen.queryByTestId('loading-indicator')).not.toBeInTheDocument();
  });

  // Test 3: Verify that clicking a user row calls the onUserSelect callback
  // with the correct user object
  it('should call onUserSelect callback when user row is clicked', () => {
    render(
      <UserList
        users={mockUsers}
        onUserSelect={mockOnUserSelect}
      />
    );

    // Find and click the first user's row
    const userRow = screen.getByTestId('user-row-1');
    userRow.click();

    // Verify the callback was called exactly once
    expect(mockOnUserSelect).toHaveBeenCalledTimes(1);
    // Verify the callback was called with the correct user object
    expect(mockOnUserSelect).toHaveBeenCalledWith({
      id: 1,
      name: 'Diana Prince',
      email: 'diana@example.com',
      role: 'Admin',
    });
  });

  // Test 4: Verify that clicking different user rows calls callback with correct data
  it('should call onUserSelect with correct data when different users are clicked', () => {
    render(
      <UserList
        users={mockUsers}
        onUserSelect={mockOnUserSelect}
      />
    );

    // Click the second user's row
    const secondUserRow = screen.getByTestId('user-row-2');
    secondUserRow.click();

    // Verify callback was called with the second user's data
    expect(mockOnUserSelect).toHaveBeenCalledWith({
      id: 2,
      name: 'Edward Norton',
      email: 'edward@example.com',
      role: 'User',
    });
  });

  // Test 5: Verify that onUserSelect is called multiple times when multiple rows are clicked
  it('should call onUserSelect multiple times when multiple rows are clicked', () => {
    render(
      <UserList
        users={mockUsers}
        onUserSelect={mockOnUserSelect}
      />
    );

    // Click first user
    screen.getByTestId('user-row-1').click();
    // Click second user
    screen.getByTestId('user-row-2').click();

    // Verify callback was called twice
    expect(mockOnUserSelect).toHaveBeenCalledTimes(2);
  });

  // Test 6: Verify that isLoading transitions correctly from true to false
  it('should transition from loading to loaded state', () => {
    const { rerender } = render(
      <UserList
        users={mockUsers}
        onUserSelect={mockOnUserSelect}
        isLoading={true}
      />
    );

    // Initially should show loading
    expect(screen.getByTestId('loading-indicator')).toBeInTheDocument();

    // Rerender with isLoading set to false
    rerender(
      <UserList
        users={mockUsers}
        onUserSelect={mockOnUserSelect}
        isLoading={false}
      />
    );

    // Now should show user list, not loading
    expect(screen.queryByTestId('loading-indicator')).not.toBeInTheDocument();
    expect(screen.getByTestId('user-list-table')).toBeInTheDocument();
  });
});
```

**Run tests from the root folder:** (Tests should fail - RED)

```bash
npm test -- ./lab018/test/UserList.test.js
```

Expected output: 6 new failing tests (PASS 1 tests should still pass)

### GREEN Phase - Make Tests Pass

The UserList component from PASS 1 already handles most of these requirements. Update the component to ensure onClick handlers work correctly:

**File:** `lab018/src/UserList.js`

No changes needed! The implementation from PASS 1 already includes:
- Loading state handling with the `isLoading` prop
- Proper onClick handlers on user rows that call `onUserSelect` with the correct user object

**Run tests from the root folder:** (All tests should pass - GREEN)

```bash
npm test -- ./lab018/test/UserList.test.js
```

Expected output: 13 passing tests (7 from PASS 1 + 6 from PASS 2)

### Refactor

- Mock functions are properly created fresh in each test's `beforeEach`
- Test names clearly describe what is being verified
- Assertions are specific and verify both the callback invocation and its arguments

---

## PASS 3: Sorting, Snapshots & Mocking Modules

### Business Requirements Addressed in This Pass
```
Feature: User List Management
  Scenario: Sort users by name ✓
  Scenario: Sort users by email ✓
  Scenario: Snapshot testing for consistency ✓
```

### Jest Features Covered in This Pass
- **Mocking Modules with jest.mock():** Mocking the userService module
- **Snapshot Testing:** Capturing full list structure with sorting states
- **Parameterized Tests:** Testing multiple sorting scenarios

### RED Phase - Add Failing Tests

**File:** `lab018/test/UserList.test.js`

**Important:** Add the following code AFTER the closing brace of the PASS 2 describe block, but BEFORE the end of the file. This is a new describe block for PASS 3 tests.

```js
describe('UserList Component - PASS 3: Sorting, Snapshots & Module Mocking', () => {
  // Setup: Test data for sorting tests
  const mockUsers = [
    { id: 3, name: 'Zoe Wilson', email: 'zoe@example.com', role: 'User' },
    { id: 1, name: 'Alice Wonderland', email: 'alice@example.com', role: 'Admin' },
    { id: 2, name: 'Bob Anderson', email: 'bob@example.com', role: 'Manager' },
  ];

  beforeEach(() => {
    jest.clearAllMocks();
  });

  afterEach(() => {
    // Cleanup
  });

  // Test 1: Verify snapshot of rendered list matches expected structure
  it('should match snapshot when rendering sorted user list', () => {
    const { container } = render(
      <UserList
        users={mockUsers}
        onUserSelect={() => {}}
      />
    );

    expect(container.firstChild).toMatchSnapshot();
  });

  // Test 2: Verify users are sorted by name when sortBy is 'name'
  it('should display users sorted by name in ascending order', () => {
    render(
      <UserList
        users={mockUsers}
        onUserSelect={() => {}}
        sortBy="name"
      />
    );

    // Get all user names in the rendered order
    const nameElements = screen.getAllByTestId(/^user-name-/);
    const displayedNames = nameElements.map(el => el.textContent);

    // Verify they are in alphabetical order
    expect(displayedNames).toEqual([
      'Alice Wonderland',
      'Bob Anderson',
      'Zoe Wilson',
    ]);
  });

  // Test 3: Verify users are sorted by email when sortBy is 'email'
  it('should display users sorted by email in ascending order', () => {
    render(
      <UserList
        users={mockUsers}
        onUserSelect={() => {}}
        sortBy="email"
      />
    );

    // Get all user emails in the rendered order
    const emailElements = screen.getAllByTestId(/^user-email-/);
    const displayedEmails = emailElements.map(el => el.textContent);

    // Verify they are in alphabetical order
    expect(displayedEmails).toEqual([
      'alice@example.com',
      'bob@example.com',
      'zoe@example.com',
    ]);
  });

  // Test 4: Verify sorting works correctly when combined with filtering
  it('should sort filtered users correctly', () => {
    render(
      <UserList
        users={mockUsers}
        onUserSelect={() => {}}
        filterText="a" // Should match Alice and Bob
        sortBy="name"
      />
    );

    const nameElements = screen.getAllByTestId(/^user-name-/);
    const displayedNames = nameElements.map(el => el.textContent);

    // Should only show Alice and Bob, sorted by name
    expect(displayedNames).toEqual([
      'Alice Wonderland',
      'Bob Anderson',
    ]);
  });

  // Test 5: Parameterized test for different sort orders
  describe('Sort order variations', () => {
    test.each([
      ['name', ['Alice Wonderland', 'Bob Anderson', 'Zoe Wilson']],
      ['email', ['alice@example.com', 'bob@example.com', 'zoe@example.com']],
    ])('should sort by %s correctly', (sortBy, expectedOrder) => {
      render(
        <UserList
          users={mockUsers}
          onUserSelect={() => {}}
          sortBy={sortBy}
        />
      );

      if (sortBy === 'name') {
        const nameElements = screen.getAllByTestId(/^user-name-/);
        const displayedNames = nameElements.map(el => el.textContent);
        expect(displayedNames).toEqual(expectedOrder);
      } else if (sortBy === 'email') {
        const emailElements = screen.getAllByTestId(/^user-email-/);
        const displayedEmails = emailElements.map(el => el.textContent);
        expect(displayedEmails).toEqual(expectedOrder);
      }
    });
  });

  // Test 6: Snapshot of loading state remains consistent
  it('should match snapshot for loading state', () => {
    const { container } = render(
      <UserList
        users={mockUsers}
        onUserSelect={() => {}}
        isLoading={true}
      />
    );

    expect(container.firstChild).toMatchSnapshot();
  });

  // Test 7: Snapshot of empty state remains consistent
  it('should match snapshot for empty state', () => {
    const { container } = render(
      <UserList
        users={[]}
        onUserSelect={() => {}}
      />
    );

    expect(container.firstChild).toMatchSnapshot();
  });

  // Test 8: Verify sorting is case-sensitive for email sorting
  it('should handle case-insensitive sorting correctly', () => {
    const usersWithMixedCase = [
      { id: 1, name: 'Zoe Wilson', email: 'Zoe@example.com', role: 'User' },
      { id: 2, name: 'alice Wonderland', email: 'alice@example.com', role: 'Admin' },
    ];

    render(
      <UserList
        users={usersWithMixedCase}
        onUserSelect={() => {}}
        sortBy="email"
      />
    );

    const emailElements = screen.getAllByTestId(/^user-email-/);
    const displayedEmails = emailElements.map(el => el.textContent);

    // alice@example.com should come before Zoe@example.com (case-insensitive)
    expect(displayedEmails[0]).toBe('alice@example.com');
  });
});
```

**Run tests from the root folder:** (Tests should fail - RED)

```bash
npm test -- ./lab018/test/UserList.test.js
```

Expected output: 8 new failing tests (count can be less, as some of the test may pass because of existing code)

### GREEN Phase - Make Tests Pass

**File:** `lab018/src/UserList.js`

Update the UserList component to support sorting. Replace the entire `UserList.js` file with the updated version below:

```js
import React, { useMemo } from 'react';

// UserList component for displaying and managing users in the system
// Props:
//   - users (array): Array of user objects to display
//   - onUserSelect (function): Callback when a user row is clicked
//   - isLoading (boolean, optional): Shows loading state when true
//   - filterText (string, optional): Text to filter users by name
//   - sortBy (string, optional): Sort users by 'name' or 'email'
function UserList({ 
  users = [], 
  onUserSelect, 
  isLoading = false, 
  filterText = '',
  sortBy = null 
}) {
  // Memoize the filtered and sorted users to avoid unnecessary recalculations
  // Dependencies: users array, filterText, and sortBy - if any change, recalculate
  const processedUsers = useMemo(() => {
    if (!Array.isArray(users)) {
      return [];
    }

    // Step 1: Filter users by name if filterText is provided
    // toLowerCase() makes the filter case-insensitive
    let result = users.filter(user =>
      user.name.toLowerCase().includes(filterText.toLowerCase())
    );

    // Step 2: Sort the filtered results if sortBy is specified
    if (sortBy === 'name') {
      // Sort by user name in ascending alphabetical order
      result = result.sort((a, b) =>
        a.name.toLowerCase().localeCompare(b.name.toLowerCase())
      );
    } else if (sortBy === 'email') {
      // Sort by user email in ascending alphabetical order
      result = result.sort((a, b) =>
        a.email.toLowerCase().localeCompare(b.email.toLowerCase())
      );
    }

    return result;
  }, [users, filterText, sortBy]);

  // Show loading indicator if isLoading prop is true
  if (isLoading) {
    return <div data-testid="loading-indicator">Loading users...</div>;
  }

  // Show empty state message if no users match the filter
  if (processedUsers.length === 0) {
    return <div data-testid="empty-state">No users found</div>;
  }

  // Render the user list table with sorted and filtered users
  return (
    <div data-testid="user-list-container" className="user-list">
      <table data-testid="user-list-table" className="user-table">
        <thead>
          <tr>
            <th>Name</th>
            <th>Email</th>
            <th>Role</th>
          </tr>
        </thead>
        <tbody>
          {processedUsers.map((user) => (
            <tr
              key={user.id}
              data-testid={`user-row-${user.id}`}
              onClick={() => onUserSelect(user)}
              className="user-row"
              style={{ cursor: 'pointer' }}
            >
              <td data-testid={`user-name-${user.id}`}>{user.name}</td>
              <td data-testid={`user-email-${user.id}`}>{user.email}</td>
              <td data-testid={`user-role-${user.id}`}>{user.role}</td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}

export default UserList;
```

**Key Changes Made:**
- Added `sortBy` prop parameter to component function signature (line 7)
- Changed `filteredUsers` to `processedUsers` to reflect that it now includes both filtering and sorting
- Added Step 2 in the useMemo hook to sort users based on the `sortBy` prop (lines 24-34)
- Used `localeCompare()` for case-insensitive sorting of both names and emails

**Run tests from the root folder:** (All tests should pass - GREEN)

```bash
npm test -- ./lab018/test/UserList.test.js
```

Expected output: 21 passing tests
- 7 from PASS 1
- 6 from PASS 2
- 8 from PASS 3

When prompted to create snapshots, press `u` to update them.

### Refactor

- Sorting logic is clean and easy to understand
- `useMemo` dependency array includes all variables that affect the result
- `localeCompare()` provides proper alphabetical sorting that handles case-insensitivity
- Variable name changed from `filteredUsers` to `processedUsers` to better reflect its dual purpose
- Comments clearly explain each step of the filtering and sorting process

---

## Complete Test File Reference

**File:** `lab018/test/UserList.test.js` (Final Version)

```js
import React from 'react';
import { render, screen, within } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import UserList from '../src/UserList';
import '@testing-library/jest-dom';

// PASS 1: Basic Rendering & Filtering
describe('UserList Component - PASS 1: Basic Rendering & Filtering', () => {
  const mockUsers = [
    { id: 1, name: 'Alice Johnson', email: 'alice@example.com', role: 'Admin' },
    { id: 2, name: 'Bob Smith', email: 'bob@example.com', role: 'User' },
    { id: 3, name: 'Charlie Brown', email: 'charlie@example.com', role: 'Manager' },
  ];

  beforeEach(() => {
    jest.clearAllMocks();
  });

  afterEach(() => {
    // Cleanup
  });

  it('should render all users when users array is provided', () => {
    render(
      <UserList 
        users={mockUsers} 
        onUserSelect={() => {}}
      />
    );
    
    expect(screen.getByText('Alice Johnson')).toBeInTheDocument();
    expect(screen.getByText('Bob Smith')).toBeInTheDocument();
    expect(screen.getByText('Charlie Brown')).toBeInTheDocument();
  });

  it('should display user email and role for each user', () => {
    render(
      <UserList 
        users={mockUsers} 
        onUserSelect={() => {}}
      />
    );
    
    expect(screen.getByText('alice@example.com')).toBeInTheDocument();
    expect(screen.getByText('bob@example.com')).toBeInTheDocument();
    expect(screen.getByText('Admin')).toBeInTheDocument();
    expect(screen.getByText('Manager')).toBeInTheDocument();
  });

  it('should display empty state message when no users provided', () => {
    render(
      <UserList 
        users={[]} 
        onUserSelect={() => {}}
      />
    );
    
    const emptyMessage = screen.getByTestId('empty-state');
    expect(emptyMessage).toBeInTheDocument();
    expect(emptyMessage).toHaveTextContent('No users found');
  });

  it('should match snapshot when rendering user list', () => {
    const { container } = render(
      <UserList 
        users={mockUsers} 
        onUserSelect={() => {}}
      />
    );
    
    expect(container.firstChild).toMatchSnapshot();
  });

  it('should filter users by name when filterText prop is provided', () => {
    const { rerender } = render(
      <UserList 
        users={mockUsers} 
        onUserSelect={() => {}}
        filterText=""
      />
    );
    
    expect(screen.getByText('Alice Johnson')).toBeInTheDocument();
    expect(screen.getByText('Bob Smith')).toBeInTheDocument();
    expect(screen.getByText('Charlie Brown')).toBeInTheDocument();
    
    rerender(
      <UserList 
        users={mockUsers} 
        onUserSelect={() => {}}
        filterText="Charlie"
      />
    );
    
    expect(screen.getByText('Charlie Brown')).toBeInTheDocument();
    expect(screen.queryByText('Alice Johnson')).not.toBeInTheDocument();
    expect(screen.queryByText('Bob Smith')).not.toBeInTheDocument();
  });

  it('should filter users case-insensitively', () => {
    render(
      <UserList 
        users={mockUsers} 
        onUserSelect={() => {}}
        filterText="alice"
      />
    );
    
    expect(screen.getByText('Alice Johnson')).toBeInTheDocument();
    expect(screen.queryByText('Bob Smith')).not.toBeInTheDocument();
  });

  it('should display empty state when filter returns no matches', () => {
    render(
      <UserList 
        users={mockUsers} 
        onUserSelect={() => {}}
        filterText="NonExistent"
      />
    );
    
    expect(screen.getByTestId('empty-state')).toBeInTheDocument();
  });
});

// PASS 2: Loading States & Callback Functions
describe('UserList Component - PASS 2: Loading States & Callback Functions', () => {
  const mockUsers = [
    { id: 1, name: 'Diana Prince', email: 'diana@example.com', role: 'Admin' },
    { id: 2, name: 'Edward Norton', email: 'edward@example.com', role: 'User' },
  ];

  let mockOnUserSelect;

  beforeEach(() => {
    mockOnUserSelect = jest.fn();
    jest.clearAllMocks();
  });

  afterEach(() => {
    jest.clearAllMocks();
  });

  it('should display loading indicator when isLoading is true', () => {
    render(
      <UserList
        users={mockUsers}
        onUserSelect={mockOnUserSelect}
        isLoading={true}
      />
    );

    const loadingIndicator = screen.getByTestId('loading-indicator');
    expect(loadingIndicator).toBeInTheDocument();
    expect(screen.queryByTestId('user-list-table')).not.toBeInTheDocument();
  });

  it('should display user list when isLoading is false', () => {
    render(
      <UserList
        users={mockUsers}
        onUserSelect={mockOnUserSelect}
        isLoading={false}
      />
    );

    expect(screen.getByTestId('user-list-table')).toBeInTheDocument();
    expect(screen.queryByTestId('loading-indicator')).not.toBeInTheDocument();
  });

  it('should call onUserSelect callback when user row is clicked', () => {
    render(
      <UserList
        users={mockUsers}
        onUserSelect={mockOnUserSelect}
      />
    );

    const userRow = screen.getByTestId('user-row-1');
    userRow.click();

    expect(mockOnUserSelect).toHaveBeenCalledTimes(1);
    expect(mockOnUserSelect).toHaveBeenCalledWith({
      id: 1,
      name: 'Diana Prince',
      email: 'diana@example.com',
      role: 'Admin',
    });
  });

  it('should call onUserSelect with correct data when different users are clicked', () => {
    render(
      <UserList
        users={mockUsers}
        onUserSelect={mockOnUserSelect}
      />
    );

    const secondUserRow = screen.getByTestId('user-row-2');
    secondUserRow.click();

    expect(mockOnUserSelect).toHaveBeenCalledWith({
      id: 2,
      name: 'Edward Norton',
      email: 'edward@example.com',
      role: 'User',
    });
  });

  it('should call onUserSelect multiple times when multiple rows are clicked', () => {
    render(
      <UserList
        users={mockUsers}
        onUserSelect={mockOnUserSelect}
      />
    );

    screen.getByTestId('user-row-1').click();
    screen.getByTestId('user-row-2').click();

    expect(mockOnUserSelect).toHaveBeenCalledTimes(2);
  });

  it('should transition from loading to loaded state', () => {
    const { rerender } = render(
      <UserList
        users={mockUsers}
        onUserSelect={mockOnUserSelect}
        isLoading={true}
      />
    );

    expect(screen.getByTestId('loading-indicator')).toBeInTheDocument();

    rerender(
      <UserList
        users={mockUsers}
        onUserSelect={mockOnUserSelect}
        isLoading={false}
      />
    );

    expect(screen.queryByTestId('loading-indicator')).not.toBeInTheDocument();
    expect(screen.getByTestId('user-list-table')).toBeInTheDocument();
  });
});

// PASS 3: Sorting, Snapshots & Module Mocking
describe('UserList Component - PASS 3: Sorting, Snapshots & Module Mocking', () => {
  const mockUsers = [
    { id: 3, name: 'Zoe Wilson', email: 'zoe@example.com', role: 'User' },
    { id: 1, name: 'Alice Wonderland', email: 'alice@example.com', role: 'Admin' },
    { id: 2, name: 'Bob Anderson', email: 'bob@example.com', role: 'Manager' },
  ];

  beforeEach(() => {
    jest.clearAllMocks();
  });

  afterEach(() => {
    // Cleanup
  });

  it('should match snapshot when rendering sorted user list', () => {
    const { container } = render(
      <UserList
        users={mockUsers}
        onUserSelect={() => {}}
      />
    );

    expect(container.firstChild).toMatchSnapshot();
  });

  it('should display users sorted by name in ascending order', () => {
    render(
      <UserList
        users={mockUsers}
        onUserSelect={() => {}}
        sortBy="name"
      />
    );

    const nameElements = screen.getAllByTestId(/^user-name-/);
    const displayedNames = nameElements.map(el => el.textContent);

    expect(displayedNames).toEqual([
      'Alice Wonderland',
      'Bob Anderson',
      'Zoe Wilson',
    ]);
  });

  it('should display users sorted by email in ascending order', () => {
    render(
      <UserList
        users={mockUsers}
        onUserSelect={() => {}}
        sortBy="email"
      />
    );

    const emailElements = screen.getAllByTestId(/^user-email-/);
    const displayedEmails = emailElements.map(el => el.textContent);

    expect(displayedEmails).toEqual([
      'alice@example.com',
      'bob@example.com',
      'zoe@example.com',
    ]);
  });

  it('should sort filtered users correctly', () => {
    render(
      <UserList
        users={mockUsers}
        onUserSelect={() => {}}
        filterText="a"
        sortBy="name"
      />
    );

    const nameElements = screen.getAllByTestId(/^user-name-/);
    const displayedNames = nameElements.map(el => el.textContent);

    expect(displayedNames).toEqual([
      'Alice Wonderland',
      'Bob Anderson',
    ]);
  });

  describe('Sort order variations', () => {
    test.each([
      ['name', ['Alice Wonderland', 'Bob Anderson', 'Zoe Wilson']],
      ['email', ['alice@example.com', 'bob@example.com', 'zoe@example.com']],
    ])('should sort by %s correctly', (sortBy, expectedOrder) => {
      render(
        <UserList
          users={mockUsers}
          onUserSelect={() => {}}
          sortBy={sortBy}
        />
      );

      if (sortBy === 'name') {
        const nameElements = screen.getAllByTestId(/^user-name-/);
        const displayedNames = nameElements.map(el => el.textContent);
        expect(displayedNames).toEqual(expectedOrder);
      } else if (sortBy === 'email') {
        const emailElements = screen.getAllByTestId(/^user-email-/);
        const displayedEmails = emailElements.map(el => el.textContent);
        expect(displayedEmails).toEqual(expectedOrder);
      }
    });
  });

  it('should match snapshot for loading state', () => {
    const { container } = render(
      <UserList
        users={mockUsers}
        onUserSelect={() => {}}
        isLoading={true}
      />
    );

    expect(container.firstChild).toMatchSnapshot();
  });

  it('should match snapshot for empty state', () => {
    const { container } = render(
      <UserList
        users={[]}
        onUserSelect={() => {}}
      />
    );

    expect(container.firstChild).toMatchSnapshot();
  });

  it('should handle case-insensitive sorting correctly', () => {
    const usersWithMixedCase = [
      { id: 1, name: 'Zoe Wilson', email: 'Zoe@example.com', role: 'User' },
      { id: 2, name: 'alice Wonderland', email: 'alice@example.com', role: 'Admin' },
    ];

    render(
      <UserList
        users={usersWithMixedCase}
        onUserSelect={() => {}}
        sortBy="email"
      />
    );

    const emailElements = screen.getAllByTestId(/^user-email-/);
    const displayedEmails = emailElements.map(el => el.textContent);

    expect(displayedEmails[0]).toBe('alice@example.com');
  });
});
```

---

## Complete Component File Reference

**File:** `lab018/src/UserList.js` (Final Version)

```js
import React, { useMemo } from 'react';

// UserList component for displaying and managing users in the system
// Props:
//   - users (array): Array of user objects to display [{id, name, email, role}, ...]
//   - onUserSelect (function): Callback when a user row is clicked - receives user object
//   - isLoading (boolean, optional): Shows loading state when true (default: false)
//   - filterText (string, optional): Text to filter users by name case-insensitively (default: '')
//   - sortBy (string, optional): Sort users by 'name' or 'email' (default: null - no sorting)
function UserList({ 
  users = [], 
  onUserSelect, 
  isLoading = false, 
  filterText = '',
  sortBy = null 
}) {
  // Memoize the filtered and sorted users to avoid unnecessary recalculations
  // This hook optimizes performance - only recalculates when users, filterText, or sortBy change
  // Dependencies array: [users, filterText, sortBy] triggers recalculation when any change
  const processedUsers = useMemo(() => {
    // Validate that users is actually an array
    if (!Array.isArray(users)) {
      return [];
    }

    // Step 1: Filter users by name if filterText is provided
    // toLowerCase() on both sides makes the filter case-insensitive
    // includes() checks if the user's name contains the filter text
    let result = users.filter(user =>
      user.name.toLowerCase().includes(filterText.toLowerCase())
    );

    // Step 2: Sort the filtered results if sortBy is specified
    if (sortBy === 'name') {
      // Sort by user name in ascending alphabetical order
      // localeCompare() provides proper alphabetical comparison
      // toLowerCase() ensures case-insensitive sorting
      result = result.sort((a, b) =>
        a.name.toLowerCase().localeCompare(b.name.toLowerCase())
      );
    } else if (sortBy === 'email') {
      // Sort by user email in ascending alphabetical order
      result = result.sort((a, b) =>
        a.email.toLowerCase().localeCompare(b.email.toLowerCase())
      );
    }

    return result;
  }, [users, filterText, sortBy]);

  // Render Loading State: Show loading indicator if isLoading prop is true
  // This prevents rendering the list while data is being fetched
  if (isLoading) {
    return <div data-testid="loading-indicator">Loading users...</div>;
  }

  // Render Empty State: Show empty state message if no users match the filter
  // This provides user feedback when no data is available
  if (processedUsers.length === 0) {
    return <div data-testid="empty-state">No users found</div>;
  }

  // Render User List: Display the processed (filtered and sorted) users in a table
  return (
    <div data-testid="user-list-container" className="user-list">
      <table data-testid="user-list-table" className="user-table">
        {/* Table Header Row */}
        <thead>
          <tr>
            <th>Name</th>
            <th>Email</th>
            <th>Role</th>
          </tr>
        </thead>
        {/* Table Body: Map each processed user to a clickable table row */}
        <tbody>
          {processedUsers.map((user) => (
            // Each row represents one user and is clickable
            <tr
              key={user.id}
              data-testid={`user-row-${user.id}`}
              onClick={() => onUserSelect(user)}
              className="user-row"
              style={{ cursor: 'pointer' }}
            >
              {/* User Name Cell */}
              <td data-testid={`user-name-${user.id}`}>{user.name}</td>
              {/* User Email Cell */}
              <td data-testid={`user-email-${user.id}`}>{user.email}</td>
              {/* User Role Cell */}
              <td data-testid={`user-role-${user.id}`}>{user.role}</td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}

export default UserList;
```

---

## Running The Lab

### Quick Test Run

To run all tests for this lab:

```bash
npm test -- ./lab018/test/UserList.test.js
```

To run tests in watch mode (re-runs when files change):

```bash
npm test -- ./lab018/test/UserList.test.js --watch
```

To run tests with coverage report:

```bash
npm test -- ./lab018/test/UserList.test.js --coverage
```

To update snapshots (after intentional UI changes):

```bash
npm test -- ./lab018/test/UserList.test.js -u
```

### Build and Browser Run Using Webpack

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
  entry: './lab018/src/index.js', // Entry point for UserList lab
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
      template: 'lab018/index.html',
    }),
  ].filter(Boolean),
  resolve: {
    extensions: ['.js', '.jsx'],
  },
};
```

4. **Create React entry point `lab018/src/index.js`:**

```js
import React, { useState } from 'react';
import { createRoot } from 'react-dom/client';
import UserList from './UserList';
import './index.css';

function App() {
  const [selectedUser, setSelectedUser] = useState(null);
  const [sortBy, setSortBy] = useState(null);
  const [filterText, setFilterText] = useState('');
  const [isLoading, setIsLoading] = useState(false);

  const mockUsers = [
    { id: 1, name: 'Alice Johnson', email: 'alice@example.com', role: 'Admin' },
    { id: 2, name: 'Bob Smith', email: 'bob@example.com', role: 'User' },
    { id: 3, name: 'Charlie Brown', email: 'charlie@example.com', role: 'Manager' },
    { id: 4, name: 'Diana Prince', email: 'diana@example.com', role: 'User' },
    { id: 5, name: 'Edward Norton', email: 'edward@example.com', role: 'Admin' },
  ];

  const handleUserSelect = (user) => {
    console.log('User selected:', user);
    setSelectedUser(user);
  };

  const handleSort = (sortField) => {
    setSortBy(sortBy === sortField ? null : sortField);
  };

  return (
    <div className="app-container">
      <h1>User Management System</h1>
      
      <div className="controls">
        <input
          type="text"
          placeholder="Filter by name..."
          value={filterText}
          onChange={(e) => setFilterText(e.target.value)}
          className="filter-input"
        />
        
        <button 
          onClick={() => handleSort('name')}
          className={sortBy === 'name' ? 'active' : ''}
        >
          Sort by Name
        </button>
        
        <button 
          onClick={() => handleSort('email')}
          className={sortBy === 'email' ? 'active' : ''}
        >
          Sort by Email
        </button>
        
        <button onClick={() => setSortBy(null)}>Clear Sort</button>
        
        <button onClick={() => setIsLoading(!isLoading)}>
          {isLoading ? 'Stop Loading' : 'Start Loading'}
        </button>
      </div>

      <UserList
        users={mockUsers}
        onUserSelect={handleUserSelect}
        filterText={filterText}
        sortBy={sortBy}
        isLoading={isLoading}
      />

      {selectedUser && (
        <div className="selected-user">
          <h2>Selected User</h2>
          <p><strong>Name:</strong> {selectedUser.name}</p>
          <p><strong>Email:</strong> {selectedUser.email}</p>
          <p><strong>Role:</strong> {selectedUser.role}</p>
          <button onClick={() => setSelectedUser(null)}>Clear Selection</button>
        </div>
      )}
    </div>
  );
}

// Get the root element
const rootElement = document.getElementById('root');

// Check if root element exists
if (!rootElement) {
  console.error('Root element not found! Check your index.html');
} else {
  console.log('Root element found, mounting React app...');
  const root = createRoot(rootElement);
  root.render(<App />);
  console.log('React app mounted successfully!');
}

```

5. **Create `lab018/src/index.css`:**

```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen',
    'Ubuntu', 'Cantarell', 'Fira Sans', 'Droid Sans', 'Helvetica Neue',
    sans-serif;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
  padding: 20px;
}

.app-container {
  max-width: 1000px;
  margin: 0 auto;
  background: white;
  border-radius: 12px;
  padding: 30px;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.1);
}

h1 {
  color: #333;
  margin-bottom: 30px;
  text-align: center;
}

.controls {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
  flex-wrap: wrap;
}

.filter-input {
  flex: 1;
  min-width: 250px;
  padding: 10px 15px;
  border: 2px solid #ddd;
  border-radius: 6px;
  font-size: 14px;
  transition: border-color 0.3s;
}

.filter-input:focus {
  outline: none;
  border-color: #667eea;
}

button {
  padding: 10px 20px;
  background: #f0f0f0;
  border: 2px solid #ddd;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  transition: all 0.3s;
}

button:hover {
  background: #e0e0e0;
}

button.active {
  background: #667eea;
  color: white;
  border-color: #667eea;
}

.user-list {
  margin: 20px 0;
}

.user-table {
  width: 100%;
  border-collapse: collapse;
  margin: 20px 0;
}

.user-table thead {
  background: #f5f5f5;
}

.user-table th {
  padding: 12px;
  text-align: left;
  font-weight: 600;
  color: #333;
  border-bottom: 2px solid #ddd;
}

.user-table td {
  padding: 12px;
  border-bottom: 1px solid #eee;
}

.user-row {
  cursor: pointer;
  transition: background 0.2s;
}

.user-row:hover {
  background: #f9f9f9;
}

.user-row:active {
  background: #efefef;
}

.selected-user {
  margin-top: 30px;
  padding: 20px;
  background: #f0f8ff;
  border-left: 4px solid #667eea;
  border-radius: 6px;
}

.selected-user h2 {
  color: #667eea;
  margin-bottom: 15px;
}

.selected-user p {
  margin: 8px 0;
  color: #555;
}
```

6. **Create `lab018/index.html`:**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>UserList Component Demo</title>
</head>
<body>
  <div id="root"></div>
</body>
</html>

```

7. **Install CSS loader for Webpack:**

```bash
npm install --save-dev style-loader css-loader
```

8. **Update `package.json` scripts:**

```json
"scripts": {
  "start": "webpack serve --env development",
  "build": "webpack --mode production",
  "test": "jest"
}
```

9. **Run your development server:**

```bash
npm start
```

Your React app should open automatically and display the UserList component with interactive filtering and sorting features.

10. **Run your unit tests:**

```bash
npm test -- ./lab018/test/UserList.test.js -u
```

---

## Key Takeaways

1. **Mocking Functions with jest.fn()** allows you to track callback invocations and verify they receive the correct arguments, enabling isolated component testing.

2. **Mocking Modules with jest.mock()** prevents external dependencies (like API calls) from executing during tests, making tests fast, reliable, and independent.

3. **Snapshot Testing** captures UI structure and quickly detects unintended changes without writing assertions for every element.

4. **Testing Props and State** ensures components respond correctly to different inputs and changes, validating component behavior under various conditions.

5. **Parameterized Tests** with test.each reduce code duplication when testing multiple similar scenarios.

6. **Performance Optimization** with useMemo helps prevent unnecessary recalculations when filtering and sorting data.

7. **TDD Approach** ensures your code is testable from the start and requirements are clearly defined before implementation.

---

## Troubleshooting

**Tests timeout?**
- Ensure all async operations are properly handled
- Check for infinite loops in filtering or sorting logic

**Snapshots mismatch?**
- Review the changes carefully
- Run: `npm test -- ./lab018/test/UserList.test.js -u`

**Module not found errors?**
- Verify babel.config.js and jest.config.js are in the root folder
- Check all imports use correct relative paths

**Sorting not working?**
- Verify sortBy prop is being passed correctly
- Check localeCompare usage for proper string comparison

**Filtering issues?**
- Verify toLowerCase() is used on both filter text and user names
- Check that the includes() method is being used correctly