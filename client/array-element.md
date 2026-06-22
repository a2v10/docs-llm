# Array Element (IArrayElement)

> An object that lives inside an array — extends a base element with selection and checked state and methods to remove, select, and move itself.

## Overview

Any object that is an element of an array extends [`IElement`](https://docs-llm.a2v10.com/client/element.md) with members for its position in the collection. It can be selected (highlighted) or checked (marked), removed from its parent, and moved up or down within the array.

Selection is exclusive — selecting an element clears the previously selected one. The set of checked elements is read from the parent array's [`$checked`](https://docs-llm.a2v10.com/client/array.md) property.

## Syntax

```ts
declare const enum MoveDir {
  up   = 'up',
  down = 'down'
}

interface IArrayElement extends IElement {
  // properties
  readonly $parent: IElementArray<IElement>;
  $selected: boolean;
  $checked:  boolean;
  // methods
  $remove(): void;
  $select(root?: IElementArray<IElement>): void;
  $move(dir: MoveDir): void;
  $canMove(dir: MoveDir): boolean;
}
```

### Properties

| Member | Type | Description |
|--------|------|-------------|
| `$parent` | `IElementArray` | The array the element belongs to (read-only) |
| `$selected` | boolean | Whether the element is highlighted (writable) |
| `$checked` | boolean | Whether the element is marked (writable) |

### Methods

| Member | Description |
|--------|-------------|
| `$remove()` | Removes the element from its parent array |
| `$select(root?)` | Highlights the element and clears the previous selection; for a selection inside a tree, pass the tree root as `root` |
| `$move(dir)` | Moves the element `up` or `down` within the array; returns the element |
| `$canMove(dir)` | Whether the element can move in `dir` — `false` for the first element moving up or the last moving down |

## Example

```js
const template = {
  commands: {
    moveUp(row) {
      if (row.$canMove('up')) row.$move('up');
    },
    deleteRow(row) { row.$remove(); }
  }
};

module.exports = template;
```

## Notes

- `$selected` and `$checked` are writable, unlike most service members. Selecting through `$select()` also resets the previous selection; setting `$selected = true` directly does not.
- The collection of checked elements is obtained from the parent array's [`$checked`](https://docs-llm.a2v10.com/client/array.md) property.
- Guard `$move()` with `$canMove()` to avoid moving past the array boundaries.
- A tree node is an array element too — see [tree element](https://docs-llm.a2v10.com/client/tree-element.md) for the additional members.
