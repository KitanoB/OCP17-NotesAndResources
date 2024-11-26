---
title: '1.0 - Class Structure and Components'
date: 2024-11-25T21:16:39+10:00
draft: false
weight: 2
summary: Understanding Java class declarations, modifiers, and common traps [READ THE ARTICLE](https://kitanob.github.io/OCP17-NotesAndResources/docs/chapter1/)
---

## **1.1 Class Declarations and Modifiers**

### **Key Concepts**

- **Access Modifiers**: `public`, `protected`, `private`, and default (package-private).
- **Non-Access Modifiers**: `abstract`, `final`, `static`, `strictfp`, `sealed`, `non-sealed`.

### **Understanding Sealed and Non-Sealed Classes in Java**

Java 17 introduced **sealed classes** and **interfaces** to provide more control over inheritance and to model restricted class hierarchies. This feature allows developers to explicitly specify which classes or interfaces are **permitted to extend or implement** a sealed class or interface. Additionally, **non-sealed** classes complement sealed classes by allowing subclasses to "open up" the hierarchy again.

#### **1.1.1 Definition of Sealed Classes**

A **sealed class** is a class that restricts which other classes or interfaces can extend or implement it. This is achieved by declaring the class with the `sealed` modifier and specifying the permitted subclasses using the `permits` clause. Sealed classes help in:

- **Restricting Extensibility**: They limit the classes that can inherit from them, providing better control over the class hierarchy.
- **Enhancing Maintainability**: By knowing all subclasses, developers can reason about the code more effectively.
- **Exhaustive Pattern Matching**: They enable the compiler to perform exhaustive checks when using pattern matching with sealed classes.

**Example**:

```java
public sealed class Shape permits Circle, Rectangle {
    // Class code
}
```

#### **1.1.2 Definition of Non-Sealed Classes**

A **non-sealed class** is a subclass of a sealed class that lifts the sealing restriction, allowing further subclasses to extend it without any limitations. By declaring a class as `non-sealed`, you essentially "open up" the class hierarchy from that point onward.

**Example**:

```java
public non-sealed class Rectangle extends Shape {
    // Class code
}
```

---

## **1.2 Syntax and Rules**

### **1.2.1 Declaring a Sealed Class**

```java
public sealed class Vehicle permits Car, Truck, Motorcycle {
    // Class body
}
```

- **Modifiers**: The class is declared with the `sealed` keyword.
- **Permits Clause**: Specifies the allowed subclasses (`Car`, `Truck`, `Motorcycle`).

### **1.2.2 Declaring Permitted Subclasses**

Each class specified in the `permits` clause must directly extend the sealed class and must explicitly declare its own status:

- **Final**: Cannot be extended further.
- **Sealed**: Continues the sealing with its own set of permitted subclasses.
- **Non-Sealed**: Opens up the hierarchy, allowing unrestricted subclassing.

**Examples**:

1. **Final Subclass**

   ```java
   public final class Car extends Vehicle {
       // Class body
   }
   ```

2. **Sealed Subclass**

   ```java
   public sealed class Truck extends Vehicle permits PickupTruck, SemiTruck {
       // Class body
   }
   ```

3. **Non-Sealed Subclass**

   ```java
   public non-sealed class Motorcycle extends Vehicle {
       // Class body
   }
   ```

### **1.2.3 Rules and Constraints**

- **Direct Extension**: Permitted subclasses must directly extend the sealed class.
- **Modifiers in Subclasses**: Permitted subclasses must explicitly declare one of `final`, `sealed`, or `non-sealed`.
- **All Subclasses Known**: The `permits` clause must list all permitted subclasses at compile-time.

---

## **1.3 Benefits of Sealed and Non-Sealed Classes**

### **1.3.1 Enhanced Control Over Inheritance**

By sealing a class, you have explicit control over which classes can extend it. This is beneficial when you want to:

- **Enforce Invariants**: Ensure that only known classes can modify or extend certain behaviors.
- **Prevent Unauthorized Extension**: Avoid unintended use or misuse of your classes by external code.

### **1.3.2 Improved Code Safety and Maintainability**

- **Exhaustive Pattern Matching**: When using sealed classes in pattern matching (e.g., in `switch` expressions), the compiler knows all possible subclasses, allowing for exhaustive checks and reducing the likelihood of runtime errors.
- **Clear Hierarchies**: Makes the class hierarchy more predictable and easier to understand.

### **1.3.3 Flexibility with Non-Sealed Classes**

- **Selective Unsealing**: Non-sealed subclasses allow parts of your hierarchy to be open for extension, providing a balance between control and flexibility.
- **Gradual Migration**: Useful when transitioning from an open hierarchy to a sealed one without breaking existing code.

---

## **1.4 Practical Examples**

### **Example 1: Vehicle Hierarchy**

**Objective**: Model a vehicle hierarchy where only specific types of vehicles are allowed.

#### **1.4.1 Defining the Sealed Class**

```java
// Vehicle.java
public sealed class Vehicle permits Car, Truck, Motorcycle {
    // Common properties and methods
}
```

#### **1.4.2 Permitted Subclasses**

**Final Subclass: Car**

```java
// Car.java
public final class Car extends Vehicle {
    // Car-specific properties and methods
}
```

**Sealed Subclass: Truck**

```java
// Truck.java
public sealed class Truck extends Vehicle permits PickupTruck, SemiTruck {
    // Truck-specific properties and methods
}
```

**Non-Sealed Subclass: Motorcycle**

```java
// Motorcycle.java
public non-sealed class Motorcycle extends Vehicle {
    // Motorcycle-specific properties and methods
}
```

#### **1.4.3 Extending Non-Sealed Class**

Since `Motorcycle` is non-sealed, it can be extended without restrictions.

```java
// SportBike.java
public class SportBike extends Motorcycle {
    // SportBike-specific properties and methods
}
```

#### **1.4.4 Extending Sealed Subclass**

The `Truck` class is sealed and permits specific subclasses.

**Permitted Subclass: PickupTruck**

```java
// PickupTruck.java
public final class PickupTruck extends Truck {
    // PickupTruck-specific properties and methods
}
```

**Permitted Subclass: SemiTruck**

```java
// SemiTruck.java
public final class SemiTruck extends Truck {
    // SemiTruck-specific properties and methods
}
```

**Attempting to Extend Truck Illegally**

```java
// UnauthorizedTruck.java
public class UnauthorizedTruck extends Truck {
    // Compilation error: UnauthorizedTruck is not permitted to extend Truck
}
```

**Explanation**: `UnauthorizedTruck` is not listed in `Truck`'s `permits` clause, so extending `Truck` is prohibited.

#### **1.4.5 Benefits in Pattern Matching**

Using sealed classes enables exhaustive pattern matching.

```java
public String getVehicleType(Vehicle vehicle) {
    return switch (vehicle) {
        case Car c        -> "Car";
        case Truck t      -> "Truck";
        case Motorcycle m -> "Motorcycle";
        // No default case needed as all subclasses are covered
    };
}
```

**Explanation**: The compiler knows all permitted subclasses of `Vehicle`, so it can ensure that all cases are handled.

---

## **1.5 Use Cases and Advantages**

### **1.5.1 Domain Modeling**

Sealed classes are useful when modeling domains with a fixed set of known types.

- **Financial Instruments**: Stocks, Bonds, Futures.
- **AST Nodes**: Different types of expressions in a compiler.

### **1.5.2 API Design**

When designing libraries or APIs, sealed classes allow you to:

- **Control Extensibility**: Prevent external code from extending internal classes in unintended ways.
- **Enhance Security**: Restricting subclassing can prevent certain types of attacks or misuse.

### **1.5.3 Evolution of Class Hierarchies**

Sealed classes can aid in evolving codebases:

- **Gradual Refactoring**: Introduce sealing to lock down parts of the hierarchy.
- **Deprecation Strategies**: Limit the extension of deprecated classes.

---

## **1.6 Important Considerations**

### **1.6.1 Backward Compatibility**

- **Existing Code**: Introducing sealed classes in existing hierarchies can break code that relies on extending those classes.
- **Migration Path**: Use non-sealed classes to maintain compatibility where necessary.

### **1.6.2 Limitations**

- **Cannot Partially Seal**: Once a class is sealed, all direct subclasses must be declared and follow the rules.
- **Cannot Seal Enums**: Enums are implicitly final and cannot be sealed.

### **1.6.3 Sealed Interfaces**

Sealed interfaces follow similar rules:

```java
public sealed interface Shape permits Circle, Rectangle {
    // Interface methods
}
```

- **Permitted Implementations**: Classes or interfaces that implement the sealed interface.

---

## **1.7 Differences Between Java 8 and Java 17**

- **Java 17**: Introduced `sealed` and `non-sealed` modifiers.
- **Java 8**: Does not support sealed classes; access control relies solely on traditional modifiers.

---

## **1.8 Understanding the Difference Between Sealed and Final Classes**

1. **Inheritance Restriction**:
   - **Final Class**: Cannot be subclassed by any class; inheritance is completely prohibited.
   - **Sealed Class**: Can be subclassed, but only by the classes explicitly specified in its `permits` clause.

2. **Control Over Class Hierarchy**:
   - **Final Class**: Provides absolute closure; the class hierarchy ends with the final class.
   - **Sealed Class**: Offers selective extension; only permitted subclasses can extend it, allowing controlled inheritance.

3. **Flexibility**:
   - **Final Class**: No further extension is possible, limiting flexibility.
   - **Sealed Class**: Permitted subclasses can be declared as `final`, `sealed`, or `non-sealed`, enabling a mix of restricted and open inheritance.

4. **Use Cases**:
   - **Final Class**: Used when a class's implementation should remain unchangeable and not be overridden.
   - **Sealed Class**: Used to model restricted hierarchies where a known set of subclasses is allowed, enhancing maintainability and enabling exhaustive pattern matching.

5. **Availability**:
   - **Final Class**: Available since the inception of Java (Java 1.0).
   - **Sealed Class**: Introduced in Java 15 (preview) and standardized in Java 17.

---

## **1.9 Common Traps**

### **Trap 1: Incompatible Modifiers**

**Issue**: Using conflicting modifiers in class declarations.

**Example**:

```java
public abstract final class MyClass {
    // Compilation error
}
```

**Explanation**: A class cannot be both `abstract` and `final` since `abstract` implies it can be subclassed, and `final` prevents subclassing.

### **Trap 2: Sealed Class Misconfigurations**

**Issue**: Misusing `sealed`, `non-sealed`, and `final` modifiers.

**Example**:

```java
public sealed class Vehicle permits Car {
    // Class body
}

public class Car extends Vehicle {
    // Compilation error
}
```

**Solution**: Subclasses of a `sealed` class must explicitly declare their status.

```java
public final class Car extends Vehicle {
    // Class body
}
```

---

## **1.10 References and Further Reading**

- **Official Documentation**: [Java SE 17 Documentation - Sealed Classes](https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html#jls-8.1.7)
- **JEP 409**: [Sealed Classes](https://openjdk.java.net/jeps/409)
- **Tutorials**:
  - [Sealed Classes in Java 17](https://www.baeldung.com/java-sealed-classes-interfaces)
  - [Understanding Sealed Classes](https://www.infoq.com/articles/java-sealed-classes/)
- **Books**:
  - *Java: The Complete Reference* by Herbert Schildt (Updated for Java 17)

---

**Feel free to experiment with sealed and non-sealed classes in your own projects to fully grasp their capabilities and benefits. Happy coding!**