---
title: "Rust Project Organization"
slug: "rust project organization"
excerpt: A quick guide to organising your Rust projects
tags: 
   - featured
   - Rust
   - Beginner
---

If like me, you're learning Rust and come from a C++ or Python background
you might have seen code and unit tests organized in broadly 2 different styles.


<div class="Rtable Rtable--2cols">
  <div style="order:0;" class="Rtable-cell">
    <h4>Unit tests mirror `src`</h4>
  </div>
  <div style="order:1;" class="Rtable-cell">

{% highlight console %}
project
+-- src
|   +-- lib1
|   +-- lib2
|   +-- bin1
|   +-- bin2
+-- unit_tests 
|   +-- lib1
|   +-- lib2
|   +-- bin1
|   +-- bin2
+-- integration_tests 
    +-- test_integration_1
    +-- test_integration_2
{% endhighlight %}

  </div>
  <div style="order:0;" class="Rtable-cell">
    <h4>Unit tests inside <code class="language-plaintext highlighter-rouge">src</code></h4>
  </div>
  <div style="order:1;" class="Rtable-cell">

{% highlight console %}
project
+-- src
|   +-- lib1
|   |   +-- unit_tests
|   +-- lib2
|   |   +-- unit_tests
|   +-- bin1
|   |   +-- unit_tests
|   +-- bin2
|       +-- unit_tests
+-- integration_tests
    +-- test_integration_1
    +-- test_integration_2
.
{% endhighlight %}

  </div>
</div>

This is either based on personal taste or governed by your organisation's coding standard.

The [The Rust Book](https://doc.rust-lang.org/book/ch11-03-test-organization.html#unit-tests) 
suggests that unit test be placed in the source `.rs` file of the code being tested. However,
for larger projects the files can quickly get big and hard to navigate and jumping between
tests and code is not very ergonomic, and can diffing and reviewing difficult too. 
Moreover, with Rust's native support of testing and some nuances imposed by `cargo` package
manager, things work slightly differently.

# `Cargo` and `rustc` and support for tests TL;DR

Rust has a well integrated native support for writing Unit tests, benchmarks and
integration tests via its `#[test] / #[cfg(test)]` `std` library macros, `rustc` compiler
and `cargo` package manager.

**`rustc`** is the Rust compiler that compiles the `.rs` files into binaries and libraries

**`Cargo`** is the most widely use package manager for Rust. It helps you organise your
project's code, configuration and its dependencies and it calls the `rustc` compiler
for you with the correct arguments.

Thus most of the rules and restrictions around project/folder structure we will cover is mostly
imposed and governed by `Cargo` and not `rustc` itself; i.e. if you don't use `Cargo` some of the
restrictions may not apply.

# Cargo Project and Test organisation rules

[Cargo Project Layout](https://doc.rust-lang.org/cargo/guide/project-layout.html) covers in detail
the package structure. The [Test Organization](https://doc.rust-lang.org/book/ch11-03-test-organization.html)
chapter covers the how tests are organized.

I'll cover only few important rules and quirks to consider as they will govern your project
struture:

## Rule 1: Only one library `crate` per `package`

A `package` can contain one or more `crate`s but only one of them can be a library,
i.e. a `package` can contain one library OR one library and one or more binaries OR one or more 
binaries. This is a `Cargo` imposed restriction, not `rustc`.

This can be counter intuitive as usually largish projects have more libraries than binaries;
each binary using multiple libraries. However, Rust chose this to reduce complexities with
dependency management

In most cases, multiple (small) library crates will make sense as:

- it promotes better separation of concerns, boundaries and interface design
- makes dependencies clearer and easy to visualise.

Ultimately, your context will drive the decision between single large monolith library crate OR
multiple library crates.

> :point_right: **TL;DR** Any non-trivial multi-library project will require use of `workspace`.
> In fact, I highly  recommend using `workspace` for projects from the start as it's almost no
> effort to do so.

## Rule 2: `tests` folders at root of `package` have special meaning and contain "binary only" crates

`tests` (also `examples` and `benches` - not covered here) are special folders when they appear at
root of the `package`'s folder. `tests` folder are meant to contain *integration tests*. Each `.rs`
file, or subdirectory directly under this folder is treated as binary only crates.

Only functions (including `main`) annotated by `#[test]` are executed by `cargo test`. In fact,
one doesn't need to provide `main` for binary crates in `tests` folder as it will be generated
automatically, similar to how `cargo` handles unit tests. However, there is
**one crucial difference** between *unit tests* (in `src`) and *integration tests* in `tests`
folder: Unit tests are compiled together and run as a single binary, integration tests produce
individual binaries (per file or multi-file). This impacts how the test are run and how output is
reported:
  
Output when the tests are unit tests in `src` folder

```console
    Running unittests src/lib.rs (target/debug/deps/hash_collections-6ec7669a1997a260)

running 24 tests
test graph_tests::insert_edges_multiple_times ... ok
test graph_tests::insert_edges_once ... ok
      ... snip ...
test hash_map_probe_test::out_of_capacity_error ... ok
test hash_map_probe_test::insert_and_get_colliding_items ... ok
      ... snip ...
test result: ok. 24 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Output when same tests are in `tests` folder:

```console
      Running tests/graph_tests.rs (target/debug/deps/graph_tests-5971696199c715fc)

running 3 tests
test insert_edges_multiple_times ... ok
test insert_edges_once ... ok
    ... snip ...

test result: ok. 3 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

    Running tests/hash_map_probe_test.rs (target/debug/deps/hash_map_probe_test-9c18fb88fdf8f8ec)

running 4 tests
test insert_and_get_colliding_items ... ok
test out_of_capacity_error ... ok
    ... snip ...

test result: ok. 4 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

> :point_right: **TL;DR:** The location of tests (i.e. whether in `<package>/src`
> or `<package>/tests`) will impact how they are executed and output reported.

## Rule 3: Visibility and privacy rules apply to tests

Rust [Visibility and Privacy Rules](https://doc.rust-lang.org/reference/visibility-and-privacy.html)
apply to tests. Unit tests (i.e. in `<package>/src`) will be able to access internal implementation
details if these are made available using `pub / pub(crate) / pub(super)` etc., or if tests and
code are in the same file or module. On the other hand, integration tests in `<package>/tests`
folder will only be able to access publicly visible items from the `<package>/src`.

Since each file (or multi-file test) is a binary `crate`, sharing code between 2 multi-file test
binaries is not possible, nor can a multi-file test binary access a module outside its own folder

```console
.
+-- src/
|   ... snip ...
+-- tests/
    +-- shared_test_code/
    |   +-- mod.rs
    |   +-- file_1.rs        
    |   +-- ... snip ...
    +-- some-integration-tests.rs    <-- can access shared_test_code
    +-- other-integration-tests.rs   <-- can access shared_test_code
    |   ... snip ...
    +-- multi-file-test/             <-- can NOT access shared_test_code
        +-- main.rs
        +-- test_module.rs
```

> :point_right: **TL;DR** While Rust offers a well thought out and fine-grained visibility and
> privacy controls, it can get difficult to reason about, manage and keep track of in large
> projects.

# Rust Project and Test Structure in production

Ultimately the library organisation (mono-crate v.s. multicrate), the level interface
and details being tested, and what common code needs to be shared between tests will
govern where and how test are organised and in which folder they are placed.

With all of the above in mind and after a few trials and frustrating errors here's a low-down
on the 2 broad approaches, how they look and some implications. 

Let's use an hypothetical image processing/object detection lib/app as an example, which takes 
an image and detects and localizes various pet animals... lets call it `Petective`

<div class="Rtable Rtable--2cols Rtable--collapse">
  <div style="order:0;" class="Rtable-cell">
    <h3>Single/Monolithic Library Crate</h3>
  </div>
  <div style="order:1;" class="Rtable-cell">
    <ul>
      <li> The entire library exists in a single crate. Code/functionality may be broken down,
        grouped and organised in modules and sub-modules. </li>
      <li> Dependencies, privacy and accessibility are handled internally using
        <code>pub(desired-level)</code>.</li>
      <li> Unit tests are placed in each module (here in <code>unit_tests</code> folder) and 
        shared test code is just another module. </li>
    </ul>
  </div>
  <div style="order:2;" class="Rtable-cell">
{% highlight console %}
petective_project   <-- Project Root
|-- Cargo.toml      <--+ Workspace toml file
|-- Cargo.lock
+-- petective       <-- Our monolith library + apps
    |-- Cargo.toml
    |-- Cargo.lock
    +-- src
    |   |-- lib.rs  <-- petective library interface
    |   |-- *.rs
    |   | 
    |   +-- shared_test_code  <--+ shared test code
    |   |   |-- mod.rs           | (e.g. fixtures)
    |   |   |-- *.rs
    |   |
    |   +-- image_core    <--+ a module grouping 
    |   |   |-- mod.rs       | some functionality 
    |   |   |-- *.rs
    |   |   +-- unit_tests   <--+ unit tests for the
    |   |       |-- mod.rs      | module
    |   |       |-- test_*.rs
    |   |
    |   +-- filters    <-- another module
    |   |   |-- mod.rs
    |   |   |-- *.rs
    |   |   +-- unit_tests   <-- and its unit tests
    |   |       |-- mod.rs
    |   |       |-- test_*.rs
    |   |
    |   +-- unit_tests <--+ Library level unit tests
    |       |-- mod.rs    | for the public interface
    |       |-- test_*.rs
    |
    ... 
{% endhighlight %}
  </div>
  <div style="order:3;" class="Rtable-cell">
    <ul>
      <li>Benchmarks, integration tests and examples may be placed and organised in
        <code>benches</code>, <code>tests</code> and <code>examples</code> folders
        respectively.</li>
    </ul>
  </div>
  <div style="order:4;" class="Rtable-cell">
{% highlight console %}
petective_project   <-- Project Root
|-- Cargo.toml      <--+ Workspace toml file
|-- Cargo.lock
+-- petective       <-- Our monolith library + apps
    |-- Cargo.toml
    |-- Cargo.lock
    +-- src
    |   |-- *.rs
    |   +-- shared_test_code  <--+ shared test code
    |   +-- image_core    <--+ a module grouping 
    |   +-- filters       <-- another module
    |   +-- unit_tests    <--+ Library level unit tests
    |   ...
    |-- tests     <-- Integration tests
    |-- benches   <-- Benchmarks
    |-- examples  <-- examples
    ... 
{% endhighlight %}
  </div>

  <div style="order:0;" class="Rtable-cell">
    <h3>Multi-crate (one crate per library)</h3>
  </div>
  <div style="order:1;" class="Rtable-cell">
    <ul>
      <li> The code/functionality is broken down, grouped and organised into individual
        <code>crate</code>s of "sub-libraries"</li>
      <li> Dependencies are handled using the <code>workspace</code> and <code>crate</code>
        level <code>Cargo.toml</code> files. Sub-libraries can only access the <code>pub</code>
        items of other sub-libraries.
      </li>
      <li> Unit tests are placed in a module (here in <code>unit_tests</code> folder) for each
        library. Shared test code can exist in its own library.</li>
    </ul>
  </div>
  <div style="order:2;" class="Rtable-cell">

{% highlight console %}
petective_project   <-- Project Root
|-- Cargo.toml      <--+ Workspace toml file (required)
|-- Cargo.lock
+-- shared_test_code  <--+ lib containing test code
|   |-- Cargo.toml       | shared across libraries
|   +-- src
|       |-- lib.rs
|       |-- *.rs
| 
+-- petective    <-- Public interface
|   |-- Cargo.toml
|   +-- src
|      |-- lib.rs 
|      |-- *.rs 
|      +-- unit_tests  
|          |-- mod.rs
|          |-- test_*.rs
|          |-- *.rs
|
+-- image_core     <--+ a 'sub-library' grouping 
|   |-- Cargo.toml    | some functionality 
|   +-- src
|       |-- lib.rs
|       |-- *.rs
|       +-- unit_tests   <-- unit test for the library
|           |-- mod.rs
|           |-- test_*.rs
|
+-- filters     <-- yet another library
|   |-- Cargo.toml
|   +-- src
|       |-- lib.rs 
|       |-- *.rs
|       +-- unit_tests   <-- and its unit tests
|           |-- mod.rs
|           |-- test_*.rs
|
...
{% endhighlight %}

  </div>
  <div style="order:3;" class="Rtable-cell">
    <ul>
      <li><b>For each library</b>, its benchmarks, integration tests and examples may and placed
        in its own <code>benches</code>, <code>tests</code> and <code>examples</code> folders
        respectively.</li>
      <li>Alternatively, a separate <code>crate</code> may be used to hold these.</li>
    </ul>
  </div>
  <div style="order:4;" class="Rtable-cell">
{% highlight console %}
petective_project   <-- Project Root
|-- Cargo.toml      <--+ Workspace toml file (required)
|-- Cargo.lock
+-- shared_test_code  <--+ lib containing test code 
+-- petective    <-- Public interface lib
|   |-- Cargo.toml
|   +-- src      <-- library source and unit test
|   +-- tests    <-- integration tests
|   +-- benches  <-- benchmark tests
|   +-- examples <-- usage examples
|
+-- image_core     <--+ a library  with only
|   |-- Cargo.toml    |  benchmarks
|   +-- src       <-- library source and unit test
|   +-- benches   <-- integration tests
|
+-- filters      <--+ a library with no integration
|   |-- Cargo.toml  | or benchmark tests
|   +-- src   <-- library source and unit test
|
+-- integration    <--+ Alternatively tests and benches
|   |-- Cargo.toml    | in separate crate
|   +-- src   <-- a place holder library here
|   +-- benches <--+ project-wide benchmarks and
|   +-- tests      | integration tests
...
{% endhighlight %}
  </div>
</div>

<div>
<!--
So how does it happen here:

``` c
|   |   +-- bin                 <--+ (**Option 1 for apps:**
|   |   |   +-- where-is-archie    |  all binaries/apps in
|   |   |   |   |-- main.rs        |  same crate)
|   |   |   |   |-- *.rs
|   |   |   |   +-- unit_tests 
|   |   |   |       |-- mod.rs
|   |   |   |       |-- test_*.rs
|   |   |   +-- finding-nemo
|   |   |       |-- main.rs
|   |   |       |-- *.rs
|   |   |       +-- unit_tests
|   |   |           |-- mod.rs
|   |   |           |-- test_*.rs
|   |   |
|   |   +-- unit_tests   <-- all unit tests for the lib.
|   |       |-- mod.rs
|   |       |-- test_*.rs
|   |
|   +-- benches      <-- benchmark tests
|   |   |-- *.rs
|   |
|   +-- tests        <-- integration tests
|       |-- edge-detection-test.rs
|       +-- pre-processing-integration-test
|           |-- main.rs
|           |-- *.rs
|
+-- petective_apps   <--+ (**Option 2 for apps**
    |-- Cargo.toml      |  all binaries in a separate
    |-- Cargo.lock      |  crate)
    +-- src   
    |   +-- bin
    |       +-- ace-ventura
    |       |   |-- main.rs
    |       |   |-- *.rs
    |       |   +-- unit_tests 
    |       |       +-- mod.rs    
    |       |       +-- test_*.rs
    |       +-- finding-nemo-2
    |       |   |-- main.rs
    |       | ... snip ...
    | ... snip ...   
```
-->
</div>