# Lab 011: Product Form TDD Example Using Node.js, Jest, JSDOM, & React

**Difficulty Level:** 2  
**Technologies:** Node.js v22.20.0, Jest, jsdom, React

## Domain: Product Form Walkthrough

Create and test a simple Product Form with two textboxes and a submit button using Test-Driven Development (TDD).

---

## Prerequisites

- You must already have Node.js, Jest, and JSDOM installed and working with a valid `jest.config.js`. 
- Jest should use jsdom as its environment and recognize your intended test folders.

### Step 1: Install dependencies
- Run below commands in <root-folder> 

- Install Jest and jsdom (should have been done in previous labs.)
```bash
npm install --save-dev jest jsdom
```

- The same folder where your package.json lives and your Node.js, Jest, and JSDOM installations are managed
```bash
npm install react react-dom
```
- Also install Babel dependencies for JSX compilation:
```bash
npm install @babel/core @babel/preset-env @babel/preset-react babel-jest --save-dev
```

- Install the React Testing Library as a developer dependency
```bash
npm install --save-dev @testing-library/react

```


- Add a basic Babel config file (`babel.config.js`):
- The **babel.config.js** file should be placed in the **root folder** of your project, alongside your package.json and where you run the npm install react react-dom command. This ensures Babel applies its configuration project-wide, including for your lab011/src React code during transpilation.

```js
module.exports = {
  presets: ['@babel/preset-env', '@babel/preset-react'],
};
```

---

## Project Structure

Create a new subfolder for this lab:

```
<root-folder>/lab011/
    ├── src/
    └── test/
```
All production code will go in `src`, all tests go in `test`.

---

## Product Form Specification

Form contains:
- **Product Name** (textbox)
- **Product Code** (textbox)
- **Submit** button

Each input type will have 1 positive and 1 negative requirement.
---

## Requirements

- **Product Name:**
    - Positive: Accepts non-empty strings.
    - Negative: Rejects empty strings.
- **Product Code:**
    - Positive: Accepts valid alphanumeric codes (e.g. `P123`, `X9Y4`).
    - Negative: Rejects empty/invalid (e.g. spaces only, symbols only).

---

# TDD Walkthrough (3 Passes)

For each pass, you will:
- Write fail-first tests for each input type's requirement (red)
- Implement just enough code for both tests to pass (green)
- Clean/refactor if needed (refactor)
- Include test functions with comments explaining the check

---

## PASS 1: Product Name Validation

### RED Phase - Write Failing Tests

**File:** `lab011/test/ProductForm.test.js`
```js
import React from 'react';
import { render, fireEvent } from '@testing-library/react';
import ProductForm from '../src/ProductForm';

// In this test, we are checking that submitting a non-empty product name is accepted
it('should accept non-empty product name', () => {
  // Test implementation goes here
});

// In this test, we will reject empty product name input
it('should reject empty product name', () => {
  // Test implementation goes here
});
```

**Run tests from the root folder using:** (Test should fail-red)

```bash
npm test -- ./lab011/test/ProductForm.test.js
```

### GREEN Phase - Make Tests Pass

**File:** `lab011/src/ProductForm.js`
```js
import React, { useState } from 'react';

function ProductForm() {
  const [name, setName] = useState('');
  const [code, setCode] = useState('');
  const [error, setError] = useState('');
  const handleSubmit = (e) => {
    e.preventDefault();
    if (name.trim() === '') {
      setError('Product Name required');
      return;
    }
    setError('');
    // ...submit logic
  };
  return (
    <form onSubmit={handleSubmit}>
      <input data-testid="name-input" value={name} onChange={e => setName(e.target.value)} placeholder="Product Name" />
      <input data-testid="code-input" value={code} onChange={e => setCode(e.target.value)} placeholder="Product Code" />
      <button type="submit">Submit</button>
      {error && <span data-testid="error-msg">{error}</span>}
    </form>
  );
}
export default ProductForm;
```

**Run tests from the root folder using:** (Test should pass-green)

```bash
npm test -- ./lab011/test/ProductForm.test.js
```


### Refactor
- No major refactor yet, keep code clear.

---

## PASS 2: Product Code Validation

### RED Phase - Add Failing Tests

