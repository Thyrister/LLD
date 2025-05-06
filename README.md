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

## Observer Pattern - Subscriber/Publisher System
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

---

## Coffee Machine - Decorator Pattern
```cpp

#include <iostream>
#include <string>

using namespace std;

// Abstract Component
class Coffee {
public:
    virtual string getDescription() = 0;
    virtual double cost() = 0;
    virtual ~Coffee() {}
};

// Concrete Component
class BasicCoffee : public Coffee {
public:
    string getDescription() override {
        return "Basic Coffee";
    }

    double cost() override {
        return 5.0;
    }
};

// Abstract Decorator
class CoffeeDecorator : public Coffee {
protected:
    Coffee* coffee;

public:
    CoffeeDecorator(Coffee* c) : coffee(c) {}
    virtual ~CoffeeDecorator() {
        delete coffee; // Cleanup the wrapped component
    }
};

// Concrete Decorators
class Milk : public CoffeeDecorator {
public:
    Milk(Coffee* c) : CoffeeDecorator(c) {}

    string getDescription() override {
        return coffee->getDescription() + ", Milk";
    }

    double cost() override {
        return coffee->cost() + 1.5;
    }
};

class Sugar : public CoffeeDecorator {
public:
    Sugar(Coffee* c) : CoffeeDecorator(c) {}

    string getDescription() override {
        return coffee->getDescription() + ", Sugar";
    }

    double cost() override {
        return coffee->cost() + 0.5;
    }
};

class Chocolate : public CoffeeDecorator {
public:
    Chocolate(Coffee* c) : CoffeeDecorator(c) {}

    string getDescription() override {
        return coffee->getDescription() + ", Chocolate";
    }

    double cost() override {
        return coffee->cost() + 2.0;
    }
};

// Usage
int main() {
    // Base coffee
    Coffee* myCoffee = new BasicCoffee();

    // Add decorators (ingredients)
    myCoffee = new Milk(myCoffee);
    myCoffee = new Sugar(myCoffee);
    myCoffee = new Chocolate(myCoffee);

    cout << "Order: " << myCoffee->getDescription() << endl;
    cout << "Total Cost: $" << myCoffee->cost() << endl;

    // Cleanup all dynamically allocated memory
    delete myCoffee;

    return 0;
}

```
---

---

