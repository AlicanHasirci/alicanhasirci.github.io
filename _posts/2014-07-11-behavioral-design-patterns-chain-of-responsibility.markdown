---
author: Alishex
comments: true
date: 2014-07-11 14:12:25+00:00
layout: post
slug: behavioral-design-patterns-chain-of-responsibility
title: Chain Of Responsibility
wordpress_id: 62
categories:
- Behavioral
- Design Patterns
tags:
- behavioral design patterns
- design pattern
---

**Chain Of Responsibility**
Often there comes a time where you need to make interact with an object that happens to have restricted access. Beginner way to do it, is to make every method and prop public, while expert way is to use a chain of responsibility.
Chain of Responsibility is a pattern where a request gets passed on between Handlers until the proper one found to handle it. The creation purpose of this mechanism is create an enviroment where a request/proccess can find it's handler with out comprimising encapsulation.

**Intent**




  * Prevents the sender to explicitly defining the handler, creating a intelligent mechanism if done correctly while preserving encapsulation.


  * Object becomes the responsibility of the chain until someone handles it.



Here is an example of how. First we create a Request class to act as... well a request.

    
    
    public class Request {
    	private int value;
    	private String name;
    	
    	public Request(int value,String name){
    		this.value = value;
    		this.name = name;
    	}
    	
    	public int getValue() {
    		return value;
    	}
    
    	public void setValue(int value) {
    		this.value = value;
    	}
    
    	public String getName() {
    		return name;
    	}
    
    	public void setName(String name) {
    		this.name = name;
    	}
    }
    


Here is the abstract Handler class to be extended by Concrete Handlers.

    
    
    public abstract class Handler {
    	
    	private static List<handler> chain = new ArrayList<handler>();
    	
    	public Handler(){
    		chain.add(this);
    	}
    	
    	protected void passNext(Request request){
    		int index = chain.indexOf(this);
    		if(chain.size()-1!=index){
    			chain.get(index+1).handleRequest(request);
    		}else
    			return; //Feel Free to throw exception here to handle this outcome
    	}
    	
    	public abstract void handleRequest(Request request);
    }
    


As you can see for this example i've created a static ArrayList to register all the Handler implementations that are created. This way whenever you extend Handler calling the super constructor will add it to the chain automaticly. Lets see the concrete handlers now.

    
    
    // ConcreteHandlerOne.java
    public class ConcreteHandlerOne extends Handler{
    
    	public ConcreteHandlerOne(){
    		super();
    	}
    	
    	@Override
    	public void handleRequest(Request request) {
    		if(request.getValue()==1)
    			System.out.println("Concrete 1 Handled:" + request.getName());
    		else{
    			this.passNext(request);
    		}
    	}
    }
    
    // ConcreteHandlerTwo.java
    public class ConcreteHandlerTwo extends Handler{
    
    	public ConcreteHandlerTwo(){
    		super();
    	}
    	
    	@Override
    	public void handleRequest(Request request) {
    		if(request.getValue()==2)
    			System.out.println("Concrete 2 Handled:"+ request.getName());
    		else
    			this.passNext(request);
    	}
    }
    
    //ConcreteHandlerThree.java
    public class ConcreteHandlerThree extends Handler{
    
    	public ConcreteHandlerThree(){
    		super();
    	}
    	
    	@Override
    	public void handleRequest(Request request) {
    		if(request.getValue()==3)
    			System.out.println("Concrete 3 Handled:"+ request.getName());
    		else
    			this.passNext(request);
    	}
    }
    


As you can each Handler implementation handles the request as they please and if they do not then pass it to the successor.
Lets see the main method now:

    
    
    public class Launcher {
    	public static void main(String[] args) {
    		ConcreteHandlerOne handlerOne = new ConcreteHandlerOne();
    		ConcreteHandlerTwo handlerTwo = new ConcreteHandlerTwo();
    		ConcreteHandlerThree handlerThree = new ConcreteHandlerThree();
    		
    		Request r1 = new Request(1, "Request1");
    		Request r2 = new Request(2, "Request2");
    		Request r3 = new Request(3, "Request3");
    
    		handlerOne.handleRequest(r1);	// Concrete 1 Handled:Request1
    		handlerOne.handleRequest(r2);	// Concrete 2 Handled:Request2
    		handlerOne.handleRequest(r3);	// Concrete 3 Handled:Request3
    		handlerTwo.handleRequest(r1);
    		handlerTwo.handleRequest(r2);	// Concrete 2 Handled:Request2
    		handlerTwo.handleRequest(r3);	// Concrete 3 Handled:Request3
    		handlerThree.handleRequest(r1);
    		handlerThree.handleRequest(r2);
    		handlerThree.handleRequest(r3);	// Concrete 3 Handled:Request3
    	}
    }
    
