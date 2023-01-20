# live coding notes

## challenges

reloading pure functions with constant data layout is simple

the hard part are changes in data layout (schema migrations)

https://www.reddit.com/r/Racket/comments/3as1bd/comment/cshobj3/

> Better support for live code upgrade is an active research area. I don't know of any language or system that gets it 100% right, because the interactions between macros, long-lived state, data formats, interfaces, protocols and code are so many and so complex.

## criteria

- "explicit" schema migrations or "implicit" schema migrations
  - explicit: migrations are defined manually, user has full control over the migrations
  - implicit: migrations are generated automatically, user has no control over the migrations, migrations can be wrong

## languages

### erlang

https://www.erlang.org/doc/design_principles/appup_cookbook.html#changing-internal-state

> 12.4  Changing Internal State
>
> In this case, simple code replacement is not sufficient. The process must explicitly transform its state using the callback function code_change before switching to the new version of the callback module. Thus, synchronized code replacement is used.

https://news.ycombinator.com/item?id=10669131

https://learnyousomeerlang.com/relups

#### elixir

https://kennyballou.com/blog/2016/12/elixir-hot-swapping/index.html

### lisp

#### common lisp

https://news.ycombinator.com/item?id=32362836

> Common Lisp. You can redefine any class while your system is running, and you can customize how the data from old instances is updated to be used by new instances. This is part of the [Metaobject Protocol](https://en.wikipedia.org/wiki/Metaobject#Metaobject_protocol), where the language calls functions and methods that are part of your program in order to implement many of its features.

https://lisp-journey.gitlab.io/blog/i-realized-that-to-live-reload-my-web-app-is-easy-and-convenient/

https://lispcookbook.github.io/cl-cookbook/web.html

https://github.com/vindarel/demo-web-live-reload

https://www.reddit.com/r/Racket/comments/3as1bd/comment/cshobj3/

> In Common Lisp, you can modify almost everything about the program after it starts running: You can redefine functions, macros, variables and in some implementations even constants, you can create new packages, delete existing packages, (load) new source files or FASL binaries (which can contain definitions that replace pre-existing ones in the running program) download and install new libraries via QuickLisp and then use them, etc.

https://news.ycombinator.com/item?id=30172641

> repl-driven development that is available to many (all?) common lisp implementations is unmatched in any language, and i think that this is unique to common lisp. please see the post by mikelevins in this thread that details this process. i think this can be a great productivity booster for production as well as exploratory programming.

<blockquote>

It's how I've worked day-to-day for thirty years or so, so here's my take on it:

I generally work in Emacs. I'll have one or more source files open, plus at least one REPL buffer. The REPL is my communication channel with the running Lisp.

The object of the game is to teach the Lisp interactively how to be the program I want. It's already a working program; it just isn't the one I want to build. I'll use Lisp expressions to teach it, one expression at a time, how to be what I want.

The contents of the source files depend on the stage I'm at with a program. If I'm just starting then it's probably one file with a few snippets that I C-x C-e to send to the REPL to modify the dynamic environment of the running Lisp.

If it's a more mature project, then I have a bunch of source files that fit into an ASDF system (which defines sources to build and dependencies among them and any libraries they depend on). The system as a whole will be loaded into the memory of the running Lisp, converting it into a work-in-progress version of my app.

I'll have some subset of the project files open in buffers so that I can add or edit definitions and C-x C-e them into the running Lisp and see in the REPL whether my changes have the effect that I intend.

I continue this way indefinitely, teaching the running Lisp expression-by-expression the features that I intend for the finished product to have. When the difference between my idea and what the Lisp does shrinks to epsilon, the program is implemented. I run a build function and I have a delivered application.

When something goes wrong during my work, I land in a breakloop, which is something like a backtrace, except live. Rather than a printout of an unwound stack, it's a live REPL on the dynamic environment of the stack at the moment the error occurred. I can use the breakloop to wander up and down the stack, inspect and edit variable, type, and function definitions, and resume execution at the moment of the error when I'm ready to see the effect of my changes. The Lisp then runs the same function from where it broke, except that the variables, types, and functions I modified now have the definitions I just gave them, rather than the ones they originally had.

If I change the definition of some class that already has live instances, then the Lisp reinitializes those instances with the new class definition and continues. If I neglected to show it how to reinitialize those classes, it drops me into another breakloop and offers me the opportunity to tell it how to do so, after which it will continue, as before.

If I stumble across something curious and I want to show it to a collaborator in context, I can save a heap image of the running Lisp and give it to my colleague. That colleague can start the image on their machine and see the same dynamic environment that I saw. In some older Lisps, that dynamic state could include all of the windows on my screen, and their contents, and which one was the active window, and where the text-insertion point was.

I can deploy my work-in-progress apps to a staging machine with a repl server built in. I can let it run in a testing environment indefinitely, and use Emacs to connect to a live REPL that I can use to rummage around in the running app, inspect and edit variables, types, and functions, and generally do anything I would do if it were running in my normal dev environment. If I see something odd, I can again dump a heap image, copy it back to my development machine, and crawl around in its running state while I let the deployed test version go back to doing what it was doing.

There's quite a bit more to it, but with luck, that gives a general idea of what the workflow is like. Common Lisp environments are generally like this; some have richer sets of affordances than others. Smalltalks are like that, too, and so is Factor.

Most other Languages and repls are not so much like that. Clojurescript with figwheel gives you part of it, and the part it implements is pretty good.

</blockquote>

<blockquote>

> What do you prefer about using C-x C-e on individual sexps rather than reloading everything and keeping the entire program synced between source files and the running environment?

Generally speaking, I work by building some data that represent my understanding of the model I'm working with, and some functions to operate on that model. My models start out simple and more or less wrong, and as I work, I build improved understanding. Mostly my understanding improves incrementally, and so does my code. I build up the app piece by piece, adding and changing things bit by bit. So most of the time, changing a small piece is all I need, and that's what evaluating one expression does.
Moreover, if I recompile everything, then I'm rebuilding my in-memory state from scratch. Mostly, that's not what I want. Mostly I work like a sculptor: I make a rough approximation and then refine it stepwise. I don't want to rebuild from scratch every time I discover something new. I want to keep most of what I've built and change just what needs to be changed.

That said, sometimes a discovery does call for a larger-scale reconstruction. That's when I reload a whole file or, in more extreme cases, reload an entire ASDF system of files. Once in a while, I change enough (or make enough mistakes) that it makes sense to blow the whole thing away--kill the Lisp, restart it, and load the system from scratch. But that's fairly rare.

Mostly I want to plop down my clay and tweak it a little at a time toward my goal, discovering and refining it a little at a time.

</blockquote>

<blockquote>

For Clojure there are libraries to refresh the program: these stop the app, reload and evaluate all the source code files, and restart the app to ensure you are working with a consistent state that reflects the source code.
Between such reloads you might end up in inconsistent states, but you benefit from fast iteration:

My workflow is to update functions in the source code, send to to the repl, quickly test them in the repl, and once done, I save, commit and push the code. CI then runs the test suite from the source files.

</blockquote>

> Well, not the same thing but Java can hot-reload classes (to a degree depending on implementation). But what truly makes repl-driven development in lisps so good is the more functional approach to development. Hot reload gets better the smaller the replaceable units get. Lisps have it good with basically paren-level hot reload.

<blockquote>

> I have never used Paredit when doing Scheme or Racket, so maybe I should just hunker down with it at some point (I have looked into it before). I don’t use Emacs (tried it a few times), but there are several Paredit-based VSCode extensions.

Paredit is the editor part, but then there is also SLIME REPL. In lisp, unlike in other programming languages, you don't start writing your code with a blank slate and an empty source file.

A common approach is to start with an initial core. The idea is that all programs start the same, with an initial core, that contains runtime and standard library. You can test things out in a REPL, that you run side by side with a text editor. As you work things out in REPL, you can start moving them to the text file and the program starts taking form and shape. It's a much more **exploration driven design** and with fast structure editing of paredit you can really transplant or reorganize pieces of code really quickly. The idea is to grow the program from the initial core into something else inside-out style. It's akin to being dropped into a running program, and modifying it as it is running but that is not possible with most languages easily and as interactively as in Common Lisp for example.

With SLIME the REPL and text editor integration, you can for example write a buggy function, test it in REPL and find out about the error. Modify it in the text editor, press Ctrl-c twice, and the function is hot reloaded on the fly. No reloads, no re-imports, no-caching, no program restarts. Next time the function is called, it is replaced with a new stuff you just wrote. It really is pretty magical. It just works, and works for classes, objects, meta classes, functions, variables, etc.

The language is unlike other languages, because it offers object introspection (often mistakenly referred to as reflection) and object intercession; introspection and intercession combined give you a true reflection. It is the latter technique that supports all of the above, for an imho superior development experience. It really just works, and it works marvelously well. No yak shaving for hours on end trying to get some dependency chain to work, compiler errors, package version mismatches, etc. No need for build team, no need for testing/qa team, no need for most of your usual dev overhead we are so used to these days :)

