# 02-inputs/01-controlled-inputs

This example demonstrates how to use controlled inputs in Lustre to manage form
input state and validate input in real-time.

## What are controlled inputs?

A **controlled input** is a form input whose value is directly managed by your
application's model. Controlled inputs have two key characteristics:

1. The `value` attribute is set from your model
2. An event handler (like `on_input`) updates your model when input changes

```gleam
html.input([
  attribute.value(model),           // Value from model
  event.on_input(UserUpdatedName),  // Updates model
])
```

This creates a cycle: model → view → user input → update → model. The input
displays exactly what's in your model—**the DOM is a direct reflection of your
application state**.

## Example: Name input with length limit

This example restricts names to 10 characters:

```gleam
type Model = String

type Msg {
  UserUpdatedName(String)
}

fn update(model: Model, msg: Msg) -> Model {
  case msg {
    UserUpdatedName(name) ->
      case string.length(name) <= 10 {
        True -> name    // Accept the new value
        False -> model  // Reject—keep the old value
      }
  }
}

fn view(model: Model) -> Element(Msg) {
  html.input([
    attribute.value(model),
    event.on_input(UserUpdatedName),
  ])
}
```

When you type more than 10 characters, the input stops accepting them. This
works because `update` rejects the change by returning the old model value—even
though the user typed, the model didn't change, so the view doesn't update.

## Why use controlled inputs?

Controlled inputs give you complete control over user input:

- **Validate on every keystroke** - Reject invalid input immediately
- **Transform input** - Format values as users type (e.g., uppercase)
- **Enforce constraints** - Length limits, character restrictions
- **Synchronize state** - Input always matches your model

## Two-way binding

The combination creates "two-way binding":

- `attribute.value(model)` - Model → view
- `event.on_input(Msg)` - View → model

Together, they keep input and model in perfect sync.

## Controlled vs uncontrolled

Lustre supports both patterns:

| Pattern | State managed by | When to read | Best for |
|---------|-----------------|--------------|----------|
| Controlled | Your app | On every change | Few inputs, real-time validation |
| Uncontrolled | Browser | On form submit | Many inputs, simple forms |

**Use controlled inputs when you need real-time validation, formatting, or
constraints.** Use uncontrolled inputs for simple forms where you only need
values on submission.

## Running the Example

Run the example using Lustre's [dev tools](https://hex.pm/packages/lustre_dev_tools):

```bash
gleam run -m lustre/dev start
```

and head to [http://localhost:1234](http://localhost:1234) in your browser.

Try typing more than 10 characters—the input stops accepting them after the
limit. The greeting updates in real-time as you type!
