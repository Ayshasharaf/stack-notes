# 🧱 OOP — Object-Oriented Programming (Dead Simple Notes)

---

## 🤔 Why Does OOP Even Exist?

Imagine writing a game with 10,000 lines of code in one file, no structure, everything is just floating functions and variables. It becomes a nightmare to maintain.

**OOP says:** Model your code like the real world. Everything is an **Object**. Objects have **properties** (data) and **behaviors** (actions).

```
Real World  →  OOP
─────────────────────────────
Car         →  Object
color, brand →  Properties (fields)
drive(), honk() →  Behaviors (methods)
Blueprint   →  Class
```

---

## 🏗️ The 4 Pillars of OOP

```
┌─────────────────────────────────────────────────────────────┐
│                     4 PILLARS OF OOP                        │
│                                                             │
│   1. ENCAPSULATION    │    2. ABSTRACTION                   │
│   "Hide the data"     │    "Hide the complexity"            │
│                       │                                     │
│   3. INHERITANCE      │    4. POLYMORPHISM                  │
│   "Reuse the code"    │    "One interface, many forms"      │
└─────────────────────────────────────────────────────────────┘
```

---

## 📦 CLASS vs OBJECT

**Class** = Blueprint / Template  
**Object** = The actual thing built from that blueprint

```
CLASS (Blueprint)               OBJECTS (Actual instances)
─────────────────               ──────────────────────────
       Car                      myCar = new Car("Red", "Toyota")
   ───────────                  yourCar = new Car("Blue", "BMW")
   - color                      
   - brand                      Both are Cars, but different objects
   - speed                      with their own data.
   ───────────
   + drive()
   + honk()
   + brake()
```

**In Java:**
```java
// Define the CLASS (blueprint)
public class Car {
    String color;   // field / property
    String brand;
    int speed;

    // Constructor — called when you create an object
    public Car(String color, String brand) {
        this.color = color;
        this.brand = brand;
        this.speed = 0;
    }

    // Methods — behaviors
    public void drive() {
        speed = 60;
        System.out.println(brand + " is driving at " + speed + " km/h");
    }

    public void honk() {
        System.out.println(brand + " goes BEEP!");
    }
}

// Create OBJECTS from the class
Car myCar = new Car("Red", "Toyota");
Car yourCar = new Car("Blue", "BMW");

myCar.drive();   // Toyota is driving at 60 km/h
yourCar.honk();  // BMW goes BEEP!
```

---

## 🔒 Pillar 1: ENCAPSULATION — "Hide the Data"

> **Idea:** Keep your data private. Only expose it through controlled methods (getters/setters). Protects your object from being messed with directly.

**Real world analogy:**
```
ATM Machine
────────────────────────────────────────
You can't reach inside and grab cash.
You use the interface: insert card → enter PIN → withdraw.
The internal logic is HIDDEN. That's encapsulation.
```

**Without Encapsulation (BAD):**
```java
BankAccount account = new BankAccount();
account.balance = -99999;  // Anyone can set ANY value. Dangerous!
```

**With Encapsulation (GOOD):**
```java
public class BankAccount {
    private double balance;  // private = no one can touch this directly

    // Getter — controlled read
    public double getBalance() {
        return balance;
    }

    // Setter — controlled write with validation
    public void deposit(double amount) {
        if (amount > 0) {          // validation!
            balance += amount;
        }
    }

    public void withdraw(double amount) {
        if (amount > 0 && amount <= balance) {  // validation!
            balance -= amount;
        } else {
            System.out.println("Insufficient funds!");
        }
    }
}

// Usage
BankAccount acc = new BankAccount();
acc.deposit(1000);
acc.withdraw(500);
// acc.balance = -99999;  ← COMPILE ERROR. Can't do this anymore. ✅
```

**Rule of thumb:** Make all fields `private`. Expose them only via public methods.

---

## 🎭 Pillar 2: ABSTRACTION — "Hide the Complexity"

> **Idea:** Show only what's necessary. Hide the how. You use a TV remote without knowing how infrared signals work.

**Real world analogy:**
```
You press the brake pedal in a car.
You don't need to know about:
  - hydraulic fluid pressure
  - caliper compression
  - friction coefficient
You just know: press = stop. That's abstraction.
```

In Java, abstraction is achieved with **Abstract Classes** and **Interfaces**.

**Abstract Class:**
```java
// You CAN'T create an object directly from an abstract class
// It's a template for other classes
abstract class Shape {
    String color;

    // Abstract method — subclasses MUST implement this
    abstract double calculateArea();

    // Regular method — subclasses inherit this as-is
    public void displayColor() {
        System.out.println("Color: " + color);
    }
}

class Circle extends Shape {
    double radius;

    Circle(double radius) { this.radius = radius; }

    @Override
    double calculateArea() {
        return Math.PI * radius * radius;  // their own implementation
    }
}

class Rectangle extends Shape {
    double width, height;

    Rectangle(double w, double h) { width = w; height = h; }

    @Override
    double calculateArea() {
        return width * height;  // different implementation
    }
}

// Usage
Shape c = new Circle(5);
Shape r = new Rectangle(4, 6);

System.out.println(c.calculateArea());  // 78.53...
System.out.println(r.calculateArea());  // 24.0
```

**Interface:**
```java
// Interface = a pure contract. "If you implement me, you MUST have these methods."
interface Flyable {
    void fly();       // no body — just the contract
    void land();
}

interface Swimmable {
    void swim();
}

// A class can implement MULTIPLE interfaces (unlike extends which is only one)
class Duck implements Flyable, Swimmable {
    public void fly()  { System.out.println("Duck is flying"); }
    public void land() { System.out.println("Duck landed"); }
    public void swim() { System.out.println("Duck is swimming"); }
}
```

**Abstract class vs Interface:**
```
Abstract Class              │  Interface
────────────────────────────┼──────────────────────────────
Has some implementation     │  No implementation (Java 7)
                            │  (Can have default methods Java 8+)
Can have fields             │  Only constants
extend (one only)           │  implement (multiple!)
IS-A relationship           │  CAN-DO relationship
e.g. Animal → Dog           │  e.g. Flyable → Bird, Plane, Superman
```

---

## 👨‍👩‍👧 Pillar 3: INHERITANCE — "Reuse the Code"

> **Idea:** A child class gets all the properties and methods of the parent class. No rewriting. Just extend and add what's new.

**Real world analogy:**
```
Animal
  ├── Dog    (inherits: breathe, eat, sleep + adds: bark)
  ├── Cat    (inherits: breathe, eat, sleep + adds: meow)
  └── Bird   (inherits: breathe, eat, sleep + adds: fly)
```

**In Java:**
```java
// PARENT class
public class Animal {
    String name;
    int age;

    public Animal(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public void eat() {
        System.out.println(name + " is eating.");
    }

    public void sleep() {
        System.out.println(name + " is sleeping.");
    }
}

// CHILD class — inherits everything from Animal
public class Dog extends Animal {
    String breed;

    public Dog(String name, int age, String breed) {
        super(name, age);  // calls Animal's constructor
        this.breed = breed;
    }

    // New behavior only Dog has
    public void bark() {
        System.out.println(name + " says: WOOF!");
    }
}

public class Cat extends Animal {
    public Cat(String name, int age) {
        super(name, age);
    }

    public void meow() {
        System.out.println(name + " says: MEOW!");
    }
}

// Usage
Dog d = new Dog("Bruno", 3, "Labrador");
d.eat();    // inherited from Animal → "Bruno is eating."
d.sleep();  // inherited from Animal → "Bruno is sleeping."
d.bark();   // Dog's own method  → "Bruno says: WOOF!"

Cat c = new Cat("Kitty", 2);
c.eat();    // inherited
c.meow();   // Cat's own
```

**Key keywords:**
```
extends    → inherit from a class
super()    → call parent's constructor
super.methodName() → call parent's version of a method
@Override  → you're replacing a parent method with your own version
```

**Inheritance tree visual:**
```
          Animal
         /  |  \
       Dog  Cat  Bird
        |
     GuideDog   (Dog extended further)
```

