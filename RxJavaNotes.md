# 在看RxJava，做一些笔记  
### Author: [仍物线](https://github.com/rengwuxian)  
  
## `RxJava` 的观察者模式  
`RxJava`有四个基本概念：`Obvservable`(被观察者（`Object`）)、`Observer（观察者）`、`subscribe(订阅)`、事件。  
`RxJava`的事件回调除了`onNext()`（相当与`onClick(）`之外，还有两个特殊的事件：`onCompleted()`和`onError()`;
* `onCompleted()`:事件队列完结。`RxJava`把每个事件单独处理，并把它们看做一个队列。`RxJava`规定，没有新的`onNext()`发出的时候触发`onCompleted()`事件作为标志；
* `onError()`:事件处理异常时，`onError()`会被触发，同时队列自动终止，不会有新的事件发出。
* 在一个正确运行的事件序列中，`onCompleted()`和`onError()`有且只有一个，并且是事件序列中的最后一个。需要注意的是，`onCompleted()`和`onError()`二者也是互斥的，即在队列中强调了其中一个，就不应该再调用另一个。 
## RxJava的基本实践主要有三点：
### 1. 创建`Observer`：
``` java
		Observer<String> observer = new Observer<String>() {
		    /*一些内部实现方法（onNext()/onComplete()/onError()）*/
		};
```
除了`Observer`接口之外，`RxJava`还内置了一个实现`Observer`的抽象类。  
`RxJava`的`subscribe`过程中，`observer`总是先被转换成一个Subscriber再使用。两者基本功能完全一样，但是主要区别在于两点： 
* `Subscriber`新添加了`onStart()`方法，在`subscribe`刚开始，事件还未开始的时候调用，可以做一些准备工作，例如数据的清零和重置。这是一个可选的方法，默认的情况下它的实现为空。需要注意的是，如果对准备工作有线程的要求（例如弹出一个现实进度条的对话框，这必须在主线程中进行），这个情况下`onStart()`就不适用了，因为它总是在`subscribe`所发生的线程被调用，而不能指定线程。要在指定的线程来做准备工作，可以使用`doOnSubscribe()`方法，具体后面介绍。
* `unsubscribe()`:这是`Subcriber`所实现的另外一个接口`Subscription`的方法，用于取消订阅。在这个方法被调用后，`Subscriber`将不在接收时间。一般这个方法调用之前，可以使用`isUnsubscribed()`先判断一下状态。`unsubscribe()`这个方法很重要，因为在`subscribe（）`之后，`observable`会持有一个`subscribe`的引用，这个引用如果不能及时被释放，将有内存泄漏的风险。所以保持一个原则：要在不再使用的时候尽快在合适的地方（例如`onPause()` `onStop()`等方法中）调用`unsubscribe()`来解除订阅关系，避免内存泄漏的的发生。
### 2. 创建`Observable`  
`Observable` 即被观察者，它决定什么时候触发什么样的事件。RxJava使用`create()`方法来创建一个Observable，并为它定义触发的规则
```java
	Observable observable = Observable.create(new Observable.OnSubscribe<String>() {
	    @Override
	    public void call(Subscriber<? super String> subscriber) {
	        subscriber.onNext("Hello");
	        subscriber.onNext("Hi");
	        subscriber.onNext("Aloha");
	        subscriber.onCompleted();
	    }
	
	);
```
`create()`方法是RxJava是最基本的创造事件序列的方法。RxJava还提供了一些方法用来快捷创造事件队列，例如：
* just(T...) 将传入的参数依次发出来:
	```
	Observable observable = Observable.just("Hello", "Hi", "Aloha");
	// 将会依次调用：
	// onNext("Hello");
	// onNext("Hi");
	// onNext("Aloha");
	// onCompleted();
	```
* from(T[])/from(Iterable<? extends T>):将传入的数据或Iterable拆分成具体的对象之后，依次发送出来。
	```
	String[] words = {"Hello", "Hi", "Aloha"};
	Observable observable = Observable.from(words);
	// 将会依次调用：
	// onNext("Hello");
	// onNext("Hi");
	// onNext("Aloha");
	// onCompleted();
	```
### 3. Subscribe(订阅)
创建了被订阅者和订阅者之后，用subscribe()方法将它们关联起来，整个结构接搭起来了：  
`observable.sunscribe(observer)`  
Or  
`observable.subscribe(subscriber)`  
Observable.subscribe(Subscriber)内部实现如下:  
```java
	// 注意：这不是 subscribe() 的源码，而是将源码中与性能、兼容性、扩展性有关的代码剔除后的核心代码。
	// 如果需要看源码，可以去 RxJava 的 GitHub 仓库下载。
	public Subscription subscribe(Subscriber subscriber) {
	    subscriber.onStart();
	    onSubscribe.call(subscriber);
	    return subscriber;
	}
```
可以看到，subscriber() 做了3件事：  
1. 调用 Subscriber.onStart() 。这个方法在前面已经介绍过，是一个可选的准备方法。
2. 调用 Observable 中的 OnSubscribe.call(Subscriber) 。在这里，事件发送的逻辑开始运行。从这也可以看出，在 RxJava 中， Observable 并不是在创建的时候就立即开始发送事件，而是在它被订阅的时候，即当 subscribe() 方法执行的时候。
3. 将传入的 Subscriber 作为 Subscription 返回。这是为了方便 unsubscribe().












































