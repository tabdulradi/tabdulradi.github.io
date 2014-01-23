---
layout: post
title: 'Pausable Actors using Stash'
tags: akka stash actor model pausable 
category: blog
---

```
import akka.actor.{Actor, Stash}
class MyActor extends Actor with Stash {
  def receive = accepting
  def accepting: Receive = {
    case Do => 
      // Do logic here
    case Wait => 
      context.become(waiting)
  }
  def waiting: Receive = {
    case Do =>
      stash()
    case Resume =>
      unstashAll()
      context.become(accepting)
  }
}
object MyActor {
  case object Do
  case object Wait
  case object Resume
}
```
You should use Stash, if you want your actor to go "offline" for sometime, may be because a necessary external resource is down, like a database for example. However while the actor is offline, it should remember all the messages that it couldn't handle right now, hoping it can recover soon, and handle those messages.

The previous example waits for recovery or "Resume" message forever. A more complex example, involving timeouts to end this "waiting" status, should be like:
```
import akka.actor.{Actor, Stash}
class PausableActor extends Actor with Stash {
  val timeout = 1.minute
  def logic: Receive 
  def receive = logic orElse checkDown
  def checkDown: Receive = {
    case Down => 
      val cancellable = 
        context.scheduler.scheduleOnce(timeout, self, Kill) 
      context.become(paused(cancellable))
  }
  def paused(cancellable: Cancellable): Receive = {
    case Up =>
      cancellable.cancel()
      unstashAll()
      context.unbecome()
    case Kill =>
      context.stop(self)
    case msg =>
      stash()
  }
}
object PausableActor {
  case object Down
  case object Up
  case object Kill
}

class MyActor extends PausableActor {
  def logic = {
    case msg =>
      // handle msg
  }
}

```