</blockquote>

<blockquote>

> Is there any source to learn this way of development?

You could start with a copy of the book „the land of Lisp” or „practical common Lisp”
People seem to recommend portacle https://portacle.github.io/ for a first dev environment but I usually set everything up myself. Portacle seems to have all the essentials. You can follow SLIME installation instructions from the quicklisp page of portacle doesn’t come with it.

Then I recommend slime manual at https://slime.common-lisp.dev/doc/slime.pdf if you’re going to be using Emacs.

I’m not familiar with other tools.

</blockquote>

#### clojure

https://github.com/chr15m/awesome-clojure-likes

https://github.com/jank-lang/jank A Clojure dialect on LLVM with gradual typing, a native runtime, and C++ interop

#### racket

https://www.reddit.com/r/Racket/comments/3as1bd/hot_swapping_code/

> Some of this stuff works in certain circumstances in Racket, but Racket defaults to acting like a statically-compiled language in the tradition of Java, C++, and Pascal. "Requiring" a module in Racket is a lot like "linking" a library in C++/Pascal or "importing" a package in Java, and it's very unlike loading a file in Common Lisp.
>
> dynamic-require is somewhat like using dlopen or LoadLibrary, except the latter two are better documented. Using dynamic-require is not easy like the Common Lisp equivalent, and it has limitations that you wouldn't run into in Common Lisp.

