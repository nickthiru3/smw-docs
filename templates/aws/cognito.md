# Cognito Template

## User Pool

- Self-signup: [Enabled | Disabled]
- Password policy:
  - Minimum length: [Length]
  - Require lowercase: [Yes | No]
  - Require uppercase: [Yes | No]
  - Require digits: [Yes | No]
  - Require symbols: [Yes | No]
- Sign-in attributes: [Email | Username]
- Auto-verify: [Email | Username]
- Keep original attributes: [Email | Username]
- Case-sensitive sign-in: [Yes | No]
- User verification:
  - Email subject: [Subject]
  - Email body: [Body]
  - Email style: [CODE | CONFIRMATION | CONFIRMATION_WITH_LINK]
- Account recovery: [Email | Username]
- Standard attributes:
  - [Name]; [Type]; [Required | Optional]; [Immutable | Mutable]
- Custom attributes:
  - [Name]; [Type]; [Required | Optional]; [Immutable | Mutable]
- Removal policy: [RETAIN | DELETE]
- Lambda Triggers:
  - Custom Message: [Function Name]; [Description]

## User Pool Client

- Generate secret: [Yes | No]
- Auth flows:
  - User password
  - Admin user password
- Access token validity: [Hours]
- OAuth:
  - Flows: [Description]
  - Scopes: [OPENID, EMAIL, PROFILE]
  - Callback URLs: [URL]
- Prevent user existence errors: [Yes | No]

## User Groups

- [Name]: [Description]
