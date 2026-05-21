# Customily Engraver - JavaScript Library

`customily.js` is a browser library that renders real-time product customization previews on an HTML canvas. It's loaded on customers' eCommerce stores and exposes a global `engraver` object.

## Quick Start

```html
<script src="https://app.customily.com/customily.js"></script>

<div id="fabric-container" style="width: 50%;">
  <canvas id="preview-canvas"></canvas>
</div>

<script>
  engraver.init("preview-canvas", {});
  engraver.setProduct("0313370a-e3d1-4b88-8640-f1027a78235d").then(function () {
    engraver.setText(1, "Hello World");
  });
</script>
```

## Setup

### 1. Include the script

```html
<script src="https://app.customily.com/customily.js"></script>
```

### 2. Add a canvas inside a sized container

The canvas takes its dimensions from its parent container. Wrap it in a div with a defined width:

```html
<div id="fabric-container" style="width: 50%;">
  <canvas id="preview-canvas"></canvas>
</div>
```

### 3. Initialize

```javascript
var options = {};
engraver.init("preview-canvas", options);
```

Returns `0` on success, `-1` if the canvas element is not found.

### 4. Load a template

```javascript
engraver.setProduct("TEMPLATE_ID").then(function () {
  // Template loaded — now you can set text, images, etc.
});
```

| Parameter     | Type     | Description                                        |
|---------------|----------|----------------------------------------------------|
| `templateId`  | `String` | Customily Design Studio template ID                |
| `initialData` | `Object` | Optional initial data to apply to the template     |

Returns a `Promise` that resolves when the template is loaded.

## Initialization Options

Pass these in the options object to `engraver.init(canvasId, options)`:

| Option                      | Type      | Default   | Description                                                        |
|-----------------------------|-----------|-----------|--------------------------------------------------------------------|
| `hoverZoom`                 | `Boolean` | `false`   | Enable zoom effect on mouse hover                                  |
| `sideZoom`                  | `Boolean` | `false`   | Enable side panel zoom display                                     |
| `touchZoom`                 | `Boolean` | `false`   | Enable pinch-to-zoom on touch devices                              |
| `hoverZoomLevel`            | `Number`  | `1.5`     | Magnification level for hover zoom                                 |
| `hoverZoomSide`             | `String`  | `'right'` | Side where the hover zoom panel appears (`'left'` or `'right'`)    |
| `imagesCover`               | `Boolean` | `false`   | Images cover their container (like CSS `background-size: cover`)   |
| `allowMobileTouchEvents`    | `Boolean` | `false`   | Allow touch events on mobile for object manipulation               |
| `preventEdgeResizing`       | `Boolean` | `false`   | Prevent resizing objects from their edges                          |
| `useGroupVisibility`        | `Boolean` | `false`   | Use group-based visibility for layers/objects                      |
| `showMobileControls`        | `Boolean` | `false`   | Display mobile-specific control buttons                            |
| `disableZoomOnImageUpload`  | `Boolean` | `false`   | Prevent automatic zoom adjustment when uploading images            |
| `zoomSelection`             | `Boolean` | `false`   | Zoom to fit selected object                                        |
| `imageOverlaySelection`     | `Boolean` | `false`   | Show overlay on selected images                                    |
| `canvasImageLoading`        | `Boolean` | `false`   | Show loading indicator while canvas images load                    |

## API Reference

