---
advertising:
  image: gdscript-typed-performance
date: 2024-02-14
image: gdscript-typed-performance-report
summary: Static typing is one of the easiest and most rewarding disciplines you can add to your game development.
tags:
  - game development
  - godot
  - gdscript
title: Yes, your Godot game runs faster with static types
---

GDScript is a **dynamically-typed** language. A GDScript variable's type can change from line to line.

For example, here's a variable that starts with a string, then gets a floating-point number, and ends up with a vector:

```gdscript
var a = "hello"
a = 1.2
a = Vector2.ZERO
```

That's entirely valid GDScript, and it'll happily do what you're asking it to do.

But you _can_--and I'd argue that you _should_--use **static types**.

Here, I'm telling GDScript that I want a variable to be statically, unchangingly, unwaveringly and for evermore typed as a string:

```gdscript
var a: String = "hello"
```

With that type annotation in place, GDScript won't let me assign a floating-point number, and I'll get a warning in the Godot Editor as soon as I type it:

```gdscript
var a: String = "hello"
a = 1.2  # ERROR: Cannot assign a value of type "float" as "String".
```

Now, look: I think static typing is a Good Thing&trade; for consistency, readability and maintainability -- but that's just, like, my opinion, man.

But did you know static typing improves your code's performance?

## Performance

Here are a couple of loops that increment a couple of variables. One's dynamically-typed, and the other is statically-typed as an integer:

```gdscript
const ITERATIONS: int = 1000000000

var _untyped = 0
var _typed: int = 0

for i in range(ITERATIONS):
  _untyped += 1

for i in range(ITERATIONS):
  _typed += 1
```

Which of these loops do you reckon runs the fastest?

Well, I mean, it's _obviously_ going to be the statically-typed loop given the conceit of the blog, but how _much_ faster do you reckon?

I hacked up a quick test project to run 1,000,000,000 iterations on my M2 Max, and the results speak for themselves:

| Build   | Untyped (ms) | Typed (ms) | Difference |
| :-      |           -: |         -: |         -: |
| Debug   |       16,921 |     12,140 |     -28.2% |
| Release |        9,695 |      6,372 |     -34.2% |

Ain't that grand! According to [GDScript progress report: Typed instructions](https://godotengine.org/article/gdscript-progress-report-typed-instructions/), static typing allows the interpreter to take shortcuts -- and it shows, too.

I added a multiplication test and saw roughly the same performance improvement. And then I added vector calculations, and _that_ blew me away:

| Operation        | Build   | Untyped (ms) | Typed (ms) | Difference |
| :-               | :-      |           -: |         -: |         -: |
| Addition         | Debug   |       16,921 |     12,140 |     -28.2% |
| Addition         | Release |        9,695 |      6,372 |     -34.2% |
| Multiplication   | Debug   |       16,961 |     12,125 |     -28.5% |
| Multiplication   | Release |        9,641 |      6,171 |     -35.9% |
| Vector2 distance | Debug   |       24,980 |     11,238 |     -55.0% |
| Vector2 distance | Release |       14,728 |      6,057 |     -58.8% |

Bloody hell! Vector operations just _scream_ when they're statically-typed!

You've got nothing to lose by adding static type annotations, and potential double-digit math performance increases to gain. There's no good reason to _not_.

And if you want to verify my numbers, my test project's on GitHub at [cariad/gdscript-typed-performance](https://github.com/cariad/gdscript-typed-performance).

## Working with type annotations in the Godot editor

You don't need to change any settings to use static types--you can just start adding type annotations and away you go--but I do recommend that you enable **text_editor/completion/add_type_hints** in the Godot editor to automatically pre-fill known annotations.

For example, when overriding `_physics_process`, you'll get:

```gdscript
func _physics_process(delta: float) -> void:
```

…instead of just:

```gdscript
func _physics_process(delta):
```

You can also keep an eye out for grey line numbers, like 294 here:

{{< image name="gdscript-untyped" >}}

This indicates an "unsafe" line that the editor can't guarantee will work at runtime.

In this case, the editor can't be sure that `a` and `b` will hold values that `+` can operate on, so the line is flagged for review. We can reassure the editor by statically-typing `a` and `b` as types that can be added:

{{< image name="gdscript-semi-typed" >}}

But of course, why statically-type just two lines when you could fix all three?

{{< image name="gdscript-typed" >}}

Have fun! ❤️
