---
title: Concurrency 高并发 - CountDownLatch and CyclicBarrier
date: 2020-01-19
tags:
  - concurrency
  - multi-threads
---
*Use Case 1*: checkout of the shopping cart, calling merchant downstream services concurrently to get and confirm the latest bills, once confirm the total amount, calling multiple payment gateway APIs concurrently to get all the available payment approaches with enough balance.
*Use Case 2*: running race, 3 times, the winner is the one who get the minimum of time in total.

There are some use cases which require the running threads in certain order. CountDownLatch and CyclicBarrier are the tools to allow those threads waiting for each other.

*Sample code: _com.snippet.concurrent_ and _com.snippet.test.concurrent_ of the Snippets repository in [GitHub](https://github.com/xljiadahao/Snippets.git).*

`CountDownLatch`
Thread groups are waiting for each other within multiple thread groups.
```
public class Player extends Thread {...}

CountDownLatch latch = new CountDownLatch(4);
new Player(latch, "merchant1").start();
new Player(latch, "merchant2").start();
new Player(latch, "merchant3").start();
new Player(latch, "merchant4").start();
latch.await();

System.out.println("countinue to do something else such as calling payment gateway");
```
The sample code above shows that it starts 4 threads concurrently, registers a CountDownLatch to count the 4 threads, and sets a barrier until all of the 4 running threads are finished.
```
public class Player extends Thread {
    ...
    @Override
    public void run() {
        ...complete the task...
        latch.countDown();
    }
}
```
Once each thread completes the task, at the end it calls `latch.countDown()` to count down or -1 of the CountDownLatch. CountDownLatch will remove the barrier of the thread group when the count of CountDownLatch == 0, then the main thread could continue to execute the rest of the code.

`CyclicBarrier`
Threads are waiting for each other within a single thread group. It can be recycling used.
```
private int currentTime = 0;
private int maximumScore = 0;
private boolean isOngoing = true;
private Map<String, Integer> scoreboard = new ConcurrentHashMap<>();

CyclicBarrier cyclicBarrier = new CyclicBarrier(3, () -> {
    System.out.println("CyclicBarrier executed, round " + ++currentTime);
    isOngoing = true;
    if (currentTime >= 4) {
        scoreboard.entrySet().stream().sorted((Map.Entry<String, Integer> o1, Map.Entry<String, Integer> o2) -> o1.getValue() == o2.getValue() ? 0 : (o1.getValue() > o2.getValue() ? -1 : 1)).forEach((Map.Entry<String, Integer> runner) -> {
            if (runner.getValue() >= maximumScore) {
                maximumScore = runner.getValue();
                System.out.println("Announce Final Winner: " + runner.getKey() + ", winning times: " + runner.getValue());
            } 
        });
    }
});
```
Define a CyclicBarrier, here are two parameters of the one of constructors:
a. parties the number of threads that must invoke await before the barrier is tripped.
b. barrierAction the command to execute when the barrier is tripped, or null if there is no action.
```
for (int i = 0; i < 3; i++) {
    new Thread(()->{
        scoreboard.put(Thread.currentThread().getName(), 0);
        while (currentTime < 4) {
            try {
                Thread.sleep(new Random().nextInt(5) * 1000);
                System.out.println(Thread.currentThread().getName() + " reached barrier.");
                updateWinnerStatus(Thread.currentThread().getName());
                cyclicBarrier.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
        }
    }, "runner" + i).start(); 
}

// pessimism lock
private  synchronized void updateWinnerStatus(String runner) {
    if (isOngoing) {
        scoreboard.put(runner, scoreboard.get(runner) + 1);
        System.out.println("winner: " + runner);
        isOngoing = false;
    }
}
```
In each thread, `cyclicBarrier.await()` will be called. Once the `await()` of all the 3 threads are called, each thread of those 3 can continue execute the rest of the code. CyclicBarrier is getting reset and it can be resued again.

`CountDownLatch vs  CyclicBarrier`
a. In general, thread groups are waiting for each other, use CountDownLatch; within a group of threads, each thread is waiting for each other, use CyclicBarrier.
b. CountDownLatch is counting --, while CyclicBarrier is counting ++.
c. CountDownLatch can be used when the count reached 0 and it cannot be reset, while CyclicBarrier can be reset each time, reused multiple times.
