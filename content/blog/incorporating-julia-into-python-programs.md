---
title: "Incorporating Julia Into Python Programs"
date: 2021-08-09T11:35:27-04:00
draft: false
---

<meta name="twitter:card" content="summary">
<meta name="twitter:site" content="@pmbaumgartner">
<meta name="twitter:creator" content="@pmbaumgartner">
<meta name="twitter:title" content="Incorporating Julia Into Python Programs">
<meta name="twitter:description" content="A collection of notes on how to get started writing Julia for inclusion in python programs.">
<meta name="twitter:image" content="https://i.ibb.co/vLsQRRr/Frame-9.png">

**Context:** I've recently been experimenting with porting portions of a simulation codebase from python to Julia. Setting up a productive development environment, using the packages (PyJulia & PyCall) that allow for communicating between python and Julia, and familiarizing myself with Julia enough to use those packages took quite a bit of time and experimentation. Here's my collection of notes including stumbling blocks, adaptations, and things I took forever to understand to make this process easier for others in the future.

## Prelude: Environment Preparation
There are a lot of things that can go wrong with virtual environments, Julia installation, and python linking, so I've found the easiest way to get things setup is using a Docker image with VSCode's dev containers. 

A basic `Dockerfile` that works for us looks like this:

```Dockerfile
FROM python:3.8

WORKDIR /app

RUN pip install julia jill ipython --no-cache-dir
# julia is pyjulia, our python-julia interface
# jill is a python package for easy Julia installation
# IPython is helpful for magic (both %time and %julia)
# Include these in your requirements.txt if you have that instead

RUN jill install 1.6.2 --confirm

# PyJulia setup (installs PyCall & other necessities)
RUN python -c "import julia; julia.install()"

# Helpful Development Packages
RUN julia -e 'using Pkg; Pkg.add(["Revise", "BenchmarkTools"])'
```

If you're using VSCode, you can set up a [dev container](https://code.visualstudio.com/docs/remote/containers) to use this image as a remote container, which allows you to code inside this environment and save all changes locally. To do this, create a `devcontainer/devcontainer.json` file at your project root with the following:

```json
// For format details, see https://aka.ms/devcontainer.json. For config options, see the README at:
// https://github.com/microsoft/vscode-dev-containers/tree/v0.187.0/containers/python-3
{
	"name": "PyJulia Env",
	"build": {
		"dockerfile": "../Dockerfile",
		"context": "../.",
	},
	// Set *default* container specific settings.json values on container create.
	"settings": {
		"python.pythonPath": "/usr/local/bin/python",
		"python.languageServer": "Pylance",
		"python.linting.enabled": true,
		"python.linting.pylintEnabled": true,
	},
	// Add the IDs of extensions you want installed when the container is created.
	"extensions": [
		"ms-python.python",
		"ms-python.vscode-pylance",
		"julialang.language-julia"
	],
	// We will use this later, leave commented out for now.
	// "postStartCommand": "julia -e 'import Pkg; Pkg.develop(path=\"MyPackage\")'",
}
```

The rest of this tutorial assumes you are working in a dev container with the `Dockerfile` and `.devcontainer` setup above. Alternatively, if you're able to get Julia and Python working with whatever setup you have, congrats!

