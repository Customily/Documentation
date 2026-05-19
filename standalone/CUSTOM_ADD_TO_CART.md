# Custom Add to Cart Integration

This guide explains how to use your own "Add to Cart" button instead of Customily's built-in one. This is useful when you need full control over the checkout flow or want to integrate the add-to-cart action into your platform's native UI.

## Overview

When using Customily's built-in "Add to Cart" button, the iframe automatically calls three operations and sends the result to the parent page via `postMessage`. If you want to use your own button and keep this functionality, you'll need to hide Customily's one and call these same operations yourself using JavaScript functions exposed by the Customily engraver window object.

The three operations are:

1. **Generate the production file request** — reserves a URL for the production-ready file
2. **Generate a preview thumbnail** — uploads a preview image and returns a CDN URL
3. **Create a cart record** — saves the personalization data on Customily's server so you can see the cart details on Customily's dashboard

## Step 1: Generate the Production File Request

Call `generatePFRPostOrder()` on the engraver to create the production file request:

```javascript
const exportedFiles = await window.engraver.generatePFRPostOrder('');
// Store the url(s) — you'll need them to call item/generate after checkout
const productionFileUrls = exportedFiles.map(f => f.url);
```

**Returns:** an array of production file URLs (one per template side):

```javascript
[
    { "url": "https://cdn.customily.com/ExportFile/..." },
    { "url": "https://cdn.customily.com/ExportFile/..." }  // if the template has multiple sides
]
```

> Note: The `url` is a placeholder — the actual file won't be available until you call the [item/generate endpoint](https://github.com/Customily/Documentation/blob/main/standalone/INTEGRATION_GUIDE.md#23-generating-the-production-file) after checkout.

## Step 2: Generate the Preview Thumbnail

Call `generatePreviewImage()` to upload a preview and get a CDN URL:

```javascript
const preview = await window.engraver.generatePreviewImage({
    shop: 'yourstandalonestore.com'
});
```

**Returns:**

```javascript
{
    "filename": "c611faa2-...",
    "previewUrl": "https://cdn.customily.com/shopify/assetFiles/previews/yourstandalonestore.com/c611faa2-....jpeg",
    "thumbnailUrl": "https://cdn.customily.com/..."
}
```

- **`previewUrl`** — a 1000x1000 high-quality image, ideal for sending to the shopper via email so they can see how their personalization looks.
- **`thumbnailUrl`** — a smaller image, more suitable for use as the cart thumbnail.

## Step 3: Create the Cart Record on Customily server

Post the personalization data to Customily's cart endpoint:

```javascript
const response = await fetch('https://sh.customily.com/api/standalone/cart?shop=yourstandalonestore.com', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
        productId: window.engraver.currentProduct.id,
        quantity: 1,
        sessionId: window.engraver.getSessionId(),
        previewUrl: preview.previewUrl,
        exportedFiles: exportedFiles,
        options: [
            { name: "Name", value: "John", type: "Text Input" },
            { name: "Date", value: "June 15, 2025", type: "Text Input" },
            { name: "Font", value: "Script", type: "Dropdown" },
            { name: "Photo", value: window.engraver.getElementsUrls('image', 1)[0], type: "Image Upload" }
        ]
    })
});

const cartItem = await response.json();
// cartItem.id is the personalizationGUID
```

The `options` array stores the shopper's selections so you can see what they entered on Customily's dashboard and at fulfillment time. Populate it with the options from your form — each entry should have a `name`, `value`, and `type`.

## Complete Example

```javascript
async function customilyAddToCart(shop, quantity, options) {
    // 1. Generate production file request
    const exportedFiles = await window.engraver.generatePFRPostOrder('');

    // 2. Generate preview thumbnail
    const preview = await window.engraver.generatePreviewImage({ shop });

    // 3. Wait for any pending file uploads to complete.
    // When a shopper uploads an image or vector, Customily immediately starts uploading
    // the file in the background. For large files, it's possible the shopper clicks
    // "Add to Cart" before the upload finishes. Calling waitFilesUpload() ensures all
    // pending uploads are complete before proceeding, so no files are missing from the order.
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
            options: options
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
    // Collect the shopper's selections from your form
    const options = [
        { name: "Name", value: document.getElementById('name-input').value, type: "Text Input" },
        { name: "Date", value: document.getElementById('date-input').value, type: "Text Input" },
        { name: "Font", value: document.getElementById('font-select').value, type: "Dropdown" },
        { name: "Photo", value: window.engraver.getElementsUrls('image', 1)[0], type: "Image Upload" }
    ];

    const result = await customilyAddToCart('yourstandalonestore.com', 1, options);
    console.log('Added to cart:', result.personalizationGUID);
    console.log('Preview:', result.previewUrl);

    // Add to your platform's cart here...
});
```

## JavaScript API Reference

The following `window.engraver` methods and properties are relevant to the add-to-cart flow:

| Method | Returns | Description |
| ------ | ------- | ----------- |
| `generatePFRPostOrder('')` | `Promise<[{ url }]>` | Generates the production file request (one entry per template side) |
| `generatePreviewImage(options)` | `Promise<{ previewUrl, thumbnailUrl, filename }>` | Uploads preview image, returns CDN URLs |
| `waitFilesUpload()` | `Promise<void>` | Waits for pending image/vector uploads to complete |
| `getElementsUrls(type, id)` | `string[]` | Gets Customily-hosted URLs for uploaded images (`'image'`) or vectors (`'vector'`) by element ID |
| `getSessionId()` | `string` | Returns the current engraver session ID (used internally by Customily for tracking) |
| `currentProduct.id` | `string` | The current template/product ID |
