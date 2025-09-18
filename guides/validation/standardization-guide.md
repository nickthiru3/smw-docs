# Validation Standardization Guide

## Overview

This document provides guidelines for implementing consistent validation across the Super Deals application. Standardizing validation ensures a consistent user experience, reduces code duplication, and improves maintainability.

## Validation Principles

1. **Single Source of Truth**: Define validation schemas in one place and reuse them
2. **Progressive Disclosure**: Show validation errors at the appropriate time
3. **Consistent Messages**: Use consistent error message formats and wording
4. **Appropriate Timing**: Validate at the right moment in the user journey

## Validation Layers

The Super Deals application implements validation at multiple layers to ensure data integrity and security:

### Client-Side Validation

- **Purpose**: Immediate user feedback
- **Implementation**: Form validation in Svelte components
- **When to Use**: For basic format validation and required fields

```javascript
// Example: Client-side validation in a Svelte component
let email = $state('');
let emailError = $state('');

function validateEmail() {
  if (!email) {
    emailError = 'Email is required';
    return false;
  }
  if (!/^\S+@\S+\.\S+$/.test(email)) {
    emailError = 'Please enter a valid email address';
    return false;
  }
  emailError = '';
  return true;
}
```

### Server-Side Validation (+page.server.js)

- **Purpose**: Comprehensive validation before API calls
- **Implementation**: Validation functions in `+page.server.js` actions
- **When to Use**: For all form submissions before sending to API

```javascript
// Example: Server-side validation in +page.server.js
import { validateMerchantData } from '$lib/validation/merchant';

export const actions = {
  default: async ({ request }) => {
    const formData = await request.formData();
    const data = Object.fromEntries(formData);
    
    // Validate using shared validation function
    const validationResult = validateMerchantData(data);
    if (!validationResult.success) {
      return { success: false, errors: validationResult.errors };
    }
    
    // Proceed with API call
    // ...
  }
};
```

### API Endpoint Validation (+server.js)

- **Purpose**: Security check before backend processing
- **Implementation**: Basic validation in API endpoints
- **When to Use**: As a security measure, not for detailed user feedback

```javascript
// Example: API endpoint validation in +server.js
import { validateMerchantData } from '$lib/validation/merchant';

export async function POST({ request }) {
  try {
    const data = await request.json();
    
    // Basic validation as security measure
    const validationResult = validateMerchantData(data);
    if (!validationResult.success) {
      return new Response(JSON.stringify({
        success: false,
        message: 'Invalid data provided'
      }), { status: 400 });
    }
    
    // Proceed with backend API call
    // ...
  } catch (error) {
    // Handle errors
  }
}
```

### API Gateway Validation

- **Purpose**: Request validation at the API Gateway level
- **Implementation**: JSON Schema with API Gateway request validators
- **When to Use**: For all API endpoints to ensure valid request structure before Lambda execution

```javascript
// Example: schema.js for API Gateway validation
const schema = {
  $schema: "http://json-schema.org/draft-04/schema#",
  title: "MerchantsAccountSignUpModel",
  type: "object",
  required: [
    "email",
    "password",
    "businessName",
    // Other required fields
  ],
  properties: {
    email: {
      type: "string",
      format: "email",
    },
    password: {
      type: "string",
      minLength: 8,
      pattern: "^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[^a-zA-Z\d]).{8,}$",
      description: "Password must be at least 8 characters and contain at least one lowercase letter, one uppercase letter, one number, and one special character",
    },
    yearOfRegistration: {
      type: "integer",
      minimum: 1900,
      maximum: new Date().getFullYear(),
    },
    // Other field validations
  }
};

module.exports = schema;
```

```javascript
// Example: construct.js for setting up API Gateway validation
const { Model, RequestValidator } = require("aws-cdk-lib/aws-apigateway");
const schema = require("./schema.js");

// Create model for request validation
const model = new Model(this, `Model`, {
  restApi: http.restApi,
  contentType: "application/json",
  description: "/merchants/account/signup",
  schema,
});

// Create request validator
const requestValidator = new RequestValidator(this, `RequestValidator`, {
  restApi: http.restApi,
  validateRequestBody: true,
  validateRequestParameters: false,
});

// Add POST method with Lambda integration and request validation
signupResource.addMethod(
  "POST",
  new LambdaIntegration(lambda.function),
  {
    operationName: "MerchantAccountSignUp",
    requestValidator,
    requestModels: {
      "application/json": model,
    },
  }
);
```

### Backend Validation (Lambda)

- **Purpose**: Final validation before database operations
- **Implementation**: Schema validation in Lambda functions
- **When to Use**: For all incoming requests to protect data integrity

```javascript
// Example: Lambda validation
const validateInput = (data) => {
  const schema = {
    // Schema definition
  };
  
  const validation = validate(data, schema);
  if (!validation.valid) {
    throw new Error('Invalid input data');
  }
  
  return data;
};
```

## Shared Validation Library

Create a shared validation library in `$lib/validation/` with reusable validation functions:

```javascript
// $lib/validation/merchant.js
export function validateMerchantData(data) {
  const errors = {};
  
  // Validate required fields
  if (!data.businessName) {
    errors.businessName = 'Business name is required';
  }
  
  if (!data.email) {
    errors.email = 'Email is required';
  } else if (!/^\S+@\S+\.\S+$/.test(data.email)) {
    errors.email = 'Please enter a valid email address';
  }
  
  // Validate year of registration
  if (!data.yearOfRegistration) {
    errors.yearOfRegistration = 'Year of registration is required';
  } else {
    const year = parseInt(data.yearOfRegistration);
    const currentYear = new Date().getFullYear();
    if (isNaN(year) || year < 1900 || year > currentYear) {
      errors.yearOfRegistration = `Year must be between 1900 and ${currentYear}`;
    }
  }
  
  // More validations...
  
  return {
    success: Object.keys(errors).length === 0,
    errors
  };
}
```

## Error Message Standardization

Use consistent error message formats:

1. **Field-specific errors**: Short, specific messages tied to form fields
2. **General errors**: Clear messages for system or API errors
3. **Validation groups**: Group related fields when appropriate

## Implementation Steps

1. **Create validation library**: Implement shared validation functions in `$lib/validation/`
2. **Update client components**: Use validation library in Svelte components
3. **Standardize server validation**: Update `+page.server.js` files to use shared validation
4. **Align API validation**: Ensure API endpoints use consistent validation
5. **Synchronize JSON Schema**: Keep API Gateway JSON Schema in sync with frontend validation
6. **Document validation rules**: Maintain documentation of validation rules

## Best Practices

1. **Avoid duplication**: Don't repeat validation logic across different files
2. **Balance user experience**: Don't overwhelm users with too many validation messages at once
3. **Provide clear guidance**: Help users understand how to fix validation errors
4. **Consider accessibility**: Ensure validation errors are accessible to all users
5. **Maintain schema consistency**: Keep JSON Schema definitions consistent between API Gateway and frontend validation
6. **Use appropriate formats**: Leverage JSON Schema formats (email, uri, date-time) for standard validations
7. **Test thoroughly**: Validate your validation with comprehensive tests
