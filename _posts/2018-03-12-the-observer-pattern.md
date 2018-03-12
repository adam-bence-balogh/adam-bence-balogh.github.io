---
layout: post
title: The Observer Pattern
image: /img/observer/observer-basic implementation big border2.jpg
share-img: /img/observer/observer-basic implementation big border2.jpg
gh-repo: adam-bence-balogh/DesignPatterns
gh-badge: [star, watch, fork, follow]
tags: [designpatterns]
---

The Observer Pattern is one of the most commonly used patterns in software engineering and it is also one of my personal favorites.
It's quite useful and also easy to remember because we run into it literally every day. These are the main reasons why beginners usually start their learning path in the world of design patterns with the observer (right after singleton).

## One Publisher Many Subscriber
The Pattern defines a one-to-many relationship, with a Subject (also known as the Publisher), and several Subscribers (Observers). The observers register themselves to the subject, which means from that point forward they will be notified whenever the Subject think they should be. This is how the Subject keeps the Observers up to date.

## In everyday life
Sometimes the Observer Pattern doesn't ring a bell at first with many developers, but if we mention the Subscriber-Publisher relation suddenly everybody knows what we are talking about. Let's take a look at some real life examples.

### YouTube channels
If you subscribe to someone's channel on YouTube you will be notified every time (along with other subscribers) whenever the user uploads a new video, and you will keep getting notifications until you unsubscribe.

### Newspaper subscription 
The other frequently used example is subscribing to a newspaper. As long as you don't unsubscribe (or you forget to pay the fee) you will receive the latest journals every time the publisher releases it.

## Implementing the Pattern

### Class diagram
Let's take a peek at how the Observer Pattern looks like in a class diagram
![Observer Pattern implementation class diagram](/img/observer/observer-basic implementation big border.jpg "Observer Pattern implementation class diagram")

StockBroker IS-A Observer  
StockMarket IS-A Subject  
StockMarket has zero or more StockerBrokers  

### The code 

```java
interface Observer {
    void update();
}

interface Subject {
    void register(Observer observer);
    void unregister(Observer observer);
    void notifyObservers();
}

class StockBroker implements Observer {
    public StockBroker(Subject subject){
        subject.register(this); //the observer registers itself
    }
    @Override
    public void update() {
        System.out.println("I've just received an update.. I think I'm gonna check the market..");
    }
}

class StockMarket implements Subject {
    private List<Observer> observerList = new ArrayList<>();
    @Override
    public void register(Observer observer) {
        observerList.add(observer);
    }

    @Override
    public void unregister(Observer observer) {
        observerList.remove(observer);
    }

    @Override
    public void notifyObservers() {
        observerList.forEach(obs -> obs.update());
    }
}

class Main {
    public static void main(String[] args) {
        Subject subject = new StockMarket();
        Observer obs1 = new StockBroker(subject);
        subject.notifyObservers(); //obs1 got an update
        subject.notifyObservers(); //obs1 got an update
        subject.unregister(obs1); //ob1 has been unregistered
        subject.notifyObservers(); //there's no one to notify
    }
}
```

{: .box-note}
**Output:**  
I've just received an update.. I think I'm gonna check the market..  
I've just received an update.. I think I'm gonna check the market..  

Okay, so this is how the notifying is done, but how can the observers process the updated data? There are two options:
- The push model
- The pull model

### The Push model

Using the push model means the Subject sends the data along with the notification, and the clients process them when receiving it.

```java
interface Observer {
    void update(Object data);
}

class StockBroker implements Observer {

    private Subject subject;

    public StockBroker(Subject subject) {
        this.subject = subject;
        subject.register(this); //the observer registers itself
    }

    @Override
    public void update(Object data) {
        System.out.println("StockBroker has received an update");
        if(data instanceof Integer) {
            makeDecision((Integer)data);
        }
    }

    private void makeDecision(int excitingCorpStockPrice) {
        if(excitingCorpStockPrice > 150){
            System.out.println("I'm selling my stocks and I'm gonna be rich");
            subject.unregister(this);
        }else if(excitingCorpStockPrice < 50){
            System.out.println("I'm selling my stocks because I don't want to lose too much money");
            subject.unregister(this);
        }
    }
}

interface Subject {
    void register(Observer observer);
    void unregister(Observer observer);
    void notifyObservers(Object data);
}

class StockMarket implements Subject {
    private List<Observer> observerList = new CopyOnWriteArrayList<>();
    private int excitingCorpStockPrice = 100;

    public void changeExcitingCorpStockPrice(int amount){
        excitingCorpStockPrice += amount;
        notifyObservers(Integer.valueOf(excitingCorpStockPrice));
    }

    @Override
    public void register(Observer observer) {
        observerList.add(observer);
        System.out.println("Register observer");
    }

    @Override
    public void unregister(Observer observer) {
        observerList.remove(observer);
        System.out.println("Unregister observer");
    }

    @Override
    public void notifyObservers(Object data) {
        observerList.forEach(observer -> observer.update(data));
    }
}

class Main {
    public static void main(String[] args) {
        StockMarket stockMarket = new StockMarket();
        Observer stockBroker = new StockBroker(stockMarket);
        stockMarket.changeExcitingCorpStockPrice(20);
        stockMarket.changeExcitingCorpStockPrice(20);
        stockMarket.changeExcitingCorpStockPrice(20);
        stockMarket.changeExcitingCorpStockPrice(20);
        stockMarket.changeExcitingCorpStockPrice(20);
    }
}
```

