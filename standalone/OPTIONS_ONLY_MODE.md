# Options-Only Mode Integration

This guide explains how to embed the Customily live preview canvas directly into your product page while using the iframe only for the option set form. This gives you full control over the preview layout and placement while Customily handles the personalization form.

## Overview

In the [standard iframe integration](INTEGRATION_GUIDE.md), the entire Customily experience (live preview + option set form) runs inside a single iframe. In options-only mode, you split these two parts:

- **The live preview canvas** runs on your product page, outside the iframe
- **The option set form** runs inside a lightweight iframe

The two communicate via `postMessage` — when the shopper types text, picks a font, or uploads an image in the iframe, the preview canvas on your page updates in real time.

This approach is ideal when you want to:
- Place the live preview anywhere on your page (e.g. as the main product image)
- Control the preview size and aspect ratio
- Avoid the preview being constrained inside an iframe

## How It Works

1. You add a `<div>` on your page where the live preview canvas will render
2. You load the `customilyPreviewHost.js` script — this loads the Customily engraver and creates the canvas
3. You add an iframe with the Customily personalization link in `mode=options-only`
4. The iframe sends engraver commands to the parent page via `postMessage`
5. The `canvasHost.js` script executes those commands on the local engraver, updating the preview

## Implementation

### Step 1: Add the Canvas Container and Options Iframe

```html
<!-- Live preview canvas renders here -->
<div id="customily-canvas" style="width: 500px; height: 500px;"></div>

<!-- Option set form (no live preview, just the form) -->
<iframe
    id="customily-options"
    src="https://preview-2.customily.com/productViewer?template={TEMPLATE_ID}&set={OPTION_SET_ID}&shop={STORE_URL}&mode=options-only"
    style="width: 400px; height: 600px; border: none;">
</iframe>
```

Note the `&mode=options-only` appended at the end of the iframe `src` URL — this is what tells Customily to render only the option set form without the live preview canvas. Without it, the iframe would load the full experience (canvas + form) as usual.

### Step 2: Configure and Load the Canvas Host

```html
<script>
    // Configure the canvas host before loading the script
    window.customilyCanvasHost = {
        canvasContainerId: 'customily-canvas',    // ID of the div where the canvas will render
        iframeSelector: '#customily-options',      // CSS selector for the options iframe
        onReady: () => {
            console.log('Customily canvas is ready');
        }
    };
</script>

<!-- Load the canvas host script -->
<script src="https://preview-2.customily.com/static/js/customilyPreviewHost.js"></script>
```

The `customilyPreviewHost.js` script:
1. Loads the Customily engraver (`customily.js`)
2. Creates a canvas element inside your container div
3. Listens for `postMessage` commands from the options iframe
4. Executes engraver operations (set text, load image, etc.) on the local canvas

### Configuration Options

| Option | Type | Required | Description |
| ------ | ---- | -------- | ----------- |
| `canvasContainerId` | string | Yes | ID of the HTML element where the canvas will be created |
| `iframeSelector` | string | Yes | CSS selector for the options-only iframe |
| `engraverUrl` | string | No | Override the URL for `customily.js` (defaults to the Customily CDN). Useful if you want to self-host `customily.js` on your own server or CDN to reduce loading times. |
| `onReady` | function | No | Callback fired when the engraver is initialized and ready |

## Complete Example

This example places the canvas and options iframe side by side (see [`testOptionsOnly.html`](../product-preview/testOptionsOnly.html) for a working version you can open locally):

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Options-Only Mode Test</title>
    <style>
        body { font-family: sans-serif; margin: 20px; }
        .container {
            display: flex;
            gap: 20px;
            align-items: flex-start;
        }
        .panel { display: flex; flex-direction: column; }
        #customily-canvas {
            width: 500px;
            height: 500px;
            border: 1px solid #ccc;
        }
        #customily-options {
            width: 500px;
            height: 600px;
            border: 1px solid #ccc;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="panel">
            <h2>Canvas (parent page)</h2>
            <div id="customily-canvas"></div>
        </div>
        <div class="panel">
            <h2>Options (iframe)</h2>
            <iframe
                id="customily-options"
                src="https://preview-2.customily.com/productViewer?template=0313370a-e3d1-4b88-8640-f1027a78235d&set=53b7ddcc-880d-4303-a495-0338e1388ca2&shop=standalone.customily.com&mode=options-only"
            ></iframe>
        </div>
    </div>

    <script>
        window.customilyCanvasHost = {
            canvasContainerId: 'customily-canvas',
            iframeSelector: '#customily-options',
            onReady: function() {
                console.log('Canvas host is ready!');
            }
        };
    </script>
    <script src="https://preview-2.customily.com/static/js/customilyPreviewHost.js"></script>

    <script>
        // Listen for personalization data (same as the standard iframe integration)
        window.addEventListener('message', (event) => {
            const data = event.data;
            if (data?.action !== 'add-to-cart') return;

            console.log('Personalization complete:', data);
            // Add to your platform's cart here...
        });
    </script>
</body>
</html>
```

## Add to Cart

The add-to-cart flow works the same as in the standard iframe integration — the iframe sends a `postMessage` with the personalization data when the shopper clicks "Add to Cart". See [Capturing the Personalization Data](INTEGRATION_GUIDE.md#22-capturing-the-personalization-data) for details.

If you want to use your own "Add to Cart" button, see [Custom Add to Cart Integration](CUSTOM_ADD_TO_CART.md). Note that in options-only mode, the engraver methods (`generatePFRPostOrder`, `generatePreviewImage`, etc.) are available on `window.engraver` on the parent page (not inside the iframe), since the canvas host loads the engraver locally.
