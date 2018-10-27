```@meta
EditURL = "https://github.com/TRAVIS_REPO_SLUG/blob/master/"
```

# Introduction to julia

Be sure to follow the instructions for installation and activation
of the environment in the [README](README.md) file.

## Some Basic Syntax

A lot of these examples were taken or adapted from [Learn julia in Y minutes](https://learnxinyminutes.com/docs/julia/)

### Numbers and math

numbers work like numbers, with common infix operators:

```@example demo
2 + 2
```

```@example demo
(1 + 2) * 6
```

```@example demo
12 % 5
```

Dividing integers always produces `Float`s

```@example demo
15 / 5
```

```@example demo
15 / 6
```

One can use `div()` to truncate,
or use `//` for rational numbers

```@example demo
div(3, 2)
```

```@example demo
3//2
```

### Boolean operators

The boolean operators in julia are `true` and `false`. `!` is used for `not`

Boolean operators

```@example demo
!true
```

```@example demo
!false
```

```@example demo
1 == 1
```

```@example demo
1 != 1
```

```@example demo
1 < 10
```

```@example demo
1 >= 10
```

Comparisons can be chained

```@example demo
1 < 2 < 3
```

```@example demo
2 < 3 < 2
```

### Variable assignment

Assign variables with `=`.
Variables must start with a letter, but can contain letters, numbers, `_` and `!`.
Many unicode characters can also be used (though some are already taken)

```@example demo
x = 5
👍 = 2x + 7 # type \:+1:<tab>

println(x)
println(👍)
println(ℯ) # type \euler<tab>
println(π) # type \pi<tab>
```

### Strings

Strings must be surrounded by double quotes (`"`).
Single quotes (`'`) are reserved for character literals.
String interpolation can be accomplished with `$`,
and concatenation with   `*`

```@example demo
typeof("S")
```

```@example demo
typeof('S')
```

```@example demo
println("x is $x" * " and " * "ℯ is $ℯ")
```

## Loops, functions, collections

Mostly, whitespace is ignored.
Newlines are sometimes reqired (or can be indicated with `;`).
Code blocks are identified with keywords and `end`.

The following are equivalent:

```@example demo
function foo(x, y)
    return 2x + y
end

function foo(x, y); return 2x+y; end

function foo(    x,y    )
return 2x + y
end
```

or

```@example demo
for i in 1:2:10
    if i < 5
        println(i)
    end
end

for i in 1:2:10; if i<5; println(i); end; end
```

Though the style convention is to indent inner blocks.

A shorter form for function invocation is also possible:

```@example demo
foo(x, y) = 2x + y

foo(2.3, 5)
```

### N-dimensional arrays are taken very seriously

Default julia arrays are column major,
though there are packages that enable different behavior

```@example demo
one_d = [1, 2, 3, 0]
```

```@example demo; continued = true
two_d = [1 10 2 20; # note: newlines aren't actually necessary
         3 30 0 0]
```

```@example demo
three_d = reshape(collect(1:16), 4,2,2)
```

"Dot" syntax can be used for broadcasting,
other operators assume matrix operations.

```@example demo
three_d .^ 2
```

```@example demo
two_d * one_d
```

### Indexing

Julia uses 1-based indexing (though there are packages that can adjust this)

```@example demo
one_d[4]
```

```@example demo
two_d[1, 2:3] # first row, columns 2 to 3
```

```@example demo
three_d[2:end, :, 1] # rows 2 to the end, all columns, 1st z
```

Initialize arrays and update, use comprehensions, or grow them from empty
(note: it's customary for functions that mutate any of their arguments to end with `!`)

```@example demo
a1 = zeros(3,2)
a1[4] = 3
a1[3,2] = 6
a1
```

```@example demo
a2 = [√x for x in 1:5] # or sqrt(x) or x^-2
```

```@example demo
a3 = []
for i in 10:-2:0
    push!(a3, i)
end
a3
```

## Julia's Type System

This is complicated,
but [the docs](https://docs.julialang.org/en/stable/manual/types/) are suprisingly readable.

Julia's magic (at least a big part of it) is its type dispatch system.
Everything from functions to scalars have a concrete type,
and one or more parent abstract types.

```@example demo
typeof(a1) |> println
typeof("Hi!") |> println
typeof(String) |> println
```

```@example demo
String <: AbstractString
```

```@example demo
String <: Number
```

```@example demo
typeof(2.2) <: Float64
```

```@example demo
Float64 <: AbstractFloat <: Real <: Number
```

```@example demo
subtypes(AbstractFloat)
```

```@example demo
subtypes(Number)
```

```@example demo
typeof(complex(2.2)) <: Real
```

Everything is a subtype of `Any`

```@example demo
supertype(Number)
```

```@example demo
supertype(AbstractString)
```

```@example demo
supertype(Any)
```

Functions dispatch on the types of all of their arguments (most specific first).
This example is from a [blog post](https://white.ucc.asn.au/2018/10/03/Dispatch,-Traits-and-Metaprogramming-Over-Reflection.html)
by Lyndon White.
I made a couple modifications, but it's mostly his work.

```@example demo
display_percent(x) = println(100x, "%")
```

When arguments do not have type assertion, they default to `Any`.
In other words, the above is equivalent to:

```@example demo
display_percent(x::Any) = println(100x, "%")
```

```@example demo
display_percent(0.5) # Float64
display_percent(0.5f0) # Float32
display_percent(BigFloat(π)/10) # BigFloat
display_percent(1) # Int64
display_percent(false) # Bool
display_percent(2//3) # Rational
```

To get the `Rational` result to display a bit better,
we can write a more specialized method.
Julia's dispatch system will attempt to call the most specific method possible.

```@example demo
function display_percent(x::Rational)
    x = round(100x, digits=2)
    println(x, "%")
end

display_percent(2//3)
```

What about something that's already a String?
The generic method above fails, since `100 * ::String` isn't defined.

```@example demo
try
    display_percent("23.0%")
catch e
    showerror(stdout, e)
end
```

But we can fix that.

```@example demo
function display_percent(str::AbstractString) # could do ::String, but nice to be as generic as possible
    if !occursin(r"^\d*\.?\d+%?$", str)  # any combination of numbers, followed or not by a percent sign
        throw(DomainError(str, "Not valid percentage format"))
    end

    if endswith(str, "%")
        println(str)
    else
        println("$str%")
    end
end

display_percent("1.2")
display_percent("42%")
```

Where this really comes in handy
is working with things you as the programmer don’t know the type of
when you are writing the code.
For example if you have a hetrogenous list of elements to process.

```@example demo
for x in [0.51, 0.6, 1//2, 0.1, "2%", "15"]
    display_percent(x)
end
```

A real world example of this sort of code
is in [solving for indexing rules in TensorFlow.jl][1].

[1]: https://github.com/malmaud/TensorFlow.jl/blob/d7ac7e306ca95122c2583c9db06f9e7405102275/src/ops/indexing.jl#L69-L98

It is also nicely extensible. Lets say we created a singleton type to represent a half.

```@example demo
struct Half end

display_percent(::Half) = println("50%")

display_percent(Half())
```

So it was simple to just add a new method for it.
Users can add support for new types,
completely separate from the original definition.

Constrast it to:

```@example demo
function display_percent_bad(x)
     if x isa AbxactString
         if !occursin(r"^\d*\.?\d+%?$", x)
             throw(DomainError(x, "Not valid percentage format"))
         end

         if endswith(x, "%")
             println(x)
         else
             println("$x%")
         end
    else
        println(100x, "%")
    end
end

display_percent_bad("5.3%")
display_percent_bad(0.5)

try
    display_percent_bad(Half())
catch e
    showerror(stdout, e)
end
```

You can see, that since it doesn’t include support for Half when it was declared,
it doesn’t work.
One could add to the conditional, but that's harder to find and debug later.
In addition, if a user wants to write a new type that uses this method,
they'd have to modify the original code.

This kinda code is common in a fair bit of python code.

*This page was generated using [Literate.jl](https://github.com/fredrikekre/Literate.jl).*

