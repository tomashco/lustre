# 01-basics/04-keyed-elements

This example demonstrates the importance of keyed elements for efficient DOM
updates when rendering lists in Lustre.

## What are keyed elements?

When Lustre re-renders your application, it needs to figure out what has changed
in the DOM. For lists of elements, this can be tricky: if items move around or
the order changes, Lustre might think elements have changed when they've actually
just moved position.

**Keyed elements** solve this problem by giving each element in a list a unique
identifier (a "key"). This allows Lustre to accurately track which items have
changed, moved, or been removed, leading to more efficient DOM updates.

## Keyed vs Unkeyed Lists

This example shows the difference side-by-side with two lists of cat images:

### Unkeyed List (Inefficient)

```gleam
html.ul([attribute.class("grid grid-cols-3 gap-2")], {
  list.map(images, view_cat)
})
```

Without keys, when the list order changes, Lustre thinks **every** element has
changed. It updates all the `src` attributes, causing unnecessary DOM operations.
In the example, you'll see all images flash when the list cycles.

### Keyed List (Efficient)

```gleam
keyed.ul([attribute.class("grid grid-cols-3 gap-2")], {
  list.map(images, fn(id) { #(id, view_cat(id)) })
})
```

With keys, Lustre can recognize that the first two images are the same as the
previous render—they've just moved. Only the new image (the third one) needs to
be updated. You'll see only the new image flash.

## How to Use Keyed Elements

The `lustre/element/keyed` module provides keyed versions of common container
elements like `ul`, `ol`, and `div`.

Instead of a list of elements, keyed functions take a **list of tuples**:

```gleam
#(key, element)
```

- **key**: A unique identifier (usually a String or Int from your data)
- **element**: The Element to render

### Example

```gleam
import lustre/element/keyed

fn view_items(items: List(Item)) -> Element(msg) {
  keyed.ul([], {
    list.map(items, fn(item) {
      #(item.id, html.li([], [html.text(item.text)]))
    })
  })
}
```

## When to Use Keyed Elements

Use keyed elements when:

- Rendering lists that can change order
- Items can be added, removed, or reordered
- You want to preserve element state (like focus or scroll position)
- Performance matters for large lists

For small, static lists, the overhead of keying might not be worth it, but for
dynamic lists it's almost always the right choice.

## Running the Example

This example includes a small script that flashes elements when they're updated
in the DOM, making it easy to visualize the difference between keyed and unkeyed
lists.

Run the example using Lustre's [dev tools](https://hex.pm/packages/lustre_dev_tools):

```bash
gleam run -m lustre/dev start
```

and head to [http://localhost:1234](http://localhost:1234) in your browser.

Click the "Next" button to cycle through the images and watch how the two lists
behave differently!
