# Webflow Component Structure

## Overview
A Webflow component is exported as a JSON structure containing all necessary information for rendering, including nodes, styles, assets, and interactions. This document outlines the structure and relationships between these elements.

## Main Structure
```json
{
    "type": "@webflow/XscpData",
    "payload": {
        "nodes": [...],     // Component structure and content
        "styles": [...],    // Style definitions
        "assets": [...],    // Media assets
        "ix1": [],         // Interactions v1
        "ix2": {           // Interactions v2
            "interactions": [],
            "events": [],
            "actionLists": []
        }
    },
    "meta": {...}        // Metadata about the component
}
```

## Node Types and Mappings

### Basic Node Structure
All nodes share these common properties:
```json
{
    "_id": "unique-identifier",
    "type": "element-type",    
    "tag": "html-tag",         
    "classes": ["class-ids"],  
    "children": ["child-ids"], 
    "data": {
        "tag": "html-tag",
        "attr": {},          // HTML attributes
        "xattr": [],         // Extended attributes
        "search": {},
        "visibility": {}
    }
}
```

### 1. Block Types
- **Section**: Root container element
  ```json
  {
      "type": "Section",
      "tag": "section"
  }
  ```

- **Block**: Generic container
  ```json
  {
      "type": "Block",
      "tag": "div"
  }
  ```

### 2. Text Content
- **Text Node**: Pure text content
  ```json
  {
      "type": "Text",
      "text": true,
      "v": "Actual text content"
  }
  ```

- **Heading**: Section headers
  ```json
  {
      "type": "Heading",
      "tag": "h1" // or h2, h3, etc.
  }
  ```

- **RichText**: Formatted text content
  ```json
  {
      "type": "RichText",
      "data": {
          "html": "<p>Formatted content</p>"
      }
  }
  ```

### 3. Media Elements
- **Image**: Image elements
  ```json
  {
      "type": "Image",
      "tag": "img",
      "data": {
          "img": {
              "_id": "asset-id"
          },
          "attr": {
              "src": "image-url",
              "alt": "description",
              "width": "500",
              "height": "500"
          }
      }
  }
  ```

### 4. Embedded Content
- **HtmlEmbed**: Raw HTML content (often SVGs)
  ```json
  {
      "type": "HtmlEmbed",
      "v": "<svg>...</svg>"
  }
  ```

- **Embed/EmbedBlock**: External embeds
  ```json
  {
      "type": "Embed",
      "data": {
          "embed": {
              "meta": {
                  "html": "embedded content"
              }
          }
      }
  }
  ```

## Style Definitions

### Base Style
```json
{
    "_id": "style-id",
    "type": "class",
    "name": "component-class",
    "namespace": "",
    "comb": "",
    "styleLess": "display: grid; grid-template-columns: 1fr 1fr;",
    "variants": {
        "medium": {
            "styleLess": "grid-template-columns: 1fr;"
        }
    },
    "children": ["child-style-ids"]
}
```

### Style Properties
1. **Base Properties**
   - `_id`: Unique identifier referenced by nodes
   - `type`: Usually "class" for CSS classes
   - `name`: The actual CSS class name
   - `styleLess`: Base CSS properties

2. **Responsive Variants**
   - `variants`: Contains responsive breakpoint styles
     - `medium`: Tablet (max-width: 991px)
     - `small`: Mobile Landscape (max-width: 767px)
     - `tiny`: Mobile Portrait (max-width: 479px)

3. **Relationships**
   - `comb`: Defines style combinations
     - `""`: Standalone class
     - `"&"`: Modifier class (e.g., `cc-1x1`)
     - Other values: Parent-child relationships
   - `children`: IDs of related styles

### Style Organization
Styles are processed and organized in the following hierarchy:
1. Global CSS Variables and Settings
2. Base Reset Styles
3. Typography Styles (h- prefix)
4. Component Styles
5. Utility Classes (u- prefix)
6. Media Queries

