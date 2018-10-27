# Workshop for learning julia

By Kevin Bonham, PhD

## Getting started

To use the examples in this notebook, you must have julia installed.
You may copy the examples into the julia REPL,
or run them from the included jupyter notebook
(assuming you have jupyter installed).

- [Link to julia downloads](https://julialang.org/downloads/)
- [Link to jupyter installation instructions](http://jupyter.org/install)

### Download this repository

If you're familiar with git, you can simply clone this repository.

<!-- TODO: add URL -->
```sh
$ git clone <URL>
$ cd intro_julia
```

Alternatively, download the zip file from github and decompress it.

### Activate and Instantiate the environment

The `Project.toml` and `Manifest.toml` files ensure that you can get the same
environment that I used to create this repository.
Assuming you downloaded the folder into your `Downloads` directory,
(note: this may be different on Wondows machines)

```julia
julia> cd(expanduser("~/Downloads/intro_julia"))
```

Then, we'll `activate` the current folder environment
and `instantiate` it to get all the packages we'll need.

```julia
julia> Pkg.activate("./")
"/Users/kev/computation/julia_playground/intro_julia/Project.toml"

julia> Pkg.instantiate()
  Updating registry at `~/.julia/registries/General`
  Updating git-repo `https://github.com/JuliaRegistries/General.git`
  # ...
```

## Additional Resources

- [julia documentation](https://docs.julialang.org/en/stable/)
- Basic Syntax - [Learn julia in Y minutes](https://learnxinyminutes.com/docs/julia/)
- Discussion and questions (transient) [Julia slack](http://julialang.slack.org)
- Discussion and questions (persistant) [Julia Discourse](http://discourse.julialang.org) (really, ask your questions!)
- [Stochastic Lifestyle](http://www.stochasticlifestyle.com/) (Christopher Rakaukas' blog)
- [Dispatch and Traits](https://white.ucc.asn.au/2018/10/03/Dispatch,-Traits-and-Metaprogramming-Over-Reflection.html) (Lyndon White blog post)
