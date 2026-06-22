# Tree Element (ITreeElement)

> A node of a tree — extends an array element with expand state and methods to expand a node and select a node by path.

## Overview

A tree node extends [`IArrayElement`](https://docs-llm.a2v10.com/client/array-element.md) with members for hierarchy. It knows whether it is expanded, it can expand itself (loading children for a dynamic tree), and it can select a descendant identified by a path, expanding the intermediate nodes along the way.

These members back the navigation of both static and dynamic [tree models](https://docs-llm.a2v10.com/sql/tree.md).

## Syntax

```ts
interface ITreeElement extends IArrayElement {
  // properties
  readonly $expanded: boolean;
  // methods
  $expand<T>(this: T): Promise<IElementArray<T>>;
  $selectPath<T>(this: T, path: Array<any>, predicate: (item: T, val: any) => boolean): Promise<T>;
}
```

### Properties

| Member | Type | Description |
|--------|------|-------------|
| `$expanded` | boolean | Whether the node is expanded in the tree (read-only) |

### Methods

| Member | Returns | Description |
|--------|---------|-------------|
| `$expand()` | `Promise` | Expands a collapsed node; works for static and dynamic trees. Resolves to the array of child elements |
| `$selectPath(path, predicate)` | `Promise` | Selects a node by `path` from the root, expanding intermediate nodes; for dynamic trees. Resolves to the selected node |

### $selectPath arguments

| Argument | Description |
|----------|-------------|
| `path` | Array describing the path from the root; elements may be of any type |
| `predicate(item, val)` | Returns `true` when tree node `item` matches the path value `val` |

## Example

```js
const template = {
  commands: {
    openNode(node) { return node.$expand(); },
    selectById(tree, id) {
      return tree.$selectPath([id], (item, val) => item.Id === val);
    }
  }
};

module.exports = template;
```

## Notes

- `$expand()` resolves to the child array, so you can chain work after the children load.
- `$selectPath()` is intended for dynamic trees, where intermediate nodes may not yet be loaded — it expands them automatically as it walks the path.
- Tree selection differs from a flat array: pass the tree root to [`$select()`](https://docs-llm.a2v10.com/client/array-element.md) when selecting a node directly.
- The hierarchy itself is shaped in SQL — see [tree models](https://docs-llm.a2v10.com/sql/tree.md).
- A tree is rendered in the UI by a [DataGrid](https://docs-llm.a2v10.com/xaml/controls/datagrid.md), which calls `$expand` as the user opens nodes.
