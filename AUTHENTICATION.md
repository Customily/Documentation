# Customily API Authentication

Several Customily API endpoints require authentication. There are two ways to authenticate:

- **API key**: send your account's API key in a header. Recommended for integrations, since no password is shared.
- **JWT token**: exchange your username and password for a token.

## Using an API Key

Find your API key in Customily under **Account → Integrations** ([how to generate an API key](https://help.customily.com/articles/8574175414-how-to-generate-an-api-key)).

Include it in the `x-api-key` header of your API requests. No call to `/api/token` is needed:

```
x-api-key: YOUR_API_KEY
```

**Example using cURL:**

```bash
curl "https://app.customily.com/api/Product/GetProductsCount" \
  -H "Accept: application/json" \
  -H "x-api-key: YOUR_API_KEY"
```

Notes:

- The API key acts as the account owner, so keep it private.
- Regenerating the key in **Account → Integrations** revokes the previous one immediately.
- Template (`/api/Product/...`) and image upload endpoints accept an API key. Team management endpoints (`/api/Team`) require a JWT token.

## Obtaining a JWT Token

Call the Customily token endpoint with your username and password:

```
POST https://app.customily.com/api/token
Content-Type: application/x-www-form-urlencoded
```

**Request body:**

```
grant_type=password&username=YOUR_USERNAME&password=YOUR_PASSWORD
```

**Response:**

```json
{
    "access_token": "eyJhbGciOi...",
    "token_type": "Bearer",
    "expires_in": 2592000
}
```

The `access_token` is the JWT token to use in the `Authorization` header of subsequent API calls.

**Example using cURL:**

```bash
curl -X POST "https://app.customily.com/api/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=password&username=YOUR_USERNAME&password=YOUR_PASSWORD"
```

## Using the Token

Include the token in the `Authorization` header of your API requests:

```
Authorization: Bearer eyJhbGciOi...
```
