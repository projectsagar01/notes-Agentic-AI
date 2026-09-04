---
{"dg-publish":true,"permalink":"/src/site/notes/notes/agentic-ai/public-notes/oops/","dg-note-properties":{}}
---

Absolutely. Let’s learn **OOPs (Object-Oriented Programming) in C++ from beginner to interview/problem-solving level**, with explanations, examples, questions, and solutions.

# OOPs in C++ — Complete Guide

OOPs is a programming approach where we organize programs around **objects** rather than just functions and logic.

The main concepts are:

1. **Class**
    
2. **Object**
    
3. **Encapsulation**
    
4. **Abstraction**
    
5. **Inheritance**
    
6. **Polymorphism**
    
7. **Constructor**
    
8. **Destructor**
    
9. **Access Specifiers**
    
10. **`this` pointer**
    
11. **Static members**
    
12. **Virtual functions**
    
13. **Function/Operator overloading**
    
14. **Friend functions/classes**
    

We'll learn them one by one.

---

# 1. Why do we need OOP?

Suppose you want to create a program for students.

Without OOP, you might have:

```cpp
string name1;
int age1;
float marks1;

string name2;
int age2;
float marks2;
```

For 10,000 students, this becomes difficult to manage.

With OOP, we can create a **Student class**:

```cpp
class Student {
public:
    string name;
    int age;
    float marks;
};
```

Then create objects:

```cpp
Student s1;
Student s2;
Student s3;
```

Each object contains its own data.

So:

> **Class = blueprint**  
> **Object = real entity created from that blueprint**

---

# 2. Class

A class is a blueprint/template for creating objects.

Example:

```cpp
#include <iostream>
using namespace std;

class Student {
public:
    string name;
    int age;

    void study() {
        cout << name << " is studying." << endl;
    }
};
```

Here:

```cpp
class Student
```

creates a class named `Student`.

It has:

### Data members

```cpp
string name;
int age;
```

### Member function

```cpp
void study()
```

---

# 3. Object

An object is an instance of a class.

```cpp
Student s1;
```

Here:

- `Student` → class
    
- `s1` → object
    

We can access its members:

```cpp
s1.name = "Rahul";
s1.age = 20;

s1.study();
```

Complete program:

```cpp
#include <iostream>
using namespace std;

class Student {
public:
    string name;
    int age;

    void study() {
        cout << name << " is studying." << endl;
    }
};

int main() {
    Student s1;

    s1.name = "Rahul";
    s1.age = 20;

    cout << s1.name << endl;
    cout << s1.age << endl;

    s1.study();

    return 0;
}
```

Output:

```text
Rahul
20
Rahul is studying.
```

---

# 4. Access Specifiers

C++ has three important access specifiers:

```text
public
private
protected
```

## Public

Members can be accessed from outside the class.

```cpp
class Student {
public:
    string name;
};
```

Then:

```cpp
Student s;
s.name = "Rahul";
```

is allowed.

---

## Private

Members can only be accessed from inside the class.

```cpp
class Student {
private:
    int age;
};
```

This is NOT allowed:

```cpp
Student s;
s.age = 20;  // Error
```

Instead, we use functions.

```cpp
class Student {
private:
    int age;

public:
    void setAge(int a) {
        age = a;
    }

    int getAge() {
        return age;
    }
};
```

Then:

```cpp
Student s;

s.setAge(20);

cout << s.getAge();
```

This is an important OOP concept called **encapsulation**.

---

# 5. Encapsulation

Encapsulation means:

> **Bundling data and functions together and controlling access to the data.**

For example:

```cpp
class BankAccount {
private:
    double balance;

public:
    void deposit(double amount) {
        if (amount > 0)
            balance += amount;
    }

    double getBalance() {
        return balance;
    }
};
```

The user cannot directly do:

```cpp
account.balance = -100000;
```

because `balance` is private.

Instead:

```cpp
account.deposit(5000);
```

This protects the object's data.

### Real-life example

Think about an ATM.

You don't directly access the bank's database.

You interact through:

- deposit
    
- withdrawal
    
- balance check
    

The internal data is protected.

That's encapsulation.

---

# 6. Constructor

A constructor is a special function that is automatically called when an object is created.

Rules:

- Same name as class
    
