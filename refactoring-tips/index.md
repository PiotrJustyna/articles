

# practical functional refactoring tips for the imperative world

[TODO] Practical refactoring workflow to help transform difficult to manage legacy code into a more functional, easier to manage version of itself.

This article is also a good starting point for those wanting to experiment with adopting F# but are not sure where to start.

Translation of pure code from C# to F#. In F# it is generally more difficult to fall into imperative programming traps that usually have long term side effects.

## how did we get here

[TODO] first article, 2021 goals: pure vs effectful code separation, mission-critical business logic is generally pure. Next year: effectfullness in F#. Legacy, frantically-written-at-3-am startup code.

## terminology

[TODO] pure, effectful, impure, mutation, global state, I/O, property based tests, business logic (archaic term)

## why functional programming

Among many others reasons which are not the focus of this article:

* **code != code**
  
  There's value in recognizing differences between effectful and pure code (side effects vs no side effects). What mature functional languages (e.g. Haskell, F#) propose is to embrace those very differences and make them explicit. Those differences are what the article recognizes as one of the most appealing features of functional programming.

* **composability**

  Working with a piece of pure code one can be sure that it is perfectly composable: global state is not accessed or mutated, variables - immutable, functions - self-contained. Chaining pure operations together is generally a very pleasant experience. On the other hand though, chaining effectful operations together is not as straightforward, but making the effectfullnes of the code explicit helps prepare for complications. In the world where pure and effectful pieces of code are mixed together, composition is usually not something that comes cheaply.

* **testability**

  Pure code is a joy to unit test. No dependency mocking or global state setup required. As a bonus, it is relatively easy to expose pure code to property based tests, etc.

result:

* easier maintenance
* shorter feedback loops
* happier engineers

## refactoring workflow

[TODO] Reference the mercury project. This is loosely based on an actual legacy class found in ERS.

### step 1 - identify different code responsibilities

This should be a rather obvious step for SOLID practitioners. Some legacy code tends to mix various responsibilities. It is always a good idea to identify and separate them from each other.

In our case there are 4 responsibilities:

* input validation
* api call
* mapping
* business logic

### step 2 - categorize responsibilities: pure vs effectful

In our case the categorization is straightforward:

* input validation - it is relatively easy to categorize this responsibility as pure (the code executed many times consecutively with the same input always gives identical results), but there is more than meets the eye and it should be noted:
  
  * exception-throwing code is considered pure
  * *exception-catching* code is not considered pure ([source](https://stackoverflow.com/questions/12335245/why-is-catching-an-exception-non-pure-but-throwing-an-exception-is-pure))
  
  It is crucial to remember this while designing/refactoring code.

* api call - effectful
* mapping - pure
* business logic - pure

### step 3 - DTOs/models/value objects

Make the models immutable. In our case all pure code gets translated from C# to F#, so we are going to create some models in the `MercuryLibrary` project.

[TODO] model code here

The code is also available here as a separate branch: [Step 3](https://github.com/PiotrJustyna/mercury-pure-functional/tree/Step3).

### step 4 - isolate remaining pure responsibilities

Move all pure responsibilities (in our case input validation, mapping and business logic) to pure, testable F# functions located in responsibility-specific modules. For the sake of simplicity, the article covers null-checking on the C# side so that it doesn't have to be done on the F# side where nulls are not a natural fit.

[TODO] input validation code here

[TODO] business logic code here

[TODO] mapping logic code here

The code is also available here as a separate branch: [Step 4](https://github.com/PiotrJustyna/mercury-pure-functional/tree/Step4).

### step 5 - unit test

In our case, the legacy code did not have any unit tests, so new tests will be needed.

[TODO] branch here