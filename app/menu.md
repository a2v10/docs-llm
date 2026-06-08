# menu.json

> The application-level navigation file — declares the main menu tree, its titles, icons, and the endpoint URLs each item points to.

## Overview

There is exactly one `menu.json` per application, and it lives in the application root. It describes the whole navigation structure of the application. This file is app-based, not endpoint-based: unlike `model.json`, which is local to a single endpoint, `menu.json` is global — it defines the top-level menu, every submenu, and the links that resolve to endpoint URLs.

The file has two parts. The top-level properties (`appTitle`, `menu`) configure the application shell, and the `menu` array holds the navigation tree. Each node in the tree is an object that either points somewhere (`url`) or groups other nodes (`items`), and the same object shape is reused at every level of nesting.

A typical tree is three levels deep. The first level are the primary sections shown in the navigation bar, usually carrying an `icon`. The second level groups links into named blocks (for example `@[Documents]`, `@[Catalogs]`). The third level are the leaf links, each with a `url` that maps to an endpoint such as `/catalog/agent`.

Titles use the localization marker `@[Key]`. At render time the platform resolves `@[Sales]` against the application resource strings for the current language; a plain string like `"Розробка"` is shown verbatim.

## Syntax

```json
{
  "$schema": "@schemas/menu-json-schema.json#",
  "appTitle": "Application Title Here",
  "menu": [
    {
      "title": "@[Section]",
      "icon": "cart",
      "items": [
        {
          "title": "@[Group]",
          "items": [
            { "title": "@[Item]", "url": "/catalog/item", "create": true }
          ]
        }
      ]
    }
  ]
}
```

Root-level properties:

| Property | Type | Description |
|----------|------|-------------|
| `$schema` | string | Schema reference for editor validation; always `"@schemas/menu-json-schema.json#"` |
| `appTitle` | string | Optional. Application title shown in the shell header |
| `menu` | array | Required. The navigation tree — an array of menu nodes |

Menu node properties (the same shape at every nesting level):

| Property | Type | Description |
|----------|------|-------------|
| `title` | string | Display label. `@[Key]` is resolved from resource strings; a plain string is shown as-is |
| `icon` | string | Icon name from the platform's fixed icon set (mostly used on first-level nodes), e.g. `cart`, `home`, `gear-outline`, `database`. The set is a closed enum — unknown names are rejected by the schema |
| `url` | string | Target endpoint URL, e.g. `/document/waybillin`. A node with a `url` is a navigable leaf |
| `items` | array | Child nodes. A node with `items` is a group; combine with `url` only when grouping |
| `create` | boolean | Adds a quick "create new" (`+`) action next to the item — for catalogs that support insert |
| `id` | string | Explicit identifier for the node, used to reference it from elsewhere |
| `category` | string | Groups leaf items under a named category heading within a submenu |
| `grow` | boolean | Spacer node with no content; pushes the following top-level items to the far end of the bar |
| `underline` | boolean | Draws a separator under the node, visually splitting groups in the menu |

## Example

A navigation tree with a sales section, a purchase section, a spacer, and a settings section pinned to the end:

```json
{
  "$schema": "@schemas/menu-json-schema.json#",
  "appTitle": "A2v10 Sample",
  "menu": [
    {
      "title": "@[Sales]",
      "icon": "cart",
      "items": [
        {
          "title": "@[Documents]",
          "items": [
            { "title": "@[WaybillIn]",  "url": "/document/waybillin" },
            { "title": "@[WaybillOut]", "url": "/document/waybillout" },
            { "title": "@[Stock]",      "url": "/journal/stock" }
          ]
        },
        {
          "title": "@[Catalogs]",
          "items": [
            { "title": "@[Agents]", "url": "/catalog/agent", "create": true },
            { "title": "@[Stores]", "url": "/catalog/store", "create": true },
            {
              "title": "@[OtherCatalogs]",
              "id": "sales",
              "items": [
                { "title": "@[Units]", "category": "@[General]", "url": "/catalog/unit" },
                { "title": "@[Stores]", "url": "/catalog/store" }
              ]
            }
          ]
        }
      ]
    },
    {
      "title": "@[Purchase]",
      "icon": "cart",
      "underline": true,
      "items": [
        { "title": "@[Documents]" },
        {
          "title": "@[Journals]",
          "items": [
            { "title": "@[Stock]", "url": "/journal/stock" }
          ]
        }
      ]
    },
    { "grow": true },
    {
      "title": "@[Settings]",
      "icon": "gear-outline",
      "items": [
        {
          "title": "BPMN",
          "items": [
            { "title": "Catalog",   "url": "/$workflow/catalog" },
            { "title": "Instances", "url": "/$workflow/instance" }
          ]
        }
      ]
    }
  ]
}
```

## Notes

- Every node must carry either a `title` or `grow` — those are the only two ways to satisfy the schema. A `grow` spacer needs nothing else; every other node needs a `title`.
- The schema is strict (`additionalProperties: false`): only the documented keys are accepted at both the root and node level. A typo'd property name fails validation rather than being ignored.
- A node is a leaf when it has a `url` and a group when it has `items`. A group without a `url` is not navigable on its own — clicking it expands the children.
- A node may carry `title` with neither `url` nor `items` (for example the lone `@[Documents]` entry above). It renders as a placeholder/disabled label until links are added.
- `{ "grow": true }` is a layout-only node — it has no `title`, `icon`, or `items`. Use it to separate trailing sections (such as Settings) from the main navigation.
- Localization: any `title` may use `@[Key]`. Keys that are missing from the resource file fall through to the raw key text, which makes typos easy to spot.
- `create: true` only makes sense on a leaf whose endpoint exposes an insert/edit action; on a group it has no effect.
- URLs beginning with `$` (for example `/$workflow/catalog`, `/$meta/config`) target built-in platform endpoints rather than application endpoints.