**Important rules:**
- Java = **single inheritance** only (one parent class)
- But can implement **multiple interfaces**
- A child IS-A parent. `Dog IS-A Animal` ✅

---

## 🦎 Pillar 4: POLYMORPHISM — "One Interface, Many Forms"

> **Idea:** One method name, many different behaviors depending on which object calls it. *Poly* = many, *Morph* = forms.

**Real world analogy:**
```
You say "speak" to:
  → A Dog    → it barks
  → A Cat    → it meows
  → A Human  → it talks

Same command. Different result. That's polymorphism.
```

**Two types:**

### 1. Compile-time Polymorphism — Method Overloading

```java
// Same method name, different parameters
public class Calculator {
    public int add(int a, int b) {
        return a + b;
    }

    public double add(double a, double b) {  // overloaded
        return a + b;
    }

    public int add(int a, int b, int c) {    // overloaded again
        return a + b + c;
    }
}

Calculator calc = new Calculator();
calc.add(2, 3);         // calls first version → 5
calc.add(2.5, 3.1);     // calls second version → 5.6
calc.add(1, 2, 3);      // calls third version → 6
// Java figures out WHICH one to call at compile time
```

### 2. Runtime Polymorphism — Method Overriding

```java
class Animal {
    public void speak() {
        System.out.println("Some animal sound...");
    }
}

class Dog extends Animal {
    @Override
    public void speak() {
        System.out.println("WOOF!");
    }
}

class Cat extends Animal {
    @Override
    public void speak() {
        System.out.println("MEOW!");
    }
}

// THE MAGIC — same type reference, different behavior
Animal a1 = new Dog();  // Animal reference, Dog object
Animal a2 = new Cat();  // Animal reference, Cat object
Animal a3 = new Animal();

a1.speak();  // WOOF!   ← Java calls Dog's version at runtime
a2.speak();  // MEOW!   ← Java calls Cat's version at runtime
a3.speak();  // Some animal sound...

// Even more powerful with lists:
List<Animal> animals = new ArrayList<>();
animals.add(new Dog());
animals.add(new Cat());
animals.add(new Dog());

for (Animal a : animals) {
    a.speak();  // Each calls its OWN version automatically!
}
// Output: WOOF! MEOW! WOOF!
```

---

## 🔗 Relationships Between Classes

### 1. IS-A (Inheritance)
```
Dog IS-A Animal     → use extends
```

### 2. HAS-A (Composition)
```
Car HAS-A Engine    → Engine is a field inside Car
Person HAS-A Address

public class Car {
    private Engine engine;  // HAS-A relationship
    private String brand;

    public Car() {
        this.engine = new Engine();  // Car "owns" an Engine
    }

    public void start() {
        engine.ignite();  // delegates to Engine
    }
}
```

### 3. CAN-DO (Interface / Implementation)
```
Bird CAN-DO Flyable    → implements Flyable
Plane CAN-DO Flyable   → implements Flyable
```

---

## 🧩 Constructors — How Objects Are Born

```java
public class Person {
    String name;
    int age;

    // Default constructor (no args)
    public Person() {
        name = "Unknown";
        age = 0;
    }

    // Parameterized constructor
    public Person(String name, int age) {
        this.name = name;  // "this" = refers to THIS object's field
        this.age = age;
    }

    // Constructor chaining — one constructor calls another
    public Person(String name) {
        this(name, 0);  // calls the 2-arg constructor with default age
    }
}

Person p1 = new Person();            // calls default
Person p2 = new Person("Alice", 25); // calls parameterized
Person p3 = new Person("Bob");       // calls the chained one → age = 0
```

---

## ⚡ Static vs Instance

```java
public class Counter {
    static int totalCount = 0;  // STATIC: shared across ALL objects
    int id;                     // INSTANCE: each object has its own

    public Counter() {
        totalCount++;           // every new Counter increments this
        this.id = totalCount;
    }

    static void showTotal() {   // static method — call on class
        System.out.println("Total: " + totalCount);
    }
}

Counter c1 = new Counter();  // totalCount = 1, c1.id = 1
Counter c2 = new Counter();  // totalCount = 2, c2.id = 2
Counter c3 = new Counter();  // totalCount = 3, c3.id = 3

Counter.showTotal();  // Total: 3  ← called on CLASS, not object
```

