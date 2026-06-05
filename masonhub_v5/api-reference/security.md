# Security

## Overview

All MasonHub API endpoints are secured using JWT (JSON Web Token) encrypted Bearer tokens. These tokens provide secure authentication and authorization for all API operations.

## Security Scheme

| Security Scheme Type | HTTP |
| --- | --- |
| HTTP Authorization Scheme | bearer |
| Bearer format | JWT |

## Authentication Requirements

- **HTTPS Required**: All API requests must use HTTPS
- **Valid Bearer Token**: Every request must include a valid JWT bearer token
- **Runtime Decryption**: Tokens are encrypted and only decrypted at runtime for enhanced security
- **No Database Storage**: Tokens are not stored in databases but matched at runtime

## Token Management

### Bearer Token Header

Include your bearer token in the Authorization header of all requests:

```http
Authorization: Bearer your_jwt_token_here
```

### Token Generation

Tokens can be generated using the `/secrets` endpoint:

```http
POST /secrets
Content-Type: application/json

{
  "secret_phrase": "Your secure secret phrase"
}
```

### Token Security Best Practices

1. **Encrypted Tokens**: Use encrypted tokens for production environments
2. **Secure Storage**: Store tokens securely and avoid exposing them in client-side code
3. **Token Rotation**: Regularly rotate tokens for enhanced security
4. **Environment Separation**: Use different tokens for sandbox and production environments

## Callback Security

For webhook callbacks, MasonHub recommends:

- **Client-Supplied Tokens**: Include optional encrypted tokens in callback registrations
- **Token Verification**: Verify callback authenticity using the provided token
- **HTTPS Endpoints**: Ensure callback URLs use HTTPS

### Example Callback Registration with Token

```json
{
  "url": "https://api.clienturl.com/api/skuInventoryChange",
  "message_type": "skuInventoryChange", 
  "api_version": "1.0",
  "token": "your_encrypted_verification_token"
}
```

## API Support

For security-related questions or issues, contact: **support@masonhub.co**