(*Author's Note*: I have macOS 11.5.1 (Big Sur) and typically use `conda` for environment management and I kept getting a segfault when trying to import PyJulia, hence the docker approach.)

## Using PyJulia

Now lets write some Julia! Open up a python REPL (we're using IPython, though I'm using the standard `>>>` to indicate an input) and run the following:

```python
>>> from julia import Main
>>> Main.println("I'm printing from a Julia function!")
I'm printing from a Julia function!
```

This will import Julia, start up the Julia interpreter, and print the statement using the Julia `println` function.

`Main` is the global namespace of the Julia interpreter. What does this mean? Anything you'd have available if you started up a Julia REPL can be called with `Main.<function>`. If you were in a Julia REPL, you'd just type `<function>`, but if that worked the same way in python we'd have collisions with python functions and not know when we're using Julia. If you have some frequently used Julia functions, you can rewrite references to functions to be more clear:

```python
>>> rand_jl = Main.rand
>>> rand_jl(3)
array([0.74949659, 0.50848869, 0.67415476])
```

The above example also illustrates the automatic type conversion that's going to happen when you pass things back and forth between python and Julia. We passed our Julia function a python `int`, which Julia has a native type for (`Int`). The return object is a `numpy` array; a python `list` in Julia is a `Vector` or `Array`, and will always get converted to a `numpy` array on the return trip.

Types are very important in Julia because of multiple dispatch. If you need to know the type of something in Julia after PyJulia converts it, you can call `Main.typeof` to get the type of the thing. This returns a `PyCall.jlwrap` object, which is used when there's no automatic conversion to a python type in the returned object. The output is a Julia `DataType`, which has no equivalent in Python (this is shown in the last example).

```python
>>> Main.typeof(3)
<PyCall.jlwrap Int64>

>>> Main.typeof([1, 2, 3, 4])
<PyCall.jlwrap Vector{Int64}>

>>> Main.typeof((1, 2, 3, 4))
<PyCall.jlwrap NTuple{4, Int64}>

>>> Main.typeof({"key" : "value"})
PyCall.jlwrap Dict{Any, Any}>

>>> Main.typeof("String!")
<PyCall.jlwrap String>

# Meta!
>>> Main.typeof(Main.typeof(3))
<PyCall.jlwrap DataType>
```

This is **really important** to understand and is probably the #1 reason why something you think should work in Julia doesn't. You will need to know the types of the objects and convert them when you see a Julia function you want to use that takes an argument that is **not** one of these automatically converted types.

For example, lets say I want an array of random integers. In Julia I can do this with the `rand` function by passing it a type and the number of objects of that type I want, e.g. `rand(Int, 40)`. But how do you get the `Int` object, which is a `Type`, which doesn't exist in python, to pass this function?

Remember you can access any Julia objects in the global namespace with `Main`, so we can do this like:

```python
# Our renamed function
>>> rand_jl(Main.Int, 4000)
array([-6472840723767327579,  -781971358995470358,  4369709767867909283,
       ...,   342925672839826321, -4289054831996301088,
        2127432410244721707], dtype=int64)
```

## Packages

Rather than a distinct tool like `pip` for package management, Julia includes a Julia package for package management called `Pkg`. You can also call it using PyJulia. Here we'll install `StatsBase` (still inside ipython):

```python
>>> Main.eval('using Pkg; Pkg.add("StatsBase")')
```

Here you also see the use of `Main.eval`, which allows you to write and evaluate raw Julia code. **Note the single quotes** for the whole string (in python) and double quotes inside. This is necessary because unlike python, quote types are not interchangeable in Julia. Since this statement is evaluated by Julia, we have to understand how Julia interprets it: a single quote is used for single characters in Julia, and double quotes for strings. If you try the above with `'StatsBase'` in single quotes, you'll get:

```
JuliaError: Exception 'syntax: character literal contains multiple characters' occurred while calling julia code:
using Pkg; Pkg.add('StatsBase')
```

Once you have packages installed, you can import those into python with normal import syntax. For example, we have `StatsBase` installed, so we can do this:

```python
>>> from julia.StatsBase import sample # imports single functions
>>> sample([1, 2, 3, 4, 5])
3

>>> import StatsBase # imports whole module
>>> StatsBase.sample([1, 2, 3, 4, 5])
1
```

At this point, your brain might look like this: 🤯. You've now got a python API to ✨**any Julia package you can install**✨.

## PyJulia 201

As mentioned above, the #1 reason things will not work when you think they should is that the inputs are the wrong type. Julia uses a concept called multiple dispatch to determine which method to actually call. This means that there are really *several* versions of the same *function*, each version is called a *method*, and the choice of which method gets used depends on the type(s) of the input.

As an example let's say we wanted to get a weighted sample using `sample` in `StatsBase`.  You can see all the methods available for a function with a call to `methods(<function>)`. Here's a truncated output for `sample`:

```python
>>> Main.methods(StatsBase.sample) 
<PyCall.jlwrap # 14 methods for generic function "sample":
[1] sample(wv::StatsBase.AbstractWeights) in StatsBase at /root/.julia/packages/StatsBase/IiL4F/src/sampling.jl:558
[2] sample(a::AbstractArray) in StatsBase at /root/.julia/packages/StatsBase/IiL4F/src/sampling.jl:432
...
[14] sample(rng::Random.AbstractRNG, a::AbstractArray{T, N} where N, wv::StatsBase.AbstractWeights, dims::Tuple{Vararg{Int64, N}} where N; replace, ordered) where T in StatsBase at /root/.julia/packages/StatsBase/IiL4F/src/sampling.jl:936>
```

In the previous section, the method `[2]` was actually called when we passed it our array. We can verify this with the `which` function -- which itself takes a function and a tuple of types as its arguments and returns the method that would be called with arguments of those types.

```python
>>> a = [1, 2, 3, 4, 5]
>>> Main.which(sample, (Main.typeof(a),))
<PyCall.jlwrap sample(a::AbstractArray) in StatsBase at /root/.julia/packages/StatsBase/IiL4F/src/sampling.jl:432>
```

Now we have a dilemma: how can we do weighted sampling (i.e. call method `[1]`) when our type is automatically converted? Let's try first casting our weighted array to a type that would cause the first method to be called. 

In this example, we'll use `ProbabilityWeights`.

```python
>>> from julia.StatsBase import sample, ProbabilityWeights

>>> wv = ProbabilityWeights([0.1, 0.2, 0.3, 0.3, 0.1])
>>> sample(wv)
0.3
```

Well, that's not right! We expect this method to return the index of the sampled value (`1 to len(array)`,  and yes, we'll talk about 1-indexing). It appears as if it's still calling the wrong method because of the automatic type conversion. We can verify with `Main.typeof(wv)` as it indicates its a `Vector{Float64}`, when we want `ProbabilityWeights{Float64, Float64, Vector{Float64}}`.

