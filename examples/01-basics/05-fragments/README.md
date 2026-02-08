# 01-basics/05-fragments

This example demonstrates how to use fragments in Lustre to group multiple elements
without adding extra container nodes to the DOM.

## What are fragments?

**Fragments** are a way to group multiple elements together in your virtual DOM
without introducing an actual HTML element in the rendered output. They exist
only in Lustre's virtual representation and are "unwrapped" when the HTML is
generated.

Think of fragments as invisible containers—they help you organize your view code
and enable powerful rendering patterns, but they don't affect the final HTML
structure.

## Why use fragments?

Fragments solve several common problems:

1. **Return multiple root elements** - Your view function can return multiple
   sibling elements without wrapping them in a `<div>`

2. **Avoid unnecessary DOM nodes** - No extra markup means cleaner HTML and
   potentially better performance

3. **Group elements with keys** - You can assign a key to an entire group of
   elements, allowing efficient reordering of element groups

4. **Conditional rendering** - Add or remove element groups without affecting
   siblings

5. **Mixed keying** - Some children can be keyed while others remain unkeyed

## Types of fragments

Lustre provides two types of fragments:

### Regular fragments

```gleam
import lustre/element

element.fragment([
  html.h1([], [html.text("Title")]),
  html.p([], [html.text("Paragraph")]),
])
```

Use `element.fragment()` when you need to group elements but don't need keys.

### Keyed fragments

```gleam
import lustre/element/keyed

keyed.fragment([
  #("item-1", html.div([], [html.text("First")])),
  #("item-2", html.div([], [html.text("Second")])),
])
```

Use `keyed.fragment()` when you need to track groups of elements for efficient
updates (similar to keyed lists).

## Example: FAQ Accordion

This example builds an FAQ accordion that demonstrates both types of fragments:

```gleam
fn view(model: Model) -> Element(Msg) {
  html.div([], [
    html.h1([], [html.text("FAQ")]),

    // Keyed fragment: groups all FAQ entries and assigns each a key
    keyed.fragment(list.map(model.entries, view_entry(model.open, _))),

    html.p([], [html.text("Additional content...")]),
  ])
}

fn view_entry(open: Set(String), entry: #(String, String)) -> #(String, Element(Msg)) {
  let #(question, answer) = entry
  let is_open = set.contains(open, question)

  let html =
    // Regular fragment: groups the button and optional answer together
    element.fragment([
      html.button([...], [html.text(question)]),
      case is_open {
        True -> html.p([], [html.text(answer)])
        False -> element.none()
      },
    ])

  #(question, html)
}
```

**Key points:**

- The outer `keyed.fragment()` allows FAQ entries to be efficiently reordered if
  the list changes
- The inner `element.fragment()` groups each question with its answer, so when
  an answer appears/disappears, Lustre can update efficiently
- No extra `<div>` wrappers pollute the HTML—open your browser dev tools to see!

## When to use fragments

Use fragments when:

- **Returning multiple elements** from a function without a natural container
- **Avoiding wrapper elements** that would complicate styling or semantics
- **Grouping dynamic content** that may conditionally render
- **Applying keys to groups** of related elements
- **Mixing keyed and unkeyed content** in the same container

For simple cases where a `<div>` wrapper wouldn't hurt, you might not need
fragments. But they're a powerful tool for building clean, efficient UIs.

## Running the Example

This example uses Lustre's [dev tools](https://hex.pm/packages/lustre_dev_tools)
to build and run the app. Just run:

```bash
gleam run -m lustre/dev start
```

and head to [http://localhost:1234](http://localhost:1234) in your browser.

Click on the FAQ questions to expand and collapse them, then open your browser's
developer tools to inspect the HTML structure. Notice that there are no extra
wrapper elements around the FAQ entries or between the questions and answers!
