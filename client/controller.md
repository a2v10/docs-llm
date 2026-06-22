# Controller (IController)

> The object that drives interaction between the model, the server, and the UI — reached from any model object via `$ctrl`.

## Overview

The controller manages the interaction between the model and the view. Every object exposes it through the `$ctrl` property, and it is a standard JavaScript object implementing `IController`.

Its methods cover the operations template code needs at runtime: invoking server commands, reloading the model, showing messages and confirmations, opening dialogs, running asynchronous validation, uploading files, and managing the dirty flag. Most return a `Promise`, so they compose naturally with `async`/`await`.

## Syntax

```ts
interface IController {
  // properties
  readonly $isDirty:    boolean;
  readonly $isPristine: boolean;
  // methods
  $invoke(command: string, arg: object, path?: string, opts?: { catchError: boolean }): Promise<object>;
  $requery(): void;
  $reload(args?: any): Promise<void>;
  $modalClose(result?: any): void;
  $msg(msg: string, title?: string, style?: CommonStyle): Promise<boolean>;
  $alert(msg: string | IMessage): Promise<boolean>;
  $confirm(msg: string | IConfirm): Promise<boolean>;
  $toast(text: string, style?: CommonStyle): void;
  $defer(func: () => void): void;
  $showDialog(url: string, data?: object, query?: object): Promise<object>;
  $asyncValid(cmd: string, arg: object): any | Promise<any>;
  $saveModified(message?: string, title?: string): boolean;
  $setFilter(target: object, prop: string, value: any): void;
  $inlineOpen(id: string): void;
  $inlineClose(id: string, result: any): void;
  $focus(htmlid: string): void;
  $emitCaller(event: string, ...params: any[]): void;
  $emitSaveEvent(): void;
  $nodirty(func: () => Promise<any>): void;
  $upload(url: string, accept?: string, data?: { Id?: any, Key?: any }, opts?: { catchError?: boolean }): Promise<any>;
}

declare const enum CommonStyle {
  error = 'error', warning = 'warning', info = 'info', success = 'success'
}
```

### Properties

| Member | Type | Description |
|--------|------|-------------|
| `$isDirty` | boolean | The model has unsaved changes |
| `$isPristine` | boolean | The model has no unsaved changes |

### Methods

| Member | Returns | Description |
|--------|---------|-------------|
| `$invoke(command, arg, path?, opts?)` | `Promise` | Calls a server [command](https://docs-llm.a2v10.com/model/commands.md) by name; `arg` properties become its arguments |
| `$requery()` | void | Re-runs the current query |
| `$reload(args?)` | `Promise` | Reloads the model, or a lazy array if one is passed |
| `$modalClose(result?)` | void | Closes the current modal, returning `result` |
| `$msg(msg, title?, style?)` | `Promise` | Shows a message in a modal window |
| `$alert(msg)` | `Promise` | Shows an alert |
| `$confirm(msg)` | `Promise` | Shows a confirmation query and resolves to the user's choice |
| `$toast(text, style?)` | void | Shows a transient toast notification |
| `$defer(func)` | void | Runs `func` after the current event cycle |
| `$showDialog(url, data?, query?)` | `Promise` | Opens a dialog at `url` and resolves to its result |
| `$asyncValid(cmd, arg)` | `Promise` | Runs an async validation command at the current endpoint |
| `$saveModified(message?, title?)` | boolean | Prompts to save if the model changed; see below |
| `$setFilter(target, prop, value)` | void | Sets a filter value (filters cannot be assigned directly) |
| `$inlineOpen(id)` | void | Opens an inline dialog by `id` |
| `$inlineClose(id, result)` | void | Closes an inline dialog by `id` |
| `$focus(htmlid)` | void | Moves the cursor to the element with `htmlid` |
| `$emitCaller(event, ...params)` | void | Sends an event to the calling code (caller) |
| `$emitSaveEvent()` | void | Sends the event named by the dialog's `SaveEvent` to the caller |
| `$nodirty(func)` | void | Runs `func` (async) without setting the dirty flag |
| `$upload(url, accept?, data?, opts?)` | `Promise` | Shows a file picker and uploads the file to a file command |

## Key methods

### $invoke

Calls a server command declared in the [`commands`](https://docs-llm.a2v10.com/model/commands.md) section of `model.json`. `arg` is an arbitrary object whose properties become command arguments. `path` optionally targets another endpoint; omitted, it runs at the current one. With `opts.catchError` set, the promise rejects on error so it can be caught — otherwise the platform shows the error and the promise does not run.

### $confirm

Shows a confirmation query and resolves to the chosen button's `result`. The `IConfirm` form accepts `title`, a `list` of extra strings (for example errors), and custom `buttons` (each `{ text, result }`); a Cancel button must use `result: false`.

### $saveModified

Returns `true` immediately if the model is unchanged. If it changed, it shows a save dialog (Save / Don't save / Cancel) and returns `false`; the controller then performs the chosen action. Most commonly used in the `CanCloseDelegate` of a dialog.

### $asyncValid

Runs an async [validator](https://docs-llm.a2v10.com/template/validators.md) command at the current endpoint, usually backed by an SQL procedure that returns a single `Result` object with one `Value` property of type `bit` or `nvarchar`. Results are cached to limit server calls.

## Example

```js
const template = {
  delegates: {
    canClose(doc) { return doc.$ctrl.$saveModified(); }
  },
  commands: {
    async post(doc) {
      if (!await doc.$ctrl.$confirm('Post this document?')) return;
      await doc.$ctrl.$invoke('post', { Id: doc.Id });
      doc.$ctrl.$toast('Document posted', 'success');
    }
  }
};

module.exports = template;
```

## Notes

- The controller is reached from any object via [`$ctrl`](https://docs-llm.a2v10.com/client/element.md); template code does not construct it.
- `$invoke` and `$asyncValid` call server [commands](https://docs-llm.a2v10.com/model/commands.md) — the command must exist in `model.json`.
- Filters are tied to markup and system behavior, so set them with `$setFilter`, never by assigning the property directly.
- `$saveModified` returns `true` when there is nothing to save (`$isDirty` is `false`) — treat `true` as "safe to close".
- Wrap operations that should not dirty the model in `$nodirty`; the function passed must be async or return a promise.
