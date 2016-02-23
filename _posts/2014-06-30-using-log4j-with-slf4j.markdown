---
layout: post
title: "Using LOG4J with SLF4J"
date: 2014-06-30 14:22:03
image: '/assets/img/'
description:
tags:
- java
- log4j
- slf4j
categories:
- Java
- Programming
twitter_text: Using LOG4J with SLF4J
---

Logging is a pretty much underrated but especially while working on communicational applications it is a must and doing it proper can save a lot of time and makes you look good.
For years i've used a lot of different logging libraries, some i coded myself some third parties. In the things that i've tried out using SLF4J with LOG4J seems to be my favorite thou a bit complicated in start, it's proven to be highly customizable therefore making things a lot more easier once you get the gist of it.

{% include image.html url="http://www.slf4j.org/images/concrete-bindings.png" description="SLF4J Concrete Bindings" %}

Here you can see the bindings of SLF4J with different systems. If you are going to use SLF4J at first glance this may look a bit complicated but it is pretty useful. Since we are going to use LOG4J with SLF4J first thing you need is to have the noted libraries(slf4j-api.jar, slf4j-log412.jar, log4j.jar) under your classpath folder. Also remember that you shouldn't have more then 1 binding library in your classpath.

Next you'll need to create a log4j.properties file in your classpath to tell log4j how to log your entries. Before we proceed there are some things you need to know about Log4j like Loggers,Appenders,Layouts etc. These are different components of LOG4J all serving to a purpose, a Logger(also considered as a "Category") is seperate channel of logging if you might say. You can seperate different topics in different Categories/Loggers to maintain clarity. Also this maybe applied in an hierarchical manner where such categories as "alicanhasirci.posts" and "alicanhasirci.comments" can exist.
Also different categories have different appenders. This means you can append anything you like to entries such as timestamps, class names, log levels etc. Setting the behaviour of logger is another thing where you can tell it write to screen, or write to file or even write to a daily rolling file.

Here is an example _log4j.properties_ file:

    
    
    log4j.rootLogger=INFO, STDOUT
    log4j.logger.alicanhasirci.posts=INFO, ALICANHASIRCI_POSTS
    log4j.logger.alicanhasirci.comments=INFO, ALICANHASIRCI_COMMENTS
    
    #STDOUT APPENDER
    log4j.appender.STDOUT=org.apache.log4j.ConsoleAppender
    log4j.appender.STDOUT.layout=org.apache.log4j.PatternLayout
    log4j.appender.STDOUT.layout.ConversionPattern=%d{HH:mm:ss,SSS}: %m%n
    
    #ALICANHASIRCI_POSTS APPENDER
    log4j.appender.ALICANHASIRCI_POSTS=org.apache.log4j.DailyRollingFileAppender
    log4j.appender.ALICANHASIRCI_POSTS.File=alicanhasirci.posts.log
    log4j.appender.ALICANHASIRCI_POSTS.layout=org.apache.log4j.PatternLayout
    log4j.appender.ALICANHASIRCI_POSTS.layout.ConversionPattern=%d{HH:mm:ss,SSS}: %m%n
    
    #ALICANHASIRCI_COMMENTS APPENDER
    log4j.appender.ALICANHASIRCI_COMMENTS=org.apache.log4j.DailyRollingFileAppender
    log4j.appender.ALICANHASIRCI_COMMENTS.File=alicanhasirci.comments.log
    log4j.appender.ALICANHASIRCI_COMMENTS.layout=org.apache.log4j.PatternLayout
    log4j.appender.ALICANHASIRCI_COMMENTS.layout.ConversionPattern=%d{HH:mm:ss,SSS}: %m%n
    


Here you can see a sample properties file. May look intimidating at start but is pretty straight forward when you get the gimmic.
The first paragraph is where we define loggers. We have three diferent ones which are **rootLogger**, **alicanhasirci.posts** and **alicanhasirci.comments**. Part before the comma is where you give the logging level. Any log above "INFO" will be logged in these loggers. The part after the comma defines the names of the logger which will be  used for the rest of the properties file.
You can see that invoking custom categories is different from rootLogger the reason is as may have guessed rootLogger is a predefined logger which behaves as a master logger getting all entries.
Looking at appenders one by one, you can see that SDOUT logger which is the rootLogger is defined as a ConsoleAppender, this means what ever log it gets above INFO including info, it will write it as a console output. Next line you can see that the layout of this logger will be a pattern layout which you can see in the preceding line.
Next appender belongs to "alicanhasirci.posts" category, which is a _DailyRollingFileAppender_, this means that everyday log4j will truncate the previous day and start with a clean slate. Next lines defines where this file will be logged and under what name. Remaining two lines are same as before while at that i'll be skipping the next and last appender since it's pretty much the same with this ones.

Ok we have prepared our properties file for log4j but how do we use these categories(loggers). Well the hard part is over. All you need to do is at the start of every class that you want logging just get the logger you want in such manner :

{% highlight java %}
    	private static final Logger postLogger = LoggerFactory.getLogger("alicanhasirci.posts");
    	private static final Logger commentsLogger = LoggerFactory.getLogger("alicanhasirci.comments");
{% endhighlight %}


All you need to pay attention here is that the strings you pass to _getLogger_ method must be same as the categories you define at properties file.
log4j.logger.**alicanhasirci.posts**=INFO, ALICANHASIRCI_POSTS
log4j.logger.**alicanhasirci.comments**=INFO, ALICANHASIRCI_COMMENTS

If you have any questions of the topic, feel free to contact me i'd be more then happy to help.

Cheers!
