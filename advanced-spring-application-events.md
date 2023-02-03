advanced-spring-application-events
Terezija Semenski

In this course, we'll cover key concepts of Spring events. You will learn how to avoid bad design, breaking solid principles, cycling dependencies, and how easy it is to add new features without refactoring existing code. 

## Spring events vs. direct method calls

The one system leads to more flexible and loosely coupled architectural design.
It simplifies testing, maintenance and troubleshooting problems. Another benefit of Event system is it provides published subscribe capability.
changes in the publisher or listener will not affect each other as both are independent.

It is the observer pattern that is well known from the Gang of Four design patterns. Where in the observer pattern, we refer to observers. In spring event system we refer to listeners. When we refer to subject, we actually refer to event publisher.

Event driven are architecture also follows an open-closed principle. Which means open for extension, but closed for modification.

by using an event system, we can close order service for modification by using events and we can easily extend the behavior by implementing new listeners.

## Highlights of Spring events

pring Events are one of the core capabilities provided by Spring Framework. They allow us to publish and listen to specific Application Events that we can process as we wish. Core components of Spring Events are Events, Publisher and Listeners.

Spring Events ones are by default synchronous, meaning the publisher thread blocks until all listeners have finished processing the event. However, Spring also supports Asynchronous mode

## Implement a new listener for events

Spring gives us two ways to define Listener. We can either implement an ApplicationListener interface or use @EventListener.

Starting with Spring 4.2, it's now possible to simply annotate a method over Spring bean with @EventListener.

It is also possible to define the order in which listeners for the same event are to be invoked. To do so, we can use SpringCommon @order annotation, alongside EventListener annotation.

In order for this method to get triggered when event is sent we need to annotate it with @EventListener. And also mark our class as Spring bean with @Component.

@EventListener
public void onCustomerRemovedEvent(CustomerRemovedEvent customerEvent){
	email.sendCustomerRemovedEmail(customerEvent.getCustomer());
}

## Implement asynchronous events

To make an event listener run in async mode, all they have to do is use the @Async annotation on that listener. We also need to enable asynchronous processing by adding @EnableAsync on top of our spring configuration.

If an asynchronous event listener throws an exception, it's not propagated to the caller. However, by implementing Async Uncaught Exception Handler Interface, we can process any asynchronous exceptions.

## Filter events

@EventListener provides a conditional tribute that accepts spell expression. The event will be handled if the expression evaluates the Boolean true, or one of the following strings: true, on, yes, or one.

@EventListener(condition = "#event.customer.newsletter")
public void onRegistrationEvent(CustomerRegisteredEvent event)

## Transaction bound events

how to allow the events to be used with more flexibility when the outcome of the current transaction matters to the listener.
In Spring, whenever we want our class or methods to be executed in a transaction, we use transactional notation. It is used to combine more than one rights on a database as a single atomic operation.

Spring provides us a special kind of listener called TransactionalEventListener. This doesn't mean that the event listener is transactional itself, but event consumption is delayed until a certain transaction outcome. This gives us more fine grain control on when event listeners should get triggered based on the transaction phase.

We can use this in our event listener should only run if the current transaction was successful. After rollback, the event will be handled after the transaction has rolled back. After completion, the event will be handled when the transaction commits or is rolled back.

@EventListener(phase = TransactionPhase.AFTER_COMMIT)

@TransactionalEventListener(phase = TransactionPhase.AFTER_ROLLBACK)
