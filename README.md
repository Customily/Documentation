# Customily Developer Documentation

Customily is a product personalization platform that lets shoppers customize products in real time — text, fonts, colors, images, vectors, and more — with a live preview rendered on an HTML canvas. It integrates with e-commerce platforms via an iframe-based modal or by embedding the preview canvas directly on the product page.

This repository contains integration guides and API references for developers building on top of Customily.

## Authentication

- [API Authentication](https://app.customily.com/swagger/index.html?url=/swagger/v1/swagger.json#/Authentication/GetToken) — How to obtain and use JWT tokens for Customily API endpoints

## Design Studio API

- [Swagger Documentation](https://app.customily.com/swagger/index.html?url=/swagger/v1/swagger.json) — Interactive API reference for the Design Studio backend

## Standalone Integration

Guides for integrating Customily into any e-commerce platform using an iframe-based approach. The iframe hosts the full personalization experience (live preview + option set form) or just the form in options-only mode.

- [Integration Guide](standalone/README.md) — End-to-end guide for embedding Customily in a modal iframe, from the "Customize" button to the cart
- [Custom Add to Cart](standalone/CUSTOM_ADD_TO_CART.md) — How to replace Customily's built-in "Add to Cart" button with your own, including production file generation and cart record creation
- [Retrieving Personalization Details & Production Files](standalone/FETCH_PERSONALIZATION.md) — How to fetch personalization data and production file URLs using the `personalizationGUID` or order ID
- [Options-Only Mode](standalone/OPTIONS_ONLY_MODE.md) — How to embed the live preview canvas directly on your page while running the option set form in a lightweight iframe

## customily.js

`customily.js` is the browser library that powers the live preview canvas. It exposes a global `engraver` object with methods to set text, fonts, colors, images, vectors, and more. Use these docs when building a fully custom integration that drives the preview programmatically.

- [Engraver API Reference](customily/README.md) — Full API reference for `customily.js`: initialization, text, images, vectors, export, and all available options
- [Position Parameter](customily/position-parameter.md) — How the position parameter works across methods like `setFont`, `setFontColor`, `setPresetImage`, and others
