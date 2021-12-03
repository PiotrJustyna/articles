# practical functional refactoring tips for the imperative world - part 2: effectful f#

## introduction + previous article

This article covers a practical to follow refactoring workflow which helps transform difficult to manage legacy code
into a more functional, easier to maintain version of itself. It is a follow up to an article I published last quarter
which covered specifically the pure components of any program. You can find
it [here](https://github.com/PiotrJustyna/articles/blob/main/refactoring-tips/index.md). Every computer program consists
of a mixture of two types of code:

* **pure code** - no side effects, code executed multiple times with the same input always yields the same results (
  think Excel spreadsheet)
* **effectful/impure code** - code causing side effects: database/file system/network interactions but also things as
  trivial as telling the time (think distributed system where many machines are working together)

This article focuses on the latter: the effectful code. It should be a bit closer to real life when compared to the last
article in the "practical functional refactoring tips" series and serve as a reference for the most common day-to-day
tasks most programmers perform. The aim is to embrace side effects as something that just needs to happen, but also to
keep them manageable and explicit, so that the code is relatively easy to maintain and extend.

Finally, the article should also serve as a good starting point for those wanting to experiment with adopting F# but are
not sure where to start.

## how did we get here

Most of 2021 we spent getting familiar with functional programming and with F# by adapting it for the pure
responsibilities of our code. That gave everybody the much needed time to get to know the language, the ecosystem, the
C#-F# interop tricks and gotchas. The basic building blocks of any new language.

Now that those aspects are no longer a mystery, we are ready for the more tricky effectful code. Why tricky? Everything
boils down to how to work with asynchrous code...

## asynchronous code

The trick to understanding asynchronous programming is to accept what it is/is not about:

* **what it is not about**:
    * concurrency - when multiple computations execute in overlapping time periods
    * parallelism - when multiple computations or several parts of a single computation run at exactly the same time
* **what it is about**:
    * asynchrony - when one or more computations can execute separately from the main program flow (asynchrony quite
      literally means "not-at-the-same-time")

[source - Async programming in F#](https://docs.microsoft.com/en-us/dotnet/fsharp/tutorials/async#asynchrony-defined)

## asynchrony in f#

Let's start with the reference material and terminology:

* `Async<T>` type represents composable asynchronous operations
* `Async` module contains functions which:
    * schedule
    * compose
    * transform

  asynchronous operations like:

    * [`Async.RunSynchronously`](https://docs.microsoft.com/en-us/dotnet/fsharp/tutorials/async#asyncrunsynchronously)
    * [`Async.Start`](https://docs.microsoft.com/en-us/dotnet/fsharp/tutorials/async#asyncstart)
    * [`Async.StartChild`](https://docs.microsoft.com/en-us/dotnet/fsharp/tutorials/async#asyncstartchild)
    * [`Async.Catch`](https://docs.microsoft.com/en-us/dotnet/fsharp/tutorials/async#asynccatch)

* `async { }` - computation expression/asynchronous expression. All expressions of such form are of type `Async<T>` for
  some `T`. [*source*](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/async-padl-revised-v2.pdf)
* `expr := async { aexpr }` - complete syntax for asynchronous expressions. Selected `aexpr` examples:
    * `do!` - execute async (`Async<unit>`).
    * `let!` - execute and bind async (`Async<T>`).
    * `return! expr` - tailcall to async
    * `return expr` - return result of async expression

But here is where it gets interesting and where most implementation challenges are going to surface:

* exceptions
    * C#'s exception types are supported
    * nested
* cancellations
    * C#'s `CancellationTokenSource` and `CancellationToken` are both supported
    * cancellation tokens are implicitly propagated through the execution of an asynchronous operation.
    * cancellation tokens are provided at the entry point to the execution of an asynchronous computation,
      e.g.: `Async.RunSynchronously`, `Async.StartImmediate`, `Async.Start`
    * nested cancellations

All listed terms and concepts are going to be demonstrated with easy to digest examples below.

## mercury-functional

In the article I published previously, we went through refactoring the pure responsibilities out of C# [mercury-legacy](https://github.com/PiotrJustyna/mercury-legacy) into [mercury-pure-functional](https://github.com/PiotrJustyna/mercury-pure-functional) - a C#-F# hybrid where the pure code was written in F# and the effectful code was written in C#.

This time, we are going to need a new repository: [mercury-functional](https://github.com/PiotrJustyna/mercury-functional) written exclusively in F# and based on [mercury-pure-functional](https://github.com/PiotrJustyna/mercury-pure-functional).