## Parking System
```cpp

#include <iostream>
#include <string>
#include <vector>
#include <map>
#include <ctime>

using namespace std;

// VEHICLE CLASSES
class Vehicle {
protected:
    string plate;
public:
    Vehicle(string p) : plate(p) {}
    virtual string getType() = 0;
    string getPlate() { return plate; }
};

class Car : public Vehicle {
public:
    Car(string p) : Vehicle(p) {}
    string getType() override { return "Car"; }
};

class Bike : public Vehicle {
public:
    Bike(string p) : Vehicle(p) {}
    string getType() override { return "Bike"; }
};

class Truck : public Vehicle {
public:
    Truck(string p) : Vehicle(p) {}
    string getType() override { return "Truck"; }
};

// PARKING SPOT
class ParkingSpot {
protected:
    string id;
    bool isFree;
    Vehicle* parkedVehicle;
public:
    ParkingSpot(string id) : id(id), isFree(true), parkedVehicle(nullptr) {}
    virtual string getType() = 0;
    bool parkVehicle(Vehicle* v) {
        if (isFree) {
            parkedVehicle = v;
            isFree = false;
            return true;
        }
        return false;
    }
    void removeVehicle() {
        parkedVehicle = nullptr;
        isFree = true;
    }
    bool isAvailable() { return isFree; }
    string getId() { return id; }
};

class SmallSpot : public ParkingSpot {
public:
    SmallSpot(string id) : ParkingSpot(id) {}
    string getType() override { return "Small"; }
};

class MediumSpot : public ParkingSpot {
public:
    MediumSpot(string id) : ParkingSpot(id) {}
    string getType() override { return "Medium"; }
};

class LargeSpot : public ParkingSpot {
public:
    LargeSpot(string id) : ParkingSpot(id) {}
    string getType() override { return "Large"; }
};

// STRATEGY: Pricing
class PricingStrategy {
public:
    virtual double calculateFee(time_t start, time_t end) = 0;
};

class HourlyPricing : public PricingStrategy {
public:
    double calculateFee(time_t start, time_t end) override {
        double hours = difftime(end, start) / 3600;
        return 10.0 * (int)(hours + 1); // ₹10/hour
    }
};

// TICKET
class Ticket {
    Vehicle* vehicle;
    ParkingSpot* spot;
    time_t entryTime;
public:
    Ticket(Vehicle* v, ParkingSpot* s) : vehicle(v), spot(s), entryTime(time(nullptr)) {}
    time_t getEntryTime() { return entryTime; }
    Vehicle* getVehicle() { return vehicle; }
    ParkingSpot* getSpot() { return spot; }
};

// FLOOR
class ParkingFloor {
    string name;
    vector<ParkingSpot*> spots;
public:
    ParkingFloor(string n) : name(n) {}
    void addSpot(ParkingSpot* s) {
        spots.push_back(s);
    }
    ParkingSpot* findAvailableSpot(string vehicleType) {
        for (auto s : spots) {
            if (s->isAvailable()) {
                if ((vehicleType == "Bike" && s->getType() == "Small") ||
                    (vehicleType == "Car" && s->getType() == "Medium") ||
                    (vehicleType == "Truck" && s->getType() == "Large")) {
                    return s;
                }
            }
        }
        return nullptr;
    }
};

// SINGLETON PARKING LOT
class ParkingLot {
    static ParkingLot* instance;
    vector<ParkingFloor*> floors;
    map<string, Ticket*> activeTickets;
    PricingStrategy* pricing;

    ParkingLot() { pricing = new HourlyPricing(); }

public:
    static ParkingLot* getInstance() {
        if (!instance) instance = new ParkingLot();
        return instance;
    }

    void addFloor(ParkingFloor* f) { floors.push_back(f); }

    Ticket* parkVehicle(Vehicle* v) {
        for (auto floor : floors) {
            ParkingSpot* spot = floor->findAvailableSpot(v->getType());
            if (spot) {
                if (spot->parkVehicle(v)) {
                    Ticket* t = new Ticket(v, spot);
                    activeTickets[v->getPlate()] = t;
                    cout << "Vehicle parked at spot " << spot->getId() << endl;
                    return t;
                }
            }
        }
        cout << "No spot available!" << endl;
        return nullptr;
    }

    void unparkVehicle(string plate) {
        if (activeTickets.find(plate) == activeTickets.end()) {
            cout << "Vehicle not found!" << endl;
            return;
        }

        Ticket* t = activeTickets[plate];
        time_t exitTime = time(nullptr);
        double fee = pricing->calculateFee(t->getEntryTime(), exitTime);

        t->getSpot()->removeVehicle();
        cout << "Vehicle " << plate << " unparked. Fee: ₹" << fee << endl;

        delete t;
        activeTickets.erase(plate);
    }

    ~ParkingLot() {
        for (auto f : floors) delete f;
        delete pricing;
    }
};

ParkingLot* ParkingLot::instance = nullptr;

// DEMO
int main() {
    ParkingLot* lot = ParkingLot::getInstance();

    ParkingFloor* floor1 = new ParkingFloor("Floor 1");
    floor1->addSpot(new SmallSpot("S1"));
    floor1->addSpot(new MediumSpot("M1"));
    floor1->addSpot(new LargeSpot("L1"));

    lot->addFloor(floor1);

    Vehicle* car = new Car("MH-12-XY-1234");
    Ticket* t = lot->parkVehicle(car);

    // Simulate time pass
    sleep(2);

    lot->unparkVehicle(car->getPlate());

    delete car;
    return 0;
}


```
---


---

