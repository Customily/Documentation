# Customily API Authentication

Several Customily API endpoints require a JWT token for authentication. This guide explains how to obtain one.

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
