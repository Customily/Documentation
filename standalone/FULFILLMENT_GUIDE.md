# Retrieving Personalization Details & Production Files

This guide explains how to retrieve the personalization details for an order item using the `personalizationGUID` you stored at cart time.

## When You Need This

If you stored the full `postMessage` payload (including `options`, `previewUrl`, and `exportedFiles`) when the shopper added the item to the cart, you already have everything you need — no API call required.

However, if you only stored the `personalizationGUID`, you can retrieve the full personalization details at any time via the Customily API. Even if you stored nothing at all, you can still retrieve the personalization details by order ID — as long as you included your e-commerce platform's order ID in the [`/standalone/cart`](CUSTOM_ADD_TO_CART.md#step-3-create-the-cart-record-on-customily-server) call.

## Retrieving Personalization Details

### By Personalization GUID

```
GET https://sh.customily.com/api/standalone/item/{personalizationGUID}
Authorization: Bearer <JWT_TOKEN>
```

**Example:**

```bash
curl -X GET "https://sh.customily.com/api/standalone/item/a1b2c3d4-e5f6-7890-abcd-ef1234567890" \
  -H "Authorization: Bearer eyJhbGciOi..."
```

**Response:**

```json
{
    "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "quantity": 1,
    "productId": "0313370a-e3d1-4b88-8640-f1027a78235d",
    "previewUrl": "https://cdn.customily.com/shopify/assetFiles/previews/yourstandalonestore.com/c611faa2-....jpeg",
    "productionUrl": "https://cdn.customily.com/ExportFile/...",
    "optionsJson": "[{\"name\":\"Name\",\"value\":\"John\",\"type\":\"Text Input\"},{\"name\":\"Date\",\"value\":\"June 15, 2025\",\"type\":\"Text Input\"}]",
    "exportedFiles": [
        { "url": "https://cdn.customily.com/ExportFile/..." }
    ],
    "createdDate": "2025-06-15T14:30:00.000Z",
    "generated": false
}
```

| Field | Type | Description |
| ----- | ---- | ----------- |
| `id` | string | The personalization GUID |
| `quantity` | number | Number of items |
| `productId` | string | The Customily template ID |
| `previewUrl` | string | CDN URL of the personalized preview image |
| `productionUrl` | string | Production file URL (available after calling [item/generate](INTEGRATION_GUIDE.md#23-generating-the-production-file)) |
| `optionsJson` | string | JSON string with the shopper's selections |
| `exportedFiles` | array | Array of production file URLs (one per template side) |
| `createdDate` | string | When the cart record was created |
| `generated` | boolean | Whether the production file has been generated |

### By Order ID

If you passed an `orderId` when creating the cart record, you can retrieve all personalization items for that order:

```
GET https://sh.customily.com/api/standalone/item?orderId={orderId}
Authorization: Bearer <JWT_TOKEN>
```

**Example:**

```bash
curl -X GET "https://sh.customily.com/api/standalone/item?orderId=12345" \
  -H "Authorization: Bearer eyJhbGciOi..."
```

**Response:** an array of personalization items (same format as above).

## Authentication

These endpoints require a JWT token in the `Authorization` header. See [Authentication](../AUTHENTICATION.md) for how to obtain one.

## Typical Fulfillment Flow

1. **Order is placed** — your platform confirms payment
2. **Generate the production file** — call [`POST /standalone/item/generate`](INTEGRATION_GUIDE.md#23-generating-the-production-file) with the production file URL
3. **Retrieve personalization details** — call `GET /standalone/item/{personalizationGUID}` to get the options, preview image, and production file URL
4. **Fulfill the order** — download the production file and use it for printing/manufacturing