## Chess Game
```cpp
#include <iostream>
#include <vector>
#include <string>
using namespace std;

class Board;
class Block;

class Piece {
protected:
    bool white;
    bool killed;
public:
    Piece(bool white) : white(white), killed(false) {}
    virtual ~Piece() {}
    bool isWhite() const { return white; }
    bool isKilled() const { return killed; }
    void setKilled(bool k) { killed = k; }
    virtual bool canMove(Board* board, Block* start, Block* end) = 0;
};

class Block {
    int x, y;
    string label;
    Piece* piece;
    string assignLabel(int x, int y) {
        string xLabels[] = {"1","2","3","4","5","6","7","8"};
        string yLabels[] = {"A","B","C","D","E","F","G","H"};
        return xLabels[x] + yLabels[y];
    }
public:
    Block(int x, int y, Piece* piece) : x(x), y(y), piece(piece) {
        label = assignLabel(x, y);
    }
    Piece* getPiece() { return piece; }
    void setPiece(Piece* p) { piece = p; }
};

class Rook : public Piece {
public:
    Rook(bool white) : Piece(white) {}
    bool canMove(Board*, Block*, Block*) override { return false; }
};

class Knight : public Piece {
public:
    Knight(bool white) : Piece(white) {}
    bool canMove(Board*, Block*, Block*) override { return false; }
};

class Bishop : public Piece {
public:
    Bishop(bool white) : Piece(white) {}
    bool canMove(Board*, Block*, Block*) override { return false; }
};

class Queen : public Piece {
public:
    Queen(bool white) : Piece(white) {}
    bool canMove(Board*, Block*, Block*) override { return false; }
};

class King : public Piece {
public:
    King(bool white) : Piece(white) {}
    bool canMove(Board*, Block*, Block*) override { return false; }
};

class Pawn : public Piece {
public:
    Pawn(bool white) : Piece(white) {}
    bool canMove(Board*, Block*, Block*) override { return false; }
};

class Board {
    Block* blocks[8][8];
public:
    Board() {
        initializeBoard();
    }

    void initializeBoard() {
        // Set up white pieces
        blocks[0][0] = new Block(0, 0, new Rook(true));
        blocks[0][1] = new Block(0, 1, new Knight(true));
        blocks[0][2] = new Block(0, 2, new Bishop(true));
        blocks[0][3] = new Block(0, 3, new Queen(true));
        blocks[0][4] = new Block(0, 4, new King(true));
        blocks[0][5] = new Block(0, 5, new Bishop(true));
        blocks[0][6] = new Block(0, 6, new Knight(true));
        blocks[0][7] = new Block(0, 7, new Rook(true));
        for (int j = 0; j < 8; j++)
            blocks[1][j] = new Block(1, j, new Pawn(true));

        // Set up black pieces
        blocks[7][0] = new Block(7, 0, new Rook(false));
        blocks[7][1] = new Block(7, 1, new Knight(false));
        blocks[7][2] = new Block(7, 2, new Bishop(false));
        blocks[7][3] = new Block(7, 3, new Queen(false));
        blocks[7][4] = new Block(7, 4, new King(false));
        blocks[7][5] = new Block(7, 5, new Bishop(false));
        blocks[7][6] = new Block(7, 6, new Knight(false));
        blocks[7][7] = new Block(7, 7, new Rook(false));
        for (int j = 0; j < 8; j++)
            blocks[6][j] = new Block(6, j, new Pawn(false));

        // Empty blocks
        for (int i = 2; i <= 5; i++) {
            for (int j = 0; j < 8; j++) {
                blocks[i][j] = new Block(i, j, nullptr);
            }
        }
    }

    Block* getBlock(int x, int y) {
        return blocks[x][y];
    }
};

class Move {
    Block* start;
    Block* end;
public:
    Move(Block* start, Block* end) : start(start), end(end) {}
    bool isValid() {
        return !(start->getPiece()->isWhite() == end->getPiece()->isWhite());
    }
    Block* getStart() { return start; }
    Block* getEnd() { return end; }
};

enum class Status {
    ACTIVE, SAVED, BLACK_WIN, WHITE_WIN, STALEMATE
};

class Player {
    string name;
public:
    Player(string name) : name(name) {}
};

class Game {
    Board* board;
    Player* player1;
    Player* player2;
    bool isWhiteTurn;
    vector<Move*> gameLog;
    Status status;

public:
    Game(Player* p1, Player* p2) : player1(p1), player2(p2), isWhiteTurn(true), status(Status::ACTIVE) {
        board = new Board();
    }

    void makeMove(Move* move) {
        if (move->isValid()) {
            Piece* source = move->getStart()->getPiece();
            if (source->canMove(board, move->getStart(), move->getEnd())) {
                Piece* dest = move->getEnd()->getPiece();
                if (dest != nullptr) {
                    if (dynamic_cast<King*>(dest)) {
                        status = isWhiteTurn ? Status::WHITE_WIN : Status::BLACK_WIN;
                        return;
                    }
                    dest->setKilled(true);
                }
                move->getEnd()->setPiece(source);
                move->getStart()->setPiece(nullptr);
                gameLog.push_back(move);
                isWhiteTurn = !isWhiteTurn;
            }
        }
    }
};



```
---

