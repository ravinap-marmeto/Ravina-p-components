# Free Shipping Bar Integration

## Overview

Integrate a modular free shipping bar into your Shopify theme. This bar is built using Tailwind CSS and requires no JavaScript.

## Steps

1. **Add the Snippet:**

    Create a snippet named `free-shipping-bar.liquid`:


3. **Render the Snippet:**

    **In the Cart Drawer:**

    Add to `cart-drawer.liquid`:

    ```liquid
    {% render 'free-shipping-bar' %}
    ```


## Customization

Modify the `settings_schema.json` for text and threshold changes:
