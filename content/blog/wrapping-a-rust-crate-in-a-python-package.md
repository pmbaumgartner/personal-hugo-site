---
title: "Wrapping a Rust Crate in a Python Package"
date: 2023-01-05T06:43:13-05:00
draft: false
---

<meta property="og:title" content="Wrapping a Rust Crate in a Python Package (Development Journal)" />
<meta property="og:description" content="A development journal on the process to wrap a rust crate as a python package" />
<meta property="og:type" content="website" />
<meta property="og:url" content="https://peterbaumgartner.com/blog/wrapping-a-rust-crate-in-a-python-package/" />
<meta property="og:image" content="https://i.postimg.cc/NFhNpxK5/crapwrap.jpg" />

In this blog post I'll walk through step-by-step how I wrapped the [`voronoice`](https://docs.rs/voronoice/0.2.0/voronoice/) Rust crate and created a Python package with it. It's written as a development journal, where I walk through my thought process and document the code and errors we're getting along the way. My hope is that this makes the process a little more approachable for beginners and adds some transparency to the process.

## Motivation

Right now I primarily use `scipy.spatial.Voronoi` (which uses [Qhull](http://www.qhull.org/)) when making Voronoi diagrams. Here's here's how I typically use it:

```python
from scipy.spatial import Voronoi

v = Voronoi(
    some_points
    + [(-1000, -1000), (W + 1000, -1000), (W + 1000, H + 1000), (-1000, H + 1000)],
)

verts = v.vertices
regions = v.regions

regions = [i + i[0:1] for i in regions if len(s) > 0 and -1 not in s]
shapes = [verts[i] for i in regions]

voronoi_cells = [geo.Polygon(s) for s in shapes]
```

In this API, the `vertices` attribute returns all the points of the diagram, which are then indexed by `regions` to get the polygons for each individual cell. I also filter out those that have `-1` as an element because it means that they extend infinitely into a certain direction. I am 100% sure I took this code from StackOverflow and barely modified it.

One thing I would like is to know the neighbors of each cell. After doing some research, I learned the `voronoice` Rust crate has a way to access this - as well as provide the other information we also get above. So, let's learn how to wrap that crate with [PyO3](https://pyo3.rs/) and make it a python package.

Here are the questions I have just starting this out:
- Should I replicate the whole API? Or come up with my own data structure that has everything I need?
- `voronoice` has a `BoundingBox` struct. How should I represent that as an input? 
	- It uses center, top right, and bottom left as input arguments. I think I'd prefer (x1, y1, x2, y2), so I can modify that API as well.
- How do I set up a pipeline to make sure this can be distributed to all available platforms, since it will require compilation?

I figure I'll come up with answers to those questions as I'm developing, so let's get started!

Of course we need a name, so we'll just ask ChatGPT...

<img src="https://i.ibb.co/XjrfgHk/Pasted-image-20221229061141.png" alt="Asking ChatGPT what we should name a Voronoi diagram package">

And `voronoiville` was born!

## Development

I know I'm going to use `pyo3` and `maturin`. I'll start with reading the docs for [maturin](https://www.maturin.rs/). I use `conda` and not `pyenv`, so lets create with a fresh virtual environment, install `maturin`, and create a new project.

```bash
conda create -n voronoiville python=3.8
pip install maturin
maturin new -b pyo3 voronoiville
```

Now I just want to test whether I can call the example function they provide from python. This is our first test of this development loop. I need to compile my rust code, which I do like this:

```bash
maturin develop
```

Which outputs:

```txt
📦 Built wheel for CPython 3.8 to /var/folders/49/l0xcvz7x4hj5_6434zsf6lhc0000gn/T/.tmp3wq1UE/voronoiville-0.1.0-cp38-cp38-macosx_11_0_arm64.whl
🛠 Installed voronoiville-0.1.0
```

Okay so `maturin` outputs a python wheel to a temporary directory and installs it. Let's start up a python REPL and check things out!

```python
>>> import voronoiville
>>> voronoiville.sum_as_string(3, 4)
'7'
```

Yay! 

So we can compile rust and use it in python! That's awesome.

The main thing we'll be doing is wrapping another crate, so we need to add that crate as a dependency of our package. 

```bash
cargo add voronoice
```

Most of the setup is out of the way now, let's write a test function to see if we can return something that's converted from rust native types to python types. I'll just write a dummy function that's runs a `voronoice` documentation example with a slight modification to return some voronoi cell vertices. It looks like this:

```rust
#[pyfunction]
fn voronoi() -> Vec<(f64, f64)> {
    use voronoice::*;
    // creates a voronoi graph from generated square sites, within a square bounding box of side 5.0
    // and runs 4 lloyd relaxation iterations to spread sites in the region
    let v: Voronoi = VoronoiBuilder::default()
        .generate_square_sites(10)
        .set_bounding_box(BoundingBox::new_centered_square(5.0))
        .set_lloyd_relaxation_iterations(4)
        .build()
        .unwrap();
    let vertices = v
        .vertices()
        .iter()
        .map(|v| (v.x, v.y))
        .collect::<Vec<(f64, f64)>>();
    vertices
}
```

`#[pyfunction]` is what's doing the magic here. We've declared this function returns `Vec<f64, f64>` which `PyO3` can automatically convert to python types. Let's go back to python and see if we can run this:

```python
>>> from voronoiville import voronoi
>>> voronoi()
[(-0.03380489650770409, -0.04864118314550073), ... ]
>>> len(voronoi())
221
```

Yay! So we have our basic scaffolding for our function, we need to actually make it take some arguments now.

Let's change it so that we can provide our own input points. I tried the code below at first:

```rust
fn voronoi(points: Vec<(f64, f64)>) -> Vec<(f64, f64)> {
	let sites: Vec<Point> = points.iter().map(|(x, y)| Point { x, y }).collect();
```

Alright, our first conundrum! This is giving me:

```
error[E0308]: mismatched types
  --> src/lib.rs:12:67
   |
12 |     let sites: Vec<Point> = points.iter().map(|(x, y)| Point { x, y }).collect();
   |                                                                   ^ expected `f64`, found `&f64`
   |
help: consider dereferencing the borrow
   |
12 |     let sites: Vec<Point> = points.iter().map(|(x, y)| Point { x, y: *y }).collect();
   |                                                                   ++++
```

Okay so the rust compiler is telling us _exactly_ what we need to do. I thought I could be clever by using the [field init shorthand](https://doc.rust-lang.org/book/ch05-01-defining-structs.html#using-the-field-init-shorthand), but alas, I need to dereference the values.

Once we fix that issue, by just using the code the compiler provides us, we are able to compile (with `maturin develop`) and try again!

```python
>>> from voronoiville import voronoi
>>> points = [(0, 0), (0, 1), (1, 0)]
>>> voronoi(points)
[(-0.05610501383113076, 0.04530452444563185), (-2.5, -2.5), ...]
```

Look at that! It even automatically casted our integer arguments to floats. 

Next I would like to create our own `BoundingBox` class that can take `(x1, y1, x2, y2)` as arguments and convert it to the right structure for `voronoice`.

In this case, we already have a thing named `BoundingBox` imported  from `voronoi`, so we'll name our struct `BoundingBoxPy`. The way to expose this as a class in python is with the `#[pyclass]` [annotation](https://doc.rust-lang.org/book/ch19-06-macros.html). Then, we can change the name of this class when we use it in python, it's just called `BoundingBox`, and not awkwardly `BoundingBoxPy`.

Then we need to provide a way to create this object in python. An [important note](https://pyo3.rs/v0.17.3/class#constructor): "By default it is not possible to create an instance of a custom class from Python code." In order to be able to create this in python, we create an `impl` block for `BoundingBoxPy`, annotate it with `#[pymethods]`, then annotate this specific method with `#[new]`. This creates the equivalent of an `__init__` method in python. Note that the method name doesn't also have to be `new`, which is typically used for constructors of rust objects - so I'll call it `init` to avoid confusion.

```rust
#[pyclass(name = "BoundingBox")]
struct BoundingBoxPy {
    x1: f64,
    y1: f64,
    x2: f64,
    y2: f64,
}

#[pymethods]
impl BoundingBoxPy {
    #[new]
    fn init(x1: f64, y1: f64, x2: f64, y2: f64) -> Self {
        BoundingBoxPy { x1, y1, x2, y2 }
    }
}
```

Then we have to add this class to our module. There's a block of code in the file that was generated at the bottom we haven't explored yet. It currently looks like this (after we add `BoundingBoxPy`).

```rust
#[pymodule]
fn voronoiville(_py: Python, m: &PyModule) -> PyResult<()> {
    m.add_function(wrap_pyfunction!(sum_as_string, m)?)?;
    m.add_function(wrap_pyfunction!(voronoi, m)?)?;
    m.add_class::<BoundingBoxPy>()?;
    Ok(())
}
```

The line before `Ok(())` is where we've added our bounding box class. Let's test it out in python!

```python
>>> from voronoiville import BoundingBox
>>> BoundingBox(1,2,3,4)
<builtins.BoundingBox object at 0x1056b12f0>
```

Great, it works (with `int -> float` casting again!), but we don't have a nice representation for it. We can fix that quickly! PyO3 supports the magic methods we're used to in python in an `impl` block annotated with `#[pymethods]`, so our `impl` block looks like this now:

```rust
#[pymethods]
impl BoundingBoxPy {
    #[new]
    fn init(x1: f64, y1: f64, x2: f64, y2: f64) -> Self {
        BoundingBoxPy { x1, y1, x2, y2 }
    }
    fn __repr__(&self) -> String {
        format!(
            "BoundingBox({}, {}, {}, {})",
            self.x1, self.y1, self.x2, self.y2
        )
    }
    fn __str__(&self) -> String {
        self.__repr__()
    }
}
```

And now...

```python
>>> from voronoiville import BoundingBox
>>> BoundingBox(1,2,3,4)
BoundingBox(1, 2, 3, 4)
```

Yay! Alright, now we need to write a trait that converts `BoundingBoxPy` into `voronoice::BoundingBox`. The bounding box from `voronoice` uses a different representation of a rectangle for it's constructor, so we'll have to do some basic math here.

```rust
impl From<BoundingBoxPy> for BoundingBox {
    fn from(value: BoundingBoxPy) -> Self {
        let width = value.x2 - value.x1;
        let height = value.y2 - value.y1;
        let center = Point {
            x: (value.x1 + value.x2) / 2.0,
            y: (value.y1 + value.y2) / 2.0,
        };
        BoundingBox::new(center, width, height)
    }
}
```

Now we update our function and add the bounding box as an argument. Since we've implemented `From` for `BoundingBox`, we can add a type annotation for the `boundingbox` variable and call the `into` method on `BoundingBoxPy`:

```rust
#[pyfunction]
fn voronoi(points: Vec<(f64, f64)>, bounding_box: BoundingBoxPy) -> Vec<(f64, f64)> {
    let sites: Vec<Point> = points.iter().map(|(x, y)| Point { x: *x, y: *y }).collect();
    let boundingbox: BoundingBox = bounding_box.into();
    let v: Voronoi = VoronoiBuilder::default()
        .set_sites(sites)
        .set_bounding_box(boundingbox)
        .set_lloyd_relaxation_iterations(4)
        .build()
        .unwrap();
    let vertices = v
        .vertices()
        .iter()
        .map(|v| (v.x, v.y))
        .collect::<Vec<(f64, f64)>>();
    vertices
}
```

Let's test this out!

```python
from voronoiville import BoundingBox, voronoi
points = [(0, 0), (0, 1), (1, 0)]
voronoi(points, BoundingBox(-2.5, -2.5, 2.5, 2.5))
[(-0.05610501383113076, 0.04530452444563185), (-2.5, -2.5), (-2.5, 2.5), (2.5, 2.5), (2.5, -2.5), (4435173994.765353, -8962657621.199606), (-9999541625.05059, 95745966.03161913), (4690381324.290731, 8831779154.45233), (1.2475346735079387, 2.5), (1.2034397454112338, -2.5), (-2.5, 0.06870490568260464)]
```

We did it! That makes me think it would be a good idea to create a test to make sure our BoundingBox works correctly. We'll add this test at the bottom of our main `lib.rs` file.

```rust
#[cfg(test)]
mod tests {

    use voronoice::BoundingBox;

    use crate::BoundingBoxPy;

    #[test]
    fn bbox_conversion() {
        let voronoi_bbox = BoundingBox::new_centered_square(5.0);
        let python_bbox: BoundingBox = BoundingBoxPy::init(-2.5, -2.5, 2.5, 2.5).into();
        assert_eq!(voronoi_bbox, python_bbox);
    }
}
```

And we run `cargo test` and get...

```txt
error[E0369]: binary operation `==` cannot be applied to type `voronoice::BoundingBox`
  --> src/lib.rs:86:9
   |
86 |         assert_eq!(voronoi_bbox, python_bbox);
   |         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
   |         |
   |         voronoice::BoundingBox
   |         voronoice::BoundingBox
   |
   = note: this error originates in the macro `assert_eq` (in Nightly builds, run with -Z macro-backtrace for more info)

For more information about this error, try `rustc --explain E0369`.
```

Okay so I think this is because `BoundingBox` doesn't implement `Eq` or `PartialEq`. This means `voronoice` hasn't defined what it means for two bounding boxes to be equal. Which makes sense because when would you ever want to compare two bounding boxes when you're just using a single one to construct Voronoi diagrams?

Rather than implement that ourselves, we'll  just check whether some attribute of our two bounding boxes is equal, like `.corners()`. Changing the assertion to this:

```rust
assert_eq!(voronoi_bbox.corners(), python_bbox.corners());
```

And then our test passes!

Some experimentation now. What I really want is a collection of `VoronoiCell` [structs](https://docs.rs/voronoice/0.2.0/voronoice/struct.VoronoiCell.html) from `voronoice`, so I can get all the associated data (vertices, points, neighbors). What happens if I just attempt to return `Vec<VoronoiCell>`? Let's just try and change the method signature first to return a this type.

```txt
error[E0106]: missing lifetime specifier
  --> src/lib.rs:49:73
   |
49 | fn voronoi(points: Vec<(f64, f64)>, bounding_box: BoundingBoxPy) -> Vec<VoronoiCell> {
   |                                                                         ^^^^^^^^^^^ expected named lifetime parameter
   |
   = help: this function's return type contains a borrowed value, but there is no value for it to be borrowed from
help: consider using the `'static` lifetime
   |
49 | fn voronoi(points: Vec<(f64, f64)>, bounding_box: BoundingBoxPy) -> Vec<VoronoiCell<'static>> {
   |                                                                                    +++++++++
```

Okay I'll make that change... then we get:

```txt
error[E0599]: the method `assert_into_py_result` exists for struct `Vec<voronoice::VoronoiCell<'_>>`, but its trait bounds were not satisfied
   --> src/lib.rs:48:1
    |
48  | #[pyfunction]
    | ^^^^^^^^^^^^^ method cannot be called on `Vec<voronoice::VoronoiCell<'_>>` due to unsatisfied trait bounds
    |
   ::: /Users/peter/.rustup/toolchains/stable-aarch64-apple-darwin/lib/rustlib/src/rust/library/alloc/src/vec/mod.rs:400:1
    |
400 | pub struct Vec<T, #[unstable(feature = "allocator_api", issue = "32838")] A: Allocator = Global> {
    | ------------------------------------------------------------------------------------------------
    | |
    | doesn't satisfy `Vec<voronoice::VoronoiCell<'_>>: IntoPy<Py<PyAny>>`
    | doesn't satisfy `_: IntoPyResult<Vec<voronoice::VoronoiCell<'_>>>`
    |
    = note: the following trait bounds were not satisfied:
            `Vec<voronoice::VoronoiCell<'_>>: IntoPy<Py<PyAny>>`
            which is required by `Vec<voronoice::VoronoiCell<'_>>: IntoPyResult<Vec<voronoice::VoronoiCell<'_>>>`
            `[voronoice::VoronoiCell<'_>]: Sized`
            which is required by `[voronoice::VoronoiCell<'_>]: IntoPyResult<[voronoice::VoronoiCell<'_>]>`
            `[voronoice::VoronoiCell<'_>]: IntoPy<Py<PyAny>>`
            which is required by `[voronoice::VoronoiCell<'_>]: IntoPyResult<[voronoice::VoronoiCell<'_>]>`
    = note: this error originates in the attribute macro `pyfunction` (in Nightly builds, run with -Z macro-backtrace for more info)

```

Okay so this is actually REALLY AWESOME. The compiler and [rust-analyzer](https://rust-analyzer.github.io/) will just tell us when we're returning a thing it doesn't know how to convert back into a python type. 

We could implement `IntoPy<Py<PyAny>>` for `VoronoiCell`, but I think that would be very painful because it contains a reference back to the `Voronoi` object that would be built. And that means we'd really be implementing something like that for `Voronoi` eventually. 

I think we've answered our first question about whether we should recreate the whole `voronoice` API: no, we should not.

Let's instead follow the pattern we came up with before and develop a `VoronoiCellPy` object that we can use instead.

To do this, I'm going to look at what methods are available on the `VoronoiCell` struct, then convert any data they provide to fields on our new struct.

Here's my initial sketch of this:

```rust
#[pyclass(name = "VoronoiCell")]
struct VoronoiCellPy {
    position: (f64, f64),
    site: usize,
    // These are originally Iterator<Item = &Point>, but we'll collect + convert to tuples
    vertices: Vec<(f64, f64)>,
    neighbors: Vec<usize>,
    is_on_hull: bool,
}

impl From<VoronoiCell> for VoronoiCellPy {
    fn from(cell: VoronoiCell) -> Self {
        let position: (f64, f64) = {
            let pos = cell.site_position();
            (pos.x, pos.y)
        };
        let site: usize = cell.site();
        let vertices: Vec<(f64, f64)> = cell
            .iter_vertices()
            .map(|point| (point.x, point.y))
            .collect();
        let neighbors: Vec<usize> = cell.iter_neighbors().collect();
        let is_on_hull = cell.is_on_hull();
        VoronoiCellPy {
            position,
            site,
            vertices,
            neighbors,
            is_on_hull,
        }
    }
}
```

and now we get...

```txt
error[E0726]: implicit elided lifetime not allowed here
  --> src/lib.rs:52:11
   |
52 | impl From<VoronoiCell> for VoronoiCellPy {
   |           ^^^^^^^^^^^ expected lifetime parameter
   |
   = note: assuming a `'static` lifetime...
help: indicate the anonymous lifetime
   |
52 | impl From<VoronoiCell<'_>> for VoronoiCellPy {
   |                      ++++

```

Okay so I made the change that's suggested, but I'm not sure why. Luckily the compiler tells me I can learn more by running `rustc --explain E0726`. So I run that and see if I can figure things out...

That leads me to look at the original implementation of `VoronoiCell`, which looks like this:

```rust
#[derive(Clone)]
pub struct VoronoiCell<'v> {
    site: usize,
    voronoi: &'v Voronoi
}
```

What I think this is saying is that since `VoronoiCell` contains a reference to `Voronoi`, it's lifetime has to be at least as long as that. Because this is declared for `VoronoiCell`, I have to be sure that when I'm using it to create my `VoronoiCellPy`, it also knows about that lifetime. I'm still a little fuzzy on lifetimes, so I'll write this example down and come back to it once I've learned a little more to see if I can figure out the details.

We make that suggested correction and continue on our way.

I am now getting this nice warning on my `VoronoiCellPy` struct (which normally I would care about):

```txt
fields `position`, `site`, `vertices`, `neighbors` and `is_on_hull` are never read
`#[warn(dead_code)]` on by default
```

We're not creating any `VoronoiCellPy` structs yet, but this will go away when we do.

Let's check if this works in python again!

```python
>>> from voronoiville import BoundingBox, voronoi
>>> points = [(0, 0), (0, 1), (1, 0)]
>>> cells = voronoi(points, BoundingBox(-2.5, -2.5, 2.5, 2.5))
>>> cells[0]
<builtins.VoronoiCell object at 0x104cee730>
>>> cells[0].position
AttributeError: 'builtins.VoronoiCell' object has no attribute 'position'
```

Uh oh. We don't have an attribute `position`? But I thought we just defined that. Nope! It looks like we need to declare that we can get these attributes as object properties in python using the [`#[pyo3(get, set)]`](https://pyo3.rs/v0.17.3/class.html?highlight=getter#object-properties-using-pyo3get-set) annotation.

This is actually nice because this is basically a read-only object and we'd never want a user to be able to `set` these attributes, only `get` them. So our `VoronoiCellPy` looks like this now:

```rust
#[allow(dead_code)]
#[pyclass(name = "VoronoiCell")]
struct VoronoiCellPy {
    #[pyo3(get)]
    position: (f64, f64),
    #[pyo3(get)]
    site: usize,
    // These are originally Iterator<Item = &Point>, but we'll collect + convert to tuples
    #[pyo3(get)]
    vertices: Vec<(f64, f64)>,
    #[pyo3(get)]
    neighbors: Vec<usize>,
    #[pyo3(get)]
    is_on_hull: bool,
}
```

And we recompile and test in python:

```python
# previous commands executed.
>>> cells = voronoi(points, BoundingBox(-2.5, -2.5, 2.5, 2.5))
>>> c = cells[0]
>>> (c.position, c.site, c.vertices, c.neighbors, c.is_on_hull)
((-0.9662767249429629, -1.1960951507454378),
 0,
 [(-0.05610501383113076, 0.04530452444563185),
  (1.2034397454112338, -2.5),
  (-2.5, -2.5),
  (-2.5, 0.06870490568260464)],
 [2, 1],
 True)
```

We did it! Just for fun, let's try and overwrite `c.site`:

```python
>>> c.site = 3
AttributeError: attribute 'site' of 'builtins.VoronoiCell' objects is not writable
```

Perfect.

### Adding default arguments and better errors

I want to make returning the neighbors is optional, so I added a helper function to create our `VoronoiCellPy` objects without neighbors. This means the type for the `neighbors` field is now `Option<Vec<usize>>`. 

I also want to allow a user to determine the number of iterations for the Lloyd relaxation algorithm. However, I want default values for these arguments. The way to do that is to pass them as arguments to the `#pyfunction` annotation.

In addition, I also want to provide a better error message when we fail to build a diagram. For example, if a user passes an empty list. The `build()` method returns an `Option<Voronoi>`, so we'll use `ok_or` to return/throw the error. We also need to change our function signature to return `PyResult<Vec<VoronoiCellPy>>`.

```rust
#[pyfunction(return_neighbors = true, lloyd_relaxation_iterations = 0)]
fn voronoi(
    points: Vec<(f64, f64)>,
    bounding_box: BoundingBoxPy,
    return_neighbors: bool,
    lloyd_relaxation_iterations: usize,
) -> PyResult<Vec<VoronoiCellPy>> {
    let sites: Vec<Point> = points.iter().map(|(x, y)| Point { x: *x, y: *y }).collect();
    let bounding_box: BoundingBox = bounding_box.into();
    let v: Voronoi = VoronoiBuilder::default()
        .set_sites(sites)
        .set_bounding_box(bounding_box)
        .set_lloyd_relaxation_iterations(lloyd_relaxation_iterations)
        .build()
        .ok_or(PyRuntimeError::new_err("Can't build Voronoi diagram from given points."))?;
    let cells: Vec<VoronoiCellPy> = match return_neighbors {
        true => v.iter_cells().map(|cell| cell.into()).collect(),
        false => v
            .iter_cells()
            .map(VoronoiCellPy::into_no_neighbors)
            .collect(),
    };
    Ok(cells)
}
```

Without updating the error message and return type, here's what a user sees if `build()` fails:

```txt
thread '<unnamed>' panicked at 'called `Option::unwrap()` on a `None` value', src/lib.rs:136:22
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
---------------------------------------------------------------------------
PanicException                            Traceback (most recent call last)
Cell In[20], line 1
----> 1 voronoi([], BoundingBox(0, 0, 1,1))

PanicException: called `Option::unwrap()` on a `None` value
```

Yikes! So it's partially informative if you know Rust and what's going on, but not really useful if you're only using python. 

### Docstrings & Type Annotations

I think we need some documentation for our library! There's a nice [page on python typing hints](https://pyo3.rs/v0.17.3/python_typing_hints) in the PyO3 documentation. Because we have a simple package, all we need to do is add a `voronoiville.pyi` file to our root directory and add in type stubs there. 

In addition to type stubs, this is also where we can add docstrings. You can also add docstrings in the Rust code, but I find it easier to write them in python so I can use the [AutoDocstring](https://marketplace.visualstudio.com/items?itemName=njpwerner.autodocstring) extension once I add the type hints.

## Building & Delivery

This process probably deserves it's own blog post, so I'll keep this section short. Our goal is to have someone be able to call `pip install voronoiville`. In python, if your package is only python code, you can typically just build a single wheel or sdist and upload that to PyPI. That's not the case here!  In our case, we're actually building an [extension module](https://llllllllll.github.io/c-extension-tutorial/what-is-an-extension-module.html) which requires a bit more involvement. The `PyO3` docs have a [helpful explanation](https://pyo3.rs/v0.17.3/building_and_distribution#building-python-extension-modules):

>Python extension modules need to be compiled differently depending on the OS (and architecture) that they are being compiled for. As well as multiple OSes (and architectures), there are also many different Python versions which are actively supported. Packages uploaded to [PyPI](https://pypi.org/) usually want to upload prebuilt "wheels" covering many OS/arch/version combinations so that users on all these different platforms don't have to compile the package themselves. Package vendors can opt-in to the "abi3" limited Python API which allows their wheels to be used on multiple Python versions, reducing the number of wheels they need to compile, but restricts the functionality they can use.

Rather than figure out the details of a build process, I copied and modified the GitHub Action (GHA) that [`graphlib2`](https://github.com/adriangb/graphlib2) uses to build and release.  I found this example because the [`matiurin-action` GitHub Action repo](https://github.com/PyO3/maturin-action#examples) has a list of example repositories that use that to build. If you just want to get your package built, I'd encourage you to do something similar and copy from an existing open-source repos build process.

One thing we'll want to do is use the "abi3" limited Python API mentioned. This will allow us to build a single wheel for all python versions 3.7+. To do this, we have to modify a line in our dependencies in the `Cargo.toml` file:


```toml
pyo3 = { version = "0.17.3", features = ["extension-module", "abi3-py37"] }
```

We've added the `abi3-py37` feature to our dependencies. A `feature` is like an `extra` if you're used to python packages.

Once our packages are built, we need to upload them to PyPI. The GitHub Action we use references a secret `PYPI_TOKEN` you can include to upload your package's wheels. 

I added some modifications to the GitHub Action to only build when I modify a relevant source code file (in `src`) and to only publish to PyPI when I push a tag.

In short, to avoid build pains, just copy what someone else is doing and modify it to your liking. In that spirit, [here's the build process I'm currently using](https://github.com/pmbaumgartner/voronoiville/blob/72ba0c90332e129d52c6e53e68b5bbc64991d3c2/.github/workflows/ci.yml).

## Wrap Up

We did it! We wrapped a rust crate and transformed it into a python package. The result is we can build Voronoi diagrams [4-5x faster](https://github.com/pmbaumgartner/voronoiville/blob/main/extra/benchmark.ipynb) than our previous method, which is great! 

The Rust development process was painless after we got our tooling setup. In this case, I really felt comfortable relying on the compiler and got this sensation of the [compiler teaching me](https://www.youtube.com/watch?v=CJtvnepMVAU). 

I think the primary obstacle was thinking about _what_ we wanted to return from our function and ensuring it was a struct composed only of native python types. Once we had figured that out, the rest of the development process was straightforward and the quality of the PyO3  API and documentation really made developing a breeze.

Here's the final result: https://github.com/pmbaumgartner/voronoiville/

[![pmbaumgartner/voronoiville - GitHub](https://gh-card.dev/repos/pmbaumgartner/voronoiville.svg?fullname=)](https://github.com/pmbaumgartner/voronoiville)