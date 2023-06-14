---
type: "post"
title: "The Rustification of data engineering, and why it matters for Python users"
date: 2023-06-12T14:40:42-04:00
draft: true
showTableOfContents: true
image: "/images/Tributaries11539-0.jpg"
tags: ["rust", "data-engineering", "databases"]
---

## Rust is eating the PyData ecosystem

Imagine you're on a boat sailing along with the current on a river. Now, imagine that the lower decks of this boat _are being rebuilt and modified_ with you still on it, continuing to sail forward! That's what it feels like to be a data engineer using Python today, because, as I'll describe in this post, there are significant movements happening right now in the internals of the PyData ecosystem that are worthy of note.

Let's first talk about Rust, the [much loved](https://www.infoworld.com/article/3546337/rust-language-tops-stack-overflow-survey.html) programming language. Having had its stable 1.0 release in 2015, the initial few years for the language were mostly about finding its niche among systems developers, which is the main purpose for which language was designed, and it quickly became the most loved programming language in subsequent years' StackOverflow surveys. I won't go into the benefits of Rust as a language in full, as there is a _lot_ of content online that covers this. Fast forward to today, and we can be sure that [the Rust language is here to stay](https://www.youtube.com/watch?v=A3AdN7U24iU), having built a robust, stable and welcoming community.

## What makes for widely adopted languages?

For a programming language ecosystem to flourish, it's not enough to just have good design principles for the language itself. It also takes the following:

- Adoption from a core "power user" base that builds foundational tooling
- Buy-in from more developers that see the need for a new language
- Open, inclusive community
- Great learning material, available largely for free
- Useful real world applications that justify the need and use of the language

It's safe to say that, since 2015, the Rust community, and the language itself, has delivered on all counts. The following key features of Rust set it apart from a lot of other languages, making it resonate deeply with systems programmers in particular.

- Blazing fast 🔥 performance
- Memory safety
- Great support for concurrency

## A confluence of great tooling

What's most exciting, however, are the specific developments in Rust's tooling ecosystem that have come together in a very nice way, and whose combined potential hasn't been unleashed yet. Going back to our river analogy, the image below shows a number of streams joining a river that's flowing from left to right.


![](./tributaries.png "Image modified from: https://travellingacrosstime.com/2015/08/23/rivers/")

The following projects and their associated Rust crates ("crates" are packages in the Rust ecosystem) have come together really nicely in 2023:

* Actix web
* Hyper
* PyO3 + Maturin
* Arrow
* DataFusion
* Ballista

It's okay if all the terms don't ring any bells yet. Read on!

### The ubiquity of Python

With [more than 15 million developers worldwide](https://www.statista.com/statistics/1241923/worldwide-software-developer-programming-language-communities/#:~:text=Python%20is%20also%20a%20popular,programmers%2C%20with%2015.7%20million%20developers.), it's safe to say that Python won the race for ML, data science and general purpose computing. The reasons for Python's popularity are its readable syntax, low barrier to entry, and universal applicability in nearly all domains.

But because Python is a dynamically-typed interpreted language, it suffers in performance compared to statically typed, compiled languages. As a result, Python always relied on lower-level systems languages to perform the heavy lifting for expensive computations (things that involve loops or iteration). In the past, these languages used to be C, C++, Cython or even Fortran.

### The impact of PyO3

It's hard to overstate the importance of one particular Rust crate, [PyO3], which came out in 2017. PyO3 allows Rust developers to _very easily_ produce Rust bindings that can be used in Python, giving Python developers access to the full power of Rust. Although similar tools like pybind11 and SWIG exist for C++, portability and ease of sharing the binaries were always a challenge, and this is where Rust + PyO3 really stood out. The quality of Rust's crates.io package management system, combined with a thriving community that [documented the project](https://pyo3.rs/v0.19.0/) really well for developers from all languages, helped bring a large number of really smart people into the Rust ecosystem.

