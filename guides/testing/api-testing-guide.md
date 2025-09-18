# API Testing Guide

This document provides comprehensive guidance on testing the SuperDeals API endpoints using Supertest and Jest.

## Table of Contents

- [Overview](#overview)
- [Testing Architecture](#testing-architecture)
- [Test Configuration](#test-configuration)
- [Writing API Tests](#writing-api-tests)
- [Test Data Management](#test-data-management)
- [Running Tests](#running-tests)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

## Overview

The SuperDeals API testing strategy focuses on end-to-end testing of API Gateway endpoints using Supertest. These tests validate that our API endpoints correctly handle requests, perform business logic, and return appropriate responses.

### Key Testing Components

- **Supertest**: HTTP assertion library that allows testing API endpoints
- **Jest**: Testing framework for running tests and assertions
- **Test fixtures**: Reusable test data for consistent testing
- **Configuration system**: Dynamic configuration that adapts to environment changes

## Testing Architecture

The API tests are organized to mirror the API structure itself:

```
backend/
├── tests/
│   ├── api/                          # All API tests
│   │   ├── config.js                 # API testing configuration
│   │   ├── merchants/                # Tests for merchant endpoints
│   │   │   ├── accounts/             # Tests for merchant account endpoints
│   │   │   │   ├── sign-up/          # Tests for sign-up endpoint
│   │   │   │   │   ├── fixtures.js   # Test data for sign-up tests
│   │   │   │   │   ├── successful.test.js  # Happy path tests
│   │   │   │   │   └── errors.test.js      # Error case tests
│   │   │   │   └── ...
│   │   │   └── ...
│   │   └── ...
├── scripts/
│   └── update-test-config.js         # Script to update test configuration
```

This structure ensures:
1. Tests are easy to find (mirroring the API structure)
2. Test files are focused on specific functionality
3. Test data is separated from test logic
4. Configuration is centralized and automatically updated

## Test Configuration

The API testing configuration is managed through two key files:

### 1. `tests/api/config.js`

This file exports the API URL and other configuration needed by tests:

```javascript
const fs = require("fs");
const path = require("path");

function getApiUrl() {
  try {
    const outputsPath = path.resolve(__dirname, "../../outputs.json");
    const outputs = JSON.parse(fs.readFileSync(outputsPath, "utf8"));
    
    const apiUrl = outputs.BackendStackDevApiStackHttpStackB0C9C4D3?.RestApiUrldev;
    
    if (!apiUrl) {
      throw new Error("API URL not found in outputs.json");
    }
    
    return apiUrl;
  } catch (error) {
    console.error("Error reading API URL from outputs.json:", error);
    throw error;
  }
}

module.exports = {
  apiUrl: process.env.TEST_API_URL || getApiUrl(),
};
```

### 2. `scripts/update-test-config.js`

This script automatically updates the test configuration when the API URL changes:

```javascript
const fs = require("fs");
const path = require("path");

// Paths configuration
const OUTPUTS_PATH = path.resolve(__dirname, "../outputs.json");
const CONFIG_PATH = path.resolve(__dirname, "../tests/api/config.js");

function updateTestConfig() {
  try {
    // Read outputs.json
    const outputs = JSON.parse(fs.readFileSync(OUTPUTS_PATH, "utf8"));
    
    // Extract API URL from outputs
    const apiUrl = outputs.BackendStackDevApiStackHttpStackB0C9C4D3?.RestApiUrldev;
    
    if (!apiUrl) {
      throw new Error("API URL not found in outputs.json");
    }

    // Generate new config content
    const configContent = `const fs = require("fs");
const path = require("path");

/**
 * Get API URL from outputs.json
 * @returns {string} The API URL
 */
function getApiUrl() {
  try {
    const outputsPath = path.resolve(__dirname, "../../outputs.json");
    const outputs = JSON.parse(fs.readFileSync(outputsPath, "utf8"));
    
    // Extract API URL from outputs
    const apiUrl = outputs.BackendStackDevApiStackHttpStackB0C9C4D3?.RestApiUrldev;
    
    if (!apiUrl) {
      throw new Error("API URL not found in outputs.json");
    }
    
    return apiUrl;
  } catch (error) {
    console.error("Error reading API URL from outputs.json:", error);
    throw error;
  }
}

module.exports = {
  apiUrl: process.env.TEST_API_URL || getApiUrl(),
};`;

    // Write new config
    fs.writeFileSync(CONFIG_PATH, configContent, "utf8");
    console.log("Successfully updated test config with latest API URL");
  } catch (error) {
    console.error("Error updating test config:", error);
    process.exit(1);
  }
}

updateTestConfig();
```

This configuration system ensures:
1. Tests always use the latest API URL from CDK deployments
2. Environment variables can override the configuration for special test scenarios
3. Configuration is automatically updated before running tests

## Writing API Tests

### Test Structure

API tests follow this general structure:

```javascript
const request = require("supertest");
const { apiUrl } = require("../../config");
const { testData } = require("./fixtures");

describe("POST /endpoint-path", () => {
  beforeEach(async () => {
    // Setup: Create any necessary test data
  });

  afterEach(async () => {
    // Cleanup: Remove test data
  });

  test("successfully performs action", async () => {
    const response = await request(apiUrl)
      .post("/endpoint-path")
      .send(testData)
      .expect(201)
      .expect("Content-Type", /json/);

    // Assertions on response body
    expect(response.body).toHaveProperty("expectedProperty");
  });

  test("returns error for invalid input", async () => {
    const invalidData = { ...testData, requiredField: undefined };

    const response = await request(apiUrl)
      .post("/endpoint-path")
      .send(invalidData)
      .expect(400)
      .expect("Content-Type", /json/);

    expect(response.body).toHaveProperty("error");
  });
});
```

### Example: Merchant Sign-Up Test

Here's a real example testing the merchant sign-up endpoint:

```javascript
const request = require("supertest");
const { apiUrl } = require("#tests/api/config");
const { validMerchantData } = require("./fixtures");

describe("POST /merchants/account/signup", () => {
  beforeEach(async () => {
    // Clean up test data if needed
  });

  afterEach(async () => {
    // Additional cleanup if needed
  });

  test("successfully creates a merchant account", async () => {
    const response = await request(apiUrl)
      .post("/merchants/account/signup")
      .send(validMerchantData)
      .expect(201)
      .expect("Content-Type", /json/);

    // Verify response structure
    expect(response.body).toHaveProperty(
      "message",
      "Merchant registered. Needs to submit OTP to complete sign-up"
    );
    expect(response.body).toHaveProperty("userConfirmed");
    expect(response.body.userConfirmed).toBe(false);
    expect(response.body).toHaveProperty("merchantId");
    expect(response.body).toHaveProperty("codeDeliveryDetails");
    expect(response.body.codeDeliveryDetails).toHaveProperty("Destination");
    expect(response.body.codeDeliveryDetails).toHaveProperty("DeliveryMedium");
    expect(response.body.codeDeliveryDetails.DeliveryMedium).toBe("EMAIL");
  });

  // Additional test cases for error scenarios
});
```

## Test Data Management

Test data is managed through fixture files to keep tests clean and maintainable:

### Example: `fixtures.js`

```javascript
/**
 * Test fixtures for merchant sign-up tests
 */

exports.validMerchantData = {
  businessName: 'Test Business LLC',
  email: 'test@business.com',
  password: 'Test123!@#',
  registrationNumber: 'REG123456',
  yearOfRegistration: '2020',
  businessType: 'LLC',
  website: 'https://testbusiness.com',
  businessPhone: '+1234567890',
  address: {
    buildingNumber: '123',
    street: 'Test Street',
    city: 'Test City',
    state: 'Test State',
    zip: '12345',
    country: 'Test Country'
  },
  primaryContact: {
    name: 'John Doe',
    email: 'john@business.com',
    phone: '+1234567891'
  },
  productCategories: ['Electronics', 'Computers'],
  termsAccepted: true
};
```

Benefits of separate fixture files:
1. Reusable test data across multiple test files
2. Cleaner test files focused on test logic
3. Easier to update test data when API requirements change
4. Provides documentation of expected request structures

## Running Tests

The project includes npm scripts for running API tests:

```json
"scripts": {
  "test": "jest",
  "test:api": "jest tests/api",
  "pretest:api": "node scripts/update-test-config.js"
}
```

To run all API tests:

```bash
npm run test:api
```

This will:
1. Run the `pretest:api` script to update the config with the latest API URL
2. Run all tests in the `tests/api` directory

To run a specific test file:

```bash
npx jest tests/api/merchants/accounts/sign-up/successful.test.js
```

## Best Practices

### 1. Test Organization

- Mirror the API structure in test directory structure
- Group tests by endpoint and functionality
- Separate happy path and error case tests

### 2. Test Data

- Use fixture files for test data
- Create realistic test data that matches production scenarios
- Include edge cases in test data

### 3. Assertions

- Test both response status and content type
- Verify all important properties in response bodies
- Use specific assertions rather than general ones

### 4. Cleanup

- Always clean up test data in `afterEach` or `afterAll` hooks
- Ensure tests don't interfere with each other
- Consider using unique identifiers for test data to avoid conflicts

### 5. Error Cases

- Test all error scenarios defined in the API spec
- Verify error responses have the correct structure
- Test validation errors for all required fields

## Troubleshooting

### Common Issues

1. **API URL not found**
   - Ensure you've deployed the API using `npm run deploy`
   - Check that `outputs.json` exists and contains the API URL
   - Verify the path to the API URL in `config.js` matches your `outputs.json` structure

2. **Authentication errors**
   - For protected endpoints, ensure you're providing valid credentials
   - Check that the authentication token is being sent correctly

3. **Test data conflicts**
   - Use unique identifiers for test data
   - Ensure proper cleanup between tests
   - Consider using a test-specific database or isolation strategy

4. **Intermittent failures**
   - Add retries for flaky tests
   - Increase timeouts for slow operations
   - Add more detailed logging to identify issues

### Debugging Tips

1. Use `console.log` to inspect response bodies and request data
2. Check API Gateway logs in CloudWatch
3. Run tests with `--verbose` flag for more detailed output
4. Use `--testNamePattern` to run specific test cases
