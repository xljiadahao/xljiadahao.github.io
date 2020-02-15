---
title: Concurrency 高并发 - Future and CompletableFuture
date: 2020-01-20
tags:
  - concurrency
  - multi-threads
---
Future class provides a very easy way to execute async operations. As of AIO, Future class manages a seperated thread to do the IO operation asynchronously. CompletableFuture is introduced by JDK 1.8 as the enhanced Future async model. It brings the complex future tasks pipeline with the functional interfaces which are also introduced by JDK 1.8. To understand this blog better, the pre-condition is the knowledge of the functional interfaces of JDK 1.8 [Java 8 Features](https://xljiadahao.github.io/2018/02/04/Java-8-Features/).

*Sample: _com.snippet.concurrent.future_ of the Snippets repository in [GitHub](https://github.com/xljiadahao/Snippets.git).*

`Future`
Future is a interface which defines the basic operations to manage the asynchronous task, but Future itself cannot init a thread. Thus, let's use FutureTask for the demo code. `FutureTask` implements RunnableFuture, while RunnableFuture is a interface which extends Runnable, Future, so FutureTask has all the Future async task features as well as the threading capability.
```
public class AsyncCallableTask implements Callable<String> {
    public String call() throws Exception { ... }
}

AsyncCallableTask task = new AsyncCallableTask();
FutureTask<String> futureTask = new FutureTask<>(task);
new Thread(futureTask).start();
boolean isDone = futureTask.isDone();
while(!isDone) {
    System.out.println("isDone: " + isDone + ", waiting 4 seconds for the downstream service call, do something else. "
            + Thread.currentThread().getName());
    Thread.sleep(4000);
    isDone = futureTask.isDone();
    Random r = new Random();
    int randomTimeout = r.nextInt(10);
    boolean isTimeout = randomTimeout > 8 ? true : false;
    if (isTimeout) {
        System.out.println("timeout " + randomTimeout + ", cancel job.");
        futureTask.cancel(true);
        break;
    }
}
if (futureTask.isCancelled()) {
    System.out.println("cancelled, isDone: " + futureTask.isDone() + ". " + Thread.currentThread().getName());
} else {
    System.out.println("isDone: " + futureTask.isDone() + ", result: " + futureTask.get() + ". " + Thread.currentThread().getName());
}
```
When we initialize the FutureTask, it accepts a Callable task which is the task to be executed asynchronously. Then we create a thread for the FutureTask, and internally it will call the Callable task. `futureTask.isDone()` checks the current running task status, and if the task is completed, return true. In our use case, if it's timeout, `futureTask.cancel(true)` attempts to cancel execution of this task. Finally `futureTask.get()` fetches result from the Callable task.

`CompletableFuture`
CompletableFuture is one of implementations of the Future interface. It has been introduced by JDK 1.8, so those methods of CompletableFuture are easily integrated with the functional interfaces. CompletableFuture provide a very easy way to chain those async tasks together as a complex async tasks pipeline.
a. `CompletableFuture basic methods`
```
CompletableFuture<String> futureAsyncTask = CompletableFuture.supplyAsync(() -> {
    try {
        System.out.println("do something");
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    return "ok";
});
System.out.println(futureAsyncTask.getNow("not_found"));
System.out.println(futureAsyncTask.join());
```
`CompletableFuture.supplyAsync()` accepts a Supplier functional interface which defines the task. Once the CompletableFuture is created, the task will be running in a seperated thread. `futureAsyncTask.getNow("not_found")` fetches the result from the task, and if the task is not completed, return the default result which is the arguments. `futureAsyncTask.join()` makes the main thread synchronized to wait and fetch the result until the async task is completed. The join() method is similar to the get() method, but join() method does not require to catch the checked Exception which may be throwed in the seperated thread specifically in the main thread. Here is one more example below.
```
List<String> visitList = Arrays.asList("Lasha", "Linzhi", "Yangzhuoyongcuo", "Yaluzangbujiang");
List<CompletableFuture<String>> futureAsyncTaskList = visitList.stream()
        .map((String visit) -> CompletableFuture.supplyAsync(() -> {
            Random r = new Random();
            int time = (r.nextInt(5) + 1) * 1000;
            try {
                Thread.sleep(time);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
             return String.format("visit %s for %d hours, async task %s", visit, time/1000, Thread.currentThread().getName());}))
        .collect(Collectors.toList());
try {
    Thread.sleep(4000);
} catch (InterruptedException e) {
    e.printStackTrace();
}
futureAsyncTaskList.stream().map((CompletableFuture<String> task) -> task.getNow("not_ready")).forEach((String plan) -> System.out.println(plan));
```
b. `Compare with Executors`
```
ExecutorService es = Executors.newCachedThreadPool();
List<Future<String>> futureList = visitList.stream()
        .map((String visit) -> es.submit(() -> {
            Random r = new Random();
            int time = (r.nextInt(5) + 1) * 1000;
            try {
                Thread.sleep(time);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return String.format("visit %s for %d hours, async task %s", visit, time/1000, Thread.currentThread().getName());}))
        .collect(Collectors.toList());
futureList.stream().map((Future<String> task) -> {
    try {
        return task.get();
    } catch (InterruptedException e) {
        e.printStackTrace();
    } catch (ExecutionException e) {
        e.printStackTrace();
    }
    return null;
}).forEach((String plan) -> System.out.println(plan));
```
c. `CompletableFuture advanced features`
As we mentioned earlier, CompletableFuture can build the complex future tasks pipeline.
```
CompletableFuture<String> futureWhenCompleteTask0 = CompletableFuture.supplyAsync(() -> {
    try {
        System.out.println("do something futureWhenCompleteTask0");
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    return "Singapore";
});
```
c.1 `whenComplete()`
```
CompletableFuture<String> futureWhenCompleteTask1 = futureWhenCompleteTask0.whenComplete((String result, Throwable e) -> {
    try {
        System.out.println("do something futureWhenCompleteTask1");
        Thread.sleep(1000);
    } catch (InterruptedException e1) {
        e1.printStackTrace();
    }
    System.out.println(String.format("now: %s -> future: China", result));
});
System.out.println("futureWhenCompleteTask1 final result: " + futureWhenCompleteTask1.join());
```
When futureWhenCompleteTask0 stage is completed, taking the result of first stage, the given action of futureWhenCompleteTask1 is getting called. But eventually the futureWhenCompleteTask1 result is still from futureWhenCompleteTask0, and the action of futureWhenCompleteTask1 can be considerd as the additional action after futureWhenCompleteTask0 produces result, because whenComplete() takes a BiConsumer as the action. Console log, now: Singapore -> future: China, futureWhenCompleteTask1 final result: Singapore.
c.2 `thenApply()`
```
CompletableFuture<String> futureWhenCompleteTask2 = futureWhenCompleteTask0.thenApply((String result) -> {
    try {
        System.out.println("do something futureWhenCompleteTask2");
        Thread.sleep(1000);
    } catch (InterruptedException e1) {
        e1.printStackTrace();
    }
    System.out.println(String.format("now: %s -> future: China", result));
    return String.format("now: %s -> future: China", result);
});
System.out.println("futureWhenCompleteTask2 final result: " + futureWhenCompleteTask2.join());
```
Compare with the whenComplete() method, due to thenApply() taking a java.util.function.Function, thenApply() method returns the computed result. Console log, now: Singapore -> future: China, futureWhenCompleteTask2 final result: now: Singapore -> future: China.
c.3 `thenCompose()`
```
CompletableFuture<String> futureWhenCompleteTask3 = futureWhenCompleteTask0.thenCompose((String result) -> 
    CompletableFuture.supplyAsync(() -> result + " -> Chengdu"));
System.out.println("futureWhenCompleteTask3 final result: " + futureWhenCompleteTask3.join());
```
Console log, futureWhenCompleteTask3 final result: Singapore -> Chengdu.
c.4. `thenCombine()`
```
CompletableFuture<String> futureWhenCompleteTask4 = futureWhenCompleteTask2.thenCombine(
        CompletableFuture.supplyAsync(() -> "-Chengdu"), (String t2, String t4) -> t2 + t4);
System.out.println("futureWhenCompleteTask4 final result: " + futureWhenCompleteTask4.join());
```
futureWhenCompleteTask4 and futureWhenCompleteTask2 run concurrently, once both of the two tasks are completed, it returns the computed result. Console log, futureWhenCompleteTask4 final result: now: Singapore -> future: China-Chengdu.
c.5 more methods
`thenAccept()`, `allOf()` (similar to CountDownLatch or CyclicBarrier), `anyOf()`, etc.
d. `why was CompletableFuture introduced by JDK 1.8?`
As mentioned above, CompletableFuture provide a very easy way to build the complex future tasks pipeline such as  IO tasks. Its default thread pool is ForkJoinPool which of the pool size is based on CPU cores. CompletableFuture provides multiple methods for pipeline such as whenComplete, thenApply, thenCompose, thenCombine, allOf, anyOf, etc.
e. `Additional knowledge, thread pool size`
CPU denseness task: thread pool size should be based on CPU cores.
IO intensive task: thread pool size should be based on CPU cores * CPU utilization ratio * (1 + thread waiting time/thread CPU holding time).
In summary, regarding the IO operations, the default ForkJoinPool cannot get the best performance, and the customized thread pool is recommended.
