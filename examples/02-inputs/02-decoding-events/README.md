# 02-inputs/02-decoding-events

This example demonstrates how to create custom event handlers by decoding data
from JavaScript event objects.

## What is event decoding?

Lustre provides convenience functions like `event.on_click` and `event.on_input`
for common cases, but for custom scenarios you can use `event.on()` with a
decoder to extract specific data from event objects.

**Event decoding** lets you access any property from browser events—mouse
coordinates, keyboard keys, modifiers, or custom event data.

## Using event.on() with decoders

The `event.on()` function takes two arguments:

1. The event name (e.g., `"mousemove"`, `"keydown"`)
2. A decoder that extracts data and produces a message

```gleam
event.on("mousemove", {
  use x <- decode.field("offsetX", decode.int)
  use y <- decode.field("offsetY", decode.int)

  decode.success(UserMovedMouse(x, y))
})
```

The decoder uses Gleam's `use` syntax to chain operations—extract `offsetX`,
extract `offsetY`, then create a message with both values.

## Example: XY tracking pad

This example tracks mouse movement and displays coordinates:

```gleam
type Msg {
  UserMovedMouse(x: Int, y: Int)
}

fn view_xy_pad() -> Element(Msg) {
  let on_mousemove =
    event.on("mousemove", {
      use x <- decode.field("offsetX", decode.int)
      use y <- decode.field("offsetY", decode.int)

      decode.success(UserMovedMouse(x, y))
    })

  html.div([on_mousemove], [
    html.text("x: " <> int.to_string(x) <> ", y: " <> int.to_string(y))
  ])
}
```

**How it works:**

- `offsetX` and `offsetY` are coordinates relative to the element
- The decoder extracts both values from the JavaScript `MouseEvent`
- A message is created with the coordinates
- The model updates on every mouse move

## Common event properties

Different event types have different properties you can decode. Here are some
useful ones:

**MouseEvent:** `offsetX`, `offsetY`, `clientX`, `clientY`, `button`, `shiftKey`, `ctrlKey`, `altKey`
**KeyboardEvent:** `key`, `code`, `shiftKey`, `ctrlKey`, `altKey`, `repeat`
**FocusEvent:** Access nested values with `decode.subfield(["target", "value"], decode.string)`

See [MDN MouseEvent](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent)
and [MDN KeyboardEvent](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent)
for complete references.

## When to use custom decoders

**Use convenience functions when possible:**

- `event.on_click(msg)` for simple clicks
- `event.on_input(fn)` for input values
- `event.on_submit(msg)` for form submission

**Use custom decoders when you need:**

- Mouse coordinates or keyboard modifiers
- Multiple properties from a single event
- Custom or third-party event types
- Access to nested event properties

## Running the Example

Run the example using Lustre's [dev tools](https://hex.pm/packages/lustre_dev_tools):

```bash
gleam run -m lustre/dev start
```

and head to [http://localhost:1234](http://localhost:1234) in your browser.

Move your mouse over the gray square to see the coordinates update in real-time!
