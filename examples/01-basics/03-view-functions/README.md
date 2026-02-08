# 01-basics/03-view-functions

This example demonstrates how to create reusable UI components using view functions
in Lustre.

## What are view functions?

View functions are the primary way to create reusable UI components in Lustre. They
are just regular Gleam functions that return `Element` values, making them simple
to understand and compose.

### Basic View Functions

```gleam
fn view_button(on_click handle_click: msg, label text: String) -> Element(msg) {
  html.button([event.on_click(handle_click)], [html.text(text)])
}
```

This example shows a reusable button component. View functions can:

- Accept any number of arguments
- Use labelled arguments for a props-like experience
- Include event handlers as parameters
- Be composed together to build complex UIs

### Using View Functions

```gleam
fn view(model: Model) -> Element(Msg) {
  html.div([], [
    view_button(on_click: UserClickedDecrement, label: "-"),
    view_count(model),
    view_button(on_click: UserClickedIncrement, label: "+"),
  ])
}
```

View functions are called like any other function and can be mixed with regular
HTML elements.

### Generic View Functions

```gleam
fn view_count(count: Int) -> Element(msg) {
  html.p([...], [html.text("Count: "), html.text(int.to_string(count))])
}
```

View functions that don't produce events use a **generic `msg` type variable**
(lowercase) instead of a concrete `Msg` type. This makes them reusable in any
context since there's no way to construct a value of type `msg` - the function
cannot possibly produce events.

This is better than using `Element(Nil)` because it allows these components to
be used anywhere without type conflicts.

## Running the Example

This example and all the others in this repository use Lustre's
[dev tools](https://hex.pm/packages/lustre_dev_tools) package to build and run
the app. Just run:

```bash
gleam run -m lustre/dev start
```

and head to [http://localhost:1234](http://localhost:1234) in your browser.
