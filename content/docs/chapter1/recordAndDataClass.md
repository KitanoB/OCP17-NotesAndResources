---
title: '2.0 - Records and Data Classes'
date: 2024-11-25T21:16:39+10:00
draft: false
weight: 6
summary: Java 16 introduced **records** as a concise way to create immutable data classes. Records simplify the task of creating classes that are primarily used to store data by automatically generating boilerplate code such as constructors, `equals()`, `hashCode()`, and `toString()` methods.
---

## **2.1 Introduction to Records**

Java 16 introduced **records** as a concise way to create immutable data classes. Records simplify the task of creating classes that are primarily used to store data by automatically generating boilerplate code such as constructors, `equals()`, `hashCode()`, and `toString()` methods.

**Example**:

```java
public record Point(int x, int y) { }
```

---

## **2.2 Key Concepts**

### **2.2.1 Structure of a Record**

- **Components**: The variables declared in the record header are called components.
- **Implicit Members**: For each component, the record automatically provides:
  - A private final field.
  - A public accessor method.
  - A canonical constructor.

**Example**:

```java
public record User(String name, int age) { }

// Automatically provides:
// - Private final fields: name, age
// - Public accessor methods: String name(), int age()
// - Canonical constructor: public User(String name, int age)
```

### **2.2.2 Immutability**

- **Fields Are Final**: Record components are final; once assigned, they cannot be modified.
- **Immutable Objects**: Records promote the creation of immutable objects, enhancing thread safety.

---

## **2.3 Common Traps**

### **Trap 1: Customizing Record Components**

**Issue**: Misunderstanding how to add validation or custom behavior in records.

**Solution**: Use a compact constructor to add validation or modify parameters before assignment.

**Example**:

```java
public record Point(int x, int y) {
    public Point {
        if (x < 0 || y < 0) {
            throw new IllegalArgumentException("Coordinates must be non-negative");
        }
    }
}
```

### **Trap 2: Inheritance Limitations with Records**

**Issue**: Records implicitly extend `java.lang.Record` and cannot extend other classes.

**Explanation**:

- **Final Classes**: Records are implicitly final; you cannot subclass a record.
- **Interfaces**: Records can implement interfaces.

**Example**:

```java
public interface Shape {
    double area();
}

public record Circle(double radius) implements Shape {
    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
}
```

### **Trap 3: Mutability of Component Objects**

**Issue**: While the record itself is immutable, if it contains mutable objects, those can still be modified.

**Example**:

```java
public record Person(String name, List<String> hobbies) { }

Person person = new Person("Alice", new ArrayList<>());
person.hobbies().add("Reading"); // Modifies the internal list
```

**Solution**: To ensure full immutability, use unmodifiable collections or make defensive copies.

---

## **2.4 Differences Between Java 8 and Java 17**

- **Java 8**: Does not support records; data classes require manual implementation of constructors, accessors, and methods like `equals()`, `hashCode()`, and `toString()`.
- **Java 17**: Records are a standard feature, providing a concise syntax for data classes.

---

## **2.5 Practical Examples**

### **2.5.1 Defining a Record**

**Record Declaration**:

```java
public record Employee(int id, String name, String department) { }
```

**Usage**:

```java
Employee emp = new Employee(101, "John Doe", "Engineering");

System.out.println(emp.id());          // Outputs: 101
System.out.println(emp.name());        // Outputs: John Doe
System.out.println(emp.department());  // Outputs: Engineering

System.out.println(emp); // Outputs: Employee[id=101, name=John Doe, department=Engineering]
```

### **2.5.2 Adding Custom Methods**

You can add instance methods to a record.

**Example**:

```java
public record Rectangle(double length, double width) {
    public double area() {
        return length * width;
    }
}
```

**Usage**:

```java
Rectangle rect = new Rectangle(5.0, 3.0);
System.out.println(rect.area()); // Outputs: 15.0
```

### **2.5.3 Custom Constructors**

Besides the canonical constructor, you can define additional constructors.

**Example**:

```java
public record Range(int start, int end) {
    public Range {
        if (start > end) {
            throw new IllegalArgumentException("Start must be less than or equal to end");
        }
    }

    public Range(int length) {
        this(0, length);
    }
}
```

**Usage**:

```java
Range range = new Range(10);
System.out.println(range.start()); // Outputs: 0
System.out.println(range.end());   // Outputs: 10
```

---

## **2.6 Advanced Features**

### **2.6.1 Static Members**

Records can contain static fields and methods.

**Example**:

```java
public record Temperature(double value) {
    public static Temperature ZERO = new Temperature(0.0);

    public static Temperature ofCelsius(double celsius) {
        return new Temperature(celsius);
    }

    public static Temperature ofFahrenheit(double fahrenheit) {
        return new Temperature((fahrenheit - 32) * 5 / 9);
    }
}
```

### **2.6.2 Nested Records**

Records can be nested within other classes or records.

**Example**:

```java
public class DataPoint {
    public record Coordinates(double x, double y) { }

    private final Coordinates coordinates;
    private final double value;

    public DataPoint(double x, double y, double value) {
        this.coordinates = new Coordinates(x, y);
        this.value = value;
    }

    // Getters and other methods
}
```

---

## **2.7 Important Considerations**

### **2.7.1 Serialization**

- **Serializable**: Records are serializable if all their components are serializable.
- **Versioning**: Be cautious when modifying records in serialized forms; adding or removing components can break compatibility.

### **2.7.2 Annotations on Components**

You can annotate record components, and these annotations will be present on the corresponding accessor methods and constructor parameters.

**Example**:

```java
public record User(@NotNull String name, @Min(0) int age) { }
```

### **2.7.3 Records and Lombok**

For projects using Lombok, consider whether records meet your needs or if Lombok's features are still required.

---

## **2.8 Limitations of Records**

- **Cannot Extend Other Classes**: Records implicitly extend `java.lang.Record` and cannot inherit from other classes.
- **Cannot Be Abstract**: Records cannot be declared as abstract.
- **No Customization of `equals()` and `hashCode()` Using Fields Outside Components**: The generated methods only consider the record components.

---

## **2.9 Best Practices**

- **Use Records for Immutable Data Classes**: When your class is primarily used to store data and you want immutability.
- **Avoid Mutable Components**: To ensure full immutability, use immutable types for components.
- **Validation in Compact Constructors**: Perform input validation in the compact constructor to maintain invariants.

---

## **2.10 Common Traps**

### **Trap 4: Overriding Accessor Methods Incorrectly**

**Issue**: Modifying the behavior of accessor methods can lead to unexpected results.

**Example**:

```java
public record Point(int x, int y) {
    public int x() {
        return x + 1;
    }
}
```

**Consequence**: The `equals()` and `hashCode()` methods rely on the component values. Altering accessor methods can break these methods' contract.

**Solution**: Avoid overriding accessor methods unless necessary, and ensure consistency.

### **Trap 5: Record Serialization with Custom Implementations**

**Issue**: Custom serialization logic may be required for complex records, which can be error-prone.

**Solution**: Implement `Serializable` carefully and consider using serialization proxies if needed.

---

## **2.11 Differences Between Records and Classes**

| Aspect             | Regular Class                           | Record                                      |
|--------------------|-----------------------------------------|---------------------------------------------|
| Declaration        | `public class ClassName { }`            | `public record ClassName(...) { }`          |
| Mutability         | Mutable by default                      | Immutable components                        |
| Inheritance        | Can extend other classes                | Extends `java.lang.Record`; cannot subclass |
| Constructors       | Must define explicitly                  | Canonical constructor auto-generated        |
| Methods            | Must implement `equals()`, `hashCode()` | Auto-generated `equals()`, `hashCode()`, `toString()` |
| Use Cases          | General-purpose                         | Data carriers (DTOs, value objects)         |

---

## **2.12 References and Further Reading**

- **Official Documentation**: [Java SE 17 - Records](https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html#jls-8.10)
- **JEP 395**: [Records](https://openjdk.java.net/jeps/395)
- **Tutorials**:
  - [Java Records - Baeldung](https://www.baeldung.com/java-record-keyword)
  - [Exploring Records in Java](https://www.infoq.com/articles/java-records-final/)
- **Books**:
  - *Effective Java* by Joshua Bloch (Third Edition) - Includes discussions on immutability and value objects.

---

**Experiment with records in your projects to simplify your data classes and embrace immutability. Happy coding!**