It's time to write some Julia code to solve this problem! 

```python
>>> weighted_sample_def = """
using StatsBase

function weighted_sample(weights)
    wv = ProbabilityWeights(weights)
    return sample(wv)
end
"""

>>> Main.eval(weighted_sample_def)
>>> Main.weighted_sample([0.1, 0.2, 0.3, 0.3, 0.1])
4
```

Here we've defined the Julia function as a string, told the Julia interpreter to evaluate that code, which brought that function into the Julia namespace, and then called that function using Julia. The function itself handles the conversion from `Vector` to `ProbabilityWeights` prior to calling `sample`. Note that we had to import `StatsBase` into our Julia namespace: we only imported it as a python module into our python namespace before. (*Authors Note*: I'm not sure this is exactly right, I was still a little surprised by having to reimport this in Julia.)

This works in a pinch, but if we have lots of Julia code we don't want to write and manage that as a series of python strings 🤢.

## Writing Julia Code in .jl files

Remember way back at the beginning when we created a docker image with Julia and connected to the container using VSCode? That's really going to come in handy now. We also installed the Julia VSCode extension in that step, so we can have our editor help out in writing Julia code.

Let's create a new file called `functions.jl`. You may have to open up a Julia REPL and install the `StatsBase` package - or run `julia -e 'using Pkg; Pkg.add("StatsBase")'` from your shell, for this to work (if you didn't do this above).

In this file, we're going to write the Julia code for our function. 

```julia
# functions.jl
using StatsBase

function weighted_sample(weights)
    wv = ProbabilityWeights(weights)
    return sample(wv)
end
```

