# Custom Add to Cart Integration

This guide explains how to use your own "Add to Cart" button instead of Customily's built-in one. This is useful when you need full control over the checkout flow or want to integrate the add-to-cart action into your platform's native UI.

## Overview

When using Customily's built-in "Add to Cart" button, the iframe automatically calls three operations and sends the result to the parent page via `postMessage`. If you want to use your own button, you need to call these same operations yourself using JavaScript functions exposed by the Customily engraver.

The three operations are:

1. **Generate the production file request** — reserves a URL for the production-ready file
2. **Generate a preview thumbnail** — uploads a preview image and returns a CDN URL
3. **Create a cart record** — saves the personalization data on Customily's server

## Step 1: Generate the Production File Request

Call `generatePFRPostOrder()` on the engraver to create the production file request:

```javascript
const exportedFiles = await window.engraver.generatePFRPostOrder('', false, 'YOUR_STORE.customily.com');
```

| Parameter     | Type    | Description                                      |
| ------------- | ------- | ------------------------------------------------ |
| `fileName`    | string  | Optional filename (can be empty)                 |
| `forceNotify` | boolean | Set to `false`                                   |
| `shop`        | string  | Your store identifier                            |

**Returns:** `ExportedFile[]`

```javascript
[
    {
        "url": "https://cdn.customily.com/ExportFile/...",
        "personalizationId": "abc-123-...",
        "epsId": "456"
    }
]
```

> Note: The `url` is a placeholder — the actual file won't be available until you call the [item/generate endpoint](INTEGRATION_GUIDE.md#23-generating-the-production-file) after checkout.

## Step 2: Generate the Preview Thumbnail

Call `generatePreviewImage()` to upload a preview and get a CDN URL:

```javascript
const preview = await window.engraver.generatePreviewImage({
    width: 1000,
    height: 1000,
    quality: 100,
    shop: 'YOUR_STORE.customily.com'
});
```

**Returns:**

```javascript
{
    "filename": "c611faa2-...",
    "previewUrl": "https://cdn.customily.com/shopify/assetFiles/previews/YOUR_STORE.customily.com/c611faa2-....jpeg",
    "thumbnailUrl": "https://cdn.customily.com/..."
}
```

Use `previewUrl` as the cart thumbnail image.

## Step 3: Create the Cart Record

Post the personalization data to Customily's cart endpoint:

```javascript
const response = await fetch('https://sh.customily.com/api/standalone/cart?shop=YOUR_STORE.customily.com', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
        productId: window.engraver.currentProduct.id,
        quantity: 1,
        sessionId: window.engraver.getSessionId(),
        previewUrl: preview.previewUrl,
        exportedFiles: exportedFiles,
        options: []    // see note below
    })
});

const cartItem = await response.json();
// cartItem.id is the personalizationGUID
```

> Note: The `options` array is used to store the shopper's selections (e.g. `[{ name: "Name", value: "John", type: "Text Input" }]`). If you don't need to retrieve them later, you can pass an empty array. However, if you need to display or reference what the shopper entered at fulfillment time, populate this array with the relevant option data.

## Complete Example

```javascript
async function customilyAddToCart(shop, quantity) {
    // 1. Generate production file request
    const exportedFiles = await window.engraver.generatePFRPostOrder('', false, shop);

    // 2. Generate preview thumbnail
    const preview = await window.engraver.generatePreviewImage({
        width: 1000,
        height: 1000,
        quality: 100,
        shop: shop
    });

    // 3. Wait for any pending file uploads (images, vectors) to complete
    await window.engraver.waitFilesUpload();

    // 4. Create cart record
    const response = await fetch(`https://sh.customily.com/api/standalone/cart?shop=${shop}`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
            productId: window.engraver.currentProduct.id,
            quantity: quantity,
            sessionId: window.engraver.getSessionId(),
            previewUrl: preview.previewUrl,
            exportedFiles: exportedFiles,
            options: []
        })
    });

    const cartItem = await response.json();

    return {
        personalizationGUID: cartItem.id,
        previewUrl: preview.previewUrl,
        exportedFiles: exportedFiles
    };
}

// Usage
document.getElementById('my-add-to-cart-btn').addEventListener('click', async () => {
    const result = await customilyAddToCart('YOUR_STORE.customily.com', 1);
    console.log('Added to cart:', result.personalizationGUID);
    console.log('Preview:', result.previewUrl);

    // Add to your platform's cart here...
});
```

## JavaScript API Reference

These methods are available on `window.engraver`:

| Method | Returns | Description |
| ------ | ------- | ----------- |
| `generatePFRPostOrder(fileName, forceNotify, shop)` | `Promise<ExportedFile[]>` | Generates the production file request |
| `generatePreviewImage(options)` | `Promise<{ previewUrl, thumbnailUrl, filename }>` | Uploads preview image, returns CDN URLs |
| `getSessionId()` | `string` | Returns the current engraver session ID |
| `waitFilesUpload()` | `Promise<void>` | Waits for pending image/vector uploads to complete |
| `getElementsUrls(type, id)` | `string[]` | Gets URLs for uploaded images (`'image'`) or vectors (`'vector'`) |
| `currentProduct.id` | `string` | The current template/product ID |