#### scheme

> the R5RS standard doesn't require there to be any way to modify the program while it's running. For example, (eval '(define (foo) 'bar) (interaction-environment)) doesn't have to work. Even the load procedure is optional. So I'd say that hotpatching is not supported by the R5RS standard. However, some specific Scheme implementations are just as hotpatchable as CL.

#### guile

#### fennel

compiles to lua

https://vxlabs.com/2018/05/18/interactive-programming-with-fennel-lua-lisp-emacs-and-lisp-game-jam-winner-exo_encounter-667/

### smalltalk

https://news.ycombinator.com/item?id=32362836

> When changing instance or class variables in Smalltalk, the system actually changing the object layout and goes and recompiles code using it. If you are interested in the details: Chapter 5 of [1] explains it a bit.
>
> [1] Hirschfeld et al. Assisting System Evolution: A Smalltalk Retrospective.
http://soft.vub.ac.be/Publications/2002/vub-prog-tr-02-25.pdf

### lively

https://lively-next.org/

### mun

https://github.com/mun-lang/mun

> First class hot-reloading
>
> Every aspect of Mun is designed with hot reloading in mind. Hot reloading is the process of changing code and resources of a live application, removing the need to start, stop and recompile an application whenever a function or value is changed.

https://mun-lang.org/blog/2020/05/01/memory-mapping/

> Whereas hot reloading of functions can be easily done in most languages (e.g. by swapping shared libraries with a fixed API), hot reloading data is more complicated because it requires information about the data layout.

implicit schema migrations

### ruby

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

https://depth-first.com/articles/2020/09/21/interactive-rust-in-a-repl-and-jupyter-notebook-with-evcxr/

### zig

https://github.com/ziglang/zig/issues/68

> I always wonder how should hot-swap deal with data (data structure layout and long-lived in-memory state), it seems rather difficult compared to hot-swap stateless code (like a pure function).

> About the state transition, I can think of a solution which serialize the state out (to a layout-agnostic format), allocate new state, then serialize in. It should work, but involves a lot of manual implementation of serializing code and is not very generic/automatic.

> Lisps have a great interactive dev experience. You can eval the last expression, the current line, the selected region, the current 'cell' (a region delimited by special comments), the current file, etc. They also autocomplete based on the living program in the REPL, and you can inspect the variables.

### nim

https://github.com/nim-lang/Nim/issues/8927

### C++

https://www.reddit.com/r/cpp/comments/wgidjl/hscpp_an_experimental_library_to_hotreload_c/

> Does C++ hot-reload in 2022 allow you to make layout changes? I feel that's one of the big advantages to the DLL approach, as making data changes is extremely common when doing game scripting.

## see also

- https://en.wikipedia.org/wiki/Interactive_programming
- On repl-driven programming
  - https://mikelevins.github.io/posts/2020-12-18-repl-driven/
  - https://news.ycombinator.com/item?id=31277149

## keywords

- hot reload
- hot reloading
- hot module replacement (HMR)
- memory mapping
- hotpatching, hot patching
- interactive programming
- interactive development