Great, now we've got the Julia code in it's own file. We can now import it into python for use in our package.

This might be good time to restart your python REPL to clear all the state and function names we had defined in there. When you restart it, rerun `from julia import Main`.

We're now going to use `include` to "import" our Julia code into our Main Julia namespace in python, then attempt to call our weighted sample function. To do that, run the following:

```python
>>> from julia import Main
>>> Main.include("functions.jl")
<PyCall.jlwrap weighted_sample>
>>> wv = [0.1, 0.2, 0.3, 0.3, 0.1]
>>> Main.weighted_sample(wv)
2
```

Excellent! We can now write and manage our Julia code in a bit more organized fashion. One thing to note is that if you update any Julia functions in your `.jl` file, all you have to do is rerun `Main.include`.

### 1-indexing and Broadcasting

Let's sidestep a trivial programming debate and get to the point: Julia is 1-indexed, python is 0-indexed. This means our `weighted_sample` function is going to return values 1 higher than we would expect in python.

If we wanted to look up the weight associated with the index returned from a sample, we'll see this problem:

```python
>>> wv = [0.1, 0.2, 0.3, 0.3, 0.1]
>>> for _ in range(100):
>>>     index = Main.weighted_sample(wv)
>>>     print(wv[index])
0.1
IndexError: list index out of range
```

This highlights an important issue: **it's a good idea to unit test your functions with very basic assertions.** This will expose any small issues like this, which will inevitably arise due to the complexity of shipping objects back and forth between languages.

To fix this, we need to update our function to subtract 1 from the return value:

```julia
function weighted_sample(weights)
    wv = ProbabilityWeights(weights)
    return sample(wv) - 1
end
```

If we have a function that returns multiple index values, we can use broadcasting to apply this subtraction to the whole vector. To broadcast a function (depending on the function), you either prefix it (if it's an operator) or suffix it (if it's a function) with a period (`.`).

For example, let's say we want to implement a function that finds the index of all values in an array greater than some number, like so:

```julia
function array_gt(array, value)
    return findall(>(value), array) .- 1 
end
```