- No return type
    
- Automatically called
    

Example:

```cpp
class Student {
public:
    Student() {
        cout << "Constructor called" << endl;
    }
};
```

Now:

```cpp
Student s;
```

automatically calls:

```cpp
Student()
```

Output:

```text
Constructor called
```

---

# 7. Parameterized Constructor

A constructor can accept parameters.

```cpp
class Student {
public:
    string name;
    int age;

    Student(string n, int a) {
        name = n;
        age = a;
    }
};
```

Now:

```cpp
Student s1("Rahul", 20);
Student s2("Aman", 21);
```

Each object gets different values.

---

# 8. Constructor Initialization List

A better way of initializing members is:

```cpp
class Student {
private:
    string name;
    int age;

public:
    Student(string n, int a) : name(n), age(a) {}
};
```

This is called an **initializer list**.

You will see it frequently in professional C++ code.

---

# 9. Types of Constructors

Commonly discussed constructors are:

### 1. Default constructor

```cpp
Student() {}
```

### 2. Parameterized constructor

```cpp
Student(string name, int age) {}
```

### 3. Copy constructor

```cpp
Student(const Student& other) {
    name = other.name;
    age = other.age;
}
```

Example:

```cpp
Student s1("Rahul", 20);

Student s2 = s1;
```

`s2` is created as a copy of `s1`.

---

# 10. Destructor

A destructor is called automatically when an object is destroyed.

Syntax:

```cpp
~Student() {
    cout << "Destructor called";
}
```

Notice:

```cpp
~
```

before the class name.

Example:

```cpp
class Student {
public:
    Student() {
        cout << "Constructor" << endl;
    }

    ~Student() {
        cout << "Destructor" << endl;
    }
};
```

When:

```cpp
Student s;
```

is created:

```text
Constructor
```

When the object is destroyed:

```text
Destructor
```

### Important

A class has only **one destructor**.

A destructor cannot take parameters.

---

# 11. Abstraction

Abstraction means:

> **Showing only necessary details and hiding implementation details.**

For example, when you use:

```cpp
cout << "Hello";
```

you don't need to know how `cout` internally sends characters to the output system.

Similarly, when you drive a car:

You use:

- steering
    
- accelerator
    
- brake
    

You don't need to know every internal engine operation.

That's abstraction.

---

# 12. How to implement abstraction in C++

One common technique is an **abstract class**.

Example:

```cpp
class Shape {
public:
    virtual void area() = 0;
};
```

The function:

```cpp
virtual void area() = 0;
```

is called a **pure virtual function**.

A class containing a pure virtual function is an **abstract class**.

You cannot do:

```cpp
Shape s; // Error
```

Instead, another class implements it.

```cpp
class Circle : public Shape {
public:
    void area() override {
        cout << "Area of circle";
    }
};
```

---

# 13. Inheritance

Inheritance allows one class to acquire properties and behavior of another class.

Example:

```cpp
class Animal {
public:
    void eat() {
        cout << "Animal is eating" << endl;
    }
};

class Dog : public Animal {
public:
    void bark() {
        cout << "Dog is barking" << endl;
    }
};
```

Now:

```cpp
Dog d;

d.eat();
d.bark();
```

Output:

```text
Animal is eating
Dog is barking
```

Here:

```text
Animal
   ↓
  Dog
```

`Animal` is the **base/parent class**.

`Dog` is the **derived/child class**.

---

# 14. Why use inheritance?

Suppose:

```text
Vehicle
 ├── Car
 ├── Bike
 └── Bus
```

Common functionality can be placed in `Vehicle`.

```cpp
class Vehicle {
public:
    void start() {
        cout << "Vehicle started";
    }
};
```

Then:

```cpp
class Car : public Vehicle {};
class Bike : public Vehicle {};
```

Both automatically get `start()`.

This reduces code duplication.

---

# 15. Types of Inheritance

Important types:

### Single inheritance

```text
A
↓
B
```

```cpp
class B : public A {};
```

### Multilevel inheritance

```text
A
↓
B
↓
C
```

### Multiple inheritance

```text
A     B
 \   /
   C
```

```cpp
class C : public A, public B {};
```

### Hierarchical inheritance

```text
      A
    /   \
   B     C
```

