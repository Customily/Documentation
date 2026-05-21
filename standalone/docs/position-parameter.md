# Understanding the Position Parameter

Several Engraver methods accept a **position** parameter to select from a list of options configured on an element. This guide explains how it works.

## What is the position?

The position is a **1-based index** that corresponds to the order in which options were added to an element in the [Design Studio](https://help.customily.com/articles/2111674173-text-box?lang=en#font-option). The first option added is position `1`, the second is position `2`, and so on.

If options are reordered by dragging them in the Design Studio, the position reflects the new order.

## Where do options come from?

Options can come from two sources:

### 1. Element options

These are fonts, colors, or images added directly to the element in the Design Studio. For example, a text element might have three font options configured on it:

| Position | Font       |
|----------|------------|
| 1        | Lato       |
| 2        | Roboto     |
| 3        | Montserrat |

```javascript
engraver.setFont(1, 2); // Sets Roboto on text element 1
```

### 2. Libraries

Libraries are shared collections of fonts, colors, or images configured at the template level. When an element is linked to a library instead of having its own options, the position indexes into the library's options.

For example, a color library shared across multiple elements:

| Position | Color  | Hex       |
|----------|--------|-----------|
| 1        | Red    | `#ff0000` |
| 2        | Blue   | `#0000ff` |
| 3        | Green  | `#00ff00` |

```javascript
engraver.setFontColor(1, 3);   // Sets Green on text element 1
engraver.setImageColor(5, 3);  // Sets Green on image element 5 (same library)
```

The position parameter works the same way regardless of whether the options are stored on the element or in a library.

## Methods that use the position parameter

| Method                    | What the position selects          |
|---------------------------|------------------------------------|
| `setFont`                 | Font from font options/library     |
| `setFontColor`            | Color from color options/library   |
| `setPresetImage`          | Image from image options/library   |
| `setImageColor`           | Color for single-color images      |
| `setPresetVectorColor`    | Color for vector placeholders      |

## Examples

### Selecting a font

```javascript
// Text element 1 has fonts: [Lato, Roboto, Montserrat]
engraver.setFont(1, 1); // Lato
engraver.setFont(1, 2); // Roboto
engraver.setFont(1, 3); // Montserrat
```

### Selecting a font color

```javascript
// Text element 1 has colors: [Black, White, Red]
engraver.setFontColor(1, 1); // Black
engraver.setFontColor(1, 3); // Red
```

### Selecting a preset image

```javascript
// Image element 3 has preset images: [Cat, Dog, Bird]
engraver.setPresetImage(3, 1); // Cat
engraver.setPresetImage(3, 2); // Dog
```

### Selecting an image color

```javascript
// Image element 5 has colors: [Gold, Silver, Bronze]
engraver.setImageColor(5, 1); // Gold
```

### Selecting a vector color

```javascript
// Vector element 2 has colors: [Red, Blue]
engraver.setPresetVectorColor(2, 2); // Blue
```
