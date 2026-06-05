# Tokens

The Tokens API manages JWT bearer tokens for authentication and authorization. These tokens provide secure access to all MasonHub API endpoints.

## Endpoints

### Generate a New Bearer Token

**POST** `/secrets`

Generate a new JWT bearer token using a secret phrase.

#### Request Body Schema

```json
{
  "secret_phrase": "Your secure secret phrase here"
}
```

#### Example Request

```json
{
  "secret_phrase": "My name is Inigo Montoya. As You WISH!!! You must have studied to be a greeper."
}
```

#### Response

```json
{
  "encrypted_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

### Delete Secret Phrase

**POST** `/delete_secrets`

Remove secret phrases from your account, invalidating associated tokens.

#### Request Body Schema

```json
{
  "secret_phrase": "Enter secret phrase exactly as entered during creation process"
}
```

#### Important Notes

- Secret phrase must match exactly as entered during token creation
- Deleting a secret phrase invalidates all tokens generated with that phrase
- This action cannot be undone

## Token Management

### Token Security

- **Encryption**: All tokens are encrypted for enhanced security
- **No Database Storage**: Tokens are not stored in databases
- **Runtime Verification**: Tokens are decrypted and verified at request time
- **Expiration**: Tokens have configurable expiration times

### Secret Phrase Requirements

Your secret phrase should be:
- **Unique**: Different from passwords or other credentials
- **Complex**: Include multiple words, numbers, and special characters
- **Memorable**: You'll need it for token management
- **Secure**: Not easily guessable or derivable

### Example Secret Phrases

```
"Marketing Campaign 2024 - Secure Token #1!"
"Production API Access - Department Finance v2.1"
"Integration Testing Token - Project Phoenix @2024"
```

## Token Usage

### Authentication Header

Include your bearer token in all API requests:

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### Complete Request Example

```http
GET /skus?limit=10
Host: sandbox.masonhub.co
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json
```

## Environment-Specific Tokens

### Sandbox Environment

Generate tokens specifically for sandbox testing:

```bash
curl -X POST https://sandbox.masonhub.co/account/api/v1/secrets \
  -H "Content-Type: application/json" \
  -d '{"secret_phrase": "Sandbox Testing Token 2024"}'
```

### Production Environment

Use different secret phrases for production:

```bash
curl -X POST https://app.masonhub.co/account/api/v1/secrets \
  -H "Content-Type: application/json" \
  -d '{"secret_phrase": "Production API Token - Secure Phrase 2024"}'
```

## Security Best Practices

### Token Storage

1. **Environment Variables**: Store tokens in environment variables
2. **Secure Vaults**: Use secure credential management systems
3. **No Source Code**: Never commit tokens to source control
4. **Rotation**: Regularly rotate tokens for security

### Token Rotation

Implement a token rotation strategy:

1. Generate new token with different secret phrase
2. Update applications to use new token
3. Test functionality with new token
4. Delete old secret phrase to invalidate previous token

### Access Control

- **Principle of Least Privilege**: Generate tokens with minimal required access
- **Environment Separation**: Use different tokens for different environments
- **Team Access**: Provide team members with individual tokens when possible
- **Audit Trail**: Monitor token usage and access patterns

## Error Handling

### Invalid Secret Phrase

```json
{
  "error": "Invalid secret phrase",
  "message": "The provided secret phrase is incorrect or does not exist"
}
```

### Token Expired

```json
{
  "error": "Token expired",
  "message": "The provided token has expired and must be regenerated"
}
```

### Malformed Token

```json
{
  "error": "Invalid token format",
  "message": "The provided token is malformed or corrupted"
}
```

## Integration Examples

### Node.js Example

```javascript
const axios = require('axios');

// Generate token
async function generateToken(secretPhrase) {
  const response = await axios.post(
    'https://sandbox.masonhub.co/account/api/v1/secrets',
    { secret_phrase: secretPhrase }
  );
  return response.data.encrypted_token;
}

// Use token in API request
async function getSkus(token) {
  const response = await axios.get(
    'https://sandbox.masonhub.co/account/api/v1/skus',
    {
      headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json'
      }
    }
  );
  return response.data;
}
```

### Python Example

```python
import requests

# Generate token
def generate_token(secret_phrase):
    response = requests.post(
        'https://sandbox.masonhub.co/account/api/v1/secrets',
        json={'secret_phrase': secret_phrase}
    )
    return response.json()['encrypted_token']

# Use token in API request
def get_skus(token):
    headers = {
        'Authorization': f'Bearer {token}',
        'Content-Type': 'application/json'
    }
    response = requests.get(
        'https://sandbox.masonhub.co/account/api/v1/skus',
        headers=headers
    )
    return response.json()
```

### cURL Example

```bash
# Generate token
TOKEN=$(curl -s -X POST https://sandbox.masonhub.co/account/api/v1/secrets \
  -H "Content-Type: application/json" \
  -d '{"secret_phrase": "My secure phrase"}' \
  | jq -r '.encrypted_token')

# Use token in API request  
curl -X GET https://sandbox.masonhub.co/account/api/v1/skus \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json"
```

## Troubleshooting

### Common Issues

1. **401 Unauthorized**: Token missing, invalid, or expired
2. **403 Forbidden**: Token valid but insufficient permissions
3. **Malformed Header**: Incorrect Authorization header format
4. **Network Issues**: HTTPS required for all token operations

### Token Validation

Test your token with a simple API call:

```bash
curl -X GET https://sandbox.masonhub.co/account/api/v1/skus?limit=1 \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -H "Content-Type: application/json"
```

### Support

For token-related issues, contact: **support@masonhub.co**

Include:
- Environment (sandbox/production)
- Error messages received
- Token generation timestamp
- Account information