# Client Template

> The client-side JavaScript template that defines model behavior in A2v10 — initial values, computed properties, validation, lifecycle events, view commands, and delegates. One exported object, one section per concern.

- [Template Overview](https://docs-llm.a2v10.com/template/overview.md): The exported template object and its sections — structure, wiring, and a minimal example
- [Options](https://docs-llm.a2v10.com/template/options.md): Model-wide flags — dirty-flag control, bind-once roots, and persisted array selection
- [Defaults](https://docs-llm.a2v10.com/template/defaults.md): Initial values for a new model, literal or computed
- [Properties](https://docs-llm.a2v10.com/template/properties.md): Scalar and computed properties added to model elements
- [Validators](https://docs-llm.a2v10.com/template/validators.md): Data-bound validation rules — standard, function, object, and async forms
- [Events](https://docs-llm.a2v10.com/template/events.md): Model, object, and array lifecycle handlers
- [Commands](https://docs-llm.a2v10.com/template/commands.md): Named operations invoked from the view via BindCmd, with confirmation and guards
- [Delegates](https://docs-llm.a2v10.com/template/delegates.md): Callback functions attached to controls, such as a selector fetch