Now if we checked this in python, we should get the following (assuming you've rerun `include(<yourfile>)`):

```python
>>> Main.array_gt([5, 4, 3, 2, 1], 2)
array([0, 1, 2], dtype=int64)
```

Look at that, broadcasting and avoiding `IndexError` like a pro.

## Helpful tools for `include`

If we want to include this code in a python codebase, there are a few helpful tools in python's `pathlib` to help.

Getting the parent directory of the currently running file can be helpful, if you know where your Julia code is relative to that file. Or in the REPL, it might be helpful to know the current working directory and import from there. For example:

```python
>>> from pathlib import Path

# File being executed Example
>>> file_dir = Path(__file__).parent
>>> Main.include(f"{file_dir}/functions.jl")
<PyCall.jlwrap weighted_sample>

# Working Directory Example (for REPL)
>>> Main.include(str(Path.cwd() / "functions.jl"))
<PyCall.jlwrap weighted_sample>
```

## Benchmarking & Timing

Ready to get into the nitty-gritty? Let's go!

It's a good idea to time our Julia functions, especially if they have python equivalents. Let's do that below. (We're in an IPython REPL here, and we're going to re-import our functions)

```python
>>> Main.include(str(Path.cwd() / "functions.jl"))
<PyCall.jlwrap weighted_sample>
>>> wv = [0.1, 0.2, 0.3, 0.3, 0.1]

>>> %time Main.weighted_sample(wv)
CPU times: user 14.7 ms, sys: 0 ns, total: 14.7 ms
Wall time: 14.5 ms
1

>>> %time Main.weighted_sample(wv)
CPU times: user 787 µs, sys: 0 ns, total: 787 µs
Wall time: 891 µs
1
```

Wait, so the first time I use it it takes `14.5ms`, and the second time it's 16x faster and takes `891µs`? What? 

If you've ever used `numba`, you might know what's going on. The first time we call a function in Julia, it's compiling it. 

Not only that, but each package that's imported for the first time also has a loading time cost. See the example below.

```python
>>> from julia import Main

>>> %time Main.eval("using StatsBase")
CPU times: user 211 ms, sys: 7.06 ms, total: 218 ms
Wall time: 220 ms

>>> %time Main.eval("using StatsBase")
CPU times: user 270 µs, sys: 35 µs, total: 305 µs
Wall time: 322 µs
```

In projects with code that runs for a long time, this isn't going to matter too much. There are python packages that also take as long, if not longer, to import. But, if you are doing benchmarking at this level, it's very good to be aware of this fact if you're doing direct comparisons between Julia and Python equivalents. 

## Getting Organized & Scaling Up: Developing a Julia Package

As soon as you have more than a handful of functions, you'll probably want some additional structure to organize your code. Wouldn't it be nice to have something like `from julia import MyPackage` and have all your Julia functions nicely namespaced in python instead of a hodge-podge of include statements and file path shenanigans? (Yes, it would!) Additionally, if you were looking to give this codebase to a colleague to help you develop, one problem is that the environment and project isn't replicable. We added `StatsBase` from the REPL, but if we gave our current codebase to a colleague, we'd have to tell them to open a REPL and run the install commands above after they got set up. We could add it to the `Dockerfile`, since we already have `Revise` and `BenchmarkTools`, but those are there solely to ease the development process: they aren't requirements for executing our Julia code.

To facilitate organizing our code, we're going to create a Julia package. To do that, we'll load up the Julia REPL and run the following:

```
julia> ]
(@v1.6) pkg> generate MyPackage
  Generating  project MyPackage:
    MyPackage/Project.toml
    MyPackage/src/MyPackage.jl
```

This might seem weird, but, Julia comes with a package management REPL. Up to this point, we've been using the functional API in `Pkg`. This time, we opened a Julia REPL and hit `]` which entered the `Pkg` REPL. If we wanted to use the functional API, we could have done the following as well:

```julia
julia> import Pkg; Pkg.generate("MyPackage")
```

We've now got a new directory named for our package, and two generated files for our package within it: a `Project.toml` which will list the dependencies when we add them, and a `src/MyPackage.jl` which will house our package code.

While we're in the package REPL, let's add our `StatsBase` dependency. We first have to _activate_ the environment for our package by typing `activate MyPackage`. Once you do this, you should see the REPL change to indicate the active environment. Then, type `add StatsBase` to add the dependency.

```julia
(@v1.6) pkg> activate MyPackage
  Activating environment at `/workspaces/pyjulia-demo/MyPackage/Project.toml`

(MyPackage) pkg> add StatsBase
    Updating registry at `~/.julia/registries/General`
	[... packages being installed ...]
Precompiling project...
  1 dependency successfully precompiled in 2 seconds (11 already precompiled)
```

You should see that the `Project.toml` file now includes a line with `StatsBase` and a UUID. Great, we've successfully declared our dependency.

Now let's update our package with our functions. Since we already had our code in a `functions.jl` file, copy this to the `MyPackage/src` folder. Then, update the `src/MyPackage.jl` file to the following:

```julia
module MyPackage

export weighted_sample

include("functions.jl")

end # module
```

The one new line here, `export weighted_sample`, is indicating that we want to export that function as a part of the public API of this module. 

(*Authors Note:* I had to do this initially, but then I removed the `export` statement later and things still worked. 🤷)

Finally, we need to tell our development container that we want to use this package locally in development mode. Back in the `.devcontainer/devcontainer.json` file, you can uncomment the last line, which looks like this:

```json
"postStartCommand": "julia -e 'import Pkg; Pkg.develop(path=\"MyPackage\")'",
```

This will add the local file path as a package in development mode.

Now, rebuild your dev container in VSCode. At the end of the rebuild process, you should see a new terminal window that will run the above command, installing our package in development mode.

To check that everything is working correctly, open the Julia REPL and try loading our package:

```julia
julia> using MyPackage
[ Info: Precompiling MyPackage [e94922da-031a-4938-8ed5-8bef1659fee7]
```

We're in business! 

Since our objective is to be able to use Julia from python, let's open up our IPython REPL and also verify that our package works.

```python
>>> from julia import MyPackage
>>> MyPackage.weighted_sample([0.1, 0.2, 0.3, 0.3, 0.1])
4
```

Look at that: we can just import the package like a regular python package! No more `include` scattered throughout. 

## Continuing Development

You'll likely want to be able to continue development of your package now that we've got things setup. One helpful tool here is the `Revise.jl` package we installed through the `Dockerfile`. You can even set this up to work with ipython, similar to the `%autoreload` magic, so that any change in your Julia package code is reflected in your python environment. Let's check that out.

Start up an ipython REPL, and then immediately type these magic commands:

```
%config JuliaMagics.revise = True
%load_ext julia.magic
```

This will turn on `Revise` for this REPL session. Let's load our package and play around.

```python
>>> from julia import MyPackage
>>> MyPackage.weighted_sample([0.1, 0.2, 0.3, 0.3, 0.1])
1
```

Nothing new here. Keeping your IPython REPL session open, let's add a new function to our package, in `MyPackage/src/functions.jl`:

```julia
# MyPackage/src/functions.jl
newfunc() = println("A new function, cool!")
```

Now, back in the REPL, you should be able to call this new function:

```python
>>> MyPackage.newfunc()
A new function, cool!
```

You can also add these commands to your [ipython config](https://ipython.org/ipython-doc/3/config/intro.html) to automatically do this upon startup.

## Wrap Up & Next Steps

You now have a workflow that should allow you to develop a Julia package alongside your python code. As you continue with development, some natural next steps might be:

- Finalize your Julia package: pulling it into it's own repo, checking it into version control, and publishing it to a remote and optionally a registry. You'll then need to [free](http://pkgdocs.julialang.org/v1/managing-packages/#developing) your development version and add the remote version. 
- Create a Julia [sysimage](https://julialang.github.io/PackageCompiler.jl/dev/sysimages.html) to speed up the startup time. This will be helpful if you add a lot of dependencies with longer startup times. In addition, [PyJulia](https://pyjulia.readthedocs.io/en/latest/sysimage.html) has a function to do this specifically for the dependencies it installs. 


## Helpful References

- [PyJulia Documentation](https://pyjulia.readthedocs.io/en/latest/index.html)
- [Rewriting Pieces of a Python Codebase in Julia | Satvik Souza Beri | JuliaCon2021](https://www.youtube.com/watch?v=EnkfGuH6Qhg)
- Stack Overflow: [R renv vs Python virtual environments vs Julia environments](https://stackoverflow.com/a/66047402) 
- [Pkg.jl Glossary](http://pkgdocs.julialang.org/v1/glossary/)


## The "Why Didn't You" Section

At several points you might be saying "but Peter, why didn't you X?" Here's why:

- **Why didn't you** add the package in development mode through the Dockerfile?

In development mode, Julia links to a local package path. Within our `Dockerfile` during the build process, we're operating in a different directory than the working directory of the development container once we're in it. Thus, we need to declare the path to the dev version of the package relative to the source tree we're working in, not the docker build context.

- **Why didn't you** create a new Julia environment, then declare your local package as a dependency?

Coming from python, my mental model of environment and package management is a little different than how Julia's works. I wanted to get a reader up and running without getting bogged down in Julia's package/project/environment documentation. My understanding is that a "more correct" way to do this, which would be required once the package is finalized, would be to create a root level project and `Project.toml`, then declare our finalized version of `MyPackage` as a dependency. This would also be required if we were developing locally, not inside the dev container. However, getting there within this tutorial means potentially managing nested `Project.toml` files and possibly confusing the reader... but maybe I'm the only one who is confused.