### Hybrid inheritance

Combination of multiple inheritance types.

---

# 16. Polymorphism

This is one of the most important OOP concepts.

**Poly** = many  
**Morphism** = forms

So:

> Polymorphism means one interface/name can have multiple forms or behaviors.

There are two major types:

```text
Polymorphism
├── Compile-time
└── Runtime
```

---

# 17. Compile-Time Polymorphism

Usually achieved through:

1. Function overloading
    
2. Operator overloading
    

---

## Function Overloading

Same function name but different parameters.

```cpp
class Calculator {
public:

    int add(int a, int b) {
        return a + b;
    }

    double add(double a, double b) {
        return a + b;
    }
};
```

Now:

```cpp
Calculator c;

cout << c.add(2, 3) << endl;
cout << c.add(2.5, 3.5) << endl;
```

The compiler decides which function to call.

That's compile-time polymorphism.

---

# 18. Operator Overloading

C++ allows operators to work with user-defined objects.

For example:

```cpp
a + b
```

where `a` and `b` are objects.

Example:

```cpp
class Complex {
public:
    int real;
    int imag;

    Complex(int r, int i) {
        real = r;
        imag = i;
    }

    Complex operator+(Complex c) {
        return Complex(real + c.real,
                       imag + c.imag);
    }
};
```

Now:

```cpp
Complex c1(2, 3);
Complex c2(4, 5);

Complex c3 = c1 + c2;
```

Result:

```text
real = 6
imag = 8
```

---

# 19. Runtime Polymorphism

Runtime polymorphism is usually achieved through:

> **Inheritance + virtual function**

Example:

```cpp
class Animal {
public:
    virtual void sound() {
        cout << "Animal sound" << endl;
    }
};

class Dog : public Animal {
public:
    void sound() override {
        cout << "Dog barks" << endl;
    }
};

class Cat : public Animal {
public:
    void sound() override {
        cout << "Cat meows" << endl;
    }
};
```

Now:

```cpp
Animal* a;

Dog d;
Cat c;

a = &d;
a->sound();

a = &c;
a->sound();
```

Output:

```text
Dog barks
Cat meows
```

The function is selected at **runtime**.

---

# 20. Why do we need `virtual`?

Consider:

```cpp
class Animal {
public:
    void sound() {
        cout << "Animal sound";
    }
};

class Dog : public Animal {
public:
    void sound() {
        cout << "Dog barks";
    }
};
```

Now:

```cpp
Animal* a = new Dog();

a->sound();
```

Without `virtual`, this can call:

```text
Animal sound
```

because the pointer is of type `Animal*`.

But if we write:

```cpp
virtual void sound()
```

then C++ performs **dynamic dispatch**, and the derived implementation is called:

```text
Dog barks
```

---

# 21. Four Pillars of OOP

The four major pillars are:

|Pillar|Meaning|
|---|---|
|Encapsulation|Binding data and methods + controlling access|
|Abstraction|Hiding implementation details|
|Inheritance|Reusing/extending existing classes|
|Polymorphism|Same interface, different behavior|

Remember these four for interviews.

---

# 22. `this` Pointer

Every non-static member function has access to a pointer called:

```cpp
this
```

It points to the current object.

Example:

```cpp
class Student {
private:
    int age;

public:
    void setAge(int age) {
        this->age = age;
    }
};
```

Here there are two `age`s:

```cpp
int age
```

parameter

and:

```cpp
this->age
```

object's data member.

So:

```cpp
this->age = age;
```

means:

> Put the parameter `age` into the current object's `age`.

---

# 23. Static Data Member

A static data member belongs to the **class**, not to each individual object.

Example:

```cpp
class Student {
public:
    static int count;

    Student() {
        count++;
    }
};

int Student::count = 0;
```

Now:

```cpp
Student s1;
Student s2;
Student s3;

cout << Student::count;
```

Output:

```text
3
```

There is only one shared `count`.

---

# 24. Static Function

A static member function can be called using the class name.

```cpp
class Test {
public:
    static void hello() {
        cout << "Hello";
    }
};
```

Call:

```cpp
Test::hello();
```

A static member function doesn't have a normal `this` pointer because it isn't tied to one particular object.

---

# 25. Friend Function

