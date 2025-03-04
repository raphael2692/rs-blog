
+++
title = "SOLID principles in python"
tags = ["coding", "python"]
date = "2022-01-03"
draft = false
+++


## Key Points

* The SOLID principles are five guidelines for better object-oriented design in Python, likely improving code maintainability and scalability.
* Each principle—Single Responsibility, Open-Closed, Liskov Substitution, Interface Segregation, and Dependency Inversion—has a specific role in creating robust software.
* Research suggests following these principles can make code easier to test and extend, though their impact can vary by project complexity.

## Introduction to SOLID Principles

The SOLID principles, introduced by Robert C. Martin (Uncle Bob), are a set of five object-oriented design principles that help developers write cleaner, more maintainable, and scalable code. These principles are particularly useful in Python, where object-oriented programming (OOP) is widely used. They are not strict rules but guidelines that, when applied, can significantly enhance the quality of software, especially in collaborative environments.

## Why SOLID Principles Matter

SOLID principles matter because they address common challenges in software development, such as code rigidity, fragility, and immobility. By following these principles, developers can create systems that are easier to maintain, test, and extend. For instance, they help reduce tight coupling between classes, making the code more modular and reusable. This is especially important in large projects where changes in one part of the system shouldn't break other parts.

An unexpected detail is that while these principles are rooted in OOP, their application can sometimes feel less "Pythonic" to developers used to Python's dynamic and flexible nature, leading to debates about their relevance in Python programming.

## Each Principle with Python Examples

Below, we explore each SOLID principle, why it matters, and provide a Python example to illustrate its application. Each example is designed to be memorable and easy to remember, with a simple analogy to help you recall the concept.

### Single-Responsibility Principle (SRP)

**Definition:** A class should have only one reason to change, meaning it should do one job. Think of it as **"one class, one job."**

**Why it matters:** If a class does too much, changes to one part can break another, making maintenance harder. It also makes testing easier when each class has a single focus.

**Python Example:**

```python
# Violation of SRP
class UserManager:
    def __init__(self):
        self.users = []
    def add_user(self, name, email):
        self.users.append({"name": name, "email": email})
    def send_notification(self, user, message):
        # Logic to send email to user's email address
        pass

# Following SRP
class UserRepository:
    def __init__(self):
        self.users = []
    def add_user(self, name, email):
        self.users.append({"name": name, "email": email})

class NotificationService:
    def send_notification(self, email, message):
        # Logic to send email
        pass
```

Here, `UserManager` originally handled both managing users and sending notifications, violating SRP. By splitting into `UserRepository` and `NotificationService`, each class has one job, making it easier to maintain. **Remember it as "one class, one job."**

### Open-Closed Principle (OCP)

**Definition:** Software should be open for extension but closed for modification. Think **"extend, don't modify."**

**Why it matters:** This allows adding new features without changing existing code, reducing the risk of introducing bugs in working systems.

**Python Example:**

```python
# Violation of OCP
class AreaCalculator:
    def calculate_area(self, shape):
        if shape == "rectangle":
            # calculate rectangle area
            pass
        elif shape == "circle":
            # calculate circle area
            pass
        else:
            raise ValueError("Unknown shape")

# Following OCP
from abc import ABC, abstractmethod

class Shape(ABC):
    @property
    @abstractmethod
    def area(self):
        pass

class Rectangle(Shape):
    def __init__(self, length, width):
        self.length = length
        self.width = width
    @property
    def area(self):
        return self.length * self.width

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius
    @property
    def area(self):
        return 3.14 * self.radius ** 2

class AreaCalculator:
    def calculate_area(self, shape):
        return shape.area

# To add a new shape, just create a new subclass of Shape
class Square(Shape):
    def __init__(self, side):
        self.side = side
    @property
    def area(self):
        return self.side ** 2
```
In the first example, adding a new shape requires changing `AreaCalculator`, violating OCP. Using abstract base classes, we can extend by adding new shapes like `Square` without modifying existing code. **Remember it as "extend, don't modify."**

### Liskov Substitution Principle (LSP)

