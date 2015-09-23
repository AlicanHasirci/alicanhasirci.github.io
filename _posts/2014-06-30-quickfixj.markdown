---
author: Alishex
comments: true
date: 2014-06-30 11:59:20+00:00
layout: post
slug: quickfixj
title: QuickFix/J
wordpress_id: 28
categories:
- Java
- Programming
tags:
- fix
- java
- quickfix
- quickfix/j
- stock exchange
---

As my first post i thought i'd write on what i am working on nowadays. Few months ago i accepted a work to write a QuickFix integration solution since Borsa Istanbul was having a transition to quickfix.
Even though i never had worked with QuickFix i thought i'd give it a try. I am writing this hoping that my experiances will help someone since there are not much resources on QuickFix/J.

Here are the layers of application that i've created:



	
  * **GUI** - Where the interaction happens

	
  * **Initiator** - Where the initiation happens. Not To be mixed with a socket initiator.

	
  * **Application** - Extends _quickfix.Application_

	
  * **Outbound** - This part takes care of sending messages

	
  * **Inbound** - Recieves and proccesses the messages


I am going to skip the GUI part since it does not have anything to do with quickfix itself.



**QuickFixInitiator**

This layer gets active when user click to _connect _button from GUI and approves the connection settings. A new QuickFixInitiator gets started and this initiator basicly creates whatever needed to create a proper _QuickFixApplication_.

    
    	public static void start(FixGUI context, boolean messageNoReseteSelected, 
                boolean changePasswordSelected, String newPassword) throws FileNotFoundException, ConfigError, UnsupportedEncodingException {
    		
    		SessionSettings sessionSettings = new SessionSettings(createSettingsInputStream());
    		MessageStoreFactory messageStoreFactory = new FileStoreFactory(sessionSettings);
    		LogFactory logFactory = new SLF4JLogFactory(sessionSettings);
            MessageFactory messageFactory = new DefaultMessageFactory();
            if(application == null){
            	application = new QuickFixApplication(context,messageNoReseteSelected,changePasswordSelected,newPassword);	        	
            }
    		socketInitiator = new SocketInitiator(application, messageStoreFactory, sessionSettings, logFactory, messageFactory);
    		socketInitiator.start();
    	}
    


This is how the initiation phase looks like for me. First the SessionSettings gets created with a method called _createSettingsInputStream()_ which passes all the settings configurations as an InputStream.
After creating SessionSettings a MessageStoreFactory gets created. This is where QuickFix will store the messages you recieve. LogFactory gets created right after that. Mine is a SLF4JFactory which i use with LOG4J which is must say can look a bit complicated at start but provides huge benefits. Maybe i'll prepare tutorial on how to integrate SLF4J with LOG4J. Rest of the code is all straight forward. This pretty much sums up this layer.