**File:** `lab011/test/ProductForm.test.js`
```js
//**Important** : below exiting lines of code add this .....

// We are testing that a valid alphanumeric code is accepted by the form
it('should accept alphanumeric product code', () => {
  // Test implementation goes here
});

// This test will ensure submitting an invalid code gives error
it('should reject invalid product code', () => {
  // Test implementation goes here
});
```



**Run tests from the root folder using:** 

```bash
npm test -- ./lab011/test/ProductForm.test.js
```
**Note** : Ideally test should fail, but some times they pass because sometimes, provious implementation already satisfy these test, Or, test functions are overly permessive to fail the test, or sometimes there are issues with 'jest cache'.


### GREEN Phase - Make Tests Pass

**File:** `lab011/src/ProductForm.js` (update handleSubmit)
```js
import React, { useState } from 'react';

function ProductForm() {
  const [name, setName] = useState('');
  const [code, setCode] = useState('');
  const [error, setError] = useState('');

  const codePattern = /^[a-zA-Z0-9]+$/;  // Add here at component top

  const handleSubmit = (e) => {           // Replace existing handleSubmit
    e.preventDefault();
    if (name.trim() === '') {
      setError('Product Name required');
      return;
    }
    if (!codePattern.test(code)) {
      setError('Product Code invalid');
      return;
    }
    setError('');
    // ...submit logic
  };

  return (
    <form onSubmit={handleSubmit}>
      <input data-testid="name-input" value={name} onChange={e => setName(e.target.value)} placeholder="Product Name" />
      <input data-testid="code-input" value={code} onChange={e => setCode(e.target.value)} placeholder="Product Code" />
      <button type="submit">Submit</button>
      {error && <span data-testid="error-msg">{error}</span>}
    </form>
  );
}

export default ProductForm;

```
**Run tests from the root folder using:** 

```bash
npm test -- ./lab011/test/ProductForm.test.js
```

### Refactor
- Consider moving patterns to top for clarity.

---

## PASS 3: Edge/Error Handling

Add user feedback and form clearing after successful submit. 

### RED Phase - Error Display And Success Feedback

**File:** `lab011/test/ProductForm.test.js`
```js
//**Important** : below exiting lines of code add this .....

// In this test we ensure error message clears on valid submit
it('should clear error and form on successful submit', () => {
  // Test implementation goes here
});
```

### GREEN Phase - Update Implementation

**File:** `lab011/src/ProductForm.js`
```js
// Important-->Make sure you replace below code st the right place in existing code 

const handleSubmit = (e) => {
  e.preventDefault();
  if (name.trim() === '') {
    setError('Product Name required');
    return;
  }
  if (!codePattern.test(code)) {
    setError('Product Code invalid');
    return;
  }
  setError('');
  setName('');
  setCode('');
  // ...submit logic
};
```

---

# Running The Lab

### Build And Browser Run### Build And Browser Run Using Webpack

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
  entry: './lab011/src/index.js', // Entry point
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
      template: 'lab011/index.html',
    }),
  ].filter(Boolean),
  resolve: {
    extensions: ['.js', '.jsx'],
  },
};
```

4. **Your React entry point `lab011/src/index.js`:**

No renaming needed, just ensure JSX is allowed via Babel.

```js
import React from 'react';
import { createRoot } from 'react-dom/client';
import ProductForm from './ProductForm';

const root = createRoot(document.getElementById('root'));
root.render(<ProductForm />);
```

5. **Create `lab011/index.html` (or use existing):**
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Product Form Demo</title>
</head>
<body>
  <div id="root"></div> <!-- React mounts here -->
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

Your React app should open automatically and display your `ProductForm` in the browser with hot reloading.

8. **Run your unit tests:**
```bash
npm test
```


***

### Notes:

- Babel config allows JSX in `.js` files without renaming.
- Hot Module Replacement (HMR) enabled for smooth dev experience.
- The HTML Webpack Plugin injects your bundle into `index.html`.

***
---

# Summary

This walkthrough guided you to:
- Install React in an existing Node/Jest/JSDOM project
- Apply TDD in three distinct passes for two form fields and a button
- Construct and improve both test and implementation code for each requirement
- Build and run your React solution in the browser

**End of Lab Instructions**
