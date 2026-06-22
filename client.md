# Client API

> The client-side object model and runtime API of A2v10 — the `$`-prefixed service members on model objects, the controller, and the standard utilities. This is the API that client template code is written against.

- [Client Object Model](https://docs-llm.a2v10.com/client/overview.md): How the model is extended on the client — the five object shapes and where to find each
- [Element (IElement)](https://docs-llm.a2v10.com/client/element.md): Base members on every object — `$root`, `$parent`, `$ctrl`, identity, and validation state
- [Root Object (IRoot)](https://docs-llm.a2v10.com/client/root.md): The model root (TRoot) — model-wide state, the template reference, validation and dirty control
- [Array (IElementArray)](https://docs-llm.a2v10.com/client/array.md): Array members — selection, cross keys, lazy loading, and insert/remove/find/sum methods
- [Array Element (IArrayElement)](https://docs-llm.a2v10.com/client/array-element.md): Members of an element inside an array — select, check, remove, and move
- [Tree Element (ITreeElement)](https://docs-llm.a2v10.com/client/tree-element.md): Tree-node members — expand state, `$expand`, and `$selectPath`
- [Controller (IController)](https://docs-llm.a2v10.com/client/controller.md): The `$ctrl` object — server commands, reload, messages, dialogs, validation, and file upload
- [Standard Utilities (std:utils)](https://docs-llm.a2v10.com/client/utils.md): The std:utils module — date, currency, and text helpers
- [require](https://docs-llm.a2v10.com/client/require.md): Loading modules — application modules by path, standard modules by `std:` name
