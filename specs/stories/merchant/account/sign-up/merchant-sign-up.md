# Merchant Sign-Up Workflow

## Overview

This document describes the complete merchant sign-up workflow in the Super Deals application, from initial form submission to email verification and successful account creation.

## Workflow Steps

1. **User Enters Sign-Up Form**
   - Route: `/merchants/sign-up`
   - User enters merchant account details
   - Form validation occurs on client-side

2. **Form Submission**
   - Form data is submitted to `+page.server.js`
   - Server-side validation occurs
   - User type (`merchant`) is extracted from the route path
   - Data is sent to `/api/accounts/merchants` endpoint

3. **API Processing**
   - API endpoint forwards request to AWS Lambda function
   - Lambda function validates data and creates Cognito user
   - User is added to the appropriate Cognito group (`merchants`)
   - Response is returned to the client

4. **Verification Email**
   - AWS Cognito triggers the custom message Lambda
   - Lambda customizes the email based on user type
   - Verification email is sent to the user
   - User is redirected to verification page

5. **Email Verification**
   - User clicks link in email or enters verification code
   - Verification request is processed by `confirm-sign-up/+page.server.js`
   - User type is retrieved from cookies
   - Amplify `confirmSignUp` API is called directly

6. **Verification Success**
   - Upon successful verification, cookies are cleared
   - User is redirected to `/auth/sign-up-success` with user type parameter
   - Success page displays confirmation message and sign-in link

## Implementation Details

### User Type Handling

The user type is captured from the route path and passed explicitly to the accounts service:

```javascript
// In merchants/sign-up/+page.server.js
export const actions = {
  default: async ({ request, fetch }) => {
    // Extract user type from route
    const userType = 'merchant';
    
    // Process form data
    const formData = await request.formData();
    const merchantData = Object.fromEntries(formData);
    
    // Pass user type to accounts service
    const result = await accountsService.signUp(userType, merchantData, fetch);
    
    // Handle result...
  }
};
```

### Verification Flow

The verification process uses cookies to maintain context between steps:

```javascript
// In confirm-sign-up/+page.server.js
export const actions = {
  default: async ({ request, cookies }) => {
    // Get user type from cookies
    const userType = cookies.get('pendingUserType') || 'merchant';
    
    // Process verification
    const result = await accountsService.verifyEmail(userType, email, code);
    
    // On success, redirect to success page
    if (result.isSignUpComplete) {
      throw redirect(303, `/auth/sign-up-success?userType=${userType}`);
    }
  }
};
```

### Error Handling

Comprehensive error handling is implemented throughout the flow:

```javascript
try {
  // Authentication operation
} catch (error) {
  // Extract error message if available
  if (error && typeof error === 'object') {
    if ('message' in error && typeof error.message === 'string') {
      errorMessage = error.message;
    } else if ('code' in error && typeof error.code === 'string') {
      // Handle specific error codes
      switch (error.code) {
        case 'CodeMismatchException':
          errorMessage = 'The verification code is incorrect. Please check and try again.';
          break;
        // Other cases...
      }
    }
  }
}
```

## Validation Strategy

Validation occurs at multiple levels:

1. **Client-side**: Basic form validation for immediate user feedback
2. **Server-side**: Comprehensive validation in `+page.server.js` before API calls
3. **API Gateway**: Schema validation using JSON Schema
4. **Lambda**: Final validation before Cognito operations

This multi-layered approach ensures data integrity and security throughout the process.

## AWS Services Integration

The workflow integrates with several AWS services:

- **Cognito**: User pool for authentication and user management
- **Lambda**: Custom logic for sign-up and verification
- **API Gateway**: RESTful API endpoints with validation
- **SES**: Email delivery for verification codes

## Best Practices

1. **User Type Separation**: Pass user type as a separate parameter
2. **Validation Layers**: Implement validation at each appropriate level
3. **Error Handling**: Provide user-friendly error messages
4. **State Management**: Use cookies for maintaining state during multi-step processes
5. **Security**: Perform authentication operations server-side in `+page.server.js`
