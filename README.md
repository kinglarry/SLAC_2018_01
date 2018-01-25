# Web Assembly and Rust SLAC

[The Rust Programming Language]: https://www.rust-lang.org

* [Pre-SLAC](#pre-slac)
* [What is Rust?](#what-is-rust)
* [What is WebAssembly?](#what-is-webassembly)
* [Labs](#labs)

## Pre-SLAC

We will be running this application off a simple HTTP server (so you can test the browser rendering of wasm code). You will also need to install the latest Rust toolchain (instructions below). Finally, we'll go through a simple "Hello World" example before trying to get a small game coded in Rust working in WebAssembly.

#### Rustup - please follow the directions for installing rustup at https://rustup.rs.
 Rustup installs the Rust programming language, and a number of tools used for package management, compilation, and toolchain management.

 After installation, you can run the following command from your command prompt:
 ```
 rustc --version
 ```

 If you see something like `rustc 1.23.0 (a5d1e7a59 2018-01-01)` then you are good to go.

####  Python 2.7
You'll need to install this run some post compilation instructions, and to host a simple HTTP server. Python is not entirely necessary for this SLAC so if you're allergic to Python, you can use an alternative HTTP server.

After installing Python 2.7, try this command from your command prompt:

```
python --version
```

If you see something like `Python 2.7.10` then you have a Python installation with a simple HTTP server.

If you want to do any other coding of Rust, and compilation to non-wasm targets (i.e. you want to write a Rust program to run as native code on your machine), you'll need to install Visual Studio Community Edition (https://www.visualstudio.com/vs/community/) and specify the appropriate workloads for your application. This also applies to installation on a Mac (for which I have no instructions).



## What is Rust

`rust` is a *systems programming language sponsored by Mozilla Research, which describes it as a "safe, concurrent, practical language", supporting functional and imperative-procedural paradigms.*. It won first place for "most loved programming language" in recent Stack Overflow Developer Surveys:

[2016]: https://insights.stackoverflow.com/survey/2016
[2017]: https://insights.stackoverflow.com/survey/2017

`rust` syntax looks similar to C and C++, but offers some elements of a meta-language. From the `rust` programmers' book: *Its design lets you create programs that have the performance and control of a low-level language, but with the powerful abstractions of a high-level language.*

Here are a few tid-bits about `rust`:

#### Rust code is compiled into a binary for a target platform

`rust` code is compiled into an executable binary for a target platform using a specified toolchain. This is made possible by both `rustup` and a compiler + toolchain project called `LLVM` (https://llvm.org/). For example, you can develop `rust` on a Windows machine and choose to compile it for a Mac. Or perhaps, as you'll see in this SLAC, we will compile `rust` code into `wasm`, which can run on all three major web browsers. While we won't explore too much today (and frankly, I'm not as knowledgeable about systems programming as I want to be), this is part of what makes coding in `rust` so friendly.


#### How does rust address variables and memory?  

When declaring a literal in `rust`, it is declared immutable by default. This is probably the clearest indication I saw of the functional/meta-language aspect of rust.

```
let foo = 5; //immutable
let mut bar = 5; //mutable
```

Variables are also statically typed - which means variable types must be derived or declared at compile-time.  The compiler does provide type-safety checks but for more granular control (especially in situations with polymorphism), a specific type can be specified or annotated in the example below. The language doesn't seem to be require super strict-typing (as the compiler seems to be able to derive variable type pretty well):

```
let mut helloString = String::new("42)"); //declaring a string
let helloString: u32 = helloString.trim().parse();  //now helloString is an unsigned 32 bit integer with value of 42
```

Other helpful information:
* Tuples are supported, and can contain distinct types
* Scalars exist for integers, floating point numbers, booleans, and characters. These are all single-value types
* Arrays can only contain a single type and are accessed like other languages

`rust` enables passing of data by reference (&) through method calls, as an alternative to passing by value (which is what Java does). The `rust` language also supports pointers (which stores the address of data) but typically does not recommend using them (given that they cannot be guaranteed to be safe at compile-time).  


#### Functions and statements

The function naming convention appears to be snakecase (all lower case with underscores) and need to be declared with the *fn* declaration, and have a set of parentheses after the function name. Functions can accept parameters, but the parameter type must be declared.

```
fn hi_function(x: u32) {
  println!("The value of x is: {}", x);
}
```

Statements in `rust` do not have return values. So you can't do *let x = (let y = 1);*. This is another example of the functional language aspect of `rust` creeping up...statement don't return values but expressions do.   


#### Conditionals and loops work as expected

Conditionals and loops don't have any special behaviors as far as I can tell.


#### Memory management is a bit confusing to me still

I don't fully appreciate the way `rust` manages memory yet. It doesn't perform run-time garbage collection (so there's no CPU cost which is great), nor does it require the programmer to explicitly de-allocate or allocate memory (so there's no programmer cost which is also great). It seems similar to the C++ concept of Resource Acquisitions Is Initialization (but I'm so rusty on C++ I don't remember this too well).

The idea of *scope* still applies so declaring a variable within the scope of a function means it's invisible and invalid for anything outside of that scope.

However, when it comes to memory management, there are some differences between how literals (which take up predictable memory space) are managed, and types (which don't take up predictable memory space).

```
let x = 5;
let y = x; //Both y and x now are equal to 5

let s1 = String::from("hello");
let s2 = s1; //Now s1 is no longer valid. When the compiler reaches here, it drops s1 from memory.
```
In some languages this is known as a "shallow copy". In order to actually copy the data (not just the reference), `rust` provides a function called *clone*.  

We won't spend too much time on this topic today but the `rust` book covers this in great detail in Chapter 4.


#### Everything else!

We won't go into any more classroom details on `rust` since the labs below can be done without getting into more advanced concepts (which I feel unprepared to speak about), but I would love to be a better `rustacean` (`rust` developer) so let's chat if you're interested too!


## What is WebAssembly

*WebAssembly* or `wasm` is a new portable, size- and load-time-efficient format suitable for compilation to the web. It recently achieved cross-browser compatibility on Firefox, Chrome, and Edge. Not too sure about Internet Expectorator. Here are some of the main reasons why WebAssembly saw massive adoption by the major browsers last year because of some of these reasons:

* The kind of binary format being considered for WebAssembly can be natively decoded much faster than JavaScript can be parsed (experiments show more than 20× faster). On mobile, large compiled codes can easily take 20–40 seconds just to parse, so native decoding (especially when combined with other techniques like streaming for better-than-gzip compression) is critical to providing a good cold-load user experience.

* Mostly HTML/CSS/JavaScript app with a few high-performance WebAssembly modules (e.g., graphing, simulation, image/sound/video processing, visualization, animation, compression, etc., examples which we can already see in asm.js today) allowing developers to reuse popular WebAssembly libraries just like JavaScript libraries today.

Some of the more gory details:

* Medium has an article on the JavaScript V8 Engine

[V8 Engine]: https://blog.sessionstack.com/how-javascript-works-inside-the-v8-engine-5-tips-on-how-to-write-optimized-code-ac089e62b12e

* WebAssembly Site
[Stack Engine]: http://webassembly.org/docs/semantics/
[FAQ]: http://webassembly.org/docs/faq/

I won't pretend to know how browsers work very well so hopefully the browser geeks among you can appreciate this suggested reading some more.

But to show off the power of this enhancement, the Tanks game (Unity engine) was recreated in `wasm` on the *WebAssembly* site - http://webassembly.org/demo/


## Labs

#### Alert On Meaning of Life

This lab is an alternative to a Hello World. It is really in that a simple `rust` file exists for you in the downloadable "life.zip" file. Here are the instructions.

Download the "life.zip" file from GitHub and unzip it. Use your command prompt to navigate to that folder and follow the instructions below:

```
$ rustup update nightly
$ rustup target add wasm32-unknown-unknown --toolchain nightly
$ rustc +nightly --target wasm32-unknown-unknown -O .\add.rs --crate-type=cdylib
$ cargo install --git https://github.com/alexcrichton/wasm-gc
$ wasm-gc .\add.wasm small-add.wasm
```

After you've followed those instructions, you'll notice another file has been created "small-add.wasm". Now execute your Python server to load up the index.html file:

```
python -m SimpleHTTPServer
```

You should see an alert explaining the meaning of life.



####  Rocket Game

This lab is pretty simple if you completed the last one. You just download and install this open-source game I found on Git. You can download the "rocket_wasm-master.zip" file and unzip it to a directory of your choice.

The creator of this game coded it in `rust`, and provided some instructions to compile to `wasm` so that it runs on your browser. There is a GitHub page on this topic but the instructions there aren't 100% correct. We will use a slightly modified version on this page:

After you've installed `rustup` per instructions on this site:

[Rustup]: https://www.hellorust.com/setup/wasm-target/

Follow these instructions to compile and optimize the game:

```
cargo +nightly build --release --target wasm32-unknown-unknown
python post_build.py
````

The generated wasm will be copied to the html directory of the installation folder. Navigate to that folder and start up your Python HTTP server:

```
python -m SimpleHTTPServer
```

Open the page on your browser http://localhost:8000/ to check if it works. Playing Instructions:

Keyboard                | Action
----------------------- | ------------
<kbd>&uparrow;</kbd>    | Boost
<kbd>&leftarrow;</kbd>  | Rotate left
<kbd>&rightarrow;</kbd> | Rotate right
<kbd>Space</kbd>        | Shoot