A friend function is not a member of the class, but it can access private/protected members if declared with `friend`.

```cpp
class Student {
private:
    int marks = 90;

public:
    friend void display(Student s);
};

void display(Student s) {
    cout << s.marks;
}
```

Normally `marks` is private.

But `display()` can access it because it's a friend.

---

# 26. Important OOP Interview Question

### Question 1

What is the difference between a class and an object?

### Answer

**Class:** Blueprint/template.

**Object:** Instance of the class.

Example:

```cpp
class Car {
public:
    string color;
};
```

Here `Car` is the class.

```cpp
Car c1;
Car c2;
```

`c1` and `c2` are objects.

---

# 27. Question 2 — Encapsulation

Create a `BankAccount` class where balance cannot be directly accessed.

### Solution

```cpp
#include <iostream>
using namespace std;

class BankAccount {
private:
    double balance;

public:
    BankAccount(double initialBalance) {
        balance = initialBalance;
    }

    void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }

    void withdraw(double amount) {
        if (amount > 0 && amount <= balance) {
            balance -= amount;
        }
    }

    double getBalance() {
        return balance;
    }
};

int main() {
    BankAccount account(1000);

    account.deposit(500);
    account.withdraw(200);

    cout << account.getBalance();

    return 0;
}
```

Output:

```text
1300
```

### Concept tested

**Encapsulation**

because:

```cpp
private:
    double balance;
```

prevents direct modification.

---

# 28. Question 3 — Constructor

Create a `Student` class that stores name and age and initializes them using a constructor.

### Solution

```cpp
#include <iostream>
using namespace std;

class Student {
private:
    string name;
    int age;

public:
    Student(string name, int age)
        : name(name), age(age) {}

    void display() {
        cout << name << " " << age << endl;
    }
};

int main() {
    Student s1("Rahul", 20);
    Student s2("Aman", 21);

    s1.display();
    s2.display();

    return 0;
}
```

Output:

```text
Rahul 20
Aman 21
```

---

# 29. Question 4 — Inheritance

Create a `Vehicle` class with a `start()` function. Create a `Car` class that inherits from it.

### Solution

```cpp
#include <iostream>
using namespace std;

class Vehicle {
public:
    void start() {
        cout << "Vehicle started" << endl;
    }
};

class Car : public Vehicle {
public:
    void drive() {
        cout << "Car is driving" << endl;
    }
};

int main() {
    Car c;

    c.start();
    c.drive();

    return 0;
}
```

Output:

```text
Vehicle started
Car is driving
```

---

# 30. Question 5 — Runtime Polymorphism

Create an `Animal` base class and make `Dog` and `Cat` provide different sounds.

### Solution

```cpp
#include <iostream>
using namespace std;

class Animal {
public:
    virtual void sound() {
        cout << "Animal makes sound" << endl;
    }

    virtual ~Animal() = default;
};

class Dog : public Animal {
public:
    void sound() override {
        cout << "Dog barks" << endl;
    }
};

class Cat : public Animal {
public:
    void sound() override {
        cout << "Cat meows" << endl;
    }
};

int main() {
    Animal* a;

    Dog d;
    a = &d;
    a->sound();

    Cat c;
    a = &c;
    a->sound();

    return 0;
}
```

Output:

```text
Dog barks
Cat meows
```

The important part:

```cpp
virtual void sound()
```

allows runtime polymorphism.

---

# 31. Question 6 — Abstract Class

Create an abstract `Shape` class and derive `Circle` from it.

### Solution

```cpp
#include <iostream>
using namespace std;

class Shape {
public:
    virtual double area() = 0;

    virtual ~Shape() = default;
};

class Circle : public Shape {
private:
    double radius;

public:
    Circle(double r) : radius(r) {}

    double area() override {
        return 3.14159 * radius * radius;
    }
};

int main() {
    Circle c(5);

    cout << c.area();

    return 0;
}
```

Output approximately:

```text
78.5398
```

Here:

```cpp
virtual double area() = 0;
```

makes `Shape` abstract.

---

# 32. Question 7 — Function Overloading

Create an `add()` function that works with both integers and doubles.

### Solution

