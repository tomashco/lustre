# 02-inputs/04-forms

This example demonstrates how to use uncontrolled forms in Lustre with the
`formal` library for validation.

## What are uncontrolled forms?

Unlike controlled inputs (example 01), **uncontrolled forms** let the browser
manage input state. Your application only receives data when the form is
submitted, not on every keystroke.

This is perfect for traditional forms where you only need values when the user
submits.

## How it works

With uncontrolled forms:

1. Inputs have `name` attributes but no `value` or `on_input` handlers
2. The browser manages the input state
3. On submit, you receive all form data at once
4. Validate and process the data

```gleam
html.form([event.on_submit(HandleSubmit)], [
  html.input([
    attribute.name("username"),
    attribute.type_("text"),
    // No value or on_input - uncontrolled!
  ]),
  html.button([attribute.type_("submit")], [html.text("Submit")]),
])
```

## Example: Login form with validation

This example shows a login form with username and password validation using the
`formal` library:

```gleam
type Model {
  Login(Form(LoginData))
  LoggedIn(username: String)
}

type LoginData {
  LoginData(username: String, password: String)
}

// Create a form with validation rules
fn new_login_form() -> Form(LoginData) {
  form.new({
    use username <- form.field(
      "username",
      form.parse_string |> form.check_not_empty,
    )
    use password <- form.field(
      "password",
      form.parse_string |> form.check(check_password),
    )
    form.success(LoginData(username:, password:))
  })
}

// Handle submission
fn update(model: Model, msg: Msg) -> Model {
  case msg {
    UserSubmittedForm(Ok(LoginData(username:, ..))) ->
      LoggedIn(username:)  // Validation passed!

    UserSubmittedForm(Error(form)) ->
      Login(form)  // Validation failed - show errors
  }
}
```

**How it works:**

- The browser manages input state
- On submit, `event.on_submit` provides all form data as name-value pairs
- `formal` validates the data against your rules
- If valid, you get typed data (`LoginData`)
- If invalid, you get the form back with error messages

## Using event.on_submit

The `event.on_submit` handler automatically:

- Prevents default browser form submission
- Extracts form data as `List(#(String, String))`
- Passes it to your handler function

```gleam
let handle_submit = fn(values) {
  form |> form.add_values(values) |> form.run |> UserSubmittedForm
}

html.form([event.on_submit(handle_submit)], [...])
```

## Controlled vs uncontrolled

| Pattern | State managed by | Updates | Best for |
|---------|-----------------|---------|----------|
| Controlled | Your app | Every keystroke | Real-time validation, few inputs |
| Uncontrolled | Browser | On submit | Traditional forms, many inputs |

**Use uncontrolled forms when:**
- You have many input fields
- You only need values on submission
- You want simpler state management
- You're building traditional server-style forms

## Running the Example

Run the example using Lustre's [dev tools](https://hex.pm/packages/lustre_dev_tools):

```bash
gleam run -m lustre/dev start
```

and head to [http://localhost:1234](http://localhost:1234) in your browser.

Try submitting the form with:
- Empty fields (validation error)
- Wrong password (must be "strawberry")
- Correct credentials (success!)