Most methods require an element `id` parameter. This is the Design Studio ID assigned to each element in your template. See [What's the element's ID?](https://help.customily.com/articles/9397457321-what-s-the-elements-id) to learn how to find it.

### Understanding the position parameter

Several methods (`setFont`, `setFontColor`, `setPresetImage`, `setImageColor`, `setPresetVectorColor`) take a **position** parameter. This is a 1-based index into the options configured on the element (or its linked library) in the [Design Studio](https://help.customily.com/articles/2111674173-text-box?lang=en#font-option).

```javascript
engraver.setFont(1, 2);       // Sets the 2nd font option on text element 1
engraver.setImageColor(3, 1); // Sets the 1st color option on image element 3
```

For a full explanation of how positions work with element options and libraries, see [Position Parameter](docs/position-parameter.md).

### Text

#### `setText(id, text)` → `Number`

Sets the text value for a text element.

```javascript
engraver.setText(1, "John");
```

| Parameter | Type     | Description                     |
|-----------|----------|---------------------------------|
| `id`      | `Number` | [Design Studio ID](https://help.customily.com/articles/9397457321-what-s-the-elements-id) of the text element |
| `text`    | `String` | Text to set                     |

Returns `0` on success, `-1` on error.

#### `setFont(id, fontPosition)` → `Promise<Number>`

Sets the font for all text elements matching the given ID.

```javascript
await engraver.setFont(1, 2); // Apply the 2nd font option
```

| Parameter      | Type              | Description                                    |
|----------------|-------------------|------------------------------------------------|
| `id`           | `String\|Number`  | [Design Studio ID](https://help.customily.com/articles/9397457321-what-s-the-elements-id) of the text element(s)        |
| `fontPosition` | `Number`          | [Position](docs/position-parameter.md) of the font from the element's font options |

Returns `0` on success, `-1` if no matching elements found.

#### `setFontColor(id, fontColorPosition)` → `Promise<Number>`

Sets the font color for all text elements matching the given ID.

```javascript
await engraver.setFontColor(1, 0); // Apply the 1st color option
```

| Parameter           | Type              | Description                                      |
|---------------------|-------------------|--------------------------------------------------|
| `id`                | `String\|Number`  | [Design Studio ID](https://help.customily.com/articles/9397457321-what-s-the-elements-id) of the text element(s)          |
| `fontColorPosition` | `Number`          | [Position](docs/position-parameter.md) of the color from the element's color options |

Returns `0` on success, `-1` if no matching elements found.

### Images

#### `setImage(id, img, threshold)` → `Promise`

Sets a user-uploaded image on an image placeholder.

```javascript
// From a file input
var reader = new FileReader();
reader.addEventListener("load", function () {
  engraver.setImage(1, reader.result);
});
reader.readAsDataURL(file);

// From a URL
engraver.setImage(1, "https://example.com/photo.jpg");
```

| Parameter   | Type              | Description                                          |
|-------------|-------------------|------------------------------------------------------|
| `id`        | `Number`          | [Design Studio ID](https://help.customily.com/articles/9397457321-what-s-the-elements-id) of the image placeholder            |
| `img`       | `String\|Image`   | Base64 string, URL, or Image object                  |
| `threshold` | `Number`          | Image tracing threshold (0-255, defaults to 150)     |

Returns a `Promise` that resolves when the image is loaded on the preview.

#### `setPresetImage(placeholderId, imagePosition)` → `Promise<void>`

Renders a preset image (configured in the Design Studio) on a dynamic image placeholder.

```javascript
await engraver.setPresetImage(3, 2); // Apply the 2nd preset image
```

| Parameter       | Type              | Description                                           |
|-----------------|-------------------|-------------------------------------------------------|
| `placeholderId` | `String\|Number`  | [Design Studio ID](https://help.customily.com/articles/9397457321-what-s-the-elements-id) of the image placeholder             |
| `imagePosition` | `Number`          | [Position](docs/position-parameter.md) of the image from the placeholder's presets     |

#### `setImageColor(imageId, colorPosition)`

Sets the color for a single-color image placeholder.

```javascript
engraver.setImageColor(1, 2);
```

#### `setImageColorHex(imageId, hexColor)`

Sets the color for a single-color image placeholder using a hex value.

```javascript
engraver.setImageColorHex(1, "#ff0000");
```

### Vectors

#### `setVector(vectorId, vector)`

Loads a vector into a dynamic vector placeholder.

```javascript
engraver.setVector(1, "<svg>...</svg>");       // SVG string
engraver.setVector(1, "https://example.com/file.svg"); // URL
engraver.setVector(1, fileObject);              // File object (EPS, PDF, SVG)
```

| Parameter  | Type     | Description                                                    |
|------------|----------|----------------------------------------------------------------|
| `vectorId` | `Number` | [Design Studio ID](https://help.customily.com/articles/9397457321-what-s-the-elements-id) of the vector placeholder                     |
| `vector`   | `String` | SVG string, URL to SVG file, or File object (EPS, PDF, SVG)   |

#### `setPresetVectorColor(vectorId, colorPosition)` → `Promise<void>`

Sets the color for a dynamic vector placeholder using a preset color index.

```javascript
await engraver.setPresetVectorColor(1, 3);
```

#### `setPresetVectorColorHex(vectorId, hexColor)`

Sets the color for a dynamic vector placeholder using a hex value.

```javascript
engraver.setPresetVectorColorHex(1, "#00ff00");
```

### Special Elements

#### `setWordCloud(id, text, shapePosition)` → `Promise<void>`

Generates a word cloud image and sets it on an image placeholder.

```javascript
await engraver.setWordCloud(1, "love family friends happiness");
```

| Parameter       | Type     | Description                                    |
|-----------------|----------|------------------------------------------------|
| `id`            | `Number` | [Design Studio ID](https://help.customily.com/articles/9397457321-what-s-the-elements-id) of the image placeholder      |
| `text`          | `String` | Words to use in the word cloud                 |
| `shapePosition` | `Number` | Optional. [Position](docs/position-parameter.md) of the shape to use            |

#### `setWordSearchPuzzle(id, words)` → `Promise<{base64, seed}>`

Generates a word search puzzle and sets it on a placeholder.

```javascript
var result = await engraver.setWordSearchPuzzle(1, "cat dog bird fish");
// result.base64 - the puzzle image
// result.seed   - the puzzle seed for reproducibility
```

#### `setCalendar(id, date)` → `Promise<{calendar}>`

Sets a calendar for a specific placeholder.

```javascript
var result = await engraver.setCalendar(1, new Date(2025, 0, 1));
```

### Export

#### `exportFileAuth(apiKey)` → `Promise<String>`

Exports the current personalized preview to production file(s).

```javascript
var fileUrls = await engraver.exportFileAuth("your-api-key");
```

Returns a `Promise` that resolves with the URL(s) of the generated file(s), separated by newlines if multiple.

### Utilities

#### `getActionableElements()`

Returns the currently selected image placeholder. If none is selected, selects and returns the first one found.

#### `setImagePlaceHolderScale()`

Programmatic control of image placeholder scale.

#### `setImagePlaceHolderRotation()`

Programmatic control of image placeholder rotation.

## Full Example

```html
<html lang="en">
<head>
  <meta charset="utf-8">
</head>
<body>
  <div>
    <input id="textfield1" oninput="processText(1, value)" type="text">
    <input id="imageInput1" type="file" onchange="previewFile(1, id)">
    <select onchange="changePreset(value)">
      <option value="1">Option A</option>
      <option value="2">Option B</option>
    </select>
  </div>

  <script src="https://app.customily.com/customily.js"></script>
  <div id="fabric-container" style="width: 50%;">
    <canvas id="preview-canvas"></canvas>
  </div>

  <script>
    engraver.init("preview-canvas", {});
    engraver.setProduct("0313370a-e3d1-4b88-8640-f1027a78235d").then(function () {
      engraver.setText(1, "Default text");
    });

    function processText(id, value) {
      engraver.setText(id, value);
    }

    function previewFile(placeholderId, fileInputId) {
      var file = document.getElementById(fileInputId).files[0];
      var reader = new FileReader();
      reader.addEventListener("load", function () {
        engraver.setImage(placeholderId, reader.result);
      });
      if (file) {
        reader.readAsDataURL(file);
      }
    }

    function changePreset(value) {
      engraver.setPresetImage(3, parseInt(value));
    }
  </script>
</body>
</html>
```

## Development

```bash
npm run dev              # Webpack dev server
npm test                 # Jest unit tests
npm run build:release    # Production build → customily.js
npm run build:beta       # Beta variant
npm run docs             # Generate JSDoc documentation
npm run docs:serve       # Serve JSDoc at http://localhost:3000
```
