---
layout: page
title: Async (Task Support)
---

From the beginning Caliburn.Micro uses IResult and Coroutines for asynchronous operations.

With .NET 4.0 Microsoft added Task to the .NET Framework for better supporting parallel/async operations. And in .NET 4.5 they added async/await keywords to C# and VB.NET.

---
><font color="#63aebb" face="微软雅黑">从一开始，Caliburn.Micro就使用IResult和Coroutines进行异步操作。

>微软在 .NET 4.0 中 将任务添加到 .NET Framework 以更好地支持并行/异步操作。在 .NET 4.5 中，为 C# 和 VB.NET 增加了async / await 关键字。</font>

#### Caliburn.Micro has extensions to support Task - Caliburn.Micro支持扩展Task

Instead of registering an callback when a coroutine is completed you now can await it:

---
><font color="#63aebb" face="微软雅黑">在完成协同程序时，你现在可以等待它，而不是注册回调：</font>

``` csharp
public static class Coroutine {

    public static void BeginExecute(
		IEnumerator<IResult> coroutine, 
		CoroutineExecutionContext context = null, 
		EventHandler<ResultCompletionEventArgs> callback = null) { }

    public static Task ExecuteAsync(
		IEnumerator<IResult> coroutine,
		CoroutineExecutionContext context = null)  { }

}
```

A Task object can be wrapped in an IResult and used inside a coroutine as if it were an IResult: 

---
><font color="#63aebb" face="微软雅黑">Task 对象可以封装在 IResult 中，并在协程中使用，就像它是 IResult 一样:</font>

``` csharp
public static class TaskExtensions {

    public static Task ExecuteAsync(
		this IResult result, CoroutineExecutionContext context = null) { }

    public static Task<TResult> ExecuteAsync<TResult>(
		this IResult<TResult> result,
		CoroutineExecutionContext context = null) { }

    public static TaskResult AsResult(this Task task) { }

    public static TaskResult<TResult> AsResult<TResult>(this Task<TResult> task) { }

}
```

##### Example - 示例

``` csharp
yield return Task.Delay(500).AsResult();
```

The other way round also works as you can wrap an IResult in a Task by ExecuteAsync.

---
><font color="#63aebb" face="微软雅黑">另一种方法也可以通过 ExecuteAsync 将 IResult 包装到 Task 中。</font>

##### Example

``` csharp
await new SimpleResult().ExecuteAsync();
```

And finally also EventAggregator supports Task with the IHandleWithTask<TMessage> interface.

---
><font color="#63aebb" face="微软雅黑">最后，EventAggregator还支持使用IHandleWithTask的Task 接口。</font>

[目录](.index.md)