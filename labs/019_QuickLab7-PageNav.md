# Lab 019: Page Navigation Component TDD Example Using Node.js, Jest, JSDOM, & React

**Technologies:** Node.js v22.20.0, Jest, jsdom, React

## Domain: Page Navigation Component with Hash Routing

Create and test a Page Navigation component with hash-based routing (#home, #about, #contact), page content display, and browser navigation support using Test-Driven Development (TDD). This lab covers event listener testing, DOM event simulation, hash change detection, and error state handling.

---

## Jest Features Covered in This Lab

### 1. **Event Listener Testing**
Testing that event listeners are properly attached and triggered. In our navigation component, we'll verify that `hashchange` events are listened to and trigger the correct handlers. This ensures the component responds correctly when users click links or use browser back/forward buttons.

**Why useful:** Confirms that your component properly responds to external events and user interactions without manual browser testing.

### 2. **Mocking Window Events**
Jest allows us to mock and simulate browser events like `hashchange`. We'll test how the component handles route changes without relying on actual browser navigation. This includes testing what happens when the hash changes externally (e.g., manual URL modification).

**Why useful:** Enables testing of browser-level behavior in isolation without actual page reloads or complex browser manipulation.

### 3. **DOM Query Assertions (getByTestId, queryByTestId)**
Testing DOM queries to verify that the correct content is rendered. We'll use `getByTestId` to find specific page elements and verify they're displayed when their route is active, and `queryByTestId` to confirm content is NOT displayed when routes are inactive.

**Why useful:** Ensures the correct page content displays for each route without relying on brittle CSS selectors or text content that might change.

### 4. **Setup and Teardown with beforeEach/afterEach**
These hooks ensure test isolation by resetting the hash and component state between tests. Critical for navigation testing since tests might pollute the URL hash.

**Why useful:** Prevents tests from interfering with each other and guarantees clean state for each test run.

### 5. **Error State Testing (expect.not.toBeNull)**
Testing error conditions and edge cases. We'll verify that invalid routes display a 404 message and that the component gracefully handles unexpected hash values.

**Why useful:** Ensures your component fails gracefully and provides user feedback for invalid navigation attempts.

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

- Now React (Should have been done in precious assignments)
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

Ensure your `jest.config.js` in the root folder includes the lab019 path:

```js
module.exports = {
  testEnvironment: "jsdom",
  roots: ["<rootDir>/basic-js-tdd", "<rootDir>/lab003", "<rootDir>/lab015", "<rootDir>/lab019", "<rootDir>/"],
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
<root-folder>/lab019/
    ├── src/
    │   ├── PageNavigation.js
    │   ├── pages/
    │   │   ├── HomePage.js
    │   │   ├── AboutPage.js
    │   │   ├── ContactPage.js
    │   │   └── NotFoundPage.js
    │   └── App.js
    └── test/
        └── PageNavigation.test.js
```

All production code will go in `src`, all tests go in `test`.

---

## Page Navigation Component Specification

The PageNavigation component is a hash-based router that displays different pages based on the URL hash. The application supports the following features:

**Component Properties:**
- None required (uses window.location.hash directly)

**Routes Supported:**
- `#home` - Home page content
- `#about` - About page content
- `#contact` - Contact page content
- Empty hash or invalid route - Shows 404 Not Found page

**Features:**
- Navigation links for each route
- Active link highlighting (shows which page you're on)
- Page content updates when hash changes
- Browser back/forward button support
- 404 error page for invalid routes

**Visual Structure:**
- Navigation bar with Home, About, Contact links
- Active link styling (bold or highlighted)
- Page content container
- 404 message for invalid routes

---

## Business Requirements (Gherkin Format)

```gherkin
Feature: Page Navigation with Hash Routing

  Scenario: Display home page on initial load
    Given the application loads
    When the hash is empty or #home
    Then the home page content is displayed
    And the Home navigation link is highlighted as active

  Scenario: Navigate to about page via link click
    Given the application is displaying the home page
    When the user clicks the About navigation link
    Then the URL hash changes to #about
    And the about page content is displayed
    And the About link is highlighted as active

  Scenario: Navigate to contact page via link click
    Given the application is on any page
    When the user clicks the Contact navigation link
    Then the URL hash changes to #contact
    And the contact page content is displayed
    And the Contact link is highlighted as active

  Scenario: Handle browser back button
    Given the user navigated to #about then #contact
    When the user clicks the browser back button
    Then the hash changes back to #about
    And the about page content is displayed

  Scenario: Handle invalid route
    Given the application is running
    When the hash is set to an invalid route like #invalid
    Then the 404 Not Found page is displayed
    And no navigation link is highlighted as active

  Scenario: Highlight active navigation link
    Given the application displays the home page
    When the user clicks the About link and navigates to #about
    Then only the About link is highlighted as active
    And the Home link is no longer highlighted
```

---

# TDD Walkthrough (3 Passes)

For each pass, you will:
- Write fail-first tests (red phase)
- Implement just enough code to make tests pass (green phase)
- Clean/refactor if needed (refactor phase)
- Include descriptive comments in test functions

---

## PASS 1: Initial Route Display & Navigation Link Clicks

### Business Requirements Covered in Pass 1:
- Display home page on initial load
- Navigate to about page via link click
- Navigate to contact page via link click

### Jest Features Covered in Pass 1:
- DOM Query Assertions (getByTestId, queryByTestId)
- Event Listener Testing (click events)
- Setup and Teardown (beforeEach)

### RED Phase - Write Failing Tests

**File:** `lab019/test/PageNavigation.test.js`

```js
import React from 'react';
import { render, screen, fireEvent } from '@testing-library/react';
import PageNavigation from '../src/PageNavigation';
import '@testing-library/jest-dom';

describe('PageNavigation Component - Pass 1: Initial Route & Link Navigation', () => {
  // Setup before each test - reset hash and clear DOM
  beforeEach(() => {
    window.location.hash = '';
    document.body.innerHTML = '';
  });

  // Test 1: Initial page displays home content
  test('should display home page content when component first loads', () => {
    render(<PageNavigation />);
    
    // Query for home page content - should be present
    const homeContent = screen.getByTestId('home-page-content');
    expect(homeContent).toBeInTheDocument();
  });

  // Test 2: Home link is highlighted on initial load
  test('should highlight Home navigation link as active on initial load', () => {
    render(<PageNavigation />);
    
    // Query for navigation links
    const homeLink = screen.getByTestId('nav-home-link');
    
    // Home link should have active class/attribute
    expect(homeLink).toHaveClass('active');
  });

  // Test 3: Clicking About link changes hash and displays about content
  test('should navigate to about page when About link is clicked', () => {
    render(<PageNavigation />);
    
    // Get the About navigation link and click it
    const aboutLink = screen.getByTestId('nav-about-link');
    fireEvent.click(aboutLink);
    
    // Verify the hash changed to #about
    expect(window.location.hash).toBe('#about');
    
    // Verify about page content is now displayed
    const aboutContent = screen.getByTestId('about-page-content');
    expect(aboutContent).toBeInTheDocument();
  });

  // Test 4: About link is highlighted after clicking
  test('should highlight About link as active after navigation', () => {
    render(<PageNavigation />);
    
    // Click About link
    const aboutLink = screen.getByTestId('nav-about-link');
    fireEvent.click(aboutLink);
    
    // About link should have active class
    expect(aboutLink).toHaveClass('active');
    
    // Home link should NOT have active class anymore
    const homeLink = screen.getByTestId('nav-home-link');
    expect(homeLink).not.toHaveClass('active');
  });

  // Test 5: Clicking Contact link displays contact page
  test('should navigate to contact page when Contact link is clicked', () => {
    render(<PageNavigation />);
    
    // Get the Contact navigation link and click it
    const contactLink = screen.getByTestId('nav-contact-link');
    fireEvent.click(contactLink);
    
    // Verify the hash changed to #contact
    expect(window.location.hash).toBe('#contact');
    
    // Verify contact page content is now displayed
    const contactContent = screen.getByTestId('contact-page-content');
    expect(contactContent).toBeInTheDocument();
  });

  // Test 6: Contact link is highlighted and others are not
  test('should highlight Contact link and remove active class from others', () => {
    render(<PageNavigation />);
    
    // Click Contact link
    const contactLink = screen.getByTestId('nav-contact-link');
    fireEvent.click(contactLink);
    
    // Contact link should be active
    expect(contactLink).toHaveClass('active');
    
    // Other links should NOT be active
    const homeLink = screen.getByTestId('nav-home-link');
    const aboutLink = screen.getByTestId('nav-about-link');
    expect(homeLink).not.toHaveClass('active');
    expect(aboutLink).not.toHaveClass('active');
  });
});
```

### GREEN Phase - Implement Minimal Code

Now create the component files that will make these tests pass.

**File:** `lab019/src/PageNavigation.js`

```js
import React, { useState, useEffect } from 'react';

// Import page components
import HomePage from './pages/HomePage';
import AboutPage from './pages/AboutPage';
import ContactPage from './pages/ContactPage';

const PageNavigation = () => {
  // State to track current active route
  const [currentRoute, setCurrentRoute] = useState('home');

  // Effect to initialize route from hash on component mount
  useEffect(() => {
    // Get initial hash from URL and remove the '#' character
    const hash = window.location.hash.substring(1) || 'home';
    setCurrentRoute(hash);
  }, []);

  // Function to handle navigation link clicks
  const handleNavClick = (route) => {
    // Update the window hash
    window.location.hash = `#${route}`;
    // Update state to show correct page
    setCurrentRoute(route);
  };

  // Function to determine if a link should be highlighted
  const isActive = (route) => {
    return currentRoute === route;
  };

  // Function to render the correct page component based on current route
  const renderPage = () => {
    switch (currentRoute) {
      case 'home':
        return <HomePage />;
      case 'about':
        return <AboutPage />;
      case 'contact':
        return <ContactPage />;
      default:
        return <HomePage />;
    }
  };

  return (
    <div className="page-navigation-container">
      {/* Navigation Bar */}
      <nav className="navigation-bar">
        <button
          data-testid="nav-home-link"
          className={isActive('home') ? 'nav-link active' : 'nav-link'}
          onClick={() => handleNavClick('home')}
        >
          Home
        </button>
        <button
          data-testid="nav-about-link"
          className={isActive('about') ? 'nav-link active' : 'nav-link'}
          onClick={() => handleNavClick('about')}
        >
          About
        </button>
        <button
          data-testid="nav-contact-link"
          className={isActive('contact') ? 'nav-link active' : 'nav-link'}
          onClick={() => handleNavClick('contact')}
        >
          Contact
        </button>
      </nav>

      {/* Page Content */}
      <div className="page-content">
        {renderPage()}
      </div>
    </div>
  );
};

