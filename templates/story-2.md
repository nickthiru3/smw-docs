# Merchant sign up

Merchants can sign up for a pre-verified merchant account

## Scenario: Successful sign up

    Visits merchant sign up page

    Fills in merchant details

    Submits merchant details

    Receives email verification code

    Verifies email

    Is successfully signed up

    Receives email with follow-up instructions to complete document verification

## Backend

### API Gateway

**Method**: [METHOD]  
**Path**: [path]  
**Headers**:  
[header name]: [description]

    **Query Parameters**:
      [param name]: [type]; [required/optional]; [description]

    Body:
      {
        [field]: [type]; [required/optional]; [description]
      }

    Responses:
      [status code]:
        {
          [field]: [type]; [required/optional]; [description]
        }

### Storage

[Storage type]:
[Resource name]:
[Schema/Structure details]:

### DB

#### DB: [DB type]

&ensp; Entity: [Entity name]

    [Schema]

### Lambda

Name: [function-name]  
Purpose: [brief description]  
Triggers: [what invokes this function]  
Permissions: [required permissions]

## Frontend

### Route:

    merchants/sign-up

### Layout Components:

    ?

### Page Components:

    MerchantSignUpForm:

      Purpose: [brief description]

      Props: [if applicable]

      State:
        isLoading: [description]
        error: [description]

      Store: [if applicable]

      Events:
        submit: [description]

    Page Server:

      Loaders:
        [loader name]: [description]

      Actions:
        default: Sign up form submission

### UI Components

[List any specific UI components needed]

### State Management

[Describe any state management requirements]

## Non-Functional Requirements

[Any non-functional requirements]

## Notes

- This is a pre-verified account opening workflow. The user still hast to complete document verification to be able to sell products.