{: .box-note}
**Output:**  
Register observer  
StockBroker has received an update  
StockBroker has received an update  
StockBroker has received an update  
I'm selling my stocks and I'm gonna be rich  
Unregister observer

Notice that we used a **CopyOnWriteArrayList** in the Subject, which is necessary in this case, because a regular List implementation (like ArrayList) will cause a ConcurrentModificationException due to the observer unregistration during the iteration. In other words, we try to remove an element from the list in the middle of an iteration. If you are not familiar with this situation and you don't know why is it problematic I encourage you to try out the code above using ArrayList instead of CopyOnWriteArrayList. (In an upcoming article I will tell more details about concurrent collections)

### The Pull model

The pull model means the **Observer has a reference to the subject, and it can retrieve the data whenever it is being notified**.   
It is worth mentioning, that the StockBroker stored a reference to the Subject at the push model example as well, but it used it only to unregister itself.

```java
interface Observer {
    void update();
}

interface Subject {
    void register(Observer observer);
    void unregister(Observer observer);
    void notifyObservers();
}

class StockBroker implements Observer {
    private Subject subject;

    public StockBroker(Subject subject) {
        this.subject = subject;
        subject.register(this); //the observer registers itself
    }

    @Override
    public void update() {
        System.out.println("StockBroker has received an update.. StockBroker is going to check the ExicitingCorp's stock price");
        if(subject instanceof StockMarket){
            int excitingCorpStockPrice = ((StockMarket) subject).getExcitingCorpStockPrice();
            makeDecision(excitingCorpStockPrice);
        }
    }

    private void makeDecision(int excitingCorpStockPrice) {
        if (excitingCorpStockPrice > 150) {
            System.out.println("I'm selling my stocks and I'm gonna be rich");
            subject.unregister(this);
        } else if (excitingCorpStockPrice < 50) {
            System.out.println("I'm selling my stocks because I don't want to lose too much money");
            subject.unregister(this);
        }
    }
}

class StockMarket implements Subject {
    private List<Observer> observerList = new CopyOnWriteArrayList<>();
    private int excitingCorpStockPrice = 100;

    @Override
    public void register(Observer observer) {
        observerList.add(observer);
        System.out.println("Register observer");
    }

    @Override
    public void unregister(Observer observer) {
        observerList.remove(observer);
        System.out.println("Unregister observer");
    }

    @Override
    public void notifyObservers() {
        observerList.forEach(obs -> obs.update());
    }

    public void changeExcitingCorpStockPrice(int amount){
        excitingCorpStockPrice += amount;
        notifyObservers();
    }

    public int getExcitingCorpStockPrice() {
        return excitingCorpStockPrice;
    }
}

class Main {
    public static void main(String[] args) {
        StockMarket stockMarket = new StockMarket();
        Observer stockBroker = new StockBroker(stockMarket);
        stockMarket.changeExcitingCorpStockPrice(20);
        stockMarket.changeExcitingCorpStockPrice(20);
        stockMarket.changeExcitingCorpStockPrice(20);
        stockMarket.changeExcitingCorpStockPrice(20);
        stockMarket.changeExcitingCorpStockPrice(20);
    }
}
```

{: .box-note}
**Output:**  
Register observer  
StockBroker has received an update.. StockBroker is going to check the ExicitingCorp's stock price  
StockBroker has received an update.. StockBroker is going to check the ExicitingCorp's stock price  
StockBroker has received an update.. StockBroker is going to check the ExicitingCorp's stock price  
I'm selling my stocks and I'm gonna be rich  
Unregister observer  

### Which should be used?
Both solutions have advantages and disadvantages. Using the **push model** sometimes causes situations when the Subject sends unnecessary data to the Observers. Imagine that we have several Observer implementations, but not all of them need the whole data package which the Subject will nevertheless sends to everyone with every notification. It is totally possible that an Observer may only be interested in temperature while others need more details, such as weather condition, humidity, pressure, or whatever it may be.. The Subject cannot and shouldn't differentiate the Observers, therefore each and every one of them will receive the same data, which can significantly increase the processing time.  
On the other hand, if we implement the **pull model**, Observers can decide what kind of data they are interested in, and only pull the values they truly need. The disadvantage is that if the Subject changes its status too quickly, it is possible that by the time the Observer retrieves the data it has already been modified.  

