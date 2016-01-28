# divan's blog

## 1.Timers
In fact, you can build a simple timer with this approach - create a channel, start goroutine which writes to this channel after given duration and returns this channel to the caller of your func. The caller then blocks on reading from the channel for the exact amount of time.

```go
package main

import "time"

func timer(d time.Duration) <-chan int {
    c := make(chan int)
    go func() {
        time.Sleep(d)
        c <- 1
    }()
    return c
}

func main() {
    for i := 0; i < 24; i++ {
        c := timer(1 * time.Second)
        <-c
    }
}
```
![](timers.gif)

## 2.Ping-pong
Here we have a channel as a table of the ping-pong game. The ball is an integer variable, and two goroutines-players that ‘hit’ the ball, increasing its value (hits counter).

```go
package main

import "time"

func main() {
    var Ball int
    table := make(chan int)
    go player(table)
    go player(table)

    table <- Ball
    time.Sleep(1 * time.Second)
    <-table
}

func player(table chan int) {
    for {
        ball := <-table
        ball++
        time.Sleep(100 * time.Millisecond)
        table <- ball
    }
}
```
![](pingpong.gif)
Now, let’s run three players instead of two.
```go
go player(table)
go player(table)
go player(table)
```
![](pingpong3.gif)
We can see here that each player takes its turn sequentially and you may wonder why is it so. Why we see this strict order in goroutines receiving the ball?

The answer is because Go runtime holds waiting FIFO queue for receivers (goroutines ready to receive on the particular channel), and in our case every player gets ready just after he passed the ball on the table. Let’s check it with more complex example and run 100 table tennis players.

```go
for i := 0; i < 100; i++ {
    go player(table)
}
```
![pingpang100](pingpong100.gif)

## 3.Fan-In

One of the popular patterns in the concurrent world is a so called fan-in pattern. It’s the opposite of the fan-out pattern, which we will cover later. To be short, fan-in is a function reading from the multiple inputs and multiplexing all into the single channel.

For example:Fan-In
One of the popular patterns in the concurrent world is a so called fan-in pattern. It’s the opposite of the fan-out pattern, which we will cover later. To be short, fan-in is a function reading from the multiple inputs and multiplexing all into the single channel.

For example:
```go
package main

import (
    "fmt"
    "time"
)

func producer(ch chan int, d time.Duration) {
    var i int
    for {
        ch <- i
        i++
        time.Sleep(d)
    }
}

func reader(out chan int) {
    for x := range out {
        fmt.Println(x)
    }
}

func main() {
    ch := make(chan int)
    out := make(chan int)
    go producer(ch, 100*time.Millisecond)
    go producer(ch, 250*time.Millisecond)
    go reader(out)
    for i := range ch {
        out <- i
    }
}
```
![](fanin.gif)
As we can see, first producer generates values each 100 milliseconds, and second one - each 250 milliseconds, but reader receives values from both producers immediately. Effectively, multiplexing happens in a range loop in main.

## 4.Workers (so classic！！！)
The opposite pattern to fan-in is a fan-out or workers pattern. Multiple goroutines can read from a single channel, distributing an amount of work between CPU cores, hence the workers name. In Go, this pattern is easy to implement - just start a number of goroutines with channel as parameter, and just send values to that channel - distributing and multiplexing will be done by Go runtime, automagically :)

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func worker(tasksCh <-chan int, wg *sync.WaitGroup) {
    defer wg.Done()
    for {
        task, ok := <-tasksCh
        if !ok {
            return
        }
        d := time.Duration(task) * time.Millisecond
        time.Sleep(d)
        fmt.Println("processing task", task)
    }
}

func pool(wg *sync.WaitGroup, workers, tasks int) {
    tasksCh := make(chan int)

    for i := 0; i < workers; i++ {
        go worker(tasksCh, wg)
    }

    for i := 0; i < tasks; i++ {
        tasksCh <- i
    }

    close(tasksCh)
}

func main() {
    var wg sync.WaitGroup
    wg.Add(36)
    go pool(&wg, 36, 50)
    wg.Wait()
}
```

![](workers.gif)