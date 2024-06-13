---
title: "Unit Test Coverage"
description: Unit Tests are an important part of the testing process, but how do you know you've written tests that coverage all of your use-cases? Coverage profiles not only report on your unit test coverage but they can also show you which lines of code are lacking tests.
date: 2024-06-09T15:02:06-04:00
image:
hidden: false
comments: true
draft: false
categories:
- Technical
tags:
- go
- rust
- utility
---

Once upon a time, I was a junior dev working alongside the QA team at an image search company. It was a small team, mostly responsible for the user-facing website. Thankless work. One of the things it instilled in me, though, was a healthy dose of respect for proper testing of code. How do you know that the code you've just written is going to perform the way you intended it to? Repeatable, easy-to-run testing.

There are a lot of different ways to test software, but I want to focus on Unit Testing. Simply put, Unit Testing is the process where you test the smallest units of code.

In Unit Testing, your focus is on testing how individual units of code, like a function or macro, operate. You may use use or create "mock" objects, such as an HTTP client that simulates operations against a pretend web server. What you're doing is making sure that the function you've written performs the way you expect it to when given valid *and* invalid input.

When you do this, you're creating test *coverage* in your code base. Code coverage is a metric that states what percentage of your code has a unit test that exercises it. It tells you how much of your code you've written tests for.

Please note, though, that it doesn't mean you've written *good* tests. More on that another time.

So the goal for this post is to show you how to see your test coverage, and make it an integral part of your testing strategy. Perhaps more importantly, I'll show you how to see *where* you lack test coverage.

The testing frameworks I've used in Go/Rust typically show you the number of tests run, pass or fail, but require a bit more work to visualize your test coverage.

For convenience, I've written shell functions for each language that will generate a coverage report. I've stuck these in my `~/.aliases`, making them available at the command line.

## Go

Go has built-in support for [collecting coverage profiles](https://go.dev/doc/build-cover). That means that this will just work, out of the box. What I've created here is a function, `go-coverage`, that I can pass a package path to and it will generate and open a [coverage map](/static/coverage/go-coverage.html).

```shell
go-coverage() {
    usage="$(basename "$0") [-h] [packages] -- [args]

    where:
        -h          show this help text
        [packages]  the package(s) to test (default: ./...)
        [args]      any additional arguments to pass to 'go test'
    "

    while getopts :h flag
    do
        case "${flag}" in
            h)  echo "$usage"
                return 0
                ;;
        esac
    done

    PACKAGES="./..."
    # Check if $1 is set
    if [ -n "$1" ]; then
        PACKAGES=$1
    fi

    # Generate the coverage profile
    if go test -v -coverprofile=/tmp/coverage.$$ $PACKAGES; then
        # Convert the test coverage output to an HTML file
        go tool cover -html /tmp/coverage.$$ -o /tmp/coverage.$$.html

        # Open the coverage html, which will give us a per-file
        # breakdown of what code doesn't have test coverage.
        open /tmp/coverage.$$.html
    else
        echo "Failed to generate coverage report."
        return 1
    fi
}
```

To use it, simply pass it the path to the module you want to test.

```shell
$ cd ~/src/go/gedcom
$ go-coverage ./parser/...
=== RUN   TestScanner_Scan
--- PASS: TestScanner_Scan (0.00s)
=== RUN   TestParser_ParseGedcom
--- PASS: TestParser_ParseGedcom (0.00s)
PASS
coverage: 55.1% of statements
ok  	github.com/adamisrael/gedcom/parser	0.117s	coverage: 55.1% of statements
```

## Rust

There are multiple options to generating a coverage profile in Rust. I've tested [llvm-conv](https://lib.rs/crates/cargo-llvm-cov) and [tarpaulin](https://github.com/xd009642/tarpaulin). They're both similiar, but I prefer the output of [llvm-conv](/static/coverage/rust-coverage-llvm.html) for navigating the data. [tarpaulin](/static/coverage/rust-coverage-tarpaulin.html) displays the same data, but doesn't allow you to link to a specific line of code or jump ahead to the first block of missing coverage.

You'll need to [install llvm-conv](https://lib.rs/crates/cargo-llvm-cov#readme-installation):

```shell
# install the binary to ~/.cargo/bin
cargo +stable install cargo-llvm-cov --locked
```

Then, add this function to your aliases:

```shell
rust-coverage() {
    usage="$(basename "$0") [-h]

    where:
        -h          show this help text
    "

    while getopts :h flag
    do
        case "${flag}" in
            h)  echo "$usage"
                return 0
                ;;
        esac
    done

    # Make sure we're in a rust project
    if [ ! -f Cargo.toml ]; then
        echo "Not in a Rust project."
        return 1
    fi

    # Generate the coverage profile and open the coverage report
    cargo llvm-cov --open
}
```

Now you can run it from any Rust project:

```shell
$ rust-coverage
   Compiling gedcom-rs v0.1.0 (/Users/adam/src/rust-gedcom)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 1.21s
     Running unittests src/lib.rs (target/llvm-cov-target/debug/deps/gedcom_rs-2f049742db481cba)

running 28 tests
[...]

test result: ok. 28 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.02s

     Running unittests src/main.rs (target/llvm-cov-target/debug/deps/gedcom_rs-90daeb1e86c76303)

running 1 test
test tests::test_complete_gedcom ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.02s

    Finished report saved to /Users/adam/src/rust-gedcom/target/llvm-cov/html
     Opening /Users/adam/src/rust-gedcom/target/llvm-cov/html/index.html
```
