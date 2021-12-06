- [practical functional refactoring tips for the imperative world - part 2: effectful f&#35;](#practical-functional-refactoring-tips-for-the-imperative-world---part-2-effectful-f)
  - [introduction](#introduction)
  - [how did we get here](#how-did-we-get-here)
  - [asynchronous code](#asynchronous-code)
  - [asynchrony in f&#35;](#asynchrony-in-f)
  - [mercury-functional](#mercury-functional)
    - [mercury code](#mercury-code)
      - [main - C&#35;](#main---c)
      - [main - F&#35;](#main---f)
      - [differences](#differences)
      - [log](#log)
      - [getWhoisResponse - C&#35;](#getwhoisresponse---c)
      - [getWhoisResponse - F&#35;](#getwhoisresponse---f)
  - [further reading](#further-reading)

# practical functional refactoring tips for the imperative world - part 2: effectful f&#35;

## introduction

This article covers a practical to follow refactoring workflow which helps transform difficult to manage legacy code
into a more functional, easier to maintain version of itself. It is a follow up to an article I published last quarter
which covered specifically the pure components of any program. You can find
it [here](https://github.com/PiotrJustyna/articles/blob/main/refactoring-tips/index.md). Every computer program consists
of a mixture of two types of code:

* **pure code** - no side effects, code executed multiple times with the same input always yields the same results (
  think Excel spreadsheet)
* **effectful/impure code** - code causing side effects: database/file system/network interactions but also things as
  trivial as telling the time (think distributed system where many machines are working together)

This article focuses on the latter: the effectful code. It should be a bit closer to real life compared to the last
article in the "practical functional refactoring tips" series and serve as a reference for the most common day-to-day
tasks most programmers handle. The aim is to embrace side effects as something that just needs to happen, but also to
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

## asynchrony in f&#35;

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
    * [`Async.AwaitTask`](https://docs.microsoft.com/en-us/dotnet/fsharp/tutorials/async#asyncawaittask)

* `async { }` - computation expression/asynchronous expression. All expressions of such form are of type `Async<T>` for
  some `T`. [*source*](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/async-padl-revised-v2.pdf)
* `expr := async { aexpr }` - complete syntax for asynchronous expressions. Selected `aexpr` examples:
    * `do!` - execute async (`Async<unit>`).
    * `let!` - execute and bind async (`Async<T>`).
    * `return! expr` - tailcall to async
    * `return expr` - return result of async expression

But here is where it gets interesting and where most implementation challenges are going to surface:

* C# - F# interop (`Async` vs `Task`) TODO, covered in getWhoisResponse
* exceptions
    * C#'s exception types are supported
* cancellations
    * C#'s `CancellationTokenSource` and `CancellationToken` are both supported
    * cancellation tokens are implicitly propagated through the execution of an asynchronous operation.
    * cancellation tokens are provided at the entry point to the execution of an asynchronous computation,
      e.g.: `Async.RunSynchronously`, `Async.StartImmediate`, `Async.Start`
    * cancelling nested asynchronous executions - cancellation tokens are passed implicitly to nested asynchronous expressions, but depending on how asynchronous work gets started, cancellations are handled differently in the nested operations, e.g.:
      * [Async.Start](https://docs.microsoft.com/en-us/dotnet/fsharp/tutorials/async#asyncstart) - If the parent computation is canceled, no child computations are canceled.
      * [Async.StartChild](https://docs.microsoft.com/en-us/dotnet/fsharp/tutorials/async#asyncstartchild) - If the parent computation is canceled, the child computation is also canceled.

## mercury-functional

In the article I published previously, we went through refactoring the pure responsibilities out of
C# [mercury-legacy](https://github.com/PiotrJustyna/mercury-legacy)
into [mercury-pure-functional](https://github.com/PiotrJustyna/mercury-pure-functional) - a C#-F# hybrid where the pure
code was written in F# and the effectful code was written in C#.

This time, we are going to need a new
repository: [mercury-functional](https://github.com/PiotrJustyna/mercury-functional) written exclusively in F# and based
on [mercury-pure-functional](https://github.com/PiotrJustyna/mercury-pure-functional).

### mercury code

Compared to `mercury-pure-functional`, both repositories are quite similar with one exception: the Host project. The project contains 3 functions which represent interesting differences between both implementations and we'll take a close look at all three. Let's start with the `main`/`Main` functions (the entry point to the program):

#### main - C&#35;

```c#
public static async Task Main(string[] args)
{
    Log("function is starting...");

    var apiUrlFormat = args[0];

    var domain = args[1];

    FSharpOption<Models.WhoisResponse> response = await GetWhoisResponse(
        apiUrlFormat,
        domain);

    Log(response.ToString());
    Log("function execution finished");
}
```

#### main - F&#35;

```f#
[<EntryPoint>]
let main argv =
    log "function is starting..."

    let apiUrlFormat = argv.[0]

    let domain = argv.[1]

    let job =
        async { return! getWhoisResponse apiUrlFormat domain }

    let response = Async.RunSynchronously(job)

    printfn $"{response.ToString()}"

    log "function execution finished"

    0
```

#### differences

* In the C# version, the `response` object is a result of an asynchronous function (`GetWhoisResponse`) being started and `await`ed.
* In the F# version, we have slightly more explicitly defined options of how the asynchronous `job` can get executed:
  * very much like a traditional C# `Task`, `job` only gets created and not immediately invoked
  * `response` is a result of `job` getting executed synchronously (no real benefits running it synchronously in our case), but other options are also available, e.g. `Async.StartChild` (for asynchronous execution).
* `!` notation allows us to nearly seamlessly introduce `Async` type into the code without too many changes. The key difference beeing the `async` vs expression results:
  * `let! pat = expr in aexpr` - execute & bind async. Example:
    ```f#
    // apiResponse is of type HttpResponseMessage
    let! apiResponse = client.GetAsync(apiUrl, cancellationToken) |> Async.AwaitTask
    ```
  * `let pat = expr in aexpr` - execute & bind expression
    ```f#
    // apiResponse is of type Async<HttpResponseMessage>
    let apiResponse = client.GetAsync(apiUrl, cancellationToken) |> Async.AwaitTask
    ```
  [source - chapter 2, An Overview of F# Asynchronous Programming](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/async-padl-revised-v2.pdf)
* It is possible to pass in a `CancellationToken` to `Async.RunSynchronously` and let it trickle down the execution chain (that is done implicitly) but in our case, `getWhoisResponse` has its own `CancellationTokenSource`.

#### log

Both functions are nearly identical, synchronous and as a result, comparing them is beyond the scope of this article.

#### getWhoisResponse - C&#35;

This is the busiest function of the project, let's take a close look at what's different between both implementations:

```c#
private static async Task<FSharpOption<Models.WhoisResponse>> GetWhoisResponse(
    string apiUrlFormat,
    string domain)
{
    FSharpOption<Models.WhoisResponse> response = null;

    InputValidation.whoisInputValidation(apiUrlFormat, domain);

    var apiUrl = string.Format(apiUrlFormat, domain);

    var cancellationTokenSource = new CancellationTokenSource(TimeSpan.FromSeconds(3));

    var cancellationToken = cancellationTokenSource.Token;

    var client = new HttpClient();

    var apiResponse = await client.GetAsync(
        apiUrl,
        cancellationToken);

    if (apiResponse.IsSuccessStatusCode)
    {
        var serializer = new XmlSerializer(typeof(Models.WhoisRecord));

        await using Stream reader = await apiResponse.Content.ReadAsStreamAsync(cancellationToken);

        response = Mappers.toWhoisResponse(
            DateTime.Now,
            domain,
            (Models.WhoisRecord)serializer.Deserialize(reader));
    }

    return response;
}
```

#### getWhoisResponse - F&#35;

```f#
let getWhoisResponse (apiUrlFormat: string) (domain: string) =
    InputValidation.whoisInputValidation apiUrlFormat domain

    let apiUrl = String.Format(apiUrlFormat, domain)

    let cancellationTokenSource =
        new CancellationTokenSource(TimeSpan.FromSeconds(3.0))

    let cancellationToken = cancellationTokenSource.Token

    let client = new HttpClient()

    async {
        let! apiResponse =
            client.GetAsync(apiUrl, cancellationToken)
            |> Async.AwaitTask

        if apiResponse.IsSuccessStatusCode then
            let serializer = XmlSerializer(typeof<WhoisRecord>)

            let! stream =
                apiResponse.Content.ReadAsStreamAsync(cancellationToken)
                |> Async.AwaitTask

            let whoisRecord =
                serializer.Deserialize(stream) :?> WhoisRecord

            return Mappers.toWhoisResponse DateTime.Now domain whoisRecord
        else
            return Option.None
    }
```

## further reading

* [Async programming in F#](https://docs.microsoft.com/en-us/dotnet/fsharp/tutorials/async)
* [The F# Asynchronous Programming Model](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/async-padl-revised-v2.pdf)
* [F# Async Guide](https://medium.com/@eulerfx/f-async-guide-eb3c8a2d180a)