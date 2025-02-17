# LLD
Low Level Design

Resources:

### 1. [Watch the video on YouTube](https://www.youtube.com/watch?v=Fn0xBsmact4&list=PLvv0ScY6vfd_ocTP2ZLicgqKnvq50OCXM)
### 2. [Github.com](https://github.com/ashishps1/awesome-low-level-design?tab=readme-ov-file)

### 3. [Medium.com Must Read](https://medium.com/@yc-kuo)
---

## SOLID Principles

These principles are basically designed for writing classes and objects in OOPs programming.

**Single Responsibility**:  
  A class should have ony one responsibility.

**Open Closed**:  
 A class should be open for extension but for not modifications.

**Lisakov Principle**:  
  If a class has been derived from some parent class then the object of parent class can be assigned to the child object.

**Interface segregation**:  
  No force on any class to implement anything which is not required but have to implemented because of the deriving interface/class.

**Dependency Inversion**:  
  Instead of overriding the method of parent its better to declare a interface and let parent and child class inherit from that repsective interface.
  
---

### **Design Patterns**
- **Creational Patterns**:  
  1. Singleton - A design where the only single object of class is required. (ex- db pools, logg aggregator, etc)
  2. Factory - The Factory Method Pattern is a Creational Design Pattern that provides an interface for creating objects without specifying their concrete classes. So its kind of wrapper class to create the objects of another classes.
  3. Abstract Factory - The Abstract Factory Pattern is an extension of the Factory Method Pattern, used when we need to create families of related objects without specifying their exact classes.
  4. Builder - The Builder Pattern is a Creational Design Pattern used to construct complex objects step by step. Instead of using a long constructor with multiple parameters, it provides a flexible way to build objects.
  5. Prototype - The Prototype Pattern is a Creational Design Pattern used to create new objects by copying an existing object (cloning) instead of building from scratch.
 
- **Structural Patterns**:  
  1. Adapter - The Adapter Pattern is a Structural Design Pattern that allows incompatible interfaces to work together by acting as a bridge between them. Think of an electric adapter that lets a European plug work in an Indian socket—it converts one interface to another.
  2. Bridge - The Bridge Pattern is a Structural Design Pattern that decouples abstraction from implementation, allowing them to evolve independently. It helps avoid the explosion of subclasses when combining multiple variations of a class.
  3. 


---