```cpp
#include <iostream>
using namespace std;

class Calculator {
public:
    int add(int a, int b) {
        return a + b;
    }

    double add(double a, double b) {
        return a + b;
    }
};

int main() {
    Calculator c;

    cout << c.add(10, 20) << endl;
    cout << c.add(10.5, 20.5) << endl;

    return 0;
}
```

Output:

```text
30
31
```

This is **compile-time polymorphism**.

---

# 33. Question 8 — Copy Constructor

What happens here?

```cpp
Student s1("Rahul", 20);
Student s2 = s1;
```

`Student s2 = s1;` invokes the copy constructor.

Example:

```cpp
class Student {
private:
    string name;
    int age;

public:
    Student(string n, int a)
        : name(n), age(a) {}

    Student(const Student& other)
        : name(other.name), age(other.age) {}

    void display() {
        cout << name << " " << age << endl;
    }
};
```

---

# 34. Question 9 — Predict the Output

```cpp
#include <iostream>
using namespace std;

class Test {
public:
    Test() {
        cout << "Constructor ";
    }

    ~Test() {
        cout << "Destructor ";
    }
};

int main() {
    Test t;
    cout << "Main ";
}
```

### Solution

Output:

```text
Constructor Main Destructor
```

Why?

Object creation:

```cpp
Test t;
```

calls constructor.

Then:

```cpp
cout << "Main ";
```

runs.

At the end of `main()`, `t` is destroyed, so destructor runs.

---

# 35. Question 10 — Constructor/Destructor Order

What is the output?

```cpp
class A {
public:
    A() {
        cout << "A ";
    }

    ~A() {
        cout << "~A ";
    }
};

class B : public A {
public:
    B() {
        cout << "B ";
    }

    ~B() {
        cout << "~B ";
    }
};
```

If:

```cpp
B obj;
```

### Solution

Output:

```text
A B ~B ~A
```

### Rule

Construction:

```text
Base → Derived
```

Destruction:

```text
Derived → Base
```

This is extremely important.

---

# 36. Question 11 — Multiple Inheritance

Create classes `Father` and `Mother` and derive `Child` from both.

### Solution

```cpp
#include <iostream>
using namespace std;

class Father {
public:
    void fatherFeature() {
        cout << "Father feature" << endl;
    }
};

class Mother {
public:
    void motherFeature() {
        cout << "Mother feature" << endl;
    }
};

class Child : public Father, public Mother {
};

int main() {
    Child c;

    c.fatherFeature();
    c.motherFeature();

    return 0;
}
```

Output:

```text
Father feature
Mother feature
```

---

# 37. Diamond Problem

Suppose:

```text
       A
      / \
     B   C
      \ /
       D
```

Both `B` and `C` inherit from `A`.

Then `D` may receive **two copies of A**.

This can cause ambiguity.

C++ solves this using **virtual inheritance**.

```cpp
class A {
};

class B : virtual public A {
};

class C : virtual public A {
};

class D : public B, public C {
};
```

Now `D` contains one shared `A` subobject.

---

# 38. Overloading vs Overriding

This is a very common interview question.

### Overloading

Same function name, different parameters.

```cpp
void print(int x);
void print(double x);
```

Usually compile-time polymorphism.

### Overriding

Derived class provides its own implementation of a base class virtual function.

```cpp
class Animal {
public:
    virtual void sound() {}
};

class Dog : public Animal {
public:
    void sound() override {}
};
```

Usually runtime polymorphism.

---

# 39. Class vs Struct in C++

Both can contain:

- variables
    
- functions
    
- constructors
    
- inheritance
    
- access specifiers
    

Main default difference:

```cpp
class
```

members are **private by default**.

```cpp
struct
```

members are **public by default**.

Example:

```cpp
class A {
    int x; // private
};
```

Whereas:

```cpp
struct A {
    int x; // public
};
```

---

# 40. Composition

Composition means one class **contains** another class.

Example:

```cpp
class Engine {
public:
    void start() {
        cout << "Engine started";
    }
};

class Car {
private:
    Engine engine;

public:
    void startCar() {
        engine.start();
    }
};
```

Here:

```text
Car HAS-A Engine
```

This is called a **HAS-A relationship**.

Inheritance is generally:

```text
Dog IS-A Animal
```

Composition is:

```text
Car HAS-A Engine
```

---