**Definition:** Subtypes should be substitutable for their base types. Think **"if it quacks like a duck, it should be a duck."**

**Why it matters:** Ensures subclasses don't break the expected behavior of the base class, maintaining system consistency.

**Python Example:**

```python
# Violation of LSP
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height
    def set_width(self, width):
        self.width = width
    def set_height(self, height):
        self.height = height

class Square(Rectangle):
    def __init__(self, side):
        super().__init__(side, side)
    def set_width(self, width):
        self.width = width
        self.height = width
    def set_height(self, height):
        self.height = height
        self.width = height

# Following LSP
class Shape:
    def area(self):
        pass

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height
    def area(self):
        return self.width * self.height

class Square(Shape):
    def __init__(self, side):
        self.side = side
    def area(self):
        return self.side ** 2
```
Here, `Square` as a `Rectangle` subclass can cause issues when setting dimensions, violating LSP. By making both subclasses of `Shape`, we ensure substitutability. **Remember it as "if it quacks like a duck, it should be a duck."**

### Interface Segregation Principle (ISP)

**Definition:** Clients should not depend on interfaces they don't use. Think **"only pay for what you use."**

**Why it matters:** Prevents classes from implementing unnecessary methods, reducing complexity and potential errors.

**Python Example:**

```python
# Violation of ISP
class Device:
    def turn_on(self):
        pass
    def turn_off(self):
        pass
    def play_music(self):
        pass

class Light(Device):
    def turn_on(self):
        print("Light is on")
    def turn_off(self):
        print("Light is off")
    def play_music(self):
        raise NotImplementedError("Light cannot play music")

# Following ISP
class PowerSwitch:
    def turn_on(self):
        pass
    def turn_off(self):
        pass

class MusicPlayer:
    def play_music(self):
        pass

class Light(PowerSwitch):
    def turn_on(self):
        print("Light is on")
    def turn_off(self):
        print("Light is off")

class Speaker(PowerSwitch, MusicPlayer):
    def turn_on(self):
        print("Speaker is on")
    def turn_off(self):
        print("Speaker is off")
    def play_music(self):
        print("Playing music")
```
In the first example, `Light` must implement `play_music`, which it doesn't need, violating ISP. By splitting into `PowerSwitch` and `MusicPlayer`, `Light` only implements what it needs. **Remember it as "only pay for what you use."**

### Dependency Inversion Principle (DIP)

**Definition:** High-level modules should depend on abstractions, not details. Think **"depend on interfaces, not implementations."**

**Why it matters:** Decouples high-level logic from low-level details, making the system more flexible and easier to change.

**Python Example:**

```python
# Violation of DIP
class Database:
    def get_data(self):
        return ["data1", "data2"]

class ReportGenerator:
    def __init__(self, database):
        self.database = database
    def generate_report(self):
        data = self.database.get_data()
        # Generate report from data
        pass

# Following DIP
from abc import ABC, abstractmethod

class DataSource(ABC):
    @abstractmethod
    def get_data(self):
        pass

class Database(DataSource):
    def get_data(self):
        return ["data1", "data2"]

class ReportGenerator:
    def __init__(self, data_source):
        self.data_source = data_source
    def generate_report(self):
        data = self.data_source.get_data()
        # Generate report from data
        pass

# Using a different data source
class API(DataSource):
    def get_data(self):
        # Fetch data from API
        return ["api_data1", "api_data2"]

report_generator = ReportGenerator(API())
report_generator.generate_report()
```
In the first example, `ReportGenerator` is tied to `Database`, violating DIP. By using `DataSource` abstraction, it can work with `Database` or `API`, enhancing flexibility. **Remember it as "depend on interfaces, not implementations."**

## Conclusion and Implications

The SOLID principles provide a framework for writing maintainable and flexible code in Python. Their application, while sometimes debated for being less "Pythonic" due to Python's dynamic nature, is supported by evidence from various sources, suggesting improved code quality in collaborative and large-scale projects. The examples provided are designed to be memorable, using analogies like "one class, one job" for SRP, ensuring developers can easily recall and apply these principles.