**QuickFix Application**
This layer is where all the magic happens. QuickFix will send and recieve all the messages from this class what you'll do with them is upto you.
Right after you extend _bfix.QuickFixApplication_ you'll get you @Override methods. The most important of these methods is _fromApp_. All the messages you reiceve will end-up here. How we'll handle will be explained in **Inbound** layer.

    
    
    	public QuickFixApplication(FixGUI context, boolean messageNoReseteSelected, 
                boolean changePasswordSelected, String newPassword) {
    		this.context = context;
    		this.messageNoReset = messageNoReseteSelected;
    		this.changePassword = changePasswordSelected;
    		this.newPassword = newPassword;
    		cmc = new CustomMessageCracker();
    		
    		try {
    			int multicastPort = ConfigurationManager.getConfigInt(ConfigurationKey.MULTICAST_PORT);
    			NetworkInterface multicastInterface = NetworkInterface.getByInetAddress(InetAddress.getByName(""));
    			MulticastManager mm = new MulticastManager(multicastPort, multicastInterface, false);
    			
    			InetAddress multicastAddress = InetAddress.getByName(ConfigurationManager.getConfig("");
    			mm.joinGroup(multicastAddress);
    			
    			executor.execute(mm);
    		} catch (IOException e) {
    			logger.warn(e.getMessage());
    		}
    		
    		executor.execute(new OrderSupplier());
    	}
    


This is how the constructor looks like. In my case multicast messages were getting delivered via UDP port different from Quickfix itself so i had to listen for multicasts on UDP and proccess them just like a normal fix message.  So feel free to ignore the multicast part. The object _executor_ is an instance of ExecutorService to handle multiple Threads.  At the end of the block you can see executor starting a new Runnable instance of _OrderSupplier_ this is the **Outbound** layer which we will be talking about next. Rest of the class is overriden method stubs. As you may have noticed earlier i have same boolean values which i pass from initiator to application, these values notices the application if the user wants to "reset password" or "reset message sequence number" while sending a login request. If you want to do the same notify the application in a similar manner and then add this fields to your login message in _toAdmin_ method. This method is a checkpoint where your outbound messages out of a settled session will drop by. Here how it looks like:

    
    
    	@Override
    	public void toAdmin(Message message, SessionID sessionId) {
    		try {
    			if (message.getHeader().getField(new MsgType()).valueEquals(MsgType.LOGON)) {
    				message.setField(new Username(BROKER_ID));
    				message.setField(new Password(BROKER_PASSWORD));
    				if(messageNoReset)
    					message.setField(new ResetSeqNumFlag(true));
    				if(changePassword){
    					message.setField(new NewPassword(newPassword));
    					ConfigurationManager.setConfiguration(ConfigurationKey.BROKER_PASSWORD, newPassword);
    				}
    			}
    		} catch (Exception e) {
    			e.getStackTrace();
    		}
    	}
    



Here is _fromApp_ :

    
    
    	public void fromApp(Message message, SessionID sessionId) throws FieldNotFound,
    			IncorrectDataFormat, IncorrectTagValue, UnsupportedMessageType {
    		cmc.crack(message,sessionId);
    	}
    


Here the messages we recieve will be cracked by a _CustomMessageCracker_ which will be thoroughly explained in **Inbound** part.


**Outbound**
Outbound is the part which takes of message sending as the name suggests. There are few elements in this part. The first one is the _OrderSupplier_ which collects the orders from database and puts them in a LinkedList.

    
    
    public class OrderSupplier implements Runnable {
    	private static final Logger logger = LoggerFactory.getLogger(CustomMessageCracker.class);
    	
    	private final int MESSAGE_PER_SECOND = 6;
    
    	private static final DateFormat MESSAGE_DATE_FORMAT = new SimpleDateFormat("yyyyMMdd");
    	
    	public static volatile LinkedList<message> orderQueue = new LinkedList<message>();
    
    	@Override
    	public void run() {
    		Message message = null;
    		while (true) {
    			if(orderQueue.size() < MESSAGE_PER_SECOND){
    				if (message == null) {
    					
    					message = fetchRequest();
    				}
    				if(message != null) {
    					orderQueue.add(message);
    					message = null;
    				}
    			}
    			try {
    				// Queue is full already.Give it some time to switch threads.
    				Thread.sleep(50);
    			} catch (InterruptedException e) {
    			}				
    		}
    	}
    



As you can see this runnable here fills a queue ready to be send so that the code which sends messages don't have to wait for a new database transaction everytime it sends a message which would burden the system greatly. The obscure area here is _fetchRequest_ where a database transaction happens and the data gets decoded to a fix message.
After getting the messages in a queue we have the _OrderSender_ class which sends them according to message per second throttling.


    
    
    public class OrderSender implements Runnable {
    
    	private SessionID sessionId;
    	private int messagePerSecond;
    	
    	public OrderSender(SessionID sessionId) {
    		this.sessionId = sessionId;
    		messagePerSecond = 6;
    	}
    	
    	@Override
    	public void run() {
    		Message message = null;
    		while (true) {
    			TimerThread timerThread = new TimerThread();
    			timerThread.start();
    			
    			for (int i = 0; i < messagePerSecond; i++) {
    				if (OrderSupplier.orderQueue.peek() != null && message == null) {
    					message = OrderSupplier.orderQueue.poll();
    				}
    				if (message == null) {
    					// The queue is empty
    					break;
    				}
    				try {
    					Session.sendToTarget(message, sessionId);
    					new Thread(new OrderRecorder(message)).start();
    					message = null;
    				} catch (SessionNotFound e) {
    					e.printStackTrace();
    				}
    			}
    			try {
    				timerThread.join();
    			} catch (InterruptedException e) {
    				e.printStackTrace();
    			}
    		}
    	}
    	
    	private class TimerThread extends Thread {
    		@Override
    		public void run() {
    			try {
    				// Time Cycle
    				Thread.sleep(1000);
    			} catch (InterruptedException e) {
    				e.printStackTrace();
    			}
    		}
    	}
    }
    



Here you can see the throttling mechanism. After sending the message code starts a new thread of the runnable class _OrderRecorder_ which writes the sent message to database and with that it pretty much sums up the **outbound** layer.


**Inbound**
You may remember the a _CustomMessageCracker_ initiated in QuickFixApplication constructor. This class extends _quickfix.MessageCracker_ and handles all the messages that come to _fromApp_ method in QuickFixApplication.

    
    
    	@Override
    	protected void onMessage(Message message, SessionID sessionID)
    			throws FieldNotFound, UnsupportedMessageType, IncorrectTagValue {
    
    		try{
    			ServerMessageType msgType = ServerMessageType.findByValue(message.getHeader().getField(new MsgType()).getValue());
    			switch(msgType){
    			case EXECUTION_REPORT:
    				logger.info("Execution Report Recieved");
    				executionReportHandler((ExecutionReport) message);
    				break;
    			case SECURITY_STATUS:
    				logger.info("Security Status Recieved");
    				securityStatusHandler((SecurityStatus) message);
    				break;
    			case QUOTE_STATUS_REPORT: 
    				logger.info("Quote Status Report Recieved");
    				quoteStatusReportHandler((QuoteStatusReport) message);
    				break;
    			case TRADING_SESSION_STATUS:
    				logger.info("Trading Session Status Message Recieved");
    				tradingSessionStatusHandler((TradingSessionStatus) message);
    				break;
    			case ORDER_CANCEL_REJECT: 
    				logger.info("Order Reject Message Recieved");
    				orderCancelRejectHandler( (OrderCancelReject) message);
    				break;
    			case BUSINESS_MESSAGE_REJECT:
    				logger.info("Business Message Recieved");
    				businessMessageRejectHandler( (BusinessMessageReject) message);
    				break;
    			case NEWS:
    				logger.info("News Message Recieved");
    				newsHandler((News) message);
    				break;
    			case SEQUENCE_RESET:
    				break;
    			default:
    				break;
    			}
    		} catch(Exception e){
    			e.printStackTrace();
    		}
    	}
    


After this step every method handles it's own message and does whatever it has to do.

This pretty much sums the Quickfix/J integration that i worked on. Remember that this is only the outline and there are much more to it then these chunks of code. Feel free contact me on topic and i'd be more then happy to help.
