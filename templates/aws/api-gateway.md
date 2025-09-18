# API Gateway

**Method**: [Method]  
**Path**: [Path]  
**Headers**:  
[Header Name]: [Value]

**Files**:

- Construct: [Path]
- Schema: [Path]
- OpenAPI: [Path]

**Path Parameters**:

- [Name]: [Type]; [Required | Optional]; [Description]

**Request Body**:

```json
{
  [Name]: [Type]; [Required | Optional]; [Description]
}
```

**Responses**:

- [Status Code]: [Description]
  ```json
  {
    [Name]: [Type]; [Required | Optional]; [Description]
  }
  ```
- [Status Code]: [Description]
  ```json
  {
    [Name]: [Type]; [Required | Optional]; [Description]
  }
  ```

**Integration**: API Gateway Lambda Proxy  
**Lambda Function**: [Function Name]  
**Request Validation**: Body (using API Gateway Model and RequestValidator)  
**Authorization**: None (Public endpoint)