# 41. Inheritance vs Composition

Very important design concept.

### Inheritance

```text
Dog IS-A Animal
```

Use when there is a genuine "is-a" relationship.

### Composition

```text
Car HAS-A Engine
```

Use when one object contains/uses another.

In modern C++ design, **composition is often preferred when inheritance isn't needed**, because it can reduce coupling.

---

# 42. Virtual Destructor

Suppose:

```cpp
class Animal {
public:
    virtual ~Animal() {}
};
```

Why make destructor virtual?

Because if you do:

```cpp
Animal* a = new Dog();
delete a;
```

you want the derived `Dog` destructor to run correctly before the base destructor.

So a base class intended for polymorphic deletion should generally have a **virtual destructor**.

---

# 43. Important Interview Questions

You should be able to answer these:

### Beginner

1. What is OOP?
    
2. What is a class?
    
3. What is an object?
    
4. Difference between class and object?
    
5. What are access specifiers?
    
6. What is a constructor?
    
7. What is a destructor?
    
8. What is a copy constructor?
    
9. What is `this` pointer?
    

### Intermediate

10. What is encapsulation?
    
11. What is abstraction?
    
12. What is inheritance?
    
13. What is polymorphism?
    
14. What is function overloading?
    
15. What is function overriding?
    
16. What is operator overloading?
    
17. What is a virtual function?
    
18. What is a pure virtual function?
    
19. What is an abstract class?
    
20. What is a static member?
    

### Advanced

21. Why should a polymorphic base class have a virtual destructor?
    
22. What is the diamond problem?
    
23. What is virtual inheritance?
    
24. What is object slicing?
    
25. Difference between shallow copy and deep copy?
    
26. What is the Rule of 3?
    
27. What is the Rule of 5?
    
28. What is the Rule of 0?
    
29. What is composition?
    
30. Inheritance vs composition?
    

---

# 44. Practice Problems

Try these yourself before looking at the solutions.

### Problem 1 — Student

Create a `Student` class containing:

```text
name
rollNumber
marks
```

Create functions:

```text
display()
isPassed()
```

A student passes if marks >= 40.

---

### Problem 2 — Bank Account

Create:

```text
BankAccount
```

with:

```text
accountNumber
balance
```

Functions:

```text
deposit()
withdraw()
displayBalance()
```

Don't allow withdrawal if the amount is greater than the balance.

---

### Problem 3 — Rectangle

Create a `Rectangle` class with:

```text
length
width
```

Functions:

```text
area()
perimeter()
```

Use a constructor to initialize them.

---

### Problem 4 — Employee

Create:

```text
Employee
```

with:

```text
name
salary
```

Create:

```text
displaySalary()
```

Then create:

```text
Manager
Developer
```

that inherit from `Employee`.

---

### Problem 5 — Runtime Polymorphism

Create:

```text
Animal
Dog
Cat
Cow
```

with:

```cpp
virtual void sound();
```

Each animal should have a different sound.

Use:

```cpp
Animal*
```

to demonstrate runtime polymorphism.

---

# 45. Solution to Problem 1

```cpp
#include <iostream>
using namespace std;

class Student {
private:
    string name;
    int rollNumber;
    int marks;

public:
    Student(string n, int r, int m)
        : name(n), rollNumber(r), marks(m) {}

    void display() {
        cout << "Name: " << name << endl;
        cout << "Roll Number: " << rollNumber << endl;
        cout << "Marks: " << marks << endl;
    }

    bool isPassed() {
        return marks >= 40;
    }
};

int main() {
    Student s("Rahul", 101, 75);

    s.display();

    if (s.isPassed())
        cout << "Passed";
    else
        cout << "Failed";

    return 0;
}
```

Concepts:

- Class
    
- Object
    
- Constructor
    
- Encapsulation
    
- Member functions
    

---

# 46. Solution to Problem 2

```cpp
#include <iostream>
using namespace std;

class BankAccount {
private:
    int accountNumber;
    double balance;

public:
    BankAccount(int acc, double bal)
        : accountNumber(acc), balance(bal) {}

    void deposit(double amount) {
        if (amount > 0)
            balance += amount;
    }

    void withdraw(double amount) {
        if (amount > 0 && amount <= balance)
            balance -= amount;
        else
            cout << "Invalid withdrawal" << endl;
    }

    void displayBalance() {
        cout << "Balance: " << balance << endl;
    }
};

int main() {
    BankAccount account(12345, 5000);

    account.deposit(1000);
    account.withdraw(2000);

    account.displayBalance();

    return 0;
}
```

