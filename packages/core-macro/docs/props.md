# Props

The props derive macro allows you to define what props your component accepts and how to accept those props. Every component must either accept no arguments or accept a single argument that implements the `Properties` trait.

> Note: You should generally prefer using the `#[component]` macro instead of the `#[derive(Props)]` macro with explicit props. The `#[component]` macro will automatically generate the props struct for you and perform extra checks to ensure that your component is valid.

## Example

```rust, no_run
# use dioxus::prelude::*;
#[derive(Props, PartialEq, Clone)]
struct ButtonProps {
    /// The text of the button
    text: String,

    /// The color of the button
    color: String,
}

fn Button(props: ButtonProps) -> Element {
    rsx! {
        button {
            color: props.color,
            "{props.text}"
        }
    }
}

rsx! {
    // Any fields you defined on the props struct will be turned into props for the component.
    Button {
        text: "Click me!",
        color: "red",
    }
};
```

## Prop Modifiers

You can use the `#[props()]` attribute to modify the behavior of the props derive macro:

- [`#[props(default)]`](#default-props) - Makes the field optional in the component and uses the default value if it is not set when creating the component.
- [`#[props(!optional)]`](#optional-props) - Makes a field with the type `Option<T>` required.
- [`#[props(into)]`](#converting-props) - Converts a field into the correct type by using the [`Into`] trait.
- [`#[props(extends = GlobalAttributes)]`](#extending-elements) - Extends the props with all the attributes from an element or the global element attributes.

Props also act slightly differently when used with:

- [`Option<T>`](#optional-props) - The field is automatically optional with a default value of `None`.
- [`ReadSignal<T>`](#reactive-props) - The props macro will automatically convert `T` into `ReadSignal<T>` when it is passed as a prop.
- [`String`](#formatted-props) - The props macro will accept formatted strings for any prop field with the type `String`.
- [`children`](#children-props) - The props macro will accept child elements if you include the `children` prop.

### Default Props

The `default` attribute lets you define a default value for a field if it isn't set when creating the component

```rust, no_run
# use dioxus::prelude::*;
#[derive(Props, PartialEq, Clone)]
struct ButtonProps {
    // The default attributes makes your field optional in the component and uses the default value if it is not set.
    #[props(default)]
    text: String,

    /// You can also set an explicit default value instead of using the `Default` implementation.
    #[props(default = "red".to_string())]
    color: String,
}

fn Button(props: ButtonProps) -> Element {
    rsx! {
        button {
            color: props.color,
            "{props.text}"
        }
    }
}

rsx! {
    // You can skip setting props that have a default value when you use the component.
    Button {}
};
```

### Optional Props

When defining props, you may want to make a prop optional without defining an explicit default value. Any fields with the type `Option<T>` are automatically optional with a default value of `None`.

```rust, no_run
# use dioxus::prelude::*;
#[derive(Props, PartialEq, Clone)]
struct ButtonProps {
    // Since the `text` field is optional, you don't need to set it when you use the component.
    text: Option<String>,
}

fn Button(props: ButtonProps) -> Element {
    rsx! {
        button { {props.text.unwrap_or("button".to_string())} }
    }
}

rsx! {
    Button {}
};
```

If you want to make your `Option<T>` field required, you can use the `!optional` attribute:

```rust, no_run
# use dioxus::prelude::*;
#[derive(Props, PartialEq, Clone)]
struct ButtonProps {
    /// You can use the `!optional` attribute on a field with the type `Option<T>` to make it required.
    #[props(!optional)]
    text: Option<String>,
}

fn Button(props: ButtonProps) -> Element {
    rsx! {
        button { {props.text.unwrap_or("button".to_string())} }
    }
}

rsx! {
    Button {
        text: None
    }
};
```

### Converting Props

You can automatically convert a field into the correct type by using the `into` attribute. Any type you pass into the field will be converted with the [`Into`] trait:

```rust, no_run
# use dioxus::prelude::*;
#[derive(Props, PartialEq, Clone)]
struct ButtonProps {
    /// You can use the `into` attribute on a field to convert types you pass in with the Into trait.
    #[props(into)]
    number: u64,
}

fn Button(props: ButtonProps) -> Element {
    rsx! {
        button { "{props.number}" }
    }
}

rsx! {
    Button {
        // Because we used the into attribute, we can pass in any type that implements Into<u64>
        number: 10u8
    }
};
```

### Formatted Props

You can use formatted strings in attributes just like you would in an element. Any prop field with the type `String` can accept a formatted string:

```rust, no_run
# use dioxus::prelude::*;
#[derive(Props, PartialEq, Clone)]
struct ButtonProps {
    text: String,
}

fn Button(props: ButtonProps) -> Element {
    rsx! {
        button { "{props.text}" }
    }
}

let name = "Bob";
rsx! {
    Button {
        // You can use formatted strings in props that accept String just like you would in an element.
        text: "Hello {name}!"
    }
};
```

### Children Props

Rather than passing the RSX through a regular prop, you may wish to accept children similarly to how elements can have children. The "magic" children prop lets you achieve this:

```rust, no_run
# use dioxus::prelude::*;
#[derive(PartialEq, Clone, Props)]
struct ClickableProps {
    href: String,
    children: Element,
}

fn Clickable(props: ClickableProps) -> Element {
    rsx! {
        a {
            href: "{props.href}",
            class: "fancy-button",
            {props.children}
        }
    }
}
```

This makes providing children to the component much simpler: simply put the RSX inside the {} brackets:

```rust, no_run
# use dioxus::prelude::*;
# #[derive(PartialEq, Clone, Props)]
# struct ClickableProps {
#     href: String,
#     children: Element,
# }
#
# fn Clickable(props: ClickableProps) -> Element {
#     rsx! {
#         a {
#             href: "{props.href}",
#             class: "fancy-button",
#             {props.children}
#         }
#     }
# }
rsx! {
    Clickable {
        href: "https://www.youtube.com/watch?v=C-M2hs3sXGo",
        "How to "
        i { "not" }
        " be seen"
    }
};
```

### Reactive Props

In dioxus, when a prop changes, the component will rerun with the new value to update the UI. For example, if count changes from 0 to 1, this component will rerun and update the UI to show "Count: 1":

```rust, no_run
# use dioxus::prelude::*;
#[component]
fn Counter(count: i32) -> Element {
    rsx! {
        div {
            "Count: {count}"
        }
    }
}
```

Generally, just rerunning the component is enough to update the UI. However, if you use your prop inside reactive hooks like `use_memo` or `use_resource`, you may also want to restart those hooks when the prop changes:

```rust, no_run
# use dioxus::prelude::*;
#[component]
fn Counter(count: i32) -> Element {
    // We can use a memo to calculate the doubled count. Since this memo will only be created the first time the component is run and `count` is not reactive, it will never update when `count` changes.
    let doubled_count = use_memo(move || count * 2);
    rsx! {
        div {
            "Count: {count}"
            "Doubled Count: {doubled_count}"
        }
    }
}
```

To fix this issue you can either:

1. Make the prop reactive by wrapping it in `ReadSignal` (recommended):

`ReadSignal` is a `Copy` reactive value. Dioxus will automatically convert any value into a `ReadSignal` when it is passed as a prop.

```rust, no_run
# use dioxus::prelude::*;
#[component]
fn Counter(count: ReadSignal<i32>) -> Element {
    // Since we made count reactive, the memo will automatically rerun when count changes.
    let doubled_count = use_memo(move || count() * 2);
    rsx! {
        div {
            "Count: {count}"
            "Doubled Count: {doubled_count}"
        }
    }
}
```

2. Explicitly add the prop as a dependency to the reactive hook with [`use_reactive`](https://docs.rs/dioxus-hooks/latest/dioxus_hooks/macro.use_reactive.html):

```rust, no_run
# use dioxus::prelude::*;
#[component]
fn Counter(count: i32) -> Element {
    // We can add the count prop as an explicit dependency to every reactive hook that uses it with use_reactive.
    // The use_reactive macro takes a closure with explicit dependencies as its argument.
    let doubled_count = use_memo(use_reactive!(|count| count * 2));
    rsx! {
        div {
            "Count: {count}"
            "Doubled Count: {doubled_count}"
        }
    }
}
```

### Extending Elements

The `extends` attribute lets you extend your props with all the attributes from an element or the global element attributes.

```rust, no_run
# use dioxus::prelude::*;
#[derive(Props, PartialEq, Clone)]
struct CardProps {
    /// You can use the `extends` attribute on a field with the type `Vec<Attribute>` to extend the props with all the attributes from an element or the global element attributes.
    #[props(extends = GlobalAttributes)]
    attributes: Vec<Attribute>,
}

#[component]
fn Card(props: CardProps) -> Element {
    rsx! {
        // Instead of copying over every single attribute, we can just spread the attributes from the props into the element.
        div { ..props.attributes, "card" }
    }
}

rsx! {
    // Since we extend global attributes, you can use any attribute that would normally appear on elements.
    Card {
        width: "10px",
        height: "10px",
        color: "red",
    }
};
```

To extend the props with both the global attributes and the attributes of a specific element, you can use the `extends` attribute multiple times:

```rust, no_run
# use dioxus::prelude::*;

#[derive(Props, PartialEq, Clone)]
struct ButtonProps {
    #[props(extends = GlobalAttributes, extends = button)]
    attributes: Vec<Attribute>,
}

#[component]
fn Button(props: ButtonProps) -> Element {
    rsx! {
        button { ..props.attributes, "button" }
    }
}

rsx! {
    Button {
        // A global attribute
        width: "10px",
        // A button specific attribute
        disabled: true,
    }
};
```

Note that extending from multiple elements will only work if the elements don't have conflicting attributes.
