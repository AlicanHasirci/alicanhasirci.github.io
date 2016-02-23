---
layout: post
title: "Singleton"
date: 2014-07-11 14:17:23
image: '/assets/img/'
description: One Object To Rule Them All
tags:
- creational design patterns
- design pattern
categories:
- Creational
- Design Patterns
twitter_text: Singleton
---


Singleton is a simple object-oriented design pattern where a class only gets to have one single instance at any given time while providing a global point of access.

**Intent**

* Ensures that only one instance of class is exists at a given time.
* Provides a global point of access to the instance.
    

Here is how you create a singleton:
{% highlight java %}
    class SampleSingleton{
    	
    	private static SampleSingleton instance;
    
    	private SampleSingleton(){
    		// initiate components
    	}
    
    	// This is the global access point
    	public synchronized SampleSingleton getInstance(){
    		if(instance == null){
    			instance = new SampleSingleton();
    		}
    		return instance;
    	}
    
    	public void foreverAlone(){
    		// ...
    	}
    }
{% endhighlight %}

After doing this all you need to do is:
{% highlight java %}
    	SampleSingleton.getInstance().foreverAlone();
{% endhighlight %}

Using singleton is especially useful when handling single resources such as loggers, configurations etc.