---

# 47. Solution to Problem 3

```cpp
#include <iostream>
using namespace std;

class Rectangle {
private:
    double length;
    double width;

public:
    Rectangle(double l, double w)
        : length(l), width(w) {}

    double area() {
        return length * width;
    }

    double perimeter() {
        return 2 * (length + width);
    }
};

int main() {
    Rectangle r(10, 5);

    cout << "Area: " << r.area() << endl;
    cout << "Perimeter: " << r.perimeter() << endl;

    return 0;
}
```

Output:

```text
Area: 50
Perimeter: 30
```

---

# 48. Solution to Problem 4

```cpp
#include <iostream>
using namespace std;

class Employee {
protected:
    string name;
    double salary;

public:
    Employee(string n, double s)
        : name(n), salary(s) {}

    void displaySalary() {
        cout << name << ": " << salary << endl;
    }
};

class Manager : public Employee {
public:
    Manager(string n, double s)
        : Employee(n, s) {}
};

class Developer : public Employee {
public:
    Developer(string n, double s)
        : Employee(n, s) {}
};

int main() {
    Manager m("Rahul", 80000);
    Developer d("Aman", 60000);

    m.displaySalary();
    d.displaySalary();

    return 0;
}
```

This demonstrates **inheritance**.

---

# 49. Solution to Problem 5

```cpp
#include <iostream>
using namespace std;

class Animal {
public:
    virtual void sound() {
        cout << "Animal sound" << endl;
    }

    virtual ~Animal() = default;
};

class Dog : public Animal {
public:
    void sound() override {
        cout << "Dog: Bark" << endl;
    }
};

class Cat : public Animal {
public:
    void sound() override {
        cout << "Cat: Meow" << endl;
    }
};

class Cow : public Animal {
public:
    void sound() override {
        cout << "Cow: Moo" << endl;
    }
};

int main() {
    Animal* animals[3];

    Dog dog;
    Cat cat;
    Cow cow;

    animals[0] = &dog;
    animals[1] = &cat;
    animals[2] = &cow;

    for (int i = 0; i < 3; i++) {
        animals[i]->sound();
    }

    return 0;
}
```

Output:

```text
Dog: Bark
Cat: Meow
Cow: Moo
```

This is **runtime polymorphism**.

---

# 50. The OOP Mental Model

Try to remember OOP this way:

```text
                 OOP
                  |
       -------------------------
       |           |           |
     Data       Behavior     Relationship
       |           |           |
   Variables    Functions    Classes
       |
   Encapsulation
```

And the four pillars:

```text
             OOP
              |
    ---------------------
    |        |          |
Encapsulation Abstraction
    |
Inheritance ---- Polymorphism
```

More accurately, the four pillars are independent concepts that work together:

```text
┌─────────────────────────────┐
│            OOP              │
├─────────────┬───────────────┤
│ Encapsulation│ Abstraction  │
├─────────────┼───────────────┤
│ Inheritance │ Polymorphism  │
└─────────────┴───────────────┘
```

---

# 51. What You Should Learn Next

If you're learning C++ for **placements/interviews/DSA**, I recommend this order:

```text
C++ Basics
   ↓
Functions
   ↓
Arrays / Strings
   ↓
Pointers & References
   ↓
Classes & Objects
   ↓
Constructors / Destructors
   ↓
Encapsulation
   ↓
Inheritance
   ↓
Polymorphism
   ↓
Virtual Functions
   ↓
Abstract Classes
   ↓
STL
   ↓
DSA
```

The most important thing is **not to memorize definitions**. Write small programs and predict what happens.

### Your first OOP challenge

Try solving this without looking at the answer:

> Create a `BankAccount` class with a private `balance`. Add `deposit()` and `withdraw()` functions. Then create two objects, deposit different amounts into them, and print their final balances.

Once you can comfortably solve that, move to **inheritance + virtual functions**, because that's where OOP starts becoming much more interesting.