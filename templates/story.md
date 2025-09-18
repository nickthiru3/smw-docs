# Story: [Story Name]

As a [role]  
I want to [action/goal]  
So that [benefit/value]

## Background

[Any information that is common to all scenarios]

## Scenarios

**Scenario: [Happy path scenario name]**

[Mention the steps of the scenario and acceptance criteria]

[Include user flow diagram, if applicable]

**Scenario: [Alternative/error path scenario name]**

[Mention the steps of the scenario that lead to the alternative/error path and acceptance criteria]

## Technical Constraints

- Input/Output limits: [specify limits]
- File requirements: [specify if applicable]
- Performance requirements: [specify if applicable]
- Security requirements: [specify if applicable]

## Dependencies

- Authentication: [specify requirements]
- Storage: [specify requirements]
- External Services: [specify requirements]

## Testing Scope

- Unit Tests:
  - [Component/function to test]
  - [Expected behavior to verify]
- Integration Tests:
  - [Integration points to test]
  - [Expected system behavior]
- E2E Tests:
  - [User flow to test]
  - [Expected end-to-end behavior]

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

### Route

[path]: Components:

      [component name]:
        Purpose: [brief description]
        Props: [if applicable]
        State: [if applicable]

    Page Server:
      Actions:
        [action name]: [description]

### UI Components

[List any specific UI components needed]

### State Management

[Describe any state management requirements]

## Non-Functional Requirements

[Any non-functional requirements]

## Notes

- [Any additional implementation notes]
- [Known limitations]
- [Future considerations]
