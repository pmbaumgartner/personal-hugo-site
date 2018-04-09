---
title: How to Test IPython Magic
date: 2018-02-16
---

## TL;DR

If you want to test `ipython magics` you can do the following:

1. Import the global ipython app with `from IPython.testing.globalipapp import get_ipython`
2. Crete an object with the global ipython app with `ip = get_ipython()`
3. Load your magic with `ip.magic('load_ext your_magic_name')`
4. Run your magic with `ip.run_line_magic('your_magic_function', 'your_magic_arguments')`
5. (Optional) Access results of your magic with `ip.user_ns` (ipython user namespace).

An example test using pytest looks like this:

```python
import pytest
from IPython.testing.globalipapp import get_ipython

ip = get_ipython()
ip.magic('load_ext excelify')

def test_nonexistent_object():
    with pytest.raises(NameError):
        ip.run_line_magic('excel', 'nonexistentobject')
```

## More Details

### Background

IPython Magics are a nice way to apply command-line like behavior from within an ipython kernel or jupyter notebook. Several magics come built-in (`%%time` is my favorite), and it's also possible to write your own magics to help with common tasks.

I ventured into the business of writing a custom magic by creating one that helped export pandas objects into Excel files. The idea emerged as I was running through a notebook where I needed to document a bunch of intermediate data frames, and rather than save them until the end of the notebook and loop through them to export, I figured a magic might be a nice way to handle this export within a cell.

I've been working on testing my code more and I figured this was a good project to continue practicing. I understood how to write basic tests that excecuted python code, but running a magic to test present an intersting case. In an ipython shell, magics are prepended with a `%` and work as expected, but if you type a magic into a regular python shell, it'll give you a `SyntaxError`.

Luckily, ipython itself is well tested and they provide access to a shell object that you can send commands to. However, the documentation around how to use this to actually write tests is patchy. I pieced together my understanding from going through the tests for the [standard magics](https://bitbucket.org/rpy2/rpy2/src/d5d60e9a0f684c27015fa29c26bbb7fd75863bc2/rpy/ipython/tests/test_rmagic.py?at=default&fileviewer=file-view-default) as well as the tests for the [rpy2 magics](https://bitbucket.org/rpy2/rpy2/src/d5d60e9a0f684c27015fa29c26bbb7fd75863bc2/rpy/ipython/tests/test_rmagic.py?at=default&fileviewer=file-view-default).

### Example Detail

First, we use the `get_ipython` method to get a global ipython object. If you actually run this first line in a standard python shell, you'll see something interesting:

```python
>>> from IPython.testing.globalipapp import get_ipython
>>> ip = get_ipython()
In :
```

This will both create a shell object that you can reference with `ip` as well as turn that standard python shell into an ipython shell. 

Second, we load the magic with `ip.magic('load_ext excelify')`. The `.magic()` method basically does the of prepending the argument provided with a `%` if you were in an ipython shell. 

Third, we use the `run_line_magic()` method on the `ip` object to run our line magic. This method takes two arguments: the name of the magic function and the remaining of the arguments for that magic. If you were in an ipython shell, this is the equivalent of running `%{magic name} {magic arguments}`. Note: We can also load in your extension by running `ip.run_line_magic('load_ext', 'excelify')` instead of using the `magic()` method.

Now that we've put all those parts together, we can use these methods to write more tests. There are other related methods if you're testing magics that might be helpful, including:

- `run_cell_magic(magic_name, magic_args, cell)`
- `run_line_magic(magic_name, magic_args)`
- `magic(magic_line)`

**Potential Gotcha:** If you want to run a magic that doesn't take any arguments, you'll need to pass an empty string for `magic_args` in the methods above (rather than leave the argument empty or `None`)