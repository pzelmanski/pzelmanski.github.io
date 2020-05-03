---
layout: post
title: F# | Async test result Inconclusive: Test not run
categories: "FSharp F#"
---

I was trying to write an async test for an async result type, but every time i ran tests I got this horrible message telling me nothing about what's wrong.

First, here's a result message after just running a test:
```
CommonTests.AsyncResultTests.AsyncResult test

Test not run

Last runner error: An existing connection to remote test runner was forcibly closed by the test runner.

Last launch log file: C:\Users\Papagaio\AppData\Local\JetBrains\Rider2020.1\log\UnitTestLogs\Sessions\2020-05-03-21-56-20-346___bf60b858-d451-4bdd-9bb8-3fd04289d3e3.log
```

When debugging a test, there was an exception thrown. It was somewhere in internals of an async implementation, exactly at `async.fs`:
```
System.NullReferenceException: Object reference not set to an instance of an object.
  at Microsoft.FSharp.Control.AsyncPrimitives.QueueAsync@824.Invoke(Unit unitVar0) in E:\A\_work\130\s\src\fsharp\FSharp.Core\async.fs:826
  at Microsoft.FSharp.Control.Trampoline.Execute(FSharpFunc`2 firstAction) in E:\A\_work\130\s\src\fsharp\FSharp.Core\async.fs:109
  at Microsoft.FSharp.Control.TrampolineHolder.ExecuteWithTrampoline(FSharpFunc`2 firstAction) in E:\A\_work\130\s\src\fsharp\FSharp.Core\async.fs:177
  at <StartupCode$FSharp-Core>.$Async.-ctor@163-1.Invoke(Object o) in E:\A\_work\130\s\src\fsharp\FSharp.Core\async.fs:165
  at at System.Threading.QueueUserWorkItemCallbackDefaultContext.Execute()
  at at System.Threading.ThreadPoolWorkQueue.Dispatch()
  at at System.Threading._ThreadPoolWaitCallback.PerformWaitCallback()
```


The whole code of my tests looks like this:
```fsharp
open Xunit
module AsyncResultTests =
    let dummy : Async<Result<int, string>> =
        async { return Ok 15 }
    let dummy2 : Async<Result<int, string>> =
        15 |> Ok |> async.Return

    [<Fact>]
    let ``AsyncResult test`` () =
        let x = dummy |> Async.RunSynchronously
        Assert.Equal(x, Ok 15)
    
    [<Fact>]
    let ``AsyncResult test2`` () = async {
        let! x = dummy2
        Assert.Equal(x, Ok 15) }
```

And the solution was... adding a `Microsoft.NET.Tests.Sdk` (version 16.7.0) to test project. Just like this.
The way I found it out is also pretty funny - in an act of desperation, I created a new solution.
There it worked.
So I compared differences between two projects.

It took me only a little over an hour to solve this, ehh.