# Commands

> The `commands` section of a template — named operations invoked from the view, with optional confirmation and pre-execution checks.

## Overview

A template command is a client-side operation invoked from the view — typically by a button. The descriptor is an ordinary JavaScript object where each property name is a command name and each value is a function or an object.

Commands are wired to controls with the [`BindCmd`](https://docs-llm.a2v10.com/xaml/bind.md) binding, using `Command="Execute"` or `Command="ExecuteSelected"` and the `CommandName` property to name the template command. The control bound to a command is automatically enabled or disabled according to the command's availability — do not set the `Disabled` property by hand.

When the value is a plain function, it is simply called when the command runs. The object form adds an availability check, pre-execution requirements (save, validity, read-only), and a confirmation prompt.

## Syntax

```js
commands: {
  Command1: Function,
  Command2: {
    exec:          Function,
    canExec:       Function,
    saveRequired:  Boolean,
    validRequired: Boolean,
    checkReadOnly: Boolean,
    confirm:       String | Object
  }
}
```

### Command object properties

| Property | Type | Description |
|----------|------|-------------|
| `exec` | Function | Executes the command |
| `canExec` | Function | Returns a boolean deciding whether the command can execute |
| `saveRequired` | Boolean | Save the model before executing |
| `validRequired` | Boolean | Execute only when the model passed validation (root `$valid` is `true`) |
| `checkReadOnly` | Boolean | Forbid the command for read-only models |
| `confirm` | String \| Object | Ask the user to confirm before executing |

### Confirm object properties

| Property | Description |
|----------|-------------|
| `message` | Message text |
| `title` | Window title (defaults to `locale.$Confirm`) |
| `okText` | Confirm button text (defaults to `locale.$Ok`) |
| `cancelText` | Cancel button text (defaults to `locale.$Cancel`) |

A string `confirm` value is treated as the message text.

### Function arguments

Both `exec` and `canExec` receive:

| Argument | Description |
|----------|-------------|
| `this` | The root object (`TRoot`) |
| `arg` | The argument passed to the command — set via the `Argument` property in `BindCmd` |

## Example

The view binds a button to the command and passes the current document as its argument:

```xml
<Button Command="{BindCmd Execute, CommandName='MyCommand', Argument={Bind Document}}">
  Run command
</Button>
```

The template defines the command with a guard and a confirmation prompt:

```js
const template = {
  commands: {
    MyCommand: {
      exec(doc) { alert(doc.Id); },
      canExec(doc) { return doc.$isNew; },
      validRequired: true,
      confirm: {
        message:    'Really run this?',
        title:      'Confirmation required',
        okText:     'Yes, run it',
        cancelText: 'No, cancel'
      }
    }
  }
};

module.exports = template;
```

## Notes

- Controls bound to a command are enabled and disabled automatically from `canExec`; never set `Disabled` yourself.
- `validRequired` checks the root object's `$valid` property — define the relevant rules in [`validators`](https://docs-llm.a2v10.com/template/validators.md).
- `saveRequired` saves the model before running `exec`, so the command can rely on persisted data.
- The `arg` passed to `exec`/`canExec` comes from the `Argument` of the [`BindCmd`](https://docs-llm.a2v10.com/xaml/bind.md) in the view.
- A bare function value is shorthand for an object with only `exec`.
- Template commands run on the client; for server-side operations configured in model.json, see [model commands](https://docs-llm.a2v10.com/model/commands.md).
