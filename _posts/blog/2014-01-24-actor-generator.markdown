---
layout: post
title: 'Generators/Coroutine using Actors'
tags: akka actor generator coroutine
category: blog
---

```
class FibonacciGenerator extends Actor {
  import FibonacciGenerator._

  def receive = fib()

  def fib(a: Long = 0, b: Long = 1): Receive = {
    case Next =>
      val c: Long = a + b
      sender ! c
      context.become(fib(b, c))
  }
}

object FibonacciGenerator {
  case object Next
}
```
Back in first days using Akka, I didn't notice that you can such lazy recursion using Actors. The previous piece of code let's you generate infinite (until you hit Long.MaxValue) fibbonci numbers.

This code can tested like this:

```
object Main extends App {
  import FibonacciGenerator._
  import ActorDSL._

  implicit val system = ActorSystem("test")
  val fib = system.actorOf(Props[FibonacciGenerator])

  actor(new Act {
    whenStarting { fib ! Next }
    become {
      case x: Long if x < 9999999999L =>
        println(x)
        fib ! Next
      case x =>
        println(x)
        println("Bye")  
        system.shutdown
    }
  })
}
```
[Full code](https://gist.github.com/tabdulradi/cbc5f95b33b430edcbc5)
