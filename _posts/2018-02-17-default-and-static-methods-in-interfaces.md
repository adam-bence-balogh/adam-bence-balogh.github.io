---
layout: post
title: Default and Static methods in interfaces
gh-repo: adam-bence-balogh/Java8
gh-badge: [star, watch, fork, follow]
tags: [java8]
---

Java 8 came with a lot of improvements compared to Java 7, and one of them is interface enhancement. This means that since Java 8 we can write methods in interfaces with concrete implementation. Interfaces are no longer only responsible for defining the contract, they became a bit more complex.  
Before we dive into the details, let's talk about the reasons behind this enhancement. Imagine a situation where we have an interface with multiple implementations. Few months later we decide to add more methods into that, which results in putting new lines into the implementers as well, because they are forced to override the recently added abstract methods. It can be painful, but as long as you posses all of these classes you are fine. The unpleasant part comes when you may not have control over all of the implementations. You can imagine the anger of the developers when their classes starting to break one by one because you added a new abstract method into an interface.

## Default methods

The solution is: default methods. Since Java 8 you can provide concrete implementation to a method in an interface, which means the implementing class inherits this behavior, therefore it doesn't have to override it. Of course it can, but the point is, we have the freedom to add new methods to an already existing interface without worrying breaking implementing classes.
One of the other main reasons why they were introduced is to provide utility methods without the need of utility classes. Remember calling the Collections class static methods like sort(List<T> list)? Now sort is added to the List interface as a default method (sadly only the version which requires a Comparator).

### default keyword in a new role

Before Java 8 the default keyword was only used in switch statements, now it's also marking methods with concrete implementation in interfaces.

```java
interface Callable {
    default void call(){
        System.out.println("Callable's call");
    }
}

class GalaxyS8 implements Callable{}

class Main {
    public static void main(String[] args) {
        GalaxyS8 galaxyS8 = new GalaxyS8();
        galaxyS8.call();
    }
}
```
   
{: .box-note}
**Output:** Callable's call  

Default methods are **implicitly public** just like abstract methods. We will get back to these implicit modifiers later on. 
 
This new feature raises a few questions - what happens if a class inherits a method with the same signature from an interface and as well as a class, both of which have concrete implementations? Not to mention the situation where a class inherits default methods with the same signature from two non-related interfaces, because normally that would lead us to the diamond problem (also known as the multiple inheritance issue). Does the code compile? Will the JVM handle it silently or will it throw a giant RuntimeException? Well, let's find out.

### Inheriting method with the same signature from class and interface

```java
interface Callable {
    default void call(){
        System.out.println("Callable's call");
    }
}

class Phone {
    public void call(){
        System.out.println("Phone call");
    }
}

class GalaxyS8 extends Phone implements Callable{}

class Main {
    public static void main(String[] args) {
        GalaxyS8 galaxyS8 = new GalaxyS8();
        galaxyS8.call();
    }
}
```

{: .box-note}
**Output:** Phone call  

As you can see, no compilation error or exception occurred, and as shown in the example:  
**If a class inherits a method with the same signature from an interface and a class, the class always wins.**

### Use an overridden default method

Let's see how can the GalaxyS8 use both the interface and the Phone implementation plus provide its own:

```java
interface Callable {
    default void call(){
        System.out.println("Callable's call");
    }
}

class Phone {
    public void call(){
        System.out.println("Phone call");
    }
}

class GalaxyS8 extends Phone implements Callable{
    public void call(){
        Callable.super.call(); //accessing Callable's default method
        super.call(); //accessing the parent class (Phone) implementation
        System.out.println("GalaxyS8 call");
    }
}

class Main {
    public static void main(String[] args) {
        GalaxyS8 galaxyS8 = new GalaxyS8();
        galaxyS8.call();
    }
}
```

{: .box-note}
**Output:**   
Callable's call  
Phone call   
GalaxyS8 call 

Overridden default methods can be accessed using:  
**InterfaceName.super.defaultMethodName()**  
(Until they are not overridden by a descendant interface or class)

```java
interface Reachable {
    default void call(){
        System.out.println("Reachable's call");
    }
}

interface Callable extends Reachable{
    default void call(String caller){
        System.out.println("Callable's call");
    }
}

class GalaxyS8 implements Callable{
    public void call(){
        Callable.super.call();
        Reachable.super.call(); //Compilation error
    }
}
```

{: .box-error}
**Compilation failure**  
not an enclosing class: com.excitingjava.test.Reachable

Just like class inheritance. If class B extends class A, and overrides a method, then class C which extends class B cannot have access to class A's original method.

### Inheriting default methods with the same signature from two non-related interfaces

Multiple inheritance have never been an issue in Java because it doesn't support it. A class can only extend one class, therefore it cannot inherit a method with the same signature from different sources, however a class can implement multiple interfaces. Since interface methods can have implementations in Java 8, it raises the question how Java handles a situation like this?

```java
interface Callable {
    default void call(){
        System.out.println("Callable's call");
    }
}

interface Reachable {
    default void call(){
        System.out.println("Reachable's call");
    }
}

class GalaxyS8 implements Callable, Reachable{} //Compilation error
```

{: .box-error}
**Compilation failure**  
class GalaxyS8 inherits unrelated defaults for call() from types Callable and Reachable

To overcome this issue we have to override it to resolve the ambiguity. In that method we are able to call both default method implementations in the same way we did previously.

```java
interface Callable {
    default void call(){
        System.out.println("Callable's call");
    }
}

interface Reachable {
    default void call(){
        System.out.println("Reachable's call");
    }
}

class GalaxyS8 implements Callable, Reachable{
 public void call(){
        Callable.super.call(); //accessing Callable's default method
        Reachable.super.call(); //accessing Reachable's default method
    }
}

class Main {
    public static void main(String[] args) {
        GalaxyS8 galaxyS8 = new GalaxyS8();
        galaxyS8.call();
    }
}
```

{: .box-note}
**Output:**  
Callable's call  
Reachable's call

Don't forget that interface default methods can be overridden by abstract methods as well.

## Static methods in interfaces

Another way to put concrete implementation into an interface is using static methods. At first sight they appear to be regular static methods, and behave the same way in most cases, but there is a big difference between them:
**Interface static methods are never inherited**
Let's see these rules in action:

### Using interface static methods as regular static methods
   
```java
interface Callable{
    static void verifyCaller(String caller){
        if(caller == null || caller.equals("")){
            throw new IllegalArgumentException("Invalid caller");
        }else{
            System.out.println("Caller has been verified");
        }
    }
}

class Main{
    public static void main(String[] args) {
        try {
            Callable.verifyCaller("John");
            Callable.verifyCaller("");
        }catch(IllegalArgumentException iae){
            System.out.println(iae);
        }
    }
}
```

{: .box-note}
**Output:**  
Caller has been verified  
java.lang.IllegalArgumentException: Invalid caller

### Calling interface static methods without using the interface name

This can only be done within the interface:

```java
interface Callable{
    default void call(String caller){
        verifyCaller(caller);
        System.out.println("Callable's call");
    }

    static void verifyCaller(String caller){
        if(caller == null || caller.equals("")){
            throw new IllegalArgumentException("Invalid caller");
        }else{
            System.out.println("Caller has been verified");
        }
    }
}

class GalaxyS8 implements Callable{}

class Main{
    public static void main(String[] args) {
        try {
            Callable callable = new GalaxyS8();
            callable.call("John");
            callable.call("");
        }catch(IllegalArgumentException iae){
            System.out.println(iae);
        }
    }
}
```

{: .box-note}
**Output:**  
Caller has been verified  
Callable's call  
java.lang.IllegalArgumentException: Invalid caller

Interface static methods can be used to provide static utility methods if they cannot be implemented as defaults. 

### Interface static methods are not inherited at all

And here comes the biggest upset about interface static methods:

```java
interface Callable{
    static void verifyCaller(String caller){
        //important implementation
    }
}

class GalaxyS8 implements Callable{
    public void voiceMail(String caller){
        Callable.verifyCaller(caller); //Perfectly legal
        verifyCaller(caller); //Compilation failure
    }
}
```

{: .box-error}
**Compilation failure**  
cannot find symbol: method verifyCaller(java.lang.String)

### Interface static methods are not inherited, not even through interfaces

```java
interface Callable{
    static void verifyCaller(String caller){
        //important implementation
    }
}

interface VoiceMail extends Callable{}

class Main {
    public static void main(String[] args) {
        Callable.verifyCaller("John");
        VoiceMail.verifyCaller("John"); //Compilation failure
    }
}
```

{: .box-error}
**Compilation failure**  
cannot find symbol: method verifyCaller(java.lang.String)

<br>
## The Object class methods cannot be default or static methods in interfaces

### Interface default methods cannot have the same signature as any method in the Object class

Remember when I said that inheriting a method with the same signature from an interface and a class, the class always wins? Adding this to the well known fact that every class derives from the Object class, it wouldn't make any sense to write a default method using the same signature which appears in the Object class, because it could never be used. In fact, the compiler doesn't even allow it.

```java
public interface MyInterface {
    default String toString(){ //Compile time error
        return "MyInterface toString";
    }
}
```

{: .box-error}
**Compilation failure**  
default method toString in interface MyInterface overrides a member of java.lang.Object

<br>
### Interface static methods cannot have the same signature as any method in the Object class

**Object class methods cannot be static methods** for almost the same reason. In a class you cannot have an instance method and a static method with the same signature. Since all classes inherit the Object class' methods, one cannot define an interface which has a static method with the same method signature as one of Object class' methods, because that would instantly break the implementation.

```java
public interface MyInterface {
    static String toString(){ //Compile time error
        return "MyInterface toString";
    }
}
```

{: .box-error}
**Compilation failure**  
toString() in MyInterface cannot override toString() in java.lang.Object

<br>
## Interface implicit modifiers

Before Java 8 we knew an interface is implicitly abstract, its variables are public static final and the methods within are public abstract. Java 8 added default and static methods to the mixture, which are also implicitly public.

```java
public abstract interface MyInterface {
    public static final String COLOR = "BLUE";
    public abstract void doSomething();
    public default void doOther(){}
    public static void doStatic(){}
}
```

<br>
## Summary:
- Since Java 8, we can add concrete implementation to our methods inside an interface due to the introduction of default and interface static methods
- Default and interface static methods are implicitly public
- If a class inherits a method with the same signature from an interface and a class, the class always wins
- Overridden default methods can be accessed using InterfaceName.super.defaultMethodName() (until they are not overridden by a descendant interface or class)
- Inherited default methods with the same signature from two non-related interfaces will cause a compiler error unless the implementing class overrides them
- Interface static methods are not inherited, not even through interfaces
- The only place where interface static methods can be referenced by their name without naming the related interface is within the interface. From outside they can be called as regular static methods: InterfaceName.mystaticMethod()
- Object class methods (e.g. toString, notify, etc..) cannot be default or static methods in interfaces