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
|   +-- tests_for_lib1
|   +-- tests_for_lib2
|   +-- tests_for_bin1
|   +-- tests_for_bin2
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

This is either based on personal taste or governed by your organisation's coding 
standard.

The [The Rust Book](https://doc.rust-lang.org/book/ch11-03-test-organization.html#unit-tests) recommends unit test be placed in the source `.rs` file of the code being tested. However,
for larger projects the files can quickly get big and hard to navigate and jump between
tests and code. Moreover, with Rust's native support of testing and some nauances imposed
by `cargo` package manager, things work slightly differently.

## `Cargo` and `rustc` and support for tests TL;DR

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

## Cargo Project and Test organisation rules

[Cargo Project Layout](https://doc.rust-lang.org/cargo/guide/project-layout.html) covers in detail
the package structure. The [Test Organization](https://doc.rust-lang.org/book/ch11-03-test-organization.html)
chapter covers the how tests are organized.

It is recommened that you read these first. I'll cover only few important rules and quirks to
consider as they will govern your project struture:

### Rule 1: Only one library `crate` per `package`

A `package` can contain one or more `crate`s but only one of them can be a library, i.e. a `package` can contain one library OR one library and one or more binaries OR one or more binaries. This is a `Cargo` imposed restriction, not `rustc`.

This can be counter intuitive as usually largish projects have more libaries than binaries;
each binary using multiple libraries. However, Rust chose this to reduce dependency management
complexities.

> :point_right: **TL;DR** Any non-trivial multi-libaray project will require use of `workspace`.
> Infact, I highly  recommend using `workspace` for rojects from the start as it's almost no
> effort to do so.

### Rule 2: `tests` folders at root of `package` have special meaning and contain "binary only" crates

`tests` (also `examples` and `benches` - not covered here) are special folders when they appear at
root of the `package`'s folder. `tests` folder are meant to contain *integration tests*. Each `.rs`
file, or subdirectory directly under this folder is treated as binary only crates.

Only functions (including `main`) anotated by `#[test]` are executed by `cargo test`. In fact, one doesn't
need to provide `main` for binary crates in `tests` folder as it will be genrated automatically, similar to
how `cargo` handles unit tests. However, there is **one crucial difference** between *unit tests* (in `src`) and
*integration tests* in `tests` folder: Unit tests are compiled together and run as a single binary,
integration tests produce individual binaries (per file or multifile ). This impacts how the test are run and
how output is reported:
  
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

> :point_right: **TL;DR:** The location of tests (i.e. wheter in `<package>/src` or `<package>/tests`)
> will impact how they are executed and output reported.

### Rule 3: Visibility and privacy rules apply to tests

Rust [Visibility and Privacy Rules](https://doc.rust-lang.org/reference/visibility-and-privacy.html) apply
to tests. Unit tests (i.e. in `<package>/src`) will be able to access internal implementation details
if these are made available using `pub(crate) / pub(super)` etc., or if tests and code are in the same
file or module. On the other hand, integration tests in `<package>/tests` folder will only be able to
access publically visible items from the `<package>/src`. 

Since each file (or multi-file test) is a binary `crate`, sharing code between 2 multi-file test binaries is
not possible, nor can a multi-file test binary access a module outside its own folder

```console
.
+-- src/
|   ... snip ...
+-- tests/
    +-- shared_test_code/
    |   +-- mod.rs
    |   +-- file_1.rs        
    |   +-- ... snip ...
    +-- some-integration-tests.rs    <== can access shared_test_code
    +-- other-integration-tests.rs   <== can access shared_test_code
    |   ... snip ...
    +-- multi-file-test/             <== can NOT access shared_test_code
        +-- main.rs
        +-- test_module.rs
```

> :point_right: **TL;DR** The level interface and details need to be tested, and what common
> code needs to be shared between tests will govern where and how test are organised and in
> which folder they are placed.

## My preferred Rust Project and Test Strucuture

With all of the above in mind and after a few trials and frustrating errors here's a structure
that works best for me.

```console
project
+-- Cargo.toml    <== Workspace toml file
+-- Cargo.lock 
+-- libs          <== All libraries
|   +-- lib_1
|   |   +-- Cargo.toml
|   |   +-- src
|   |   |   +-- lib.rs 
|   |   |   +-- *.rs 
|   |   |   +-- module_1
|   |   |   |   +-- mod.rs
|   |   |   |   +-- *.rs       <== Test internal details here ONLY if necessary
|   |   |   +-- unittests
|   |   |       +-- mod.rs
|   |   |       +-- common_test_code.rs
|   |   |       +-- test_*.rs   <== Test pub(crate) interface/implementation of the library here
|   |   +-- benches
|   |   |   +- *.rs    <== benchmark tests
|   |   +-- tests      <== 
|   |                         
|   +-- lib_2
|   |   +-- ... snip ...
|   | ... snip ...
+-- apps             <== all binaries 
|   +-- app-1
|   |   +-- Cargo.toml
|   |   +-- src
|   |   |   +-- main.rs
|   |   |   +-- *.rs
|   |   |   +-- module_1
|   |   |   |   +-- mod.rs
|   |   |   |   +-- *.rs
|   |   |   +-- unittests 
|   |   |       +-- mod.rs    
|   |   |       +-- test_*.rs
|   |   +-- tests
|   |       +-- integration-test-*.rs
```