```
STATIC                     │  INSTANCE
───────────────────────────┼───────────────────────────
Belongs to the class       │  Belongs to each object
Shared by all objects      │  Each object has its own copy
Called via ClassName.method│  Called via objectName.method
Memory allocated once      │  Memory per object
```

---

## 🎯 Access Modifiers — Who Can See What?

```
Modifier    │  Same Class  │  Same Package  │  Subclass  │  Everywhere
────────────┼──────────────┼────────────────┼────────────┼────────────
private     │     ✅       │      ❌        │    ❌      │    ❌
(default)   │     ✅       │      ✅        │    ❌      │    ❌
protected   │     ✅       │      ✅        │    ✅      │    ❌
public      │     ✅       │      ✅        │    ✅      │    ✅
```

**General rule for OOP:**
- Fields → `private`
- Methods you want to expose → `public`
- Methods only subclasses should use → `protected`
- Internal helper methods → `private`

---

## 🧠 Quick Concept Summary

```
Concept          │  One Line Explanation
─────────────────┼─────────────────────────────────────────────────
Class            │  Blueprint/template for creating objects
Object           │  An actual instance built from a class
Constructor      │  Special method called when object is created
Encapsulation    │  Private fields + public getters/setters
Abstraction      │  Hide complexity, show only what's needed
Inheritance      │  Child class gets parent's fields & methods
Polymorphism     │  Same method name, different behavior per class
Interface        │  A contract — "you MUST have these methods"
Abstract Class   │  Partial blueprint — some methods left for child
this             │  Refers to the current object
super            │  Refers to the parent class
static           │  Belongs to class, not object
final            │  Can't be changed (variable) / inherited (class) / overridden (method)
```

---

## 🔨 Mini Project — Put It All Together

**Scenario:** A small employee management system.

```java
// Abstract base class
abstract class Employee {
    private String name;        // encapsulation
    private double baseSalary;

    public Employee(String name, double baseSalary) {
        this.name = name;
        this.baseSalary = baseSalary;
    }

    public String getName() { return name; }
    public double getBaseSalary() { return baseSalary; }

    // Abstract — every employee type calculates salary differently
    public abstract double calculateSalary();

    public void displayInfo() {
        System.out.println("Name: " + getName());
        System.out.println("Salary: $" + calculateSalary());
    }
}

// Interface — some employees can work overtime
interface OvertimeEligible {
    double calculateOvertime(int hours);
}

// Full-time employee
class FullTimeEmployee extends Employee implements OvertimeEligible {
    public FullTimeEmployee(String name, double salary) {
        super(name, salary);
    }

    @Override
    public double calculateSalary() {
        return getBaseSalary();  // fixed salary
    }

    @Override
    public double calculateOvertime(int hours) {
        return hours * (getBaseSalary() / 160) * 1.5;  // 1.5x rate
    }
}

// Part-time employee
class PartTimeEmployee extends Employee {
    private int hoursWorked;

    public PartTimeEmployee(String name, double hourlyRate, int hours) {
        super(name, hourlyRate);
        this.hoursWorked = hours;
    }

    @Override
    public double calculateSalary() {
        return getBaseSalary() * hoursWorked;  // hourly * hours
    }
}

// Main — POLYMORPHISM in action
public class Main {
    public static void main(String[] args) {
        List<Employee> employees = new ArrayList<>();

        employees.add(new FullTimeEmployee("Alice", 5000));
        employees.add(new PartTimeEmployee("Bob", 25, 80));
        employees.add(new FullTimeEmployee("Carol", 6000));

        for (Employee e : employees) {
            e.displayInfo();  // each calls THEIR OWN calculateSalary()
            System.out.println("---");
        }
    }
}
```

Output:
```
Name: Alice
Salary: $5000.0
---
Name: Bob
Salary: $2000.0
---
Name: Carol
Salary: $6000.0
---
```

---

*All 4 pillars used: Encapsulation (private fields), Abstraction (abstract class + interface), Inheritance (extends Employee), Polymorphism (each calculates salary differently).*
