- Name: `keyword-generic-effect-clause`
- Proposed by: [Caio](@CaioOliveira793)
- Original proposal (optional): N/A

# Design

<!-- Please fill out the snippets labeled with "fill me in". If there are any
other examples you want to show, please feel free to append more.-->

## base (reference)

<!-- This is the snippet which is being translated to various scenarios we're
translating from. Please keep this as-is, so we can reference it later.-->

```rust
/// A trimmed-down version of the `std::Iterator` trait.
pub trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
    fn size_hint(&self) -> (usize, Option<usize>);
}

/// An adaptation of `Iterator::find` to a free-function
pub fn find<I, T, P>(iter: &mut I, predicate: P) -> Option<T>
where
    I: Iterator<Item = T> + Sized,
    P: FnMut(&T) -> bool;
```

## always async

<!-- A variant where all items are always `async` -->

```rust
pub trait Iterator<effect>
where
    effect: async
{
    type Item;

    // opt-in for a async effect
    fn next(&mut self) -> Option<Self::Item>
    where
        effect: async;

    // the size_hint is left unchanged, since the effect is opt-in
    fn size_hint(&self) -> (usize, Option<usize>);
}

// the `effect` generic indicates that this function is generic
// over keywords
pub fn find<I, T, P, effect>(iter: &mut I, predicate: P) -> Option<T>
where
    effect: async,
    // iterator effect **must** be async
    I: Iterator<effect = async>
    // a short version is also possible
    // I: Iterator<effect = async, Item = T> + Sized,
    I: Iterator<Item = T> + Sized,
    P: FnMut(&T) -> bool;
```

## maybe async

<!-- A variant where all items are generic over `async` -->

```rust
pub trait Iterator<effect>
where
    effect: ?async
{
    type Item;

    fn next(&mut self) -> Option<Self::Item>
    where
        effect: ?async;

    fn size_hint(&self) -> (usize, Option<usize>);
}

pub fn find<I, T, P, effect>(iter: &mut I, predicate: P) -> Option<T>
where
    effect: ?async,
    // iterator effect is be **maybe** async
    I: Iterator<effect = ?async>
    I: Iterator<Item = T> + Sized,
    P: FnMut(&T) -> bool;
```

## generic over all modifier keywords

<!-- A variant where all items are generic over all modifier keywords (e.g.
`async`, `const`, `gen`, etc.) -->

```rust
pub trait Iterator<effect>
where
    // in order to be generic over all keywords the effect clause must specify all the keywords available
    effect: ?async + ?const
{
    type Item;

    fn next(&mut self) -> Option<Self::Item>
    where
        effect: ?async + ?const;

    // functions without effect anotatios follows the same rules as if
    // the trait wasn't generic over a keyword
    fn size_hint(&self) -> (usize, Option<usize>);

    // ... or possibilly, they could restrict the keywords allowed
    fn size_hint(&self) -> (usize, Option<usize>)
    where
        effect: !async + ?const;
}

pub fn find<I, T, P, effect>(iter: &mut I, predicate: P) -> Option<T>
where
    effect: ?async + ?const,
    I: Iterator<effect = ?async + ?const, Item = T> + Sized,
    P: FnMut(&T) -> bool;
```

# Notes

## Compatiablity

compatible with [associated type bounds](https://github.com/rust-lang/rust/issues/52662):

```rust
pub fn find<I, T, P, effect>(iter: &mut I, predicate: P) -> Option<T>
where
    effect: ?async + ?const,
    // the choice over ":" or "=" is open
    I: Iterator<effect = ?async + ?const>,
    // or
    <I as Iterator>::effect: ?async + ?const,
    // or
    I: Iterator<effect: ?async + ?const>,
    I: Iterator<Item = T> + Sized,
    P: FnMut(&T) -> bool;
```

is also compatible with [return type notation](https://smallcultfollowing.com/babysteps/blog/2023/02/13/return-type-notation-send-bounds-part-2/)

```rust
pub trait HealthCheck<effect>
where
    effect: async
{
    fn check(&mut self, server: Server)
    where
        effect: async;
}

fn start_health_check<H, effect>(health_check: H, server: Server)
where
    effect: async,
    H: HealthCheck<effect: async, check(): Send> + Send + 'static,
```

## explicit all modifiers keywords

The syntax does not give shortans for specifing all modifiers at once.

Positive

- explicit
- backwards compatible to introduce new keywords
- extendable

Negative

- cumbersome to declare all keywords

<!-- Add additional notes, context, and thoughts you want to share about your design
here -->