export default PageNavigation;
```

**File:** `lab019/src/pages/HomePage.js`

```js
import React from 'react';

const HomePage = () => {
  return (
    <div data-testid="home-page-content" className="page-content-home">
      <h1>Home Page</h1>
      <p>Welcome to the home page. This is the default page displayed when the application loads.</p>
    </div>
  );
};

export default HomePage;
```

**File:** `lab019/src/pages/AboutPage.js`

```js
import React from 'react';

const AboutPage = () => {
  return (
    <div data-testid="about-page-content" className="page-content-about">
      <h1>About Page</h1>
      <p>Learn more about us on this page.</p>
    </div>
  );
};

export default AboutPage;
```

**File:** `lab019/src/pages/ContactPage.js`

```js
import React from 'react';

const ContactPage = () => {
  return (
    <div data-testid="contact-page-content" className="page-content-contact">
      <h1>Contact Page</h1>
      <p>Get in touch with us through this page.</p>
    </div>
  );
};

export default ContactPage;
```

### Running Pass 1 Tests

Execute this command in your terminal:

```bash
npm test -- lab019/test/PageNavigation.test.js --testNamePattern="Pass 1"
```

All 6 tests should now pass. If they don't, verify that:
- All files are created in the correct locations
- Component names match the import statements
- `data-testid` attributes are spelled exactly as in the tests
- The `active` class is applied correctly

---

## PASS 2: Hash Change Events & Browser Back/Forward Support

### Business Requirements Covered in Pass 2:
- Handle browser back button
- Handle invalid route (404 detection)

### Jest Features Covered in Pass 2:
- Mocking Window Events (hashchange event)
- Event Listener Testing (external hash changes)
- Error State Testing

### RED Phase - Write Failing Tests

- Add these tests to `lab019/test/PageNavigation.test.js` AFTER the closing brace of the Pass 1 test suite but before the end if file

```js
describe('PageNavigation Component - Pass 2: Hash Change Events & Invalid Routes', () => {
  // Setup before each test - reset hash and clear DOM
  beforeEach(() => {
    window.location.hash = '';
    document.body.innerHTML = '';
  });

  // Helper function to trigger hashchange event manually
  const triggerHashChange = () => {
    const event = new Event('hashchange');
    window.dispatchEvent(event);
  };

  // Test 1: Component responds to external hash change (simulate browser back button)
  test('should update page content when hash changes externally (back button)', async () => {
    const { rerender } = render(<PageNavigation />);
    
    // Simulate user navigating to about page
    window.location.hash = '#about';
    
    // Trigger hashchange event to simulate browser recognizing the hash change
    triggerHashChange();
    
    // Re-render to pick up state changes from hashchange event
    rerender(<PageNavigation />);
    
    // About page content should now be displayed
    const aboutContent = screen.getByTestId('about-page-content');
    expect(aboutContent).toBeInTheDocument();
  });

  // Test 2: Component responds to contact hash change via back button
  test('should display contact page when hash changes to #contact externally', async () => {
    render(<PageNavigation />);
    
    // Simulate browser navigation - set hash directly and fire event
    window.location.hash = '#contact';
    triggerHashChange();
    
    // Wait a moment for state to update
    await new Promise(resolve => setTimeout(resolve, 50));
    
    // Contact page should be displayed
    const contactContent = screen.getByTestId('contact-page-content');
    expect(contactContent).toBeInTheDocument();
  });

  // Test 3: Invalid route displays 404 page
  test('should display 404 page when hash is set to invalid route', () => {
    window.location.hash = '#invalid';
    
    render(<PageNavigation />);
    
    // Query for 404 content - should be present for invalid routes
    const notFoundContent = screen.queryByTestId('not-found-page-content');
    expect(notFoundContent).toBeInTheDocument();
  });

  // Test 4: 404 page displays correct message
  test('should show 404 message for unrecognized route', () => {
    window.location.hash = '#unknown-page';
    
    render(<PageNavigation />);
    
    // The not found page should be displayed
    const notFoundContent = screen.getByTestId('not-found-page-content');
    expect(notFoundContent).toBeInTheDocument();
    
    // Verify 404 message is visible
    expect(screen.getByText(/404|not found/i)).toBeInTheDocument();
  });

  // Test 5: No navigation link is active for 404 page
  test('should not highlight any navigation link when displaying 404 page', () => {
    window.location.hash = '#nonexistent';
    
    render(<PageNavigation />);
    
    // Get all navigation links
    const homeLink = screen.getByTestId('nav-home-link');
    const aboutLink = screen.getByTestId('nav-about-link');
    const contactLink = screen.getByTestId('nav-contact-link');
    
    // None should have active class
    expect(homeLink).not.toHaveClass('active');
    expect(aboutLink).not.toHaveClass('active');
    expect(contactLink).not.toHaveClass('active');
  });

  // Test 6: Empty hash defaults to home page
  test('should display home page when hash is empty', () => {
    window.location.hash = '';
    
    render(<PageNavigation />);
    
    // Home page content should be displayed
    const homeContent = screen.getByTestId('home-page-content');
    expect(homeContent).toBeInTheDocument();
    
    // Home link should be active
    const homeLink = screen.getByTestId('nav-home-link');
    expect(homeLink).toHaveClass('active');
  });
});
```

### GREEN Phase - Update Component Code

You need to modify the PageNavigation component to handle hashchange events and invalid routes.

**File:** `lab019/src/PageNavigation.js` - Update the component

Find the import statement at the top:

```js
import React, { useState, useEffect } from 'react';

