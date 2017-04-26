***NB: This is in real bad shape. Sorry***

# Rust API guidelines

* [Checklist](#checklist)
* [Organization](#organization)
* [Naming](#naming)
* [Interoperability](#interoperability)
* [Macros](#macros)
* [Documentation](#documentation)
* [Predictability](#predictability)
* [Flexibility](#flexibility)
* [Type safety](#type-safety)
* [Dependability](#dependability)
* [Debuggability](#debuggability)
* [Future proofing](#future-proofing)
* [Necessities](#necessities)
* [External links](#external-links)


<a id="checklist"></a>
## Crate conformance checklist

<!--
# Guidelines for the guidelines:

A guideline is an indicative statement about a hypothetical crate.

  - Not an imperative like "Implement Hex, Octal, Binary for binary number types."
    Instead: "Binary number types provide Hex, Octal, Binary formatting."

  - Not an obligation like "Macros should compose well with attributes."
    Instead: "Macros compose well with attributes."

Guidelines have an explicit subject and verb.

  - Not implicit subject like "Includes all common Cargo.toml metadata."
    Instead: "Cargo.toml includes all common metadata."

  - Not implicit verb like "Thoroughly documented with examples."
    Instead: "Crate level docs are thorough and include examples."

  - Not metaphysical like "There are no out-parameters."
    Instead: "Functions do not take out-parameters."

Guidelines use active voice.

  - Not passive voice like "Function arguments are validated."
    Instead: "Functions validate their arguments."
-->

- **Organization** *(crate is structured in an intelligible way)*
  - [ ] Crate root reexports common functionality ([C-REEXPORT])
  - [ ] Modules provide a sensible API hierarchy ([C-HIERARCHY])
- **Naming** *(crate aligns with Rust naming conventions)*
  - [ ] Casing conforms to RFC 430 ([C-CASE])
  - [ ] Ad-hoc conversions follow `as_`, `to_`, `into_` conventions ([C-CONV])
  - [ ] Methods that produce iterators follow `iter`, `iter_mut`, `into_iter` ([C-ITER])
  - [ ] Iterator type names match the methods that produce them ([C-ITER-TY])
  - [ ] Ownership suffixes use `_mut` and `_ref` ([C-OWN-SUFFIX])
  - [ ] Single-element containers implement appropriate getters ([C-GETTERS])
- **Interoperability** *(crate interacts nicely with other library functionality)*
  - [ ] Types eagerly implement common traits ([C-COMMON-TRAITS])
    - `Copy`, `Clone`, `Eq`, `PartialEq`, `Ord`, `PartialOrd`, `Hash` `Debug`,
      `Display`, `Default`
  - [ ] Conversions use the standard traits `From`, `AsRef`, `AsMut` ([C-CONV-TRAITS])
  - [ ] Collections implement `FromIterator` and `Extend` ([C-COLLECT])
  - [ ] Data structures implement Serde's `Serialize`, `Deserialize` ([C-SERDE])
  - [ ] Crate has a `"serde"` cfg option that enables Serde ([C-SERDE-CFG])
  - [ ] Types are `Send` and `Sync` where possible ([C-SEND-SYNC])
  - [ ] Error types are `Send` and `Sync` ([C-SEND-SYNC-ERR])
  - [ ] Binary number types provide `Hex`, `Octal`, `Binary` formatting ([C-NUM-FMT])
- **Macros** *(crate presents well-behaved macros)*
  - [ ] Input syntax is evocative of the output ([C-EVOCATIVE])
  - [ ] Macros compose well with attributes ([C-MACRO-ATTR])
  - [ ] Item macros work anywhere that items are allowed ([C-ANYWHERE])
  - [ ] Item macros support visibility specifiers ([C-MACRO-VIS])
  - [ ] Type fragments are flexible ([C-MACRO-TY])
- **Documentation** *(crate is abundantly documented)*
  - [ ] Crate level docs are thorough and include examples ([C-CRATE-DOC])
  - [ ] All items have a rustdoc example ([C-EXAMPLE])
  - [ ] Examples use `?`, not `try!`, not `unwrap` ([C-QUESTION-MARK])
  - [ ] Function docs include error conditions in "Errors" section ([C-ERROR-DOC])
  - [ ] Function docs include panic conditions in "Panics" section ([C-PANIC-DOC])
  - [ ] Prose contains hyperlinks to relevant things ([C-LINK])
  - [ ] Cargo.toml publishes CI badges for tier 1 platforms ([C-CI])
  - [ ] Cargo.toml includes all common metadata ([C-METADATA])
    - authors, description, license, homepage, documentation, repository,
      readme, keywords, categories
  - [ ] Crate sets html_root_url attribute "https://docs.rs/$crate/$version" ([C-HTML-ROOT])
  - [ ] Cargo.toml documentation key points to "https://docs.rs/$crate" ([C-DOCS-RS])
- **Predictability** *(crate enables legible code that acts how it looks)*
  - [ ] Smart pointers do not add inherent methods ([C-SMART-PTR])
  - [ ] Conversions live on the most specific type involved ([C-CONV-SPECIFIC])
  - [ ] Functions with a clear receiver are methods ([C-METHOD])
  - [ ] Functions do not take out-parameters ([C-NO-OUT])
  - [ ] Operator overloads are unsurprising ([C-OVERLOAD])
  - [ ] Only smart pointers implement `Deref` and `DerefMut` ([C-DEREF])
  - [ ] `Deref` and `DerefMut` never fail ([C-DEREF-FAIL])
  - [ ] Constructors are static, inherent methods ([C-CTOR])
  - [ ] Constructors are available for passive `struct`s with defaults ([C-EMPTY-CTOR])
- **Flexibility** *(crate supports diverse real-world use cases)*
  - [ ] Functions expose intermediate results to avoid duplicate work ([C-INTERMEDIATE])
  - [ ] Caller decides where to copy and place data ([C-CALLER-CONTROL])
  - [ ] Functions minimize assumptions about parameters by using generics ([C-ASSUMPTION])
  - [ ] Arguments prefer passing by reference ([C-BY-REF])
  - [ ] Traits are object-safe if they may be useful as a trait object ([C-OBJ-SAFE])
  - [ ] Functions use trait-bounded generics instead of virtual dispatch ([C-GENERIC])
  - [ ] Functions accept trait objects in place of generics ([C-OBJECT])
- **Type safety** *(crate leverages the type system effectively)*
  - [ ] Newtypes provide static distinctions ([C-NEWTYPE])
  - [ ] Arguments convey meaning through types, not `bool` or `Option` ([C-CUSTOM-TYPE])
  - [ ] Types for a set of flags are `bitflags`, not enums ([C-BITFLAG])
  - [ ] Builders enable construction of complex values ([C-BUILDER])
- **Dependability** *(crate is unlikely to do the wrong thing)*
  - [ ] Functions validate their arguments ([C-VALIDATE])
  - [ ] Destructors never fail ([C-DTOR-FAIL])
  - [ ] Destructors that may block have alternatives ([C-DTOR-BLOCK])
- **Debuggability** *(crate is conducive to easy debugging)*
  - [ ] All public types implement `Debug` ([C-DEBUG])
  - [ ] `Debug` representation is never empty ([C-DEBUG-NONEMPTY])
- **Future proofing** *(crate is free to improve without breaking users' code)*
  - [ ] Structs have private fields ([C-STRUCT-PRIVATE])
  - [ ] Newtypes encapsulate implementation details ([C-NEWTYPE-HIDE])
- **Necessities** *(to whom they matter, they really matter)*
  - [ ] Public dependencies of a stable crate are stable ([C-STABLE])
  - [ ] Crate and its dependencies have a permissive license ([C-PERMISSIVE])


<a id="organization"></a>
## Organization

[C-REEXPORT]: #c-reexport
<a id="c-reexport"></a>
### Crate root reexports common functionality (C-REEXPORT)

Crates `pub use` the most common types for convenience, so that clients do not
have to remember or write the crate's module hierarchy to use these types.

[C-HIERARCHY]: #c-hierarchy
<a id="c-hierarchy"></a>
### Modules provide a sensible API hierarchy (C-HIERARCHY)


<a id="naming"></a>
## Naming

[C-CASE]: #c-case
<a id="c-case"></a>
### Casing conforms to RFC 430 (C-CASE)

Basic Rust naming conventions are described in [RFC 430].

In general, Rust tends to use `CamelCase` for "type-level" constructs (types and
traits) and `snake_case` for "value-level" constructs. More precisely:

| Item | Convention |
| ---- | ---------- |
| Crates | [unclear](https://github.com/brson/rust-api-guidelines/issues/29) |
| Modules | `snake_case` |
| Types | `CamelCase` |
| Traits | `CamelCase` |
| Enum variants | `CamelCase` |
| Functions | `snake_case` |
| Methods | `snake_case` |
| General constructors | `new` or `with_more_details` |
| Conversion constructors | `from_some_other_type` |
| Local variables | `snake_case` |
| Static variables | `SCREAMING_SNAKE_CASE` |
| Constant variables | `SCREAMING_SNAKE_CASE` |
| Type parameters | concise `CamelCase`, usually single uppercase letter: `T` |
| Lifetimes | short, lowercase: `'a` |

In `CamelCase`, acronyms count as one word: use `Uuid` rather than `UUID`. In
`snake_case`, acronyms are lower-cased: `is_xid_start`.

In `snake_case` or `SCREAMING_SNAKE_CASE`, a "word" should never consist of a
single letter unless it is the last "word". So, we have `btree_map` rather than
`b_tree_map`, but `PI_2` rather than `PI2`.

[C-CONV]: #c-conv
<a id="c-conv"></a>
### Ad-hoc conversions follow `as_`, `to_`, `into_` conventions (C-CONV)

Conversions should be provided as methods, with names prefixed as follows:

| Prefix | Cost | Consumes convertee |
| ------ | ---- | ------------------ |
| `as_` | Free | No |
| `to_` | Expensive | No |
| `into_` | Variable | Yes |

For example:

- [`as_bytes()`] gives a `&[u8]` view into a `&str`, which is a no-op.
- [`to_owned()`] copies a `&str` to a new `String`.
- [`into_bytes()`] consumes a `String` and yields the underlying `Vec<u8>`,
  which is a no-op.

[`as_bytes()`]: https://doc.rust-lang.org/std/primitive.str.html#method.as_bytes
[`to_owned()`]: https://doc.rust-lang.org/std/primitive.str.html#method.to_owned
[`into_bytes()`]: https://doc.rust-lang.org/std/string/struct.String.html#method.into_bytes

Conversions prefixed `as_` and `into_` typically _decrease abstraction_, either
exposing a view into the underlying representation (`as`) or deconstructing data
into its underlying representation (`into`). Conversions prefixed `to_`, on the
other hand, typically stay at the same level of abstraction but do some work to
change one representation into another.

More examples:

- [`Result::as_ref`](https://doc.rust-lang.org/std/result/enum.Result.html#method.as_ref)
- [`RefCell::as_ptr`](https://doc.rust-lang.org/std/cell/struct.RefCell.html#method.as_ptr)
- [`Path::to_str`](https://doc.rust-lang.org/std/path/struct.Path.html#method.to_str)
- [`slice::to_vec`](https://doc.rust-lang.org/std/primitive.slice.html#method.to_vec)
- [`Option::into_iter`](https://doc.rust-lang.org/std/option/enum.Option.html#method.into_iter)
- [`AtomicBool::into_inner`](https://doc.rust-lang.org/std/sync/atomic/struct.AtomicBool.html#method.into_inner)

[C-ITER]: #c-iter
<a id="c-iter"></a>
### Methods that produce iterators follow `iter`, `iter_mut`, `into_iter` (C-ITER)

Per [RFC 199].

For a container with elements of type `U`, iterator methods should be named:

```rust
fn iter(&self) -> Iter             // Iter implements Iterator<Item = &U>
fn iter_mut(&mut self) -> IterMut  // IterMut implements Iterator<Item = &mut U>
fn into_iter(self) -> IntoIter     // IntoIter implements Iterator<Item = U>
```

The default iterator variant yields shared references `&U`.

[C-ITER-TY]: #c-iter-ty
<a id="c-iter-ty"></a>
### Iterator type names match the methods that produce them (C-ITER-TY)

For example:

* [`iter`][Vec::iter()] should yield an [`Iter`][slice::Iter]
* [`iter_mut`][Vec::iter_mut()] should yield an [`IterMut`][slice::IterMut]
* [`into_iter`][Vec::into_iter()] should yield an [`IntoIter`][vec::IntoIter]
* [`keys`][BTreeMap::keys()] should yield [`Keys`][btree_map::Keys]

[Vec::iter()]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.iter
[slice::Iter]: https://doc.rust-lang.org/std/slice/struct.Iter.html
[Vec::iter_mut()]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.iter_mut
[slice::IterMut]: https://doc.rust-lang.org/std/slice/struct.IterMut.html
[Vec::into_iter()]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.into_iter
[vec::IntoIter]: https://doc.rust-lang.org/std/vec/struct.IntoIter.html
[BTreeMap::keys()]: https://doc.rust-lang.org/std/collections/struct.BTreeMap.html#method.keys
[btree_map::Keys]: https://doc.rust-lang.org/std/collections/btree_map/struct.Keys.html

These type names make the most sense when prefixed with their owning module,
e.g. [`vec::IntoIter`].

[`vec::IntoIter`]: https://doc.rust-lang.org/std/vec/struct.IntoIter.html

[C-OWN-SUFFIX]: #c-own-suffix
<a id="c-own-suffix"></a>
### Ownership suffixes use `_mut`, `_ref` (C-OWN-SUFFIX)

Functions often come in multiple variants: immutably borrowed, mutably borrowed,
and owned.

The right default depends on the function in question. Variants should be marked
through suffixes.

#### Exceptions

In the case of iterators, the moving variant can also be understood as an `into`
conversion, `into_iter`, and `for x in v.into_iter()` reads arguably better than
`for x in v.iter_move()`, so the convention is `into_iter`.

For mutably borrowed variants, if the `mut` qualifier is part of a type name
(e.g. [`as_mut_slice`]), it should appear as it would appear in the type.

[`as_mut_slice`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.as_mut_slice

#### Immutably borrowed by default

If `foo` uses/produces an immutable borrow by default, use:

* The `_mut` suffix (e.g. `foo_mut`) for the mutably borrowed variant.
* The `_move` suffix (e.g. `foo_move`) for the owned variant.

#### Owned by default

If `foo` uses/produces owned data by default, use:

* The `_ref` suffix (e.g. `foo_ref`) for the immutably borrowed variant.
* The `_mut` suffix (e.g. `foo_mut`) for the mutably borrowed variant.

[C-GETTERS]: #c-getters
<a id="c-getters"></a>
### Single-element containers implement appropriate getters (C-GETTERS)

Single-element contains where accessing the element cannot fail should implement
`get` and `get_mut`, with the signatures

```rust
fn get(&self) -> &V;
fn get_mut(&mut self) -> &mut V;
```

Some examples:

- [`std::io::Cursor`](https://doc.rust-lang.org/std/io/struct.Cursor.html#method.get_mut)
- [`std::ptr::Unique`](https://doc.rust-lang.org/std/ptr/struct.Unique.html#method.get_mut)
- [`std::sync::PoisonError`](https://doc.rust-lang.org/std/sync/struct.PoisonError.html#method.get_mut)
- [`std::sync::atomic::AtomicBool`](https://doc.rust-lang.org/std/sync/atomic/struct.AtomicBool.html#method.get_mut)
- [`std::collections::hash_map::OccupiedEntry`](https://doc.rust-lang.org/std/collections/hash_map/struct.OccupiedEntry.html#method.get_mut)

Single-element containers where the element is [`Copy`] (e.g. [`Cell`]-like
containers) should instead return the value directly, and not implement a
mutable accessor:

[`Copy`]: https://doc.rust-lang.org/std/marker/trait.Copy.html
[`Cell`]: https://doc.rust-lang.org/std/cell/struct.Cell.html

```rust
fn get(&self) -> V;
```

For getters that do runtime validation, consider adding unsafe `_unchecked`
variants:

```rust
unsafe fn get_unchecked(&self, index) -> &V;
```

An example of this is [`<[_]>::get_unchecked`].

[`<[_]>::get_unchecked`]: https://doc.rust-lang.org/std/primitive.slice.html#method.get_unchecked


<a id="interoperability"></a>
## Interoperability

[C-COMMON-TRAITS]: #c-common-traits
<a id="c-common-traits"></a>
### Types eagerly implement common traits (C-COMMON-TRAITS)

Rust's trait system does not allow _orphans_: roughly, every `impl` must live
either in the crate that defines the trait or the implementing type.
Consequently, crates that define new types should eagerly implement all
applicable, common traits.

To see why, consider the following situation:

* Crate `std` defines trait `Display`.
* Crate `url` defines type `Url`, without implementing `Display`.
* Crate `webapp` imports from both `std` and `url`,

There is no way for `webapp` to add `Display` to `url`, since it defines
neither. (Note: the newtype pattern can provide an efficient, but inconvenient
workaround.

The most important common traits to implement from `std` are:

- [`Copy`](https://doc.rust-lang.org/std/marker/trait.Copy.html)
- [`Clone`](https://doc.rust-lang.org/std/clone/trait.Clone.html)
- [`Eq`](https://doc.rust-lang.org/std/cmp/trait.Eq.html)
- [`PartialEq`](https://doc.rust-lang.org/std/cmp/trait.PartialEq.html)
- [`Ord`](https://doc.rust-lang.org/std/cmp/trait.Ord.html)
- [`PartialOrd`](https://doc.rust-lang.org/std/cmp/trait.PartialOrd.html)
- [`Hash`](https://doc.rust-lang.org/std/hash/trait.Hash.html)
- [`Debug`](https://doc.rust-lang.org/std/fmt/trait.Debug.html)
- [`Display`](https://doc.rust-lang.org/std/fmt/trait.Display.html)
- [`Default`](https://doc.rust-lang.org/std/default/trait.Default.html)

[C-CONV-TRAITS]: #c-conv-traits
<a id="c-conv-traits"></a>
### Conversions use the standard traits `From`, `AsRef`, `AsMut` (C-CONV-TRAITS)

The following conversion traits should be implemented where it makes sense:

- [`From`](https://doc.rust-lang.org/std/convert/trait.From.html)
- [`TryFrom`](https://doc.rust-lang.org/std/convert/trait.TryFrom.html)
- [`AsRef`](https://doc.rust-lang.org/std/convert/trait.AsRef.html)
- [`AsMut`](https://doc.rust-lang.org/std/convert/trait.AsMut.html)

The following conversion traits should never be implemented:

- [`Into`](https://doc.rust-lang.org/std/convert/trait.Into.html)
- [`TryInto`](https://doc.rust-lang.org/std/convert/trait.TryInto.html)

These traits have a blanket impl based on `From` and `TryFrom`. Implement those
instead.

[C-COLLECT]: #c-collect
<a id="c-collect"></a>
### Collections implement `FromIterator` and `Extend` (C-COLLECT)

[C-SERDE]: #c-serde
<a id="c-serde"></a>
### Data structures implement Serde's `Serialize`, `Deserialize` (C-SERDE)

Types that play the role of a data structure should implement [`Serialize`] and
[`Deserialize`].

An example of a type that plays the role of a data structure is
[`linked_hash_map::LinkedHashMap`].

An example of a type that does not play the role of a data structure is
[`byteorder::LittleEndian`].

[`Serialize`]: https://docs.serde.rs/serde/trait.Serialize.html
[`Deserialize`]: https://docs.serde.rs/serde/trait.Deserialize.html
[`byteorder::LittleEndian`]: https://docs.rs/byteorder/1.0.0/byteorder/enum.LittleEndian.html
[`linked_hash_map::LinkedHashMap`]: https://docs.rs/linked-hash-map/0.4.2/linked_hash_map/struct.LinkedHashMap.html

[C-SERDE-CFG]: #c-serde-cfg
<a id="c-serde-cfg"></a>
### Crate has a `"serde"` cfg option that enables Serde (C-SERDE-CFG)

If the crate relies on `serde_derive` to provide Serde impls, the name of the
cfg can still be simply `"serde"` by using [this workaround]. Do not use a
different name for the cfg like `"serde_impls"` or `"serde_serialization"`.

[this workaround]: https://github.com/serde-rs/serde/blob/v1.0.0/serde/src/lib.rs#L222-L260

[C-SEND-SYNC]: #c-send-sync
<a id="c-send-sync"></a>
### Types are `Send` and `Sync` where possible (C-SEND-SYNC)

[C-SEND-SYNC-ERR]: #c-send-sync-err
<a id="c-send-sync-err"></a>
### Error types are `Send` and `Sync` (C-SEND-SYNC-ERR)

[C-NUM-FMT]: #c-num-fmt
<a id="c-num-fmt"></a>
### Binary number types provide `Hex`, `Octal`, `Binary` formatting (C-NUM-FMT)

- [`std::fmt::UpperHex`](https://doc.rust-lang.org/std/fmt/trait.UpperHex.html)
- [`std::fmt::LowerHex`](https://doc.rust-lang.org/std/fmt/trait.LowerHex.html)
- [`std::fmt::Octal`](https://doc.rust-lang.org/std/fmt/trait.Octal.html)
- [`std::fmt::Binary`](https://doc.rust-lang.org/std/fmt/trait.Binary.html)

These traits control the representation of a type under the `{:X}`, `{:x}`,
`{:o}`, and `{:b}` format specifiers.

Implement these traits for any number type on which you would consider doing
bitwise manipulations like `|` or `&`. This is especially appropriate for
bitflag types. Numeric quantity types like `struct Nanoseconds(u64)` probably do
not need these.


<a id="macros"></a>
## Macros

[C-EVOCATIVE]: #c-evocative
<a id="c-evocative"></a>
### Input syntax is evocative of the output (C-EVOCATIVE)

Rust macros let you dream up practically whatever input syntax you want. Aim to
keep input syntax familiar and cohesive with the rest of your users' code by
mirroring existing Rust syntax where possible. Pay attention to the choice and
placement of keywords and punctuation.

A good guide is to use syntax, especially keywords and punctuation, that is
similar to what will be produced in the output of the macro.

For example if your macro declares a struct with a particular name given in the
input, preface the name with the keyword `struct` to signal to readers that a
struct is being declared with the given name.

```rust
// Prefer this...
bitflags! {
    struct S: u32 { /* ... */ }
}

// ...over no keyword...
bitflags! {
    S: u32 { /* ... */ }
}

// ...or some ad-hoc word.
bitflags! {
    flags S: u32 { /* ... */ }
}
```

Another example is semicolons vs commas. Constants in Rust are followed by
semicolons so if your macro declares a chain of constants, they should likely be
followed by semicolons even if the syntax is otherwise slightly different from
Rust's.

```rust
// Ordinary constants use semicolons.
const A: u32 = 0b000001;
const B: u32 = 0b000010;

// So prefer this...
bitflags! {
    struct S: u32 {
        const C = 0b000100;
        const D = 0b001000;
    }
}

// ...over this.
bitflags! {
    struct S: u32 {
        const E = 0b010000,
        const F = 0b100000,
    }
}
```

Macros are so diverse that these specific examples won't be relevant, but think
about how to apply the same principles to your situation.

[C-MACRO-ATTR]: #c-macro-attr
<a id="c-macro-attr"></a>
### Item macros compose well with attributes (C-MACRO-ATTR)

Macros that produce more than one output item should support adding attributes
to any one of those items. One common use case would be putting individual items
behind a cfg.

```rust
bitflags! {
    struct Flags: u8 {
        #[cfg(windows)]
        const ControlCenter = 0b001;
        #[cfg(unix)]
        const Terminal = 0b010;
    }
}
```

Macros that produce a struct or enum as output should support attributes so that
the output can be used with derive.

```rust
bitflags! {
    #[derive(Default, Serialize)]
    struct Flags: u8 {
        const ControlCenter = 0b001;
        const Terminal = 0b010;
    }
}
```

[C-ANYWHERE]: #c-anywhere
<a id="c-anywhere"></a>
### Item macros work anywhere that items are allowed (C-ANYWHERE)

Rust allows items to be placed at the module level or within a tighter scope
like a function. Item macros should work equally well as ordinary items in all
of these places. The test suite should include invocations of the macro in at
least the module scope and function scope.

```rust
#[cfg(test)]
mod tests {
    test_your_macro_in_a!(module);

    #[test]
    fn anywhere() {
        test_your_macro_in_a!(function);
    }
}
```

As a simple example of how things can go wrong, this macro works great in a
module scope but fails in a function scope.

```rust
macro_rules! broken {
    ($m:ident :: $t:ident) => {
        pub struct $t;
        pub mod $m {
            pub use super::$t;
        }
    }
}

broken!(m::T); // okay, expands to T and m::T

fn g() {
    broken!(m::U); // fails to compile, super::U refers to the containing module not g
}
```

[C-MACRO-VIS]: #c-macro-vis
<a id="c-macro-vis"></a>
### Item macros support visibility specifiers (C-MACRO-VIS)

Follow Rust syntax for visibility of items produced by a macro. Private by
default, public if `pub` is specified.

```rust
bitflags! {
    struct PrivateFlags: u8 {
        const A = 0b0001;
        const B = 0b0010;
    }
}

bitflags! {
    pub struct PublicFlags: u8 {
        const C = 0b0100;
        const D = 0b1000;
    }
}
```

[C-MACRO-TY]: #c-macro-ty
<a id="c-macro-ty"></a>
### Type fragments are flexible (C-MACRO-TY)

If your macro accepts a type fragment like `$t:ty` in the input, it should be
usable with all of the following:

- Primitives: `u8`, `&str`
- Relative paths: `m::Data`
- Absolute paths: `::base::Data`
- Upward relative paths: `super::Data`
- Generics: `Vec<String>`

As a simple example of how things can go wrong, this macro works great with
primitives and absolute paths but fails with relative paths.

```rust
macro_rules! broken {
    ($m:ident => $t:ty) => {
        pub mod $m {
            pub struct Wrapper($t);
        }
    }
}

broken!(a => u8); // okay

broken!(b => ::std::marker::PhantomData<()>); // okay

struct S;
broken!(c => S); // fails to compile
```


<a id="documentation"></a>
## Documentation

[C-CRATE-DOC]: #c-crate-doc
<a id="c-crate-doc"></a>
### Crate level docs are thorough and include examples (C-CRATE-DOC)

See [RFC 1687].

[C-EXAMPLE]: #c-example
<a id="c-example"></a>
### All items have a rustdoc example (C-EXAMPLE)

[C-QUESTION-MARK]: #c-question-mark
<a id="c-question-mark"></a>
### Examples use `?`, not `try!`, not `unwrap` (C-QUESTION-MARK)

See [RFC 1574].

[C-ERROR-DOC]: #c-error-doc
<a id="c-error-doc"></a>
### Function docs include error conditions in "Errors" section (C-ERROR-DOC)

See [RFC 1574].

[C-PANIC-DOC]: #c-panic-doc
<a id="c-panic-doc"></a>
### Function docs include panic conditions in "Panics" section (C-PANIC-DOC)

See [RFC 1574].

This applies to trait methods as well. Traits methods for which the
implementation is allowed or expected to panic should be documented with a
"Panics" section.

[C-LINK]: #c-link
<a id="c-link"></a>
### Prose contains hyperlinks to relevant things (C-LINK)

[C-CI]: #c-ci
<a id="c-ci"></a>
### Cargo.toml publishes CI badges for tier 1 platforms (C-CI)

[C-METADATA]: #c-metadata
<a id="c-metadata"></a>
### Cargo.toml includes all common metadata (C-METADATA)

- `authors`
- `description`
- `license`
- `homepage` (though see [rust-api-guidelines#26](https://github.com/brson/rust-api-guidelines/issues/26))
- `documentation`
- `repository`
- `readme`
- `keywords`
- `categories`

[C-HTML-ROOT]: #c-html-root
<a id="c-html-root"></a>
### Crate sets html_root_url attribute (C-HTML-ROOT)

It should point to `"https://docs.rs/$crate/$version"`.

Cargo.toml should contain a note next to the version to remember to bump the
`html_root_url` when bumping the crate version.

[C-DOCS-RS]: #c-docs-rs
<a id="c-docs-rs"></a>
### Cargo.toml documentation key points to docs.rs (C-DOCS-RS)

It should point to `"https://docs.rs/$crate"`.


<a id="predictability"></a>
## Predictability

[C-SMART-PTR]: #c-smart-ptr
<a id="c-smart-ptr"></a>
### Smart pointers do not add inherent methods (C-SMART-PTR)

[C-CONV-SPECIFIC]: #c-conv-specific
<a id="c-conv-specific"></a>
### Conversions live on the most specific type involved (C-CONV-SPECIFIC)

When in doubt, prefer `to_`/`as_`/`into_` to `from_`, because they are more
ergonomic to use (and can be chained with other methods).

For many conversions between two types, one of the types is clearly more
"specific": it provides some additional invariant or interpretation that is not
present in the other type. For example, `str` is more specific than `&[u8]`,
since it is a UTF-8 encoded sequence of bytes.

Conversions should live with the more specific of the involved types. Thus,
`str` provides both the `as_bytes` method and the `from_utf8` constructor for
converting to and from `&[u8]` values. Besides being intuitive, this convention
avoids polluting concrete types like `&[u8]` with endless conversion methods.

[C-METHOD]: #c-method
<a id="c-method"></a>
### Functions with a clear receiver are methods (C-METHOD)

Prefer

```rust
impl Foo {
    pub fn frob(&self, w: widget) { ... }
}
```

over

```rust
pub fn frob(foo: &Foo, w: widget) { ... }
```

for any operation that is clearly associated with a particular type.

Methods have numerous advantages over functions:

* They do not need to be imported or qualified to be used: all you need is a
  value of the appropriate type.
* Their invocation performs autoborrowing (including mutable borrows).
* They make it easy to answer the question "what can I do with a value of type
  `T`" (especially when using rustdoc).
* They provide `self` notation, which is more concise and often more clearly
  conveys ownership distinctions.

[C-NO-OUT]: #c-no-out
<a id="c-no-out"></a>
### Functions do not take out-parameters (C-NO-OUT)

Prefer

```rust
fn foo() -> (Bar, Bar)
```

over

```rust
fn foo(output: &mut Bar) -> Bar
```

for returning multiple `Bar` values.

Compound return types like tuples and structs are efficiently compiled and do
not require heap allocation. If a function needs to return multiple values, it
should do so via one of these types.

The primary exception: sometimes a function is meant to modify data that the
caller already owns, for example to re-use a buffer:

```rust
fn read(&mut self, buf: &mut [u8]) -> IoResult<uint>
```

[C-OVERLOAD]: #c-overload
<a id="c-overload"></a>
### Operator overloads are unsurprising (C-OVERLOAD)

Operators with built in syntax (`*`, `|`, and so on) can be provided for a type
by implementing the traits in `core::ops`. These operators come with strong
expectations: implement `Mul` only for an operation that bears some resemblance
to multiplication (and shares the expected properties, e.g. associativity), and
so on for the other traits.

[C-DEREF]: #c-deref
<a id="c-deref"></a>
### Only smart pointers implement `Deref` and `DerefMut` (C-DEREF)

The `Deref` traits are used implicitly by the compiler in many circumstances,
and interact with method resolution. The relevant rules are designed
specifically to accommodate smart pointers, and so the traits should be used
only for that purpose.

[C-DEREF-FAIL]: #c-deref-fail
<a id="c-deref-fail"></a>
### `Deref` and `DerefMut` never fail (C-DEREF-FAIL)

Because the `Deref` traits are invoked implicitly by the compiler in sometimes
subtle ways, failure during dereferencing can be extremely confusing. If a
dereference might not succeed, target the `Deref` trait as a `Result` or
`Option` type instead.

[C-CTOR]: #c-ctor
<a id="c-ctor"></a>
### Constructors are static, inherent methods (C-CTOR)

In Rust, "constructors" are just a convention:

```rust
impl<T> Vec<T> {
    pub fn new() -> Vec<T> { ... }
}
```

Constructors are static (no `self`) inherent methods for the type that they
construct. Combined with the practice of fully importing type names, this
convention leads to informative but concise construction:

```rust
use vec::Vec;

// construct a new vector
let mut v = Vec::new();
```

This convention also applied to conversion constructors (prefix `from` rather
than `new`).

[C-EMPTY-CTOR]: #c-empty-ctor
<a id="c-empty-ctor"></a>
### Constructors are available for passive `struct`s with defaults (C-EMPTY-CTOR)

Given the `struct`

```rust
pub struct Config {
    pub color: Color,
    pub size: Size,
    pub shape: Shape,
}
```

provide a constructor if there are sensible defaults:

```rust
impl Config {
    pub fn new() -> Config {
        Config {
            color: Brown,
            size: Medium,
            shape: Square,
        }
    }
}
```

which then allows clients to concisely override using `struct` update syntax:

```rust
Config { color: Red, .. Config::new() };
```


<a id="flexibility"></a>
## Flexibility

[C-INTERMEDIATE]: #c-intermediate
<a id="c-intermediate"></a>
### Functions expose intermediate results to avoid duplicate work (C-INTERMEDIATE)

Many functions that answer a question also compute interesting related data. If
this data is potentially of interest to the client, consider exposing it in the
API.

Prefer

```rust
struct SearchResult {
    found: bool,          // item in container?
    expected_index: uint  // what would the item's index be?
}

fn binary_search(&self, k: Key) -> SearchResult
```
or

```rust
fn binary_search(&self, k: Key) -> (bool, uint)
```

over

```rust
fn binary_search(&self, k: Key) -> bool
```

#### Yield back ownership:

Prefer

```rust
fn from_utf8_owned(vv: Vec<u8>) -> Result<String, Vec<u8>>
```

over

```rust
fn from_utf8_owned(vv: Vec<u8>) -> Option<String>
```

The `from_utf8_owned` function gains ownership of a vector. In the successful
case, the function consumes its input, returning an owned string without
allocating or copying. In the unsuccessful case, however, the function returns
back ownership of the original slice.

[C-CALLER-CONTROL]: #c-caller-control
<a id="c-caller-control"></a>
### Caller decides where to copy and place data (C-CALLER-CONTROL)

Prefer

```rust
fn foo(b: Bar) {
   // use b as owned, directly
}
```

over

```rust
fn foo(b: &Bar) {
    let b = b.clone();
    // use b as owned after cloning
}
```

If a function requires ownership of a value of unknown type `T`, but does not
otherwise need to make copies, the function should take ownership of the
argument (pass by value `T`) rather than using `.clone()`. That way, the caller
can decide whether to relinquish ownership or to `clone`.

Similarly, the `Copy` trait bound should only be demanded it when absolutely
needed, not as a way of signaling that copies should be cheap to make.

Prefer

```rust
fn foo(b: Bar) -> Bar { ... }
```

over

```rust
fn foo(b: Box<Bar>) -> Box<Bar> { ... }
```

[C-ASSUMPTION]: #c-assumption
<a id="c-assumption"></a>
### Functions minimize assumptions about parameters by using generics (C-ASSUMPTION)

The fewer assumptions a function makes about its inputs, the more widely usable
it becomes.

Prefer

```rust
fn foo<T: Iterator<int>>(c: T) { ... }
```

over any of

```rust
fn foo(c: &[int]) { ... }
fn foo(c: &Vec<int>) { ... }
fn foo(c: &SomeOtherCollection<int>) { ... }
```

if the function only needs to iterate over the data.

More generally, consider using generics to pinpoint the assumptions a function
needs to make about its arguments.

On the other hand, generics can make it more difficult to read and understand a
function's signature. Aim for "natural" parameter types that a neither overly
concrete nor overly abstract.

[C-BY-REF]: #c-by-ref
<a id="c-by-ref"></a>
### Arguments prefer passing by reference (C-BY-REF)

Prefer either of

```rust
fn foo(b: &Bar) { ... }
fn foo(b: &mut Bar) { ... }
```

over

```rust
fn foo(b: Bar) { ... }
```

That is, prefer borrowing arguments rather than transferring ownership, unless
ownership is actually needed.

[C-OBJ-SAFE]: #c-obj-safe
<a id="c-obj-safe"></a>
### Traits are object-safe if they may be useful as a trait object (C-OBJ-SAFE)

Trait objects have some significant limitations: methods invoked through a trait
object cannot use generics, and cannot use `Self` except in receiver position.

When designing a trait, decide early on whether the trait will be used as an
object or as a bound on generics.

If a trait is meant to be used as an object, its methods should take and return
trait objects rather than use generics.

[C-GENERIC]: #c-generic
<a id="c-generic"></a>
### Functions use trait-bounded generics instead of virtual dispatch (C-GENERIC)

The most widespread use of traits is for writing generic functions or types. For
example, the following signature describes a function for consuming any iterator
yielding items of type `A` to produce a collection of `A`:

```rust
fn from_iter<T: Iterator<A>>(iterator: T) -> SomeCollection<A>
```

Here, the `Iterator` trait is specifies an interface that a type `T` must
explicitly implement to be used by this generic function.

**Pros**:

* _Reusability_. Generic functions can be applied to an open-ended collection of
  types, while giving a clear contract for the functionality those types must
  provide.
* _Static dispatch and optimization_. Each use of a generic function is
  specialized ("monomorphized") to the particular types implementing the trait
  bounds, which means that (1) invocations of trait methods are static, direct
  calls to the implementation and (2) the compiler can inline and otherwise
  optimize these calls.
* _Inline layout_. If a `struct` and `enum` type is generic over some type
  parameter `T`, values of type `T` will be laid out _inline_ in the
  `struct`/`enum`, without any indirection.
* _Inference_. Since the type parameters to generic functions can usually be
  inferred, generic functions can help cut down on verbosity in code where
  explicit conversions or other method calls would usually be necessary.
* _Precise types_. Because generic give a _name_ to the specific type
  implementing a trait, it is possible to be precise about places where that
  exact type is required or produced. For example, a function

  ```rust
  fn binary<T: Trait>(x: T, y: T) -> T
  ```

  is guaranteed to consume and produce elements of exactly the same type `T`; it
  cannot be invoked with parameters of different types that both implement
  `Trait`.

**Cons**:

* _Code size_. Specializing generic functions means that the function body is
  duplicated. The increase in code size must be weighed against the performance
  benefits of static dispatch.
* _Homogeneous types_. This is the other side of the "precise types" coin: if
  `T` is a type parameter, it stands for a _single_ actual type. So for example
  a `Vec<T>` contains elements of a single concrete type (and, indeed, the
  vector representation is specialized to lay these out in line). Sometimes
  heterogeneous collections are useful; see [trait objects][C-OBJECT].
* _Signature verbosity_. Heavy use of generics can bloat function signatures.

[C-OBJECT]: #c-object
<a id="c-object"></a>
### Functions accept trait objects in place of generics (C-OBJECT)

Trait objects are useful primarily when _heterogeneous_ collections of objects
need to be treated uniformly; it is the closest that Rust comes to
object-oriented programming.

```rust
struct Frame { ... }
struct Button { ... }
struct Label { ... }

trait Widget { ... }

impl Widget for Frame { ... }
impl Widget for Button { ... }
impl Widget for Label { ... }

impl Frame {
    fn new(contents: &[Box<Widget>]) -> Frame {
        ...
    }
}

fn make_gui() -> Box<Widget> {
    let b: Box<Widget> = box Button::new(...);
    let l: Box<Widget> = box Label::new(...);

    box Frame::new([b, l]) as Box<Widget>
}
```

By using trait objects, we can set up a GUI framework with a `Frame` widget that
contains a heterogeneous collection of children widgets.

**Pros**:

* _Heterogeneity_. When you need it, you really need it.
* _Code size_. Unlike generics, trait objects do not generate specialized
  (monomorphized) versions of code, which can greatly reduce code size.

**Cons**:

* _No generic methods_. Trait objects cannot currently provide generic methods.
* _Dynamic dispatch and fat pointers_. Trait objects inherently involve
  indirection and vtable dispatch, which can carry a performance penalty.
* _No Self_. Except for the method receiver argument, methods on trait objects
  cannot use the `Self` type.


<a id="type-safety"></a>
## Type safety

[C-NEWTYPE]: #c-newtype
<a id="c-newtype"></a>
### Newtypes provide static distinctions (C-NEWTYPE)

Newtypes can statically distinguish between different interpretations of an
underlying type.

For example, a `f64` value might be used to represent a quantity in miles or in
kilometers. Using newtypes, we can keep track of the intended interpretation:

```rust
struct Miles(pub f64);
struct Kilometers(pub f64);

impl Miles {
    fn as_kilometers(&self) -> Kilometers { ... }
}
impl Kilometers {
    fn as_miles(&self) -> Miles { ... }
}
```

Once we have separated these two types, we can statically ensure that we do not
confuse them. For example, the function

```rust
fn are_we_there_yet(distance_travelled: Miles) -> bool { ... }
```

cannot accidentally be called with a `Kilometers` value. The compiler will
remind us to perform the conversion, thus averting certain [catastrophic bugs].

[catastrophic bugs]: http://en.wikipedia.org/wiki/Mars_Climate_Orbiter

[C-CUSTOM-TYPE]: #c-custom-type
<a id="c-custom-type"></a>
### Arguments convey meaning through types, not `bool` or `Option` (C-CUSTOM-TYPE)

Prefer

```rust
let w = Widget::new(Small, Round)
```

over

```rust
let w = Widget::new(true, false)
```

Core types like `bool`, `u8` and `Option` have many possible interpretations.

Use custom types (whether `enum`s, `struct`, or tuples) to convey interpretation
and invariants. In the above example, it is not immediately clear what `true`
and `false` are conveying without looking up the argument names, but `Small` and
`Round` are more suggestive.

Using custom types makes it easier to expand the options later on, for example
by adding an `ExtraLarge` variant.

See [the newtype pattern][C-NEWTYPE] for a no-cost way to wrap existing types
with a distinguished name.

[C-BITFLAG]: #c-bitflag
<a id="c-bitflag"></a>
### Types for a set of flags are `bitflags`, not enums (C-BITFLAG)

Rust supports `enum` types with "custom discriminants":

~~~~
enum Color {
  Red = 0xff0000,
  Green = 0x00ff00,
  Blue = 0x0000ff
}
~~~~

Custom discriminants are useful when an `enum` type needs to be serialized to an
integer value compatibly with some other system/language. They support
"typesafe" APIs: by taking a `Color`, rather than an integer, a function is
guaranteed to get well-formed inputs, even if it later views those inputs as
integers.

An `enum` allows an API to request exactly one choice from among many. Sometimes
an API's input is instead the presence or absence of a set of flags. In C code,
this is often done by having each flag correspond to a particular bit, allowing
a single integer to represent, say, 32 or 64 flags. Rust's `std::bitflags`
module provides a typesafe way for doing so.

[C-BUILDER]: #c-builder
<a id="c-builder"></a>
### Builders enable construction of complex values (C-BUILDER)

Some data structures are complicated to construct, due to their construction
needing:

* a large number of inputs
* compound data (e.g. slices)
* optional configuration data
* choice between several flavors

which can easily lead to a large number of distinct constructors with many
arguments each.

If `T` is such a data structure, consider introducing a `T` _builder_:

1. Introduce a separate data type `TBuilder` for incrementally configuring a `T`
   value. When possible, choose a better name: e.g. `Command` is the builder for
   `Process`.
2. The builder constructor should take as parameters only the data _required_ to
   make a `T`.
3. The builder should offer a suite of convenient methods for configuration,
   including setting up compound inputs (like slices) incrementally. These
   methods should return `self` to allow chaining.
4. The builder should provide one or more "_terminal_" methods for actually
   building a `T`.

The builder pattern is especially appropriate when building a `T` involves side
effects, such as spawning a task or launching a process.

In Rust, there are two variants of the builder pattern, differing in the
treatment of ownership, as described below.

#### Non-consuming builders (preferred):

In some cases, constructing the final `T` does not require the builder itself to
be consumed. The follow variant on [`std::process::Command`] is one example:

[`std::process::Command`]: https://doc.rust-lang.org/std/process/struct.Command.html

```rust
// NOTE: the actual Command API does not use owned Strings;
// this is a simplified version.

pub struct Command {
    program: String,
    args: Vec<String>,
    cwd: Option<String>,
    // etc
}

impl Command {
    pub fn new(program: String) -> Command {
        Command {
            program: program,
            args: Vec::new(),
            cwd: None,
        }
    }

    /// Add an argument to pass to the program.
    pub fn arg<'a>(&'a mut self, arg: String) -> &'a mut Command {
        self.args.push(arg);
        self
    }

    /// Add multiple arguments to pass to the program.
    pub fn args<'a>(&'a mut self, args: &[String])
                    -> &'a mut Command {
        self.args.push_all(args);
        self
    }

    /// Set the working directory for the child process.
    pub fn cwd<'a>(&'a mut self, dir: String) -> &'a mut Command {
        self.cwd = Some(dir);
        self
    }

    /// Executes the command as a child process, which is returned.
    pub fn spawn(&self) -> IoResult<Process> {
        ...
    }
}
```

Note that the `spawn` method, which actually uses the builder configuration to
spawn a process, takes the builder by immutable reference. This is possible
because spawning the process does not require ownership of the configuration
data.

Because the terminal `spawn` method only needs a reference, the configuration
methods take and return a mutable borrow of `self`.

##### The benefit

By using borrows throughout, `Command` can be used conveniently for both
one-liner and more complex constructions:

```rust
// One-liners
Command::new("/bin/cat").arg("file.txt").spawn();

// Complex configuration
let mut cmd = Command::new("/bin/ls");
cmd.arg(".");

if size_sorted {
    cmd.arg("-S");
}

cmd.spawn();
```

#### Consuming builders:

Sometimes builders must transfer ownership when constructing the final type `T`,
meaning that the terminal methods must take `self` rather than `&self`:

```rust
// A simplified excerpt from std::task::TaskBuilder

impl TaskBuilder {
    /// Name the task-to-be. Currently the name is used for identification
    /// only in failure messages.
    pub fn named(mut self, name: String) -> TaskBuilder {
        self.name = Some(name);
        self
    }

    /// Redirect task-local stdout.
    pub fn stdout(mut self, stdout: Box<Writer + Send>) -> TaskBuilder {
        self.stdout = Some(stdout);
        //   ^~~~~~ this is owned and cannot be cloned/re-used
        self
    }

    /// Creates and executes a new child task.
    pub fn spawn(self, f: proc():Send) {
        // consume self
        ...
    }
}
```

Here, the `stdout` configuration involves passing ownership of a `Writer`, which
must be transferred to the task upon construction (in `spawn`).

When the terminal methods of the builder require ownership, there is a basic
tradeoff:

* If the other builder methods take/return a mutable borrow, the complex
  configuration case will work well, but one-liner configuration becomes
  _impossible_.

* If the other builder methods take/return an owned `self`, one-liners continue
  to work well but complex configuration is less convenient.

Under the rubric of making easy things easy and hard things possible, _all_
builder methods for a consuming builder should take and returned an owned
`self`. Then client code works as follows:

```rust
// One-liners
TaskBuilder::new().named("my_task").spawn(proc() { ... });

// Complex configuration
let mut task = TaskBuilder::new();
task = task.named("my_task_2"); // must re-assign to retain ownership

if reroute {
    task = task.stdout(mywriter);
}

task.spawn(proc() { ... });
```

One-liners work as before, because ownership is threaded through each of the
builder methods until being consumed by `spawn`. Complex configuration, however,
is more verbose: it requires re-assigning the builder at each step.


<a id="dependability"></a>
## Dependability

[C-VALIDATE]: #c-validate
<a id="c-validate"></a>
### Functions validate their arguments (C-VALIDATE)

Rust APIs do _not_ generally follow the [robustness principle]: "be conservative
in what you send; be liberal in what you accept".

[robustness principle]: http://en.wikipedia.org/wiki/Robustness_principle

Instead, Rust code should _enforce_ the validity of input whenever practical.

Enforcement can be achieved through the following mechanisms (listed in order of
preference).

#### Static enforcement:

Choose an argument type that rules out bad inputs.

For example, prefer

```rust
fn foo(a: Ascii) { ... }
```

over

```rust
fn foo(a: u8) { ... }
```

where `Ascii` is a _wrapper_ around `u8` that guarantees the highest bit is
zero; see [newtype patterns][C-NEWTYPE] for more details on creating typesafe
wrappers.

Static enforcement usually comes at little run-time cost: it pushes the costs to
the boundaries (e.g. when a `u8` is first converted into an `Ascii`). It also
catches bugs early, during compilation, rather than through run-time failures.

On the other hand, some properties are difficult or impossible to express using
types.

#### Dynamic enforcement:

Validate the input as it is processed (or ahead of time, if necessary). Dynamic
checking is often easier to implement than static checking, but has several
downsides:

1. Runtime overhead (unless checking can be done as part of processing the
   input).
2. Delayed detection of bugs.
3. Introduces failure cases, either via `fail!` or `Result`/`Option` types,
   which must then be dealt with by client code.

#### Dynamic enforcement with `debug_assert!`:

Same as dynamic enforcement, but with the possibility of easily turning off
expensive checks for production builds.

#### Dynamic enforcement with opt-out:

Same as dynamic enforcement, but adds sibling functions that opt out of the
checking.

The convention is to mark these opt-out functions with a suffix like
`_unchecked` or by placing them in a `raw` submodule.

The unchecked functions can be used judiciously in cases where (1) performance
dictates avoiding checks and (2) the client is otherwise confident that the
inputs are valid.

[C-DTOR-FAIL]: #c-dtor-fail
<a id="c-dtor-fail"></a>
### Destructors never fail (C-DTOR-FAIL)

Destructors are executed on task failure, and in that context a failing
destructor causes the program to abort.

Instead of failing in a destructor, provide a separate method for checking for
clean teardown, e.g. a `close` method, that returns a `Result` to signal
problems.

[C-DTOR-BLOCK]: #c-dtor-block
<a id="c-dtor-block"></a>
### Destructors that may block have alternatives (C-DTOR-BLOCK)

Similarly, destructors should not invoke blocking operations, which can make
debugging much more difficult. Again, consider providing a separate method for
preparing for an infallible, nonblocking teardown.


<a id="debuggability"></a>
## Debuggability

[C-DEBUG]: #c-debug
<a id="c-debug"></a>
### All public types implement `Debug` (C-DEBUG)

If there are exceptions, they are rare.

[C-DEBUG-NONEMPTY]: #c-debug-nonempty
<a id="c-debug-nonempty"></a>
### `Debug` representation is never empty (C-DEBUG-NONEMPTY)

Even for conceptually empty values, the `Debug` representation should never be
empty.

```rust
let empty_str = "";
assert_eq!(format!("{:?}", empty_str), "\"\"");

let empty_vec = Vec::<bool>::new();
assert_eq!(format!("{:?}", empty_vec), "[]");
```


<a id="future-proofing"></a>
## Future proofing

[C-STRUCT-PRIVATE]: #c-struct-private
<a id="c-struct-private"></a>
### Structs have private fields (C-STRUCT-PRIVATE)

Making a field public is a strong commitment: it pins down a representation
choice, _and_ prevents the type from providing any validation or maintaining any
invariants on the contents of the field, since clients can mutate it arbitrarily.

Public fields are most appropriate for `struct` types in the C spirit: compound,
passive data structures. Otherwise, consider providing getter/setter methods and
hiding fields instead.

[C-NEWTYPE-HIDE]: #c-newtype-hide
<a id="c-newtype-hide"></a>
### Newtypes encapsulate implementation details (C-NEWTYPE-HIDE)

A newtype can be used to hide representation details while making precise
promises to the client.

For example, consider a function `my_transform` that returns a compound iterator
type `Enumerate<Skip<vec::MoveItems<T>>>`. We wish to hide this type from the
client, so that the client's view of the return type is roughly `Iterator<(uint,
T)>`. We can do so using the newtype pattern:

```rust
struct MyTransformResult<T>(Enumerate<Skip<vec::MoveItems<T>>>);
impl<T> Iterator<(uint, T)> for MyTransformResult<T> { ... }

fn my_transform<T, Iter: Iterator<T>>(iter: Iter) -> MyTransformResult<T> {
    ...
}
```

Aside from simplifying the signature, this use of newtypes allows us to make a
expose and promise less to the client. The client does not know _how_ the result
iterator is constructed or represented, which means the representation can
change in the future without breaking client code.


<a id="necessities"></a>
## Necessities

[C-STABLE]: #c-stable
<a id="c-stable"></a>
### Public dependencies of a stable crate are stable (C-STABLE)

A crate cannot be stable (>=1.0.0) without all of its public dependencies being
stable.

Public dependencies are crates from which types are used in the public API of
the current crate.

```rust
pub fn do_my_thing(arg: other_crate::TheirThing) { /* ... */ }
```

A crate containing this function cannot be stable unless `other_crate` is also
stable.

Be careful because public dependencies can sneak in at unexpected places.

```rust
pub struct Error {
    private: ErrorImpl,
}

enum ErrorImpl {
    Io(io::Error),
    // Should be okay even if other_crate isn't
    // stable, because ErrorImpl is private.
    Dep(other_crate::Error),
}

// Oh no! This puts other_crate into the public API
// of the current crate.
impl From<other_crate::Error> for Error {
    fn from(err: other_crate::Error) -> Self {
        Error { private: ErrorImpl::Dep(err) }
    }
}
```

[C-PERMISSIVE]: #c-permissive
<a id="c-permissive"></a>
### Crate and its dependencies have a permissive license (C-PERMISSIVE)


<a id="external-links"></a>
## External Links

- [RFC 199] - Ownership naming conventions
- [RFC 344] - Naming conventions
- [RFC 430] - Naming conventions
- [RFC 505] - Doc conventions
- [RFC 1574] - Doc conventions
- [RFC 1687] - Crate-level documentation

[RFC 344]: https://github.com/rust-lang/rfcs/blob/master/text/0344-conventions-galore.md
[RFC 430]: https://github.com/rust-lang/rfcs/blob/master/text/0430-finalizing-naming-conventions.md
[RFC 1687]: https://github.com/rust-lang/rfcs/pull/1687
[RFC 505]: https://github.com/rust-lang/rfcs/blob/master/text/0505-api-comment-conventions.md
[RFC 1574]: https://github.com/rust-lang/rfcs/blob/master/text/1574-more-api-documentation-conventions.md
[RFC 199]: https://github.com/rust-lang/rfcs/blob/master/text/0199-ownership-variants.md
