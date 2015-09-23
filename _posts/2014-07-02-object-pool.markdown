---
author: Alishex
comments: true
date: 2014-07-02 08:55:47+00:00
layout: post
slug: object-pool
title: Object Pool
wordpress_id: 48
categories:
- Creational
- Design Patterns
tags:
- constructional design pattern
- design pattern
- singleton
---

An object pool is a singleton object which creates, distributes and after the use acquires objects are expensive to create. 

**Intent**




  * Maximize the use of objects which are expensive to create



Now to think of it, a got analogy for an ObjectPool can be a library. Creating a book can be costly for you so when you want something from library, you first go to library(get the instance) then pick the book you want(acquire the object), read it(use the object) and then return it when you are done with it(release it).


    
    
    class LibraryObjectPool{
    
    	private static LibraryObjectPool instance;
    
    	private LibraryObjectPool(){
    		// initiation
    	}
    
    	public LibraryObjectPool getInstance(){
    		if(instance==null)
    			instance = new LibraryObjectPool();
    		return instance;
    	}
    
    	public Book rentABook(){
    		// if there is an unused ones
    			//rent the book
    		//else
    			//create the book and then rent it
    	}
    
    	public void returnBook(Book return){
    		// accept the book back
    	}
    }
    



Keep in mind that you'll also need an array to hold your objects and flags to know if there are unused ones or not.