// Import page components
import HomePage from './pages/HomePage';
import AboutPage from './pages/AboutPage';
import ContactPage from './pages/ContactPage';
```

**ADD this line after the ContactPage import:**

```js
import NotFoundPage from './pages/NotFoundPage';
```

The imports section should now look like:

```js
import React, { useState, useEffect } from 'react';

// Import page components
import HomePage from './pages/HomePage';
import AboutPage from './pages/AboutPage';
import ContactPage from './pages/ContactPage';
import NotFoundPage from './pages/NotFoundPage';
```

Now find the main component function declaration and the useEffect hook:

```js
const PageNavigation = () => {
  // State to track current active route
  const [currentRoute, setCurrentRoute] = useState('home');

  // Effect to initialize route from hash on component mount
  useEffect(() => {
    // Get initial hash from URL and remove the '#' character
    const hash = window.location.hash.substring(1) || 'home';
    setCurrentRoute(hash);
  }, []);
```

**REPLACE the existing useEffect hook above with this updated version:**

```js
  // Effect to initialize route from hash on component mount AND listen for hash changes
  useEffect(() => {
    // Function to update route based on current hash
    const updateRouteFromHash = () => {
      const hash = window.location.hash.substring(1) || 'home';
      setCurrentRoute(hash);
    };

    // Set initial route on mount
    updateRouteFromHash();

    // Listen for hashchange events (browser back/forward, manual URL changes)
    window.addEventListener('hashchange', updateRouteFromHash);

    // Cleanup: remove event listener when component unmounts
    return () => {
      window.removeEventListener('hashchange', updateRouteFromHash);
    };
  }, []);
```

Now find the `renderPage()` function. It currently has this switch statement:

```js
  const renderPage = () => {
    switch (currentRoute) {
      case 'home':
        return <HomePage />;
      case 'about':
        return <AboutPage />;
      case 'contact':
        return <ContactPage />;
      default:
        return <HomePage />;
    }
  };
```

**REPLACE the default case with a return to NotFoundPage:**

```js
  const renderPage = () => {
    switch (currentRoute) {
      case 'home':
        return <HomePage />;
      case 'about':
        return <AboutPage />;
      case 'contact':
        return <ContactPage />;
      default:
        return <NotFoundPage />;
    }
  };
```

Also update the `isActive()` function to handle the 404 case. Find this function:

```js
  const isActive = (route) => {
    return currentRoute === route;
  };
```

**REPLACE it with this updated version:**

```js
  const isActive = (route) => {
    // Only return true if current route exactly matches and it's a valid route
    const validRoutes = ['home', 'about', 'contact'];
    if (!validRoutes.includes(currentRoute)) {
      return false; // No link is active for 404 pages
    }
    return currentRoute === route;
  };
```

**File:** `lab019/src/pages/NotFoundPage.js` - Create new file

Create this new file in the pages folder:

```js
import React from 'react';

const NotFoundPage = () => {
  return (
    <div data-testid="not-found-page-content" className="page-content-not-found">
      <h1>404 - Page Not Found</h1>
      <p>Sorry, the page you requested does not exist. Please navigate using the menu above.</p>
    </div>
  );
};

export default NotFoundPage;
```

### Running Pass 2 Tests

Execute this command in your terminal:

```bash
npm test -- lab019/test/PageNavigation.test.js --testNamePattern="Pass 2"
```

All 6 new tests should now pass. If they don't, verify that:
- The `hashchange` event listener is properly attached and removed
- The NotFoundPage component exists and has the correct test ID
- The switch statement returns `<NotFoundPage />` for invalid routes
- The `isActive()` function returns false for 404 pages
- The helper function `triggerHashChange()` is properly creating and dispatching a new Event

**Important Note on hashchange Testing:** In JSDOM (the DOM environment Jest uses), you must manually create and dispatch the hashchange event using `new Event('hashchange')` and `window.dispatchEvent()`. This is different from the browser where navigation changes automatically trigger the event. The helper function in the tests handles this properly.

---

## PASS 3: Comprehensive Navigation Flow & Active State Management

### Business Requirements Covered in Pass 3:
- Highlight active navigation link
- All navigation scenarios working together

### Jest Features Covered in Pass 3:
- Combined Event Testing (multiple events in sequence)
- State Management Assertions
- Integration Testing

### RED Phase - Write Failing Tests

- Add these tests to `lab019/test/PageNavigation.test.js` AFTER the closing brace of the Pass 1 test suite but before the end if file

```js
describe('PageNavigation Component - Pass 3: Comprehensive Navigation Flow & State Management', () => {
  // Setup before each test - reset hash and clear DOM
  beforeEach(() => {
    window.location.hash = '';
    document.body.innerHTML = '';
  });

  // Helper function to trigger hashchange event manually
  const triggerHashChange = () => {
    const event = new Event('hashchange');
    window.dispatchEvent(event);
  };

  // Test 1: Multiple sequential navigations work correctly
  test('should maintain correct state through multiple sequential navigations', () => {
    render(<PageNavigation />);
    
    // Start on home
    const homeContent = screen.getByTestId('home-page-content');
    expect(homeContent).toBeInTheDocument();
    
    // Navigate to about
    const aboutLink = screen.getByTestId('nav-about-link');
    fireEvent.click(aboutLink);
    
    const aboutContent = screen.getByTestId('about-page-content');
    expect(aboutContent).toBeInTheDocument();
    expect(screen.queryByTestId('home-page-content')).not.toBeInTheDocument();
    
    // Navigate to contact
    const contactLink = screen.getByTestId('nav-contact-link');
    fireEvent.click(contactLink);
    
    const contactContent = screen.getByTestId('contact-page-content');
    expect(contactContent).toBeInTheDocument();
    expect(screen.queryByTestId('about-page-content')).not.toBeInTheDocument();
  });

  // Test 2: Only one link is active at any given time
  test('should ensure only one navigation link has active class at a time', () => {
    render(<PageNavigation />);
    
    const homeLink = screen.getByTestId('nav-home-link');
    const aboutLink = screen.getByTestId('nav-about-link');
    const contactLink = screen.getByTestId('nav-contact-link');
    
    // Initially home is active
    expect(homeLink).toHaveClass('active');
    expect(aboutLink).not.toHaveClass('active');
    expect(contactLink).not.toHaveClass('active');
    
    // Click about
    fireEvent.click(aboutLink);
    expect(homeLink).not.toHaveClass('active');
    expect(aboutLink).toHaveClass('active');
    expect(contactLink).not.toHaveClass('active');
    
    // Click contact
    fireEvent.click(contactLink);
    expect(homeLink).not.toHaveClass('active');
    expect(aboutLink).not.toHaveClass('active');
    expect(contactLink).toHaveClass('active');
  });

  // Test 3: Clicking same link twice keeps it active
  test('should keep navigation link active when clicked multiple times', () => {
    render(<PageNavigation />);
    
    const aboutLink = screen.getByTestId('nav-about-link');
    
    // First click to navigate
    fireEvent.click(aboutLink);
    expect(aboutLink).toHaveClass('active');
    
    // Second click on same link
    fireEvent.click(aboutLink);
    expect(aboutLink).toHaveClass('active');
    
    // About page should still be displayed
    const aboutContent = screen.getByTestId('about-page-content');
    expect(aboutContent).toBeInTheDocument();
  });

  // Test 4: Back/forward navigation maintains active state
  test('should update active link when navigating via browser back button', () => {
    const { rerender } = render(<PageNavigation />);
    
    // Navigate to about via click
    const aboutLink = screen.getByTestId('nav-about-link');
    fireEvent.click(aboutLink);
    
    expect(aboutLink).toHaveClass('active');
    expect(screen.getByTestId('about-page-content')).toBeInTheDocument();
    
    // Simulate browser back button - set hash back to home
    window.location.hash = '#home';
    triggerHashChange();
    rerender(<PageNavigation />);
    
    // Home link should be active now
    const homeLink = screen.getByTestId('nav-home-link');
    expect(homeLink).toHaveClass('active');
    expect(aboutLink).not.toHaveClass('active');
  });

  // Test 5: Navigation shows only relevant page content
  test('should display only the current page content, not others', () => {
    render(<PageNavigation />);
    
    // Only home content should exist
    expect(screen.getByTestId('home-page-content')).toBeInTheDocument();
    expect(screen.queryByTestId('about-page-content')).not.toBeInTheDocument();
    expect(screen.queryByTestId('contact-page-content')).not.toBeInTheDocument();
    
    // Navigate to about
    fireEvent.click(screen.getByTestId('nav-about-link'));
    
    // Now only about content should exist
    expect(screen.queryByTestId('home-page-content')).not.toBeInTheDocument();
    expect(screen.getByTestId('about-page-content')).toBeInTheDocument();
    expect(screen.queryByTestId('contact-page-content')).not.toBeInTheDocument();
  });

  // Test 6: Invalid route does not show any page content, only 404
  test('should not render any page content for invalid routes, only 404', () => {
    window.location.hash = '#invalid-route';
    
    render(<PageNavigation />);
    
    // 404 page should be shown
    expect(screen.getByTestId('not-found-page-content')).toBeInTheDocument();
    
    // No valid page content should be shown
    expect(screen.queryByTestId('home-page-content')).not.toBeInTheDocument();
    expect(screen.queryByTestId('about-page-content')).not.toBeInTheDocument();
    expect(screen.queryByTestId('contact-page-content')).not.toBeInTheDocument();
  });

  // Test 7: Navigating from 404 to valid page works correctly
  test('should properly navigate from 404 page back to valid page', () => {
    window.location.hash = '#invalid';
    
    const { rerender } = render(<PageNavigation />);
    
    // Verify 404 is shown
    expect(screen.getByTestId('not-found-page-content')).toBeInTheDocument();
    
    // Simulate clicking the home link
    const homeLink = screen.getByTestId('nav-home-link');
    fireEvent.click(homeLink);
    
    rerender(<PageNavigation />);
    
    // Home page should now be displayed
    expect(screen.getByTestId('home-page-content')).toBeInTheDocument();
    expect(screen.queryByTestId('not-found-page-content')).not.toBeInTheDocument();
    expect(homeLink).toHaveClass('active');
  });

  // Test 8: Hash is updated in URL when navigating
  test('should update window.location.hash with correct format when navigating', () => {
    render(<PageNavigation />);
    
    // Start with empty hash
    expect(window.location.hash).toBe('');
    
    // Click about
    fireEvent.click(screen.getByTestId('nav-about-link'));
    expect(window.location.hash).toBe('#about');
    
    // Click contact
    fireEvent.click(screen.getByTestId('nav-contact-link'));
    expect(window.location.hash).toBe('#contact');
    
    // Click home
    fireEvent.click(screen.getByTestId('nav-home-link'));
    expect(window.location.hash).toBe('#home');
  });
});
```

### GREEN Phase - Verify Existing Code

The existing code from Pass 1 and Pass 2 should already handle all the scenarios in Pass 3. Run the tests to verify everything passes. The component correctly:

1. Updates state when links are clicked
2. Maintains only one active link at a time (via the `isActive()` function)
3. Handles hashchange events for back/forward navigation
4. Shows only one page at a time (via the switch statement in `renderPage()`)
5. Shows 404 for invalid routes
6. Updates the hash in the URL (via `handleNavClick()`)

If all tests pass, no additional code changes are needed. If some tests fail, the most common issues are:

- **queryByTestId returning elements when it shouldn't:** Verify that only ONE page content div is being rendered (the renderPage() switch statement should only return one component)
- **Active class not updating:** Verify the `isActive()` function is called for every link in the JSX
- **Hash not updating:** Verify `handleNavClick()` includes `window.location.hash = #${route}`

### Running Pass 3 Tests

Execute this command in your terminal:

```bash
npm test -- lab019/test/PageNavigation.test.js --testNamePattern="Pass 3"
```

All 8 new tests should now pass. Your component now has comprehensive navigation functionality!

---

# Final Component Summary

The PageNavigation component is now a fully functional hash-based router with:

**Features Implemented:**
- ✅ Initial route display (home page on load)
- ✅ Navigation via link clicks
- ✅ Active link highlighting
- ✅ Browser back/forward button support
- ✅ Invalid route handling (404 page)
- ✅ Single page content display
- ✅ URL hash synchronization

**Code Structure:**
- `PageNavigation.js` - Main router component
- `HomePage.js`, `AboutPage.js`, `ContactPage.js` - Page components
- `NotFoundPage.js` - 404 error page

---

# Running The Lab

### Run All Tests for This Lab
```bash
npm test -- lab019/test/PageNavigation.test.js
```

### Run Tests by Pass
```bash
# Pass 1 only
npm test -- lab019/test/PageNavigation.test.js --testNamePattern="Pass 1"

# Pass 2 only
npm test -- lab019/test/PageNavigation.test.js --testNamePattern="Pass 2"

# Pass 3 only
npm test -- lab019/test/PageNavigation.test.js --testNamePattern="Pass 3"
```

### Generate Coverage Report
```bash
npm test -- lab019/test/PageNavigation.test.js --coverage
```

### Watch Mode (Re-run tests on file changes)
```bash
npm test -- lab019/test/PageNavigation.test.js --watch
```

### Verify Your Jest Configuration Includes Lab019
Before running tests, ensure your `jest.config.js` in the root folder includes:

```js
module.exports = {
  testEnvironment: "jsdom",
  roots: ["<rootDir>/lab019", "<rootDir>/"], // Includes lab019
  testMatch: ["**/test/**/*.js", "**/tests/**/*.js", "**/__tests__/**/*.js", "**/?(*.)+(spec|test).js"],
  coverageDirectory: "coverage",
  clearMocks: true,
};
```

### Troubleshooting

**Tests not finding components:**
- Verify all files are in `lab019/src/` and `lab019/test/`
- Check import paths use correct relative paths (`../src/PageNavigation`)

**hashchange event not working:**
- Use the helper function `triggerHashChange()` provided in the tests
- This creates a new Event and dispatches it: `window.dispatchEvent(new Event('hashchange'))`
- Do NOT use `fireEvent.hashChange()` - that method does not exist in React Testing Library
- JSDOM requires manual event creation and dispatch for hashchange events

**Active class not applying/removing:**
- Verify the `isActive()` function is called in the className rendering
- Check that the className is dynamically built: `{isActive('home') ? 'active' : ''}`

**404 page not showing:**
- Verify the switch statement has a default case that returns `<NotFoundPage />`
- Check that NotFoundPage.js exists and has the correct test ID

**Data Test IDs not found:**
- Verify exact spelling of test IDs (case-sensitive)
- All test IDs should start with `data-testid=`
- Expected format: `data-testid="home-page-content"`

---

## Extension Ideas

Once you've completed all 3 passes, consider these extensions:

1. **Add Navigation History**: Track visited pages and show a breadcrumb trail
2. **Dynamic Route Parameters**: Support routes like `#contact/:name` with parameters
3. **Lazy Loading Pages**: Import page components dynamically only when needed
4. **Animation Between Routes**: Add CSS transitions when switching pages
5. **Query String Support**: Handle both hash routes and query parameters
6. **Route Guards**: Prevent navigation to certain pages based on conditions
7. **Nested Routes**: Support sub-routes within pages like `#about/team`, `#about/history`

Each of these can be tested using the Jest features learned in this lab!