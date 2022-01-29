---
title: "Python Virtual Environment & Packaging Workflow"
date: 2022-01-28T20:19:35-05:00
draft: false
---

<meta name="twitter:card" content="summary">
<meta name="twitter:site" content="@pmbaumgartner">
<meta name="twitter:creator" content="@pmbaumgartner">
<meta name="twitter:title" content="Python Virtual Environment & Packaging Workflow">
<meta name="twitter:description" content="I use conda and poetry. That's it.">
<meta name="twitter:image" content="https://i.ibb.co/LYz2DyQ/1537215809678.jpg">

In this blog post I'll walk you through the workflow I use for managing virtual environments and creating python packages.

I have a long list of criteria I've used to develop this workflow and have honed it over time since the start of my career as a data scientist. The factors that I've determined are critically important for choosing these tools are as follows:

- They work for me

Yep, that's it. I haven't actually done a thorough evaluation of what's available. All I know is that this works for me and I hope if you're reading this it'll help you find something that works for you.

## Managing Python Versions & Virtual Environments: `conda`

I use `conda` (installed via [`miniconda`](https://docs.conda.io/en/latest/miniconda.html)) to manage virtual environments and python versions installed on my computer. This mostly a historical artifact: when I started programming as a data scientist, the easiest way to get up and running with python was through the Anaconda installer. At the time (2014ish), there were some packages that were also easier to install via a `conda install` rather than a `pip install`, but I don't think that's been true since about 2017[^1]. 

In short, I create environments with `conda`, then use `pip install` and a `requirements.txt` inside the environment. I do _not_ use `conda`'s `environment.yml` file except in rare circumstances.

The command to create a new environment is `conda create -n <environment_name> python=<python_version>`. I do this often so I created a bash function for it (put this in your `~/.bash_profile` or `~/.zshrc`).

```bash
create_conda_env() {
    DEFAULT_PYTHON_VERSION="3.8"
    conda create -n "$1" python=${2:-$DEFAULT_PYTHON_VERSION}
}
```

Then call it like this:
```bash
create_conda_env new_env 3.9
```

I create environments per project, so the environment is usually associated with a folder of the same name. Using the above example, I'd make a folder called `new_env`. If you name the folder the same as the environment, you can do a 😎 cool trick 😎 where you activate the environment by the folder name by calling `conda activate ${PWD##*/}`. However, I _don't_ typically do this to activate the environment. Rather, I have it set up to automatically activate in VSCode and use the terminal there. To do this in VSCode, regardless of the folder name, open the command palette (`⇧⌘P`) and find `Python: Select Interpreter`. Select the environment you just created from this list. There is also a setting called `Python › Terminal: Activate Env In Current Terminal` you will want to turn on, otherwise sometimes the environment won't activate if you have a terminal open in VSCode before the python extension loads.

If I'm doing a project that's more analysis focused, or won't result in a python package, I manually track the main packages installed. That is, if I need `pandas`, I'll do a `pip install pandas`. Then I'll check what version was installed with `pip freeze | grep pandas` and copy that line to my `requirements.txt`. I know I could install everything I need and then just do a big `pip freeze > requirements.txt`, but I use the `requirements.txt` to indicate to any other user of this project what packages are primarily used (i.e. anything that gets `import`ed is in `requirements.txt`) and let any dependency versions resolve themselves when someone else does `pip install -r requirements.txt`.

## Creating Python Packages: `poetry`

This is a newer addition to my workflow. For most of my work I'm not creating a python package, I'm just following the previous process with `conda` and making sure my repositories include a `requirements.txt` that allows someone else to execute my code.

More recently I have been creating packages using `poetry`. Here's the main thing I like: it sets everything up so that relative and absolute imports ✨just work✨. Every time I've tried to create a package manually I get lost for days in Stack Overflow [`ImportError` hell](https://stackoverflow.com/questions/14132789/relative-imports-for-the-billionth-time).[^2] The side effect of imports working correctly is that you also don't have to worry about some `PATH` nonsense with your test suite! It sets up `pytest` and gives an example test that also ✨just works✨.

Here's how I use `poetry` with `conda`. I create a new virtual environment _first_, where the python version in that environment is the minimum I'll choose to support for that package. Then I'll activate that environment and `pip install poetry`. The side effect of this is that in the `pyproject.toml` file the python version will be set at the version you chose for that environment. This is better than installing `poetry` inside your `base` environment because it'll use whatever version of python is there and that might not be what you want. After this, I will then use `poetry` for package and dependency management (**not** `pip`) by adding new packages with `poetry add <package>` so that the required packages are tracked in the `pyproject.toml` and `poetry.lock` files.

The command to create a new poetry package is:

```bash
poetry new my-package
```

However, this will create a new folder with the package name as well, so I run this command from the same "parent level" folder I'd be at if I was using `conda` and doing a regular `mkdir` to start a new project folder. The key is to **not** make the folder for your package before running `poetry new`, just let `poetry` handle that for you.

For package distribution, 90% of the time I'm just using a `git` repo and uploading it to github. You can install python packages from git repositories by prefixing the url with `git+<URL_TO_REPO>`. This also works in `requirements.txt` files. The advantage here is that I am often rapidly changing parts of my packages early in development and I don't want to manually track versions when changes occur so freqently. The downside is that a user can't use `pip install -U` to update the package, they have to uninstall and reinstall from the repo.

If I'm working on a package that becomes a bit more stable and I want to publish it more widely, I'll then upload to PyPI. One cool thing to know is that there's a test version of PyPI called [TestPyPI](https://test.pypi.org/)! This is awesome! You can practice uploading a package here and not worry about messing anything up on the official PyPI. There is [a little bit of configuration needed](https://stackoverflow.com/a/68901875) to set this up with `poetry`, but it's well worth it if you're like me and rarely do the package publishing process.

## Wrap Up

That's it! That's how I use `conda` and `poetry` to manage python virtual environments and create python packages. 

[^1]: I was curious why this was so I did a little digging. I think can pin it down to the invention of wheels with binaries (circa 2013). This meant that the underlying non-python libraries used for things like numpy could be included in the package when things were installed with `pip`. Until this time, the ability to install system-specific precompiled binaries was `conda`'s selling point. I think it took a few years for packages in the data science python ecosystem to actually implement this (and users to realize it) before this became the mainstream way to install python packages if you were doing data science / scientific computing. For more, see this [Stack Overflow answer](https://stackoverflow.com/questions/20994716/what-is-the-difference-between-pip-and-conda/68897551#68897551). For more on wheels, check [this RealPython article](https://realpython.com/python-wheels/#python-packaging-made-better-an-intro-to-python-wheels).

[^2]: Remember my only criteria for this workflow is that it **works**. I have _no idea_ what it is doing that makes it work and what I was doing incorrectly before that made it not work. I am [DHH's dog](https://twitter.com/dhh/status/1463822670131351555?lang=en). 