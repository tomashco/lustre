# 02-inputs/03-debouncing

This example demonstrates how to use `event.debounce()` and `event.throttle()`
to control how often events fire, improving performance and reducing unnecessary
updates.

## What are debouncing and throttling?

### Debouncing

**Debouncing** waits for events to stop before firing. The timer resets with each
new event, and only fires after the specified delay passes without new events.

```
   original : --a-b-cd--e----------f--------
  debounced : ---------------e----------f---
```

**Use for:** Search as you type, form validation, expensive operations

### Throttling

**Throttling** limits events to a maximum frequency—one event per time period,
no matter how many occur.

```
   original : --a-b-cd--e----------f--------
  throttled : -a------ d----------f---------
```

**Use for:** Mouse movement, scroll handlers, animations

## Why rate-limit events?

Events like `mousemove` and `input` fire very frequently. Without rate limiting,
your `update` function runs excessively, causing performance issues and (for
server components) excessive network traffic

## Example 1: Debounced name input

The example includes a name input that waits 500ms after typing stops before
updating:

```gleam
html.input([
  attribute.value(name),
  event.on_input(UserUpdatedName) |> event.debounce(500),
])
```

Type quickly and notice the greeting only updates after you pause. This is
perfect for search-as-you-type where you want to wait for the user to finish
before making an API call.

## Example 2: Throttled mouse tracking

The mouse tracking pad updates at most once every 250ms:

```gleam
let on_mousemove = event.on("mousemove", {
  use x <- decode.field("offsetX", decode.int)
  use y <- decode.field("offsetY", decode.int)
  decode.success(UserMovedMouse(x, y))
})

html.div([
  on_mousemove |> event.throttle(250),
], [...])
```

Move your mouse rapidly over the gray square—the coordinates only update 4 times
per second, not hundreds of times. This prevents excessive re-renders while
remaining responsive.

## How to use

Both work by piping any event handler through the function:

```gleam
// Debounce - wait for pause
event.on_input(SearchChanged) |> event.debounce(500)

// Throttle - limit frequency
event.on("mousemove", decoder) |> event.throttle(250)
```

You can chain them with other modifiers:

```gleam
event.on_input(SearchChanged)
  |> event.debounce(300)
  |> event.prevent_default
```

**When to use which:**
- **Debounce** when you want to wait for the user to "finish" (search, validation)
- **Throttle** when you want regular but controlled updates (scroll, mouse tracking)

## Running the Example

Run the example using Lustre's [dev tools](https://hex.pm/packages/lustre_dev_tools):

```bash
gleam run -m lustre/dev start
```

and head to [http://localhost:1234](http://localhost:1234) in your browser.

Try both examples—type rapidly in the input and move your mouse quickly over
the gray square to see the difference rate limiting makes!
