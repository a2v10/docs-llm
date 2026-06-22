# Array (IElementArray)

> Every model array — extends a JavaScript array with selection state, cross keys, lazy loading, and methods to insert, remove, find, and sum elements.

## Overview

Every array in the client model extends the native JavaScript array with service members. Beyond the usual array behavior it tracks selection and checked state, exposes the identifiers and names of its elements as strings, supports lazy (deferred) loading, and offers methods for inserting, removing, copying, searching, and summing elements.

Manipulating an array through these methods is what raises the array [events](https://docs-llm.a2v10.com/template/events.md) (`add`, `remove`, and so on) — mutating the underlying JavaScript array directly does not.

## Syntax

```ts
declare const enum InsertTo {
  start = 'start',
  end   = 'end',
  above = 'above',
  below = 'below'
}

interface IElementArray<T> extends Array<T> {
  // properties
  readonly $root:          IRoot;
  readonly $parent:        IElement;
  readonly $ctrl:          IController;
  readonly $isEmpty:       boolean;
  readonly $hasSelected:   boolean;
  readonly $hasChecked:    boolean;
  readonly $selectedIndex: number;
  readonly $checked:       IElementArray<T>;
  readonly $cross:         { [prop: string]: string[] };
  readonly $loaded:        boolean;
  readonly $ids:           string;
  readonly $names:         string;
  // methods
  $empty(): this;
  $reload(): Promise<IElementArray<T>>;
  $remove(elem: T): this;
  $clearSelected(): this;
  $append(src?: object): T;
  $prepend(src?: object): T;
  $insert(src: object, to: InsertTo, ref?: T): T;
  $copy(src: any[]): this;
  $sum(fn: (item: T) => number): number;
  $find(fn: (item: T, index?: number, array?: IElementArray<T>) => boolean, thisArg?: any): T;
  $isLazy(): boolean;
  $loadLazy(): Promise<IElementArray<T>>;
  $load(): void;
  $resetLazy(): void;
}
```

### Properties

| Member | Type | Description |
|--------|------|-------------|
| `$root` | `IRoot` | Reference to the model root |
| `$parent` | `IElement` | Reference to the object that owns the array |
| `$ctrl` | `IController` | Reference to the controller |
| `$isEmpty` | boolean | The array has no elements |
| `$hasSelected` | boolean | At least one element has `$selected = true` |
| `$hasChecked` | boolean | At least one element has `$checked = true` |
| `$selectedIndex` | number | Index of the selected element, or `-1` if none |
| `$checked` | `IElementArray<T>` | The subset of elements with `$checked = true` |
| `$cross` | object | Cross keys for cross-model data — property names are cross fields, values are arrays of key strings |
| `$loaded` | boolean | Whether a lazy array has been loaded |
| `$ids` | string | Comma-separated `Id` values of the elements (elements must have an `Id`) |
| `$names` | string | Space-and-comma-separated `Name` values of the elements (elements must have a `Name`) |

### Methods

| Member | Returns | Description |
|--------|---------|-------------|
| `$empty()` | `this` | Removes all elements |
| `$reload()` | `Promise` | Reloads the array from the database; only meaningful for child and lazy arrays |
| `$remove(elem)` | `this` | Removes `elem` from the array |
| `$clearSelected()` | `this` | Clears the selection flag on all elements |
| `$append(src?)` | `T` | Inserts an element at the end, copying properties from `src` if given. Returns the new element |
| `$prepend(src?)` | `T` | Inserts an element at the start, copying properties from `src` if given. Returns the new element |
| `$insert(src, to, ref?)` | `T` | Inserts an element at `to` (`start`/`end`/`above`/`below`), relative to `ref` for `above`/`below`. Returns the new element |
| `$copy(src)` | `this` | Copies an array of plain objects in; properties absent from the element are ignored |
| `$sum(fn)` | number | Sums the value returned by `fn` over all elements |
| `$find(fn, thisArg?)` | `T` | Finds the first matching element; recursive for hierarchical (tree) arrays |
| `$isLazy()` | boolean | Whether the array is lazy (deferred loading) |
| `$loadLazy()` | `Promise` | Loads a lazy array |
| `$load()` | void | Loads the array |
| `$resetLazy()` | void | Clears the lazy load flag, so the next access reloads from the server |

## Example

```js
const template = {
  properties: {
    'TDocument.Total'() { return this.Rows.$sum(r => r.Sum); }
  },
  commands: {
    addRow(doc) {
      const row = doc.Rows.$append({ Qty: 1 });
      row.$select();
    },
    removeChecked(doc) {
      doc.Rows.$checked.forEach(r => doc.Rows.$remove(r));
    }
  }
};

module.exports = template;
```

## Notes

- `$append`, `$prepend`, `$insert`, `$remove`, and `$empty` go through the platform, so they raise the array [events](https://docs-llm.a2v10.com/template/events.md). Native `push`/`splice` do not.
- `$find` differs from native `Array.find`: on a tree it descends recursively through child nodes. See [tree models](https://docs-llm.a2v10.com/sql/tree.md).
- Lazy arrays load on demand. Check `$isLazy()` / `$loaded`, trigger with `$loadLazy()`, and force a refresh with `$resetLazy()`. The shape is produced server-side — see [array models](https://docs-llm.a2v10.com/sql/array.md).
- `$ids` and `$names` require the elements to have `Id` and `Name` properties respectively.
- `$cross` exposes the keys of a cross (matrix) model; the property names are the cross fields.
- A model array is rendered in the UI by a [DataGrid](https://docs-llm.a2v10.com/xaml/controls/datagrid.md), whose `ItemsSource` binds to it; selection there surfaces as `$hasSelected` / `$selectedIndex` / `$checked` here and as `$selected` / `$checked` on each [array element](https://docs-llm.a2v10.com/client/array-element.md).
