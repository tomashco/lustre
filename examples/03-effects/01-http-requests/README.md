# 03-effects/01-http-requests

This example demonstrates how to perform HTTP requests in Lustre using effects
and the `rsvp` library.

## What are effects?

**Effects** are how Lustre handles side effects—operations that interact with
the world outside your application, like HTTP requests, timers, or local storage.

Effects are descriptions of work to be done. The Lustre runtime executes them
and sends results back to your `update` function as messages.

## Using lustre.application

To use effects, switch from `lustre.simple` to `lustre.application`:

```gleam
let app = lustre.application(init, update, view)
```

Now `init` and `update` return tuples: `#(Model, Effect(Msg))`

## Example: Todo list with API

This example fetches todos from a REST API on startup and updates them when
clicked:

```gleam
import lustre/effect.{type Effect}
import rsvp

// Init now returns a tuple
fn init(_) -> #(Model, Effect(Msg)) {
  let model = []
  let effect = fetch_todos(on_response: ApiReturnedTodos)

  #(model, effect)
}

fn fetch_todos(
  on_response handle_response: fn(Result(List(Todo), rsvp.Error)) -> msg,
) -> Effect(msg) {
  let url = "https://jsonplaceholder.typicode.com/todos/"
  let decoder = decode.list(todo_decoder())
  let handler = rsvp.expect_json(decoder, handle_response)

  rsvp.get(url, handler)
}

// Update also returns a tuple
fn update(model: Model, msg: Msg) -> #(Model, Effect(Msg)) {
  case msg {
    // When the API responds, update the model
    ApiReturnedTodos(Ok(todos)) -> #(todos, effect.none())

    // When user clicks, trigger an API request
    UserClickedComplete(id, completed) -> #(
      model,
      complete_todo(id:, completed:, on_response: ApiUpdatedTodo),
    )
  }
}
```

**How it works:**

1. App starts → `init` returns effect to fetch todos
2. Runtime executes HTTP request
3. API responds → `ApiReturnedTodos` message sent to `update`
4. Model updates with todos
5. User clicks checkbox → `UserClickedComplete` message
6. `update` returns effect to update todo via API
7. API responds → `ApiUpdatedTodo` message
8. Model updates to reflect completion

## Using rsvp for HTTP requests

The `rsvp` library provides HTTP operations as effects:

```gleam
import rsvp

// GET request
rsvp.get(url, handler)

// POST request
rsvp.post(url, body, handler)

// Custom request (PATCH, PUT, DELETE, etc.)
request
  |> request.set_method(http.Patch)
  |> request.set_body(body)
  |> rsvp.send(handler)
```

Handlers expect a response type and produce a message:

```gleam
// Expect JSON and decode it
rsvp.expect_json(decoder, ApiReturnedData)

// Expect plain text
rsvp.expect_text(ApiReturnedText)

// Expect nothing (for DELETE, etc.)
rsvp.expect_anything(ApiCompleted)
```

## When no effects are needed

Use `effect.none()` when you don't need to perform side effects:

```gleam
fn update(model: Model, msg: Msg) -> #(Model, Effect(Msg)) {
  case msg {
    // API succeeded - just update model, no more effects
    ApiReturnedTodos(Ok(todos)) -> #(todos, effect.none())
  }
}
```

## Type-safe effects

Effects are parameterized by the message type they produce, just like elements:

```gleam
fn fetch_todos() -> Effect(Msg) {
  // This effect produces Msg values
  rsvp.get(url, rsvp.expect_json(decoder, ApiReturnedTodos))
}
```

The runtime ensures effects can only send messages your `update` function can
handle.

## Running the Example

Run the example using Lustre's [dev tools](https://hex.pm/packages/lustre_dev_tools):

```bash
gleam run -m lustre/dev start
```

and head to [http://localhost:1234](http://localhost:1234) in your browser.

The app fetches 10 todos from the JSONPlaceholder API. Click the checkboxes to
toggle completion—each click makes a PATCH request to the API!
