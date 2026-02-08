# 01-basics/06-flags

This example demonstrates how to use flags to pass initial data into your Lustre
application at startup.

## What are flags?

**Flags** are the mechanism for passing data from the outside world into your
Lustre application when it starts. They're the third argument to `lustre.start()`,
and whatever value you pass in will be forwarded to your `init` function.

```gleam
let initial_data = "some data"
let assert Ok(_) = lustre.start(app, "#app", initial_data)
```

## Why use flags?

Lustre applications follow functional principles where your `init`, `update`, and
`view` functions should be **pure**—they produce the same output for the same
input and don't perform side effects.

But sometimes you need initial data that comes from side effects:

- Random values
- Data from `localStorage` or other browser APIs
- Query parameters from the URL
- Configuration from environment variables
- Server-rendered data (for hydration)

**Flags let you perform these side effects once at startup**, before your app
begins, keeping the rest of your application pure.

## How flags work

The flow is simple:

1. **Before starting** - Gather your initial data through side effects
2. **Pass to `lustre.start()`** - Provide the data as the third argument (flags)
3. **Receive in `init()`** - Your init function receives the flags and uses them
   to create the initial model

```gleam
pub fn main() {
  let app = lustre.simple(init, update, view)

  // 1. Perform side effect to get initial data
  let initial_count = int.random(20)

  // 2. Pass it as flags to lustre.start()
  let assert Ok(_) = lustre.start(app, "#app", initial_count)

  Nil
}

// 3. Your init function receives the flags
fn init(initial_count: Int) -> Model {
  initial_count / 2
}
```

## Example: Random counter

This example demonstrates a counter that starts with a random value. Instead of
hardcoding the initial count, we:

1. Generate a random number between 0 and 20
2. Pass it to the app via flags
3. Use it in `init()` to set up the model (divided by 2)

Every time you refresh the page, the counter starts at a different random value!

### The code

```gleam
pub fn main() {
  let app = lustre.simple(init, update, view)

  // Generate random initial data
  let initial_count = int.random(20)
  let assert Ok(_) = lustre.start(app, "#app", initial_count)

  Nil
}

fn init(initial_count: Int) -> Model {
  // Use the flags to initialize the model
  initial_count / 2
}
```

## Common use cases for flags

### Browser APIs

```gleam
// Read from localStorage
let saved_data = local_storage.get_item("my-app-data")
let assert Ok(_) = lustre.start(app, "#app", saved_data)
```

### URL parameters

```gleam
// Parse query params
let params = get_query_params()
let assert Ok(_) = lustre.start(app, "#app", params)
```

### Server-side hydration

When rendering HTML on the server, you can embed initial data in the HTML and
read it on the client:

```gleam
// Read embedded JSON from a script tag
let json = document.query_selector("#model")
  |> result.map(element.inner_text)
  |> result.try(json.parse(_, my_decoder()))
  |> result.unwrap(default_value)

let assert Ok(_) = lustre.start(app, "#app", json)
```

This pattern is essential for server-side rendering (SSR) and hydration.

## Type safety

Flags are fully type-safe! The type you pass to `lustre.start()` must match the
type your `init` function expects:

```gleam
// init expects an Int, so flags must be an Int
fn init(flags: Int) -> Model { ... }

// ✅ This works
lustre.start(app, "#app", 42)

// ❌ This won't compile
lustre.start(app, "#app", "not an int")
```

## Running the Example

This example uses Lustre's [dev tools](https://hex.pm/packages/lustre_dev_tools)
to build and run the app. Just run:

```bash
gleam run -m lustre/dev start
```

and head to [http://localhost:1234](http://localhost:1234) in your browser.

Each time you refresh the page, the counter will start at a different random value
(between 0 and 10, since we divide the random number by 2).