If you want to use the Observer pattern take your time to think through your application design and choose the right model which fits your needs best.

## Loose coupling in the Observer Pattern
Loose coupling means two objects only know each other as much as is absolutely necessary. In our case, the parties only need to know about the other that they are implementing an interface, they shouldn't have any knowledge about how the internal mechanism goes. Even though we violated the loose coupling a bit above in our previous examples when we casted the Subject into StockMarket, normally we must strive for loose coupling as much as possible. Keep in mind that the pull model only means the Observers will be notified, and they have to get the necessary information themselves. It can even happen from a third party, which means the observers may not have to get the data from the Subject.

## Built-in implementation in Java
If you don't want to create the interface from scratch and write the Subject methods, Java offers you a built-in implementation, but it comes at a price. Let's take a look at why:
### Built-in Observer interface
```java
package java.util;
public interface Observer {
    void update(Observable o, Object arg);
}
```
### Built-in Observable (Subject)
```java
package java.util;

public class Observable {
    private boolean changed = false;
    private Vector<Observer> obs;

    public Observable() {
        obs = new Vector<>();
    }

    public synchronized void addObserver(Observer o) {
        if (o == null)
            throw new NullPointerException();
        if (!obs.contains(o)) {
            obs.addElement(o);
        }
    }

    public synchronized void deleteObserver(Observer o) {
        obs.removeElement(o);
    }

    public void notifyObservers() {
        notifyObservers(null);
    }

    public void notifyObservers(Object arg) {
        Object[] arrLocal;

        synchronized (this) {

            if (!changed)
                return;
            arrLocal = obs.toArray();
            clearChanged();
        }

        for (int i = arrLocal.length-1; i>=0; i--)
            ((Observer)arrLocal[i]).update(this, arg);
    }

  
    public synchronized void deleteObservers() {
        obs.removeAllElements();
    }


    protected synchronized void setChanged() {
        changed = true;
    }
	
    protected synchronized void clearChanged() {
        changed = false;
    }

    public synchronized boolean hasChanged() {
        return changed;
    }

    public synchronized int countObservers() {
        return obs.size();
    }
}
```
(I have removed the comments, but you can read them by searching these classes in your favorite IDE.)  
### Downsides
The first interesting thing you may notice is that the Subject is called Observable, and it is a concrete class. In other words it is already an implementation. Sadly though, this also means that if we extend this Observable, we won't be able to extend any other class. Therefore we lose the flexibility to make an already existing class an Observable, unless it hadn't extended anything.  
The other thing is, if you look at the Observable carefully you can see that we have to call the setChanged() method before we can notify the Observers. This solution was designed to avoid situations when the Subject sends a notification, but there is no actual update. Nevertheless, it is something we have to keep in mind.

## Known usage in Java libraries
### Listeners in Java Swing
If you are familiar with Java Swing you definitely met the Observer Pattern (perhaps, without knowing it), because this is how listeners are built. When you write a new ActionListener (or ClickListener), you basically create an Observer, and you register it to a Component (e.g. a JButton), which is the Subject. When the user clicks on the button, the Listeners will be notified, and your code will be executed.
```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionListener;

class ExcitingFrame extends JFrame {
    public ExcitingFrame(String title) throws HeadlessException {
        super(title);
        //create subject
        JButton myButton = new JButton("Click me");
        //create observer
        ActionListener myListener = action -> JOptionPane.showMessageDialog(null, "www.excitingjava.com");
        //register observer
        myButton.addActionListener(myListener);
        //make things visible
        add(myButton);
        setSize(300, 200);
		setDefaultCloseOperation(WindowConstants.EXIT_ON_CLOSE);
        setVisible(true);
    }
}

class Main{
    public static void main(String[] args) {
        new ExcitingFrame("Observer example in Swing");
    }
}
```
If you run the Main class the ExcitingFrame will appear, and clicking on the button means the listener will be notified, then your lambda will be executed. By the way, this short lambda function is an implementation of the ActionListener functional interface's single abstract method, and this is how we create an implementation, then instantiate it right away. We will get back to this topic in another article focusing on lambdas, stay tuned..)

### FileListeners
There are multiple libraries available aiming to notify observers if a file changes (for example apache commons FileAlterationObserver). This solution comes handy when a system should be configurable at runtime. If the application itself notices that its configuration.properties has changed, it can load the values from it immediately making your life so much easier, but of course you can also use this capability for anything else that fits your needs.

## Summary
Knowing the Observer Pattern is essential for every software engineer. Even if you have not come across it yet, you can be certain it will surface sooner or later in your programming career, so it is very valuable to have it in your toolbox ready to go.