### CSS Variables
```css
:root {
    /* Typography */
    --font-size-base: 16px;
    --line-height-base: 1.5;
    
    /* Colors */
    --color-text: #333333;
    --color-background: #ffffff;
    
    /* Spacing */
    --spacing-unit: 1rem;
    --spacing-xs: calc(var(--spacing-unit) * 0.25);
    --spacing-sm: calc(var(--spacing-unit) * 0.5);
    --spacing-md: var(--spacing-unit);
    --spacing-lg: calc(var(--spacing-unit) * 2);
    --spacing-xl: calc(var(--spacing-unit) * 4);
}
```

## Component Hierarchy
Components follow a nested structure:
1. Root Node (can be either):
   - Section (preferred)
   - Block (top-level container)
2. Container (Block)
   - Content Blocks
     - Elements (Heading, Image, HtmlEmbed)
       - Text Nodes

Note: While Webflow components typically start with a Section node, some components might use a Block as their root node. The converter handles both cases.

## Class Mapping
Classes in nodes reference style definitions:
1. Node contains class IDs: `"classes": ["80633ea2-356e-963e-4220-c2c8019a1c7f"]`
2. Style definition provides the human-readable name: `{"_id": "80633ea2-356e-963e-4220-c2c8019a1c7f", "name": "section"}`

## Processing Guidelines

### 1. Node Processing Order
1. Check for text nodes first (`node.text === true`)
2. Process embedded content (`HtmlEmbed`, `RichText`, `Embed`)
3. Handle special elements (images, self-closing tags)
4. Process regular elements with their attributes and children

### 2. Attribute Handling
1. Convert class IDs to class names
2. Process standard HTML attributes from `data.attr`
3. Handle special cases:
   - Image sources and dimensions
   - Embedded content
   - Extended attributes (xattr)

### 3. Content Processing
1. Text nodes: Use `node.v` directly
2. HTML embeds: Insert raw content from `node.v`
3. Rich text: Use processed HTML from `data.html`
4. Images: Resolve asset references and URLs

### 4. Style Processing
- Map class IDs to readable names
- Extract and organize styles by category
- Process responsive variants
- Generate organized CSS output

### 5. Asset Handling
- Map asset IDs to CDN URLs
- Process image dimensions and metadata
- Handle responsive images

### 5. Grid and Alignment
Nodes can have grid-specific settings and inline styles:
```json
{
    "data": {
        "grid": {
            "align": "center",     // Controls align-self
            "justify": "start",    // Controls justify-self
            "id": "w-node-..."     // Grid item identifier
        },
        "style": {
            "base": {
                "main": {
                    "noPseudo": {
                        "alignSelf": "center",
                        "marginTop": "1rem"
                    }
                }
            }
        }
    }
}
```

### Style Processing Guidelines

1. **Grid Alignment**
   - Extract `align` and `justify` from `data.grid`
   - Convert to `align-self` and `justify-self` CSS properties
   - Apply as inline styles on the element

2. **Inline Styles**
   - Check `data.style.base.main.noPseudo` for inline styles
   - Convert camelCase keys to kebab-case (e.g., `marginTop` → `margin-top`)
   - Combine with grid alignment styles if present

3. **Node IDs**
   - Preserve Webflow node IDs starting with `w-node-`
   - These IDs are essential for grid positioning and layout

Example output:
```html
<div id="w-node-123" class="grid-item" style="align-self: center; margin-top: 1rem">
    <!-- content -->
</div>
```

## Output Structure
```
output/
  └── ComponentName/
      ├── index.html    # Component HTML
      └── styles.css    # Organized styles
```

## Best Practices

### 1. Component Organization
- Use semantic HTML structure
- Maintain logical class naming
- Follow responsive design principles
- Implement proper accessibility attributes

### 2. Style Management
- Use CSS variables for consistency
- Organize styles logically
- Group related styles together
- Maintain clean media queries

### 3. Performance
- Optimize asset loading
- Minimize style duplication
- Use efficient selectors
- Follow mobile-first approach
- Build node and asset maps for quick lookups
- Create class name mapping
- Process nodes recursively
- Preserve whitespace and formatting
- Handle all node types appropriately
- Validate output HTML structure 