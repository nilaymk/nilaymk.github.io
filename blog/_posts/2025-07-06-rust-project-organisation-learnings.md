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

<table>
<tr>
<th> Unit tests mirror src </th>
<th> Unit tests inside src </th>
</tr>
<tr>
<td>

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

</td>
<td>

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

</td>
</tr>
</table>

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

With all of the above in mind and after a few trials and frustrating errors here's 2 approaches
I can recommend:

Let's use an hypothetical image processing/object detection lib/app as an example, which takes 
an image and detects and localizes various pet animals... lets call it `Petective`

## Single/Monolithic Library Crate

The idea here is simply to break down the library into `modules / submodules`, and add unit tests
in each  `module`.

```console
petective_project   <-- Project Root
|-- Cargo.toml      <-- Workspace toml file (workspace optional but recommend!!)
|-- Cargo.lock 
+-- petective       <-- Our monolith library + apps
|   |-- Cargo.toml
|   |-- Cargo.lock
|   +-- src
|   |   +-- lib.rs      <-- petective library's public interface
|   |       |-- petective_apis.rs 
|   |       +-- shared_test_code      <-- test code shared by all modules (e.g. fixtures)
|   |       |   |-- mod.rs
|   |       |   |-- ... snip ...
|   |       +-- image_core            <-- in-memory image buffer representation module 
|   |       |   |-- mod.rs
|   |       |   |-- greyscale.rs
|   |       |   |-- rgb.rs
|   |       |   | ... snip ....
|   |       |   +-- unit_tests        <-- all unit tests for the module.
|   |       |       |-- mod.rs
|   |       |       |-- image_core_common_test_code.rs
|   |       |       |-- test_greyscale.rs
|   |       |       |-- test_rgb.rs
|   |       |       |-- ... snip ....
|   |       +-- filters               <-- image filters (e.g. color conv, sharpen, edge detect)
|   |       |   |-- mod.rs
|   |       |   | ... snip ...
|   |       | ... snip ...
|   |       +-- bin         <--   (**Option 1 for apps:** all binaries/apps in same crate)
|   |           +-- where-is-archie
|   |           |   |-- main.rs
|   |           |   |-- *.rs
|   |           |   +-- unit_tests 
|   |           |       +-- mod.rs    
|   |           |       +-- test_*.rs
|   |           +-- finding-nemo
|   |           |   |-- main.rs
|   |           | ... snip ...
|   |    
|   +-- benches      <-- benchmark tests
|   |   |-- *.rs   
|   |   | ... snip ...
|   |    
|   +-- tests        <-- integration tests
|       |-- edge-detection-test.rs
|       +-- pre-processing-integration-test
|       |   |-- main.rs
|       |   | ... snip ...
|       | ... snip ...
|       
+-- petective_apps   <--   (**Option 2 for apps** all binaries in a separate crate)
    |-- Cargo.toml
    |-- Cargo.lock
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
        
...
```

## Multi-crate (one crate per library)

```console
petective
+-- Cargo.toml    <-- Workspace toml file (workspace optional but recommend!!)
+-- Cargo.lock 
+-- petective     <-- Our monolith library + apps
    +-- Cargo.toml
    +-- src
    |   +-- lib.rs    <-- petective library's public interface
    |   |   +-- petective_apis.rs 
    |   |   +-- common_test_code    <-- test code shared by all modules
    |   |   |   +-- mod.rs
    |   |   |   +-- ... snip ...
    |   |   +-- image_core   <-- in-memory image buffer representation module 
    |   |   |   +-- mod.rs
    |   |   |   +-- greyscale.rs
    |   |   |   +-- rgb.rs
    |   |   |   +-- ... snip ....
    |   |   |   +-- unit_tests    <-- all unit tests for the module.
    |   |   |       +-- mod.rs
    |   |   |       +-- image_core_common_test_code.rs
    |   |   |       +-- test_greyscale.rs
    |   |   |       +-- test_rgb.rs
    |   |   |       +-- ... snip ....
    |   |   +-- filters   <-- image filters (e.g. color conv, sharpen, edge detect)
    |   |   |   +-- mod.rs
    |   |   |   +-- ... snip ...
    |   |   +-- ... snip ...
    |   +-- benches
    |   |   +-- *.rs    <-- benchmark tests
    |   +-- tests      <-- integration tests
    |   |   +-- main.rs
    |   |   +-- pre-processing-test
    |   |   |   +-- main.rs
    |   |   |   +-- ... snip ...
    |   |   +-- ... snip ...
    |   +-- apps                <-- all binaries 
    |   |   +-- where-is-archie
    |   |       +-- main.rs
    |   |       +-- *.rs
    |   |       +-- unit_tests 
    |   |           +-- mod.rs    
    |   |           +-- test_*.rs

```
