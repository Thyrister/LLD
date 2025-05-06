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

---


## Logging System
```cpp

#include <iostream>
#include <vector>
#include <string>
using namespace std;

// ---------------- Writers (Strategies) ----------------
class OutputWriter {
public:
    virtual void write(const string& msg) = 0;
    virtual ~OutputWriter() {}
};

class ConsoleWriter : public OutputWriter {
public:
    void write(const string& msg) override {
        cout << "[Console] " << msg << endl;
    }
};

class FileWriter : public OutputWriter {
public:
    void write(const string& msg) override {
        cout << "[File] " << msg << endl;  // Simulated write
    }
};

class AlertWriter : public OutputWriter {
public:
    void write(const string& msg) override {
        cout << "[ALERT] " << msg << endl;
    }
};

// ---------------- Logger Chain (CoR) ----------------
enum LogLevel { INFO, WARNING, ERROR };

class Logger {
protected:
    Logger* next;
    vector<OutputWriter*> writers;

public:
    Logger(Logger* n = nullptr) : next(n) {}

    void addWriter(OutputWriter* writer) {
        writers.push_back(writer);
    }

    virtual void log(const string& msg, LogLevel level) {
        if (next) next->log(msg, level);
    }

    virtual ~Logger() {
        for (auto w : writers) delete w;
        delete next;
    }
};

class InfoLogger : public Logger {
public:
    InfoLogger(Logger* n = nullptr) : Logger(n) {}

    void log(const string& msg, LogLevel level) override {
        if (level == INFO) {
            for (auto writer : writers) writer->write("[INFO] " + msg);
        }
        Logger::log(msg, level);
    }
};

class WarningLogger : public Logger {
public:
    WarningLogger(Logger* n = nullptr) : Logger(n) {}

    void log(const string& msg, LogLevel level) override {
        if (level == WARNING) {
            for (auto writer : writers) writer->write("[WARNING] " + msg);
        }
        Logger::log(msg, level);
    }
};

class ErrorLogger : public Logger {
public:
    ErrorLogger(Logger* n = nullptr) : Logger(n) {}

    void log(const string& msg, LogLevel level) override {
        if (level == ERROR) {
            for (auto writer : writers) writer->write("[ERROR] " + msg);
        }
        Logger::log(msg, level);
    }
};

// ---------------- Client Code ----------------
int main() {
    Logger* loggerChain =
        new InfoLogger(
            new WarningLogger(
                new ErrorLogger()
            )
        );

    // Attach writers
    dynamic_cast<InfoLogger*>(loggerChain)->addWriter(new ConsoleWriter());
    dynamic_cast<WarningLogger*>(loggerChain->next)->addWriter(new ConsoleWriter());
    dynamic_cast<WarningLogger*>(loggerChain->next)->addWriter(new FileWriter());
    dynamic_cast<ErrorLogger*>(loggerChain->next->next)->addWriter(new FileWriter());
    dynamic_cast<ErrorLogger*>(loggerChain->next->next)->addWriter(new AlertWriter());

    loggerChain->log("Everything is running fine", INFO);
    loggerChain->log("Disk usage at 90%", WARNING);
    loggerChain->log("Disk failure detected", ERROR);

    delete loggerChain;
    return 0;
}

```
---

---

## Observer Pattern
```cpp
#include <iostream>
#include <vector>
#include <string>
#include <algorithm>

using namespace std;

// Notification sending strategy
class NotificationChannel {
public:
    virtual void send(const string& user, const string& message) = 0;
    virtual ~NotificationChannel() {}
};

// Concrete notification channels
class Email : public NotificationChannel {
public:
    void send(const string& user, const string& message) override {
        cout << "[Email to " << user << "] " << message << endl;
    }
};

class SMS : public NotificationChannel {
public:
    void send(const string& user, const string& message) override {
        cout << "[SMS to " << user << "] " << message << endl;
    }
};

class AppNotification : public NotificationChannel {
public:
    void send(const string& user, const string& message) override {
        cout << "[App message to " << user << "] " << message << endl;
    }
};

// Subscriber is now a user with preferences
class User {
    string name;
    vector<NotificationChannel*> preferences;

public:
    User(string n) : name(n) {}

    void addPreference(NotificationChannel* channel) {
        preferences.push_back(channel);
    }

    void notify(const string& message) {
        for (auto channel : preferences) {
            channel->send(name, message);
        }
    }

    string getName() const { return name; }
};

// Publisher stays simple
class Publisher {
    vector<User*> subscribers;

public:
    void addSubscriber(User* user) {
        subscribers.push_back(user);
    }

    void notifyAll(const string& message) {
        for (auto user : subscribers) {
            user->notify(message);
        }
    }
};

// Main
int main() {
    Email* email = new Email();
    SMS* sms = new SMS();
    AppNotification* app = new AppNotification();

    User* alice = new User("Alice");
    alice->addPreference(email);
    alice->addPreference(app);

    User* bob = new User("Bob");
    bob->addPreference(sms);

    Publisher news;
    news.addSubscriber(alice);
    news.addSubscriber(bob);

    news.notifyAll("🚨 Breaking: System design interview in 15 mins!");

    // Cleanup
    delete email;
    delete sms;
    delete app;
    delete alice;
    delete bob;

    return 0;
}

```

  
---




