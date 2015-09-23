---
author: Alishex
comments: true
date: 2014-10-10 16:18:25+00:00
layout: post
slug: multi-threading-and-java-memory-model
title: Multi-Threading & Java Memory Model
wordpress_id: 102
categories:
- Java
- Programming
tags:
- final
- java
- jsr-133
- memory
- model
- multi
- synchronization
- synchronized
- threading
- volatile
---

Hello. Been sometime since I posted anything but I broke my silence to talk about Java Memory Model.Okay so here is the game-plan, first we'll talk about some problems Java has that you won't notice unless you are doing something multi-threaded, analyze them and then we'll look how we can avoid such problems and where the memory model swoops-in to rescue.

_Does my program get executed in the order it is coded?_ Well literally speaking, no. Since the compiler may reorder the execution trace as long as the semantics does not change. So lets say we have such a piece of goofy code:

    
    
    public static void main(String[] args){
    	int a,b;
    	a = 1;
    	b = 2;
    
    	int c;
    	c = a;
    }
    


Lets talk about what happens here:

* We create two primitive local variables whom gets the value 0 on creation.
* Perform a write actions on them both giving them new values.
* Create a new local variable.
* Write the value of first variable to the last.

Although you may expect that every proccess happens in this exact order that may not be true due to optimizations made by various components such as compiler and Just-in-Time compiler on JVM. Yet Java guarantees you this, semantics of the program cannot change. Which means that if there is a read of variable "a" during the write of variable "c" then the write to "a" before the last read must be visible therefore our poor old "c" gets the value "1" and the balance is restored, Java saves the world, go home people, there is nothing more to see here.

Everything is nice and dandy for now but what happens when a Thread takes cares of writing to "a" and another Thread that will read the value of "a"? Here comes problem:

    
    
    // Thread 1
    public void someStuff(){
    	a = 1;
    	x = b;
    }
    
    // Thread 2
    public void someOtherStuff{
    	b = 2;
    	y = a;
    }
    


What happens now? There is no way each thread can mess with intra-thread(inside thread) semantics but what about inter-thread(between threads) semantics? Does Thread 1 which is doing someStuff gets to read the variable "b" before Thread 2 who is doing someOtherStuff writes to "b" which will end up with variable "x" having the value "2". This possibility goes the other way too which will end up in either "y" having the value "1" which is intended or the value "0" which is the default value for an int. This problem is called a data race. There are such fun situations in programming which will drive you nuts if you don't know what you are doing.

Like reordering is not enough there are times when CPU may cache the variable in register to ease the traffic on shared memory bus. Here is a situation where such a problem can occur:

    
    
    // Thread 1
    public void looper(){
    	while(!done){
    		Thread.sleep(1000); // Important thing to note that sleep() and yield() does not have synchronization semantics so the compiler does not have to flush writes cached in register to shared memory.
    	}
    }
    // Thread 2
    public void changer(){
    	done = true;
    }
    



I think you understood by now that with great CPU power comes great responsibility to do things properly. What makes this even more scary is that these behaviours may vary according to your CPU architecture. Even while writing these lines I am thinking about quitting my job and become a fisherman BUT Java saves the day again! Twice in a post! How? BAM! with keywords such as **volatile, synchronized and final**.

Now before we proceed further to memory model, first we should see what are our tools to prevent unexpected executions traces then I hope we'll get a clearer view where a memory model falls in.

Let's start with synchronization. Since each Java class implicitly extends **java.lang.Object** class(In fact only class that does not do this is the Object class itself), every object comes to life with some synchronization mechanisms of their own, such as a monitor. A monitor is basically a locking system which can be locked by one thread at a time, it is like hold the balls of someone over the course of conversation, only one person can hold the balls at a time, if someone else wants a grip they should wait for you to release. So when a thread holds the lock on a monitor of an object, all the other threads that want to do something with it must wait for the monitor to be unlocked. You can interact with an objects monitor by using **synchronized** keyword and there are two places to use it.
First way is to use a synchronized block:

    
    
    public void method(){
    	synchronized(object){
    		// During this time you hold the lock on object's monitor.
    	}
    }
    



Second way is to use a synchronized method:

    
    
    public synchronized void syncMethod(){
    
    }
    


Okay then, here is the question. At first example we used the monitor of "object" but which object monitor gets locked at the second one? Well in this example since the method is not **static** this means the instance(**this**) object's monitor gets the lock. If the method is static then it is the Class object that feels the grope.
There is more to an Object then monitor such as Waiting Sets but that has to wait() till I notify() them on another topic.

Lets talk about **volatile** variables a bit. Volatiles are shared memory variables that are "special", so to speak. They get extra attention when they are getting read and write. Memory barriers gets applied to them so when somebody is doing something with them(read or write), everyone(all the other threads) should wait and watch. Well watching part is the important part since every write on a volatile variable is immediately visible to other threads. There are also some other semantics that volatile has which we will talk about later on.

Now is a good time to talk about the **final** semantics. It is the keyword used to define an immutable object. What is an immutable object? It is an object that cannot be modified after construction. So sorry to say but there are no countdowns happening here, [Europe](http://www.youtube.com/watch?v=9jK-NcRmVcw). Since these values cannot be changed after construction CPU may cache the value of **final** fields and compilers may freely reorder them. Also the immutability guarantees that any thread will see the correctly initialized value of the **final** field.

By now this post is far more long then intended so I will try to keep it brief. After all this talk about multi-threading and the problems it comes hand to hand with, lets talk about what does a memory model do in all this mayhem. Well, a memory model is a set of rules that guarantees you certain behaviour on every environmental combination you work on.
Before 2004, Java had a weak memory model where things didn't go as intended for most of the time. **final** fields observed to change value, **volatile** fields reordered a lot with non-volatile fields which messed inter-thread semantics and similar behaviours often came from **synchronized** usages UNTIL JSR-133 came down the hills on it's horse with all new **volatile,final and synchronized** semantics at it's sides, on the first light of the fifth <del>day</del> Java(which is called Tiger). Oracle talks about these set of rules in various manners but after researching a lot about using multi-thread mechanism it all boils down to few rules, the happens-before relationship. Which is :

* An unlock on a monitor _happens-before_ every subsequent lock on that monitor.
* A write to a volatile field _happens-before_ every subsequent read of that field.
* A call to start() on a thread _happens-before_ any actions in the started thread.
* All actions in a thread _happen-before_ any other thread successfully returns from a join() on that thread.
* The default initialization of any object _happens-before_ any other actions (other than default-writes) of a program.



Of course, there are a lot more to Java Memory Model than this so please check [Java Language Specifications](http://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html) for more.

Okay here are my dying words while the screen fades to black. Know what you are doing. Most programmers consider **volatile** a fairy-dust, avoid logics such as "sprinkle it here and there enough and things start to feel as they should be". Knowing what you are doing makes you feel like the boss (that you are), and keeps you from hamstringing your own program.
