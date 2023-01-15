# live coding notes

## challenges

reloading pure functions with constant data layout is simple

the hard part are changes in data layout (schema migrations)

## languages

### erlang

https://www.erlang.org/doc/design_principles/appup_cookbook.html#changing-internal-state

> 12.4  Changing Internal State
>
> In this case, simple code replacement is not sufficient. The process must explicitly transform its state using the callback function code_change before switching to the new version of the callback module. Thus, synchronized code replacement is used.

### clojure

### common lisp

https://news.ycombinator.com/item?id=32362836

> Common Lisp. You can redefine any class while your system is running, and you can customize how the data from old instances is updated to be used by new instances. This is part of the Metaobject Protocol, where the language calls functions and methods that are part of your program in order to implement many of its features.

### smalltalk

https://news.ycombinator.com/item?id=32362836

> When changing instance or class variables in Smalltalk, the system actually changing the object layout and goes and recompiles code using it. If you are interested in the details: Chapter 5 of [1] explains it a bit.
>
> [1] Hirschfeld et al. Assisting System Evolution: A Smalltalk Retrospective.
http://soft.vub.ac.be/Publications/2002/vub-prog-tr-02-25.pdf

### lively

https://lively-next.org/

### rust

https://github.com/rksm/hot-lib-reloader-rs

https://robert.kra.hn/posts/hot-reloading-rust/

https://news.ycombinator.com/item?id=32362836

> Only being able to change function implementation (and non generics only at that) seems like a substantial limitation. Its not often I’m quickly prototyping something that would benefit from hot reloading, and not also changing data layout as I’m iterating.
>
> I’m curious to see if the sharp edges will soften as time goes on. Mangling structs to be pinned to a particular loaded library version somehow, and injecting indirection where they’re used, would take this a long way. But would probably require compiler support.

> you can also serialize state or keep it in a generic form like a serde_json::Value. Serialization of course needs some kind of migration on the part of the code using it (if the data layout changes it loads an old version and needs to convert it to the new form) or defaults for newly added fields/types. However, this is true in general, even in other languages.

> Like you say, modeling the interface between the shared library like a Thrift struct (which can be serialized) may be beneficial, and would facilitate adding/removing fields by having e.g. default fields. Or just crash if a non-backwards compatible change is made, e.g. a new field is expected but not present, but ignore fields that are present but no longer expected.

> When I think hotreload I think “persist state but update code”. Persisting state safely when you change data layout is almost impossible. Even if the language is dynamic you’ll have stale objects that are missing the new members or didn’t go through the new initialization routine.

> Yes, the indirection (and also late binding) is why hot reloading is (somewhat) ergonomic in those languages, but would be difficult to do in Rust (or C++, etc).
>
> I think the shared library types would need to be indirected either via compiler support, or some kind of `Hot<T>` wrapper that acts like a `Box<T>` when hot reloading is enabled, and like a `T` in "production" mode to be ergonomic.

> what if there was a sure way to directly map your entire program state into the new version and you could be assured that you didn't miss any obscure corner of the state space? Like a borrow checker for transitioning between program versions. That'd be pretty neat.

https://github.com/irh/rust-hot-reloading

### zig

https://github.com/ziglang/zig/issues/68

> I always wonder how should hot-swap deal with data (data structure layout and long-lived in-memory state), it seems rather difficult compared to hot-swap stateless code (like a pure function).

> About the state transition, I can think of a solution which serialize the state out (to a layout-agnostic format), allocate new state, then serialize in. It should work, but involves a lot of manual implementation of serializing code and is not very generic/automatic.

> Lisps have a great interactive dev experience. You can eval the last expression, the current line, the selected region, the current 'cell' (a region delimited by special comments), the current file, etc. They also autocomplete based on the living program in the REPL, and you can inspect the variables.

### nim

https://github.com/nim-lang/Nim/issues/8927

## keywords

- hot reload
- hot reloading
- hot module replacement (HMR)
