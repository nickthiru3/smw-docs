# Resend verification code

A user who needs to verify their email can request a resend of the verification code

## Scenario: Successful sign up

    Visits merchant sign up page

    Fills in merchant details

    Submits merchant details

    Is shown a message asking them to check their email for a verification code

    They can also request a new verification code after a timeout of 5 minutes?

    Clicks on the "Resend verification code" button

    Receives a new verification code

## Backend

### API Gateway

**Method**: POST  
**Path**: /auth/resend-verification  
**Headers**:  
Content-Type: application/json

**Files**:

- Construct: `backend/lib/api/http/endpoints/auth/resend-verification/construct.js`
- Schema: `backend/lib/api/http/endpoints/auth/resend-verification/schema.js`
- OpenAPI: `docs/oas/resources/auth/resend-verification.yml`

**Request Body**:

```json
{
  // ResendConfirmationCodeRequest
  "ClientId": "STRING_VALUE", // required
  "SecretHash": "STRING_VALUE",
  "UserContextData": {
    // UserContextDataType
    "IpAddress": "STRING_VALUE",
    "EncodedData": "STRING_VALUE"
  },
  "Username": "STRING_VALUE", // required
  "AnalyticsMetadata": {
    // AnalyticsMetadataType
    "AnalyticsEndpointId": "STRING_VALUE"
  },
  "ClientMetadata": {
    // ClientMetadataType
    "<keys>": "STRING_VALUE"
  }
}
```

**Responses**:

- 200 OK:
  ```json
  {
    "success": true,
    "message": string; required; "Account registered. Complete sign-up with OTP",
    "username": string; required; Email address used for registration
  }
  ```
- 400 Bad Request:
  ```json
  {
    "success": false,
    "error": string; required; Error message describing the validation failure
  }
  ```

**Integration**: AWS Lambda Proxy  
**Lambda Function**: Merchant Sign-Up Lambda  
**Request Validation**: Body (using API Gateway Model and RequestValidator)  
**Authorization**: None (Public endpoint)

### Lambda

**Name**: MerchantSignUpLambda  
**Runtime**: Node.js 20.x  
**Handler**: handler  
**Files**:

- Handler: `backend/src/lambda/merchants/account/sign-up/handler.js`
- Construct: `backend/lib/lambda/merchants/account/sign-up/construct.js`

**Environment Variables**:

- USER_POOL_ID: Cognito User Pool ID
- USER_POOL_CLIENT_ID: Cognito User Pool Client ID

**IAM Permissions**:

- cognito-idp:SignUp (Allow)
- cognito-idp:AdminAddUserToGroup (Allow on User Pool ARN)

**Function Logic**:

1. Receives merchant sign-up data from API Gateway
2. Performs dynamic business validations:
   - Validates year of registration is not in the future
   - Validates year of registration is not too old (before 1900)
   - Validates website URL format if provided
   - Validates primary contact email is different from business email
3. Creates Cognito user with SignUpCommand
4. Sets all required attributes:
   - email
   - custom:businessName
   - custom:userGroup
   - custom:registrationNumber
   - custom:yearOfRegistration
   - custom:website
   - custom:address (JSON string)
   - custom:phone
   - custom:primaryContact (JSON string)
   - custom:productCategories (JSON string)
5. Adds user to "Merchants" group with AdminAddUserToGroupCommand
6. Returns success response with username and message

## Frontend

### Routes:

- `/merchants/sign-up`: Multi-step merchant registration form
- `/auth/verification-sent`: Verification email confirmation page
- `/auth/confirm-sign-up`: Code verification page

### Sign-up Page

**Files**:

- Component: `sveltekit/src/routes/merchants/sign-up/+page.svelte`
- Server Actions: `sveltekit/src/routes/merchants/sign-up/+page.server.js`
- Validation Schema: `sveltekit/src/routes/merchants/sign-up/schema.js`

**Features**:

- Multi-step form with 3 steps:
  - Step 1: Account Information (business name, email, password)
  - Step 2: Business Information (registration number, year, business type, website)
  - Step 3: Contact Information (address, phone, primary contact, product categories)
- Client-side validation using Svelte 5 runes
- Server-side validation using Zod schemas
- Progress indicator showing current step
- Form state persistence between steps using cookies
- Responsive design using Tailwind CSS
- Error handling and feedback

**Implementation**:

- Uses Svelte 5 runes for reactivity:
  - `$state()` for reactive variables
  - `$derived()` for computed values
  - `$effect()` for side effects
- Follows universal component format (no separate script/style sections)
- Implements progressive enhancement with server-side form handling
- Uses Tailwind CSS for styling
- Communicates with backend API through merchant service

### Verification Sent Page

**Files**:

- Component: `sveltekit/src/routes/auth/verification-sent/+page.svelte`
- Server Load: `sveltekit/src/routes/auth/verification-sent/+page.server.js`

**Features**:

- Confirmation that account was created successfully
- Instructions to check email for verification code
- Countdown timer for code expiration (5 minutes)
- Buttons to:
  - Enter verification code
  - Resend verification code
  - Go to sign-in
- Adapts messaging based on user type (merchant or customer)
- Secure access (redirects if no pending confirmation)

**Implementation**:

- Uses Svelte 5 runes for reactivity
- Loads email and user type data from cookies
- Implements countdown timer with cleanup on component unmount
- Uses Tailwind CSS for responsive design
- Provides clear user guidance on next steps

### API Integration

**Files**:

- Service: `sveltekit/src/lib/services/api/merchantService.js`

**Features**:

- Handles API communication between frontend and backend
- Supports both real API and mock API for development
- Standardized error handling
- Type definitions using JSDoc

**Implementation**:

- `signUp()` function sends merchant data to backend API
- Handles success and error responses
- Returns standardized response format
- Supports development with mock data

### Validation Schema

**Files**:

- Frontend: `sveltekit/src/routes/merchants/sign-up/schema.js`
- Backend: `backend/lib/api/http/endpoints/merchants/account/sign-up/schema.js`

**Features**:

- Step-specific validation schemas
- Comprehensive field validation
- Custom error messages
- Type safety

**Implementation**:

- Frontend: Uses Zod for form validation with three schemas:
  - `step1Schema`: Validates account information
  - `step2Schema`: Validates business information
  - `step3Schema`: Validates contact information
- Backend: Uses "JSON schema draft 4" to generate Schema for API Gateway validation

## Integration Points

1. **Frontend to Backend**:

   - SvelteKit form submits to page server action
   - Server action calls merchant service API
   - API Gateway validates request using schema
   - Lambda processes request and creates Cognito user

2. **Email Verification Flow**:

   - Cognito triggers custom message Lambda
   - Custom email sent based on user type
   - User redirected to verification sent page
   - User enters code on confirm sign-up page

3. **Post-Verification**:
   - User status changes from UNCONFIRMED to CONFIRMED
   - User can sign in with verified credentials
   - User receives follow-up instructions for document verification
