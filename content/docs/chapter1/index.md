---
title: 'Java Building Blocks Deep Dive'
date: 2024-11-25T21:16:39+10:00
draft: false
weight: 2
summary: Traps, Key Concepts, and Evolution from Java 8 to Java 17 [READ THE ARTICLE](https://kitanob.github.io/OCP17-NotesAndResources/docs/chapter1/)
---

# **Java Building Blocks Deep Dive: Traps, Key Concepts, and Evolution from Java 8 to Java 17**

## **1. Class Structure and Components**

### **1.1 Class Declarations and Modifiers**

#### **Key Concepts**

- **Access Modifiers**: `public`, `protected`, `private`, and default (package-private).
- **Non-Access Modifiers**: `abstract`, `final`, `static`, `strictfp`, `sealed`, `non-sealed`.

#### **Understanding Sealed and Non-Sealed Classes in Java**

Java 17 introduced **sealed classes** and **interfaces** as a way to provide more control over inheritance and to model restricted class hierarchies. This feature allows developers to explicitly specify which classes or interfaces are **permitted to extend or implement a sealed class or interface**. Additionally, the concept of **non-sealed** classes complements sealed classes by allowing subclasses to "open up" the hierarchy again.

##### **1.1.1 Definition of Sealed Classes**

A **sealed class** is a class that restricts/avoid which other classes or interfaces can extend or implement it. This is achieved by declaring a class with the `sealed` modifier and specifying the permitted subclasses using the `permits` clause. Sealed classes help in:

- **Restricting Extensibility**: They limit the classes that can inherit from them, providing better control over the class hierarchy.
- **Enhancing Maintainability**: By knowing all subclasses, developers can reason about the code more effectively.
- **Exhaustive Pattern Matching**: They enable the compiler to perform exhaustive checks when using pattern matching with sealed classes.

```java
public sealed class Forme permits Cercle, Rectangle {
    // code de la classe
}
```

##### **1.1.2 Definition of Non-Sealed Classes**

A **non-sealed class** is a subclass of a sealed class that lifts the sealing restriction, allowing further subclasses to extend it without any limitations. By declaring a class as `non-sealed`, you essentially "open up" the class hierarchy from that point onward.

```java
public non-sealed class Rectangle extends Forme {
    // code de la classe
}
```

---

#### **1.2. Syntax and Rules**

##### **1.2.1 Declaring a Sealed Class**

```java
public sealed class Vehicle permits Car, Truck, Motorcycle {
    // Class body
}
```

- **Modifiers**: The class is declared with the `sealed` keyword.
- **Permits Clause**: Specifies the allowed subclasses (`Car`, `Truck`, `Motorcycle`).

##### **1.2.2 Declaring Permitted Subclasses**

Each class specified in the `permits` clause must directly extend the sealed class and must explicitly declare its own status:

- **Final**: Cannot be extended further.
- **Sealed**: Continues the sealing with its own set of permitted subclasses.
- **Non-Sealed**: Opens up the hierarchy, allowing unrestricted subclassing.

**Examples**

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

##### **1.2.3 Rules and Constraints**

- **Direct Extension**: Permitted subclasses must directly extend the sealed class.
- **Modifiers in Subclasses**: Permitted subclasses must explicitly declare one of `final`, `sealed`, or `non-sealed`.
- **All Subclasses Known**: The `permits` clause must list all permitted subclasses at compile-time.

---

#### **1.3. Benefits of Sealed and Non-Sealed Classes**

##### **1.3.1 Enhanced Control Over Inheritance**

By sealing a class, you have explicit control over which classes can extend it. This is beneficial when you want to:

- **Enforce Invariants**: Ensure that only known classes can modify or extend certain behaviors.
- **Prevent Unauthorized Extension**: Avoid unintended use or misuse of your classes by external code.

##### **1.3.2 Improved Code Safety and Maintainability**

- **Exhaustive Pattern Matching**: When using sealed classes in pattern matching (e.g., in `switch` expressions), the compiler knows all possible subclasses, allowing for exhaustive checks and reducing the likelihood of runtime errors.
- **Clear Hierarchies**: Makes the class hierarchy more predictable and easier to understand.

##### **1.3.3 Flexibility with Non-Sealed Classes**

- **Selective Unsealing**: Non-sealed subclasses allow parts of your hierarchy to be open for extension, providing a balance between control and flexibility.
- **Gradual Migration**: Useful when transitioning from an open hierarchy to a sealed one without breaking existing code.

---

#### **1.4. Practical Examples**

**Example 1: Vehicle Hierarchy**

**Objective**: Model a vehicle hierarchy where only specific types of vehicles are allowed.

##### **1.4.1 Defining the Sealed Class**

```java
// Vehicle.java
public sealed class Vehicle permits Car, Truck, Motorcycle {
    // Common properties and methods
}
```

##### **1.4.2 Permitted Subclasses**

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

##### **1.4.3 Extending Non-Sealed Class**

Since `Motorcycle` is non-sealed, it can be extended without restrictions.

```java
// SportBike.java
public class SportBike extends Motorcycle {
    // SportBike-specific properties and methods
}
```

##### **1.4.4 Extending Sealed Subclass**

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

##### **1.4.5 Benefits in Pattern Matching**

Using sealed classes enables exhaustive pattern matching.

```java
public String getVehicleType(Vehicle vehicle) {
    return switch (vehicle) {
        case Car c -> "Car";
        case Truck t -> "Truck";
        case Motorcycle m -> "Motorcycle";
        // No default case needed as all subclasses are covered
    };
}
```

**Explanation**: The compiler knows all permitted subclasses of `Vehicle`, so it can ensure that all cases are handled.

### **Example 2: Expression Evaluation**

**Objective**: Model an expression tree for mathematical computations.

##### **1.4.6 Defining the Sealed Interface**

```java
// Expr.java
public sealed interface Expr permits Constant, Add, Multiply {
    double evaluate();
}
```

##### **1.4.7 Implementing Permitted Subclasses**

**Final Class: Constant**

```java
// Constant.java
public final class Constant implements Expr {
    private final double value;

    public Constant(double value) {
        this.value = value;
    }

    @Override
    public double evaluate() {
        return value;
    }
}
```

**Sealed Class: Binary Operation**

```java
// BinaryOperation.java
public sealed abstract class BinaryOperation implements Expr permits Add, Multiply {
    protected final Expr left;
    protected final Expr right;

    protected BinaryOperation(Expr left, Expr right) {
        this.left = left;
        this.right = right;
    }
}
```

**Final Class: Add**

```java
// Add.java
public final class Add extends BinaryOperation {
    public Add(Expr left, Expr right) {
        super(left, right);
    }

    @Override
    public double evaluate() {
        return left.evaluate() + right.evaluate();
    }
}
```

**Final Class: Multiply**

```java
// Multiply.java
public final class Multiply extends BinaryOperation {
    public Multiply(Expr left, Expr right) {
        super(left, right);
    }

    @Override
    public double evaluate() {
        return left.evaluate() * right.evaluate();
    }
}
```

##### **1.4.8 Evaluating Expressions**

```java
public class Calculator {
    public static void main(String[] args) {
        Expr expr = new Add(
            new Constant(5),
            new Multiply(
                new Constant(2),
                new Constant(3)
            )
        );
        System.out.println(expr.evaluate()); // Outputs 11.0
    }
}
```

##### **1.4.9 Benefits**

- **Type Safety**: By sealing `Expr`, we ensure that only known expression types are used.
- **Pattern Matching**

```java
public double evaluateExpression(Expr expr) {
    return switch (expr) {
        case Constant c -> c.evaluate();
        case Add a -> a.evaluate();
        case Multiply m -> m.evaluate();
    };
}
```

**Explanation**: The compiler can verify that all possible implementations of `Expr` are handled.

---

#### **1.5. Use Cases and Advantages**

##### **1.5.1 Domain Modeling**

Sealed classes are useful when modeling domains with a fixed set of known types.

- **Financial Instruments**: Stocks, Bonds, Futures.
- **AST Nodes**: Different types of expressions in a compiler.

##### **1.5.2 API Design**

When designing libraries or APIs, sealed classes allow you to:

- **Control Extensibility**: Prevent external code from extending internal classes in unintended ways.
- **Enhance Security**: Restricting subclassing can prevent certain types of attacks or misuse.

##### **1.5.3 Evolution of Class Hierarchies**

Sealed classes can aid in evolving codebases:

- **Gradual Refactoring**: Introduce sealing to lock down parts of the hierarchy.
- **Deprecation Strategies**: Limit the extension of deprecated classes.

---

##### **1.6. Important Considerations**

##### **1.6.1 Backward Compatibility**

- **Existing Code**: Introducing sealed classes in existing hierarchies can break code that relies on extending those classes.
- **Migration Path**: Use non-sealed classes to maintain compatibility where necessary.

##### **1.6.2 Limitations**

- **Cannot Partially Seal**: Once a class is sealed, all direct subclasses must be declared and follow the rules.
- **Cannot Seal Enums**: Enums are implicitly final and cannot be sealed.

##### **1.6.3 Sealed Interfaces**

Sealed interfaces follow similar rules:

```java
public sealed interface Shape permits Circle, Rectangle {
    // Interface methods
}
```

- **Permitted Implementations**: Classes or interfaces that implement the sealed interface.


---

#### **1.7 References and Further Reading**

- **Official Documentation**: [Java SE 17 Documentation - Sealed Classes](https://docs.oracle.com/javase/specs/jls/se17/html/jls-8.html#jls-8.1.7)
- **JEP 409**: [Sealed Classes](https://openjdk.java.net/jeps/409)
- **Tutorials**:
  - [Sealed Classes in Java 17](https://www.baeldung.com/java-sealed-classes-interfaces)
  - [Understanding Sealed Classes](https://www.infoq.com/articles/java-sealed-classes/)
- **Books**:
  - *Java: The Complete Reference* by Herbert Schildt (Updated for Java 17)

---

**Feel free to experiment with sealed and non-sealed classes in your own projects to fully grasp their capabilities and benefits. Happy coding!**


#### **Common Traps**

##### **Trap 1: Incompatible Modifiers**

**Issue**: Using conflicting modifiers in class declarations.

**Example**:

```java
public abstract final class MyClass { /*...*/ } // Compilation error
```

**Explanation**: A class cannot be both `abstract` and `final` since `abstract` implies it can be subclassed, and `final` prevents subclassing.

##### **Trap 2: Sealed Class Misconfigurations (Java 17)**

**Issue**: Misusing `sealed`, `non-sealed`, and `final` modifiers.

**Example**:

```java
public sealed class Vehicle permits Car { /*...*/ }

public class Car extends Vehicle { /*...*/ } // Compilation error
```

**Solution**: Subclasses of a `sealed` class must explicitly declare their status.

```java
public final class Car extends Vehicle { /*...*/ }
```

#### **Differences Between Java 8 and Java 17**

- **Java 17**: Introduced `sealed` and `non-sealed` modifiers.
- **Java 8**: Does not support `sealed` classes; access control relies solely on traditional modifiers.

---

### **1.2 Members of a Class**

#### **Key Concepts**

- **Fields**: Variables declared within a class.
- **Methods**: Functions associated with a class.
- **Constructors**: Special methods to initialize new objects.
- **Nested Classes**: Classes defined within another class.

#### **Common Traps**

##### **Trap 3: Initialization Order**

**Issue**: Misunderstanding the order in which static blocks, instance initializers, and constructors are executed.

**Example**:

```java
public class InitializationExample {
    static {
        System.out.println("Static Block");
    }

    {
        System.out.println("Instance Initializer");
    }

    public InitializationExample() {
        System.out.println("Constructor");
    }

    public static void main(String[] args) {
        new InitializationExample();
    }
}
```

**Output**:

```
Static Block
Instance Initializer
Constructor
```

**Explanation**:

- Static blocks execute once when the class is loaded.
- Instance initializers run before constructors for each object creation.

##### **Trap 4: Accessing Static Context from Instance Context**

**Issue**: Confusion between static and instance contexts.

**Example**:

```java
public class Example {
    int instanceVar = 10;
    static int staticVar = 20;

    public void instanceMethod() {
        System.out.println(staticVar); // Allowed
    }

    public static void staticMethod() {
        System.out.println(instanceVar); // Compilation error
    }
}
```

**Explanation**: Static methods cannot access instance variables directly.

#### **Differences Between Java 8 and Java 17**

- **No significant differences** in the basic structure of class members.
- **Java 16**: Introduced records, which are special kinds of classes.

---

## **2. Records and Data Classes**

### **Key Concepts**

- **Records**: Immutable data classes that automatically generate boilerplate code like `equals()`, `hashCode()`, and `toString()`.

**Example**:

```java
public record Point(int x, int y) { }
```

#### **Common Traps**

##### **Trap 5: Customizing Record Components**

**Issue**: Misunderstanding how to add validation or custom behavior in records.

**Solution**: Use compact constructors.

```java
public record Point(int x, int y) {
    public Point {
        if (x < 0 || y < 0) {
            throw new IllegalArgumentException("Coordinates must be positive");
        }
    }
}
```

##### **Trap 6: Inheritance Limitations with Records**

**Issue**: Records implicitly extend `java.lang.Record` and cannot extend other classes.

**Explanation**: Records are final and cannot be subclassed; they can implement interfaces.

#### **Differences Between Java 8 and Java 17**

- **Java 8**: No records; developers manually create data classes with boilerplate code.
- **Java 17**: Records simplify data class creation and enforce immutability.

---

## **3. Primitive Types and Wrapper Classes**

### **Key Concepts**

- **Primitive Types**: `byte`, `short`, `int`, `long`, `float`, `double`, `char`, `boolean`.
- **Wrapper Classes**: Object representations of primitives (`Integer`, `Double`, etc.).

#### **Common Traps**

##### **Trap 7: Autoboxing and Unboxing Pitfalls**

**Issue**: Unintended behavior due to autoboxing and unboxing.

**Example**:

```java
Integer a = 128;
Integer b = 128;
System.out.println(a == b); // Outputs false

Integer c = 127;
Integer d = 127;
System.out.println(c == d); // Outputs true
```

**Explanation**:

- Integer caching for values between -128 and 127.
- `==` compares object references; use `.equals()` for value comparison.

##### **Trap 8: Null Pointers with Wrapper Types**

**Issue**: Unboxing null values leads to `NullPointerException`.

**Example**:

```java
Integer num = null;
int value = num; // Throws NullPointerException
```

**Solution**: Check for null before unboxing.

#### **Differences Between Java 8 and Java 17**

- **Java 17**: May have performance optimizations, but no changes in autoboxing behavior.
- **Project Valhalla**: Aims to introduce value types but is not part of Java 17.

---

## **4. Variable Scope and Shadowing**

### **Key Concepts**

- **Scope**: The region of code where a variable is accessible.
- **Shadowing**: A variable declared in an inner scope with the same name as an outer scope variable.

#### **Common Traps**

##### **Trap 9: Variable Shadowing Leading to Confusion**

**Example**:

```java
public class ShadowExample {
    static int x = 10;

    public static void main(String[] args) {
        int x = 20;
        System.out.println(x); // Outputs 20
    }
}
```

**Explanation**: The local variable `x` shadows the static variable `x`.

#### **Differences Between Java 8 and Java 17**

- **No differences** in variable scope rules.
- **Java 10**: Introduced `var` for local variable type inference, affecting how variables are declared.

---

## **5. Flow Control Constructs**

### **Key Concepts**

- **If-Else Statements**
- **Switch Statements**
- **Loops**: `for`, `while`, `do-while`, enhanced `for-each`

#### **Common Traps**

##### **Trap 10: Switch Expressions and Enhanced Switch (Java 14+)**

**Issue**: Misusing switch expressions.

**Example**:

```java
int result = switch (day) {
    case MONDAY -> 1;
    case TUESDAY -> 2;
    default -> {
        yield 0;
    }
};
```

**Trap**: Forgetting that switch expressions can return values and using `break` instead of `yield`.

#### **Trap 11: Fall-Through Behavior**

**Issue**: Unintentional fall-through in switch statements.

**Example**:

```java
switch (value) {
    case 1:
        // Code for case 1
    case 2:
        // Code for case 2
        break;
}
```

**Explanation**: Without a `break`, execution falls through to the next case.

#### **Differences Between Java 8 and Java 17**

- **Java 14+**: Introduced switch expressions and enhanced case labels.
- **Pattern Matching in Switch**: Available in preview in Java 17.

---

## **6. Pattern Matching for `instanceof`**

### **Key Concepts**

- Simplifies type casting by introducing a binding variable within the `instanceof` check.

**Example**:

```java
if (obj instanceof String s) {
    System.out.println(s.toUpperCase());
}
```

#### **Common Traps**

##### **Trap 12: Scope of Pattern Variable**

**Issue**: Assuming the pattern variable is accessible outside the conditional block.

**Example**:

```java
if (!(obj instanceof String s)) {
    // s is not accessible here
}
```

**Solution**: Pattern variable `s` is only accessible when the `instanceof` condition is true.

#### **Differences Between Java 8 and Java 17**

- **Java 8**: Requires explicit casting after `instanceof`.

---

## **7. Arrays and Varargs**

### **Key Concepts**

- **Arrays**: Fixed-size data structures storing elements of the same type.
- **Varargs**: Allows methods to accept variable numbers of arguments.

#### **Common Traps**

##### **Trap 13: Array Covariance Leading to Runtime Exceptions**

**Issue**: Assigning arrays of subclass types to arrays of superclass types.

**Example**:

```java
Number[] numbers = new Integer[2];
numbers[0] = 1.5; // Throws ArrayStoreException at runtime
```

**Explanation**: The array `numbers` is actually an `Integer[]`, so assigning a `Double` causes an exception.

##### **Trap 14: Varargs Ambiguity**

**Issue**: Method overloading confusion with varargs.

**Example**:

```java
public void print(int... nums) { /*...*/ }
public void print(int num1, int num2) { /*...*/ }

print(1, 2); // Ambiguous method call
```

**Solution**: Avoid overloading methods in a way that causes ambiguity with varargs.

#### **Differences Between Java 8 and Java 17**

- **No significant differences** in array or varargs behavior.

---

## **8. Enums and Advanced Usage**

### **Key Concepts**

- **Enums**: Special classes representing a group of constants.
- **Methods in Enums**: Enums can have fields, constructors, and methods.

#### **Common Traps**

##### **Trap 15: Enum Constructor Access**

**Issue**: Enum constructors are always `private`, even if declared `public`.

**Example**:

```java
public enum Day {
    MONDAY("Start of work week"),
    TUESDAY("Second day");

    private String description;

    Day(String description) {
        this.description = description;
    }
}
```

**Explanation**: Cannot instantiate enums using `new` outside of the enum declaration.

##### **Trap 16: Extending Enums**

**Issue**: Enums cannot be subclassed.

**Solution**: Use interfaces if polymorphic behavior is needed.

#### **Differences Between Java 8 and Java 17**

- **No changes** in enum behavior between Java 8 and Java 17.

---

## **9. Exception Handling Nuances**

### **Key Concepts**

- **Checked vs. Unchecked Exceptions**
- **Try-With-Resources**: Simplifies resource management.

#### **Common Traps**

##### **Trap 17: Suppressed Exceptions in Try-With-Resources**

**Issue**: Ignoring suppressed exceptions when multiple exceptions occur.

**Example**:

```java
try (MyResource res = new MyResource()) {
    throw new Exception("Primary Exception");
} catch (Exception e) {
    for (Throwable suppressed : e.getSuppressed()) {
        System.out.println(suppressed.getMessage());
    }
}
```

**Explanation**: Exceptions thrown during resource closing are suppressed but can be retrieved.

#### **Differences Between Java 8 and Java 17**

- **Java 9+**: Enhanced try-with-resources to allow final or effectively final variables.

**Example**:

```java
MyResource res = new MyResource();
try (res) { // Valid in Java 9+, invalid in Java 8
    // Use res
}
```

---

## **10. Text Blocks and String Enhancements**

### **Key Concepts**

- **Text Blocks**: Multi-line string literals introduced in Java 15.

**Example**:

```java
String json = """
    {
        "name": "John",
        "age": 30
    }
    """;
```

#### **Common Traps**

##### **Trap 18: Misaligned Indentation**

**Issue**: Inconsistent indentation affects the resulting string.

**Solution**: Align closing `"""` with the text block content or use methods like `stripIndent()`.

#### **Differences Between Java 8 and Java 17**

- **Java 8**: No text blocks; multi-line strings require concatenation or escape sequences.

---

## **Final Tips for Experts**

- **Stay Updated**: Regularly review the Java Enhancement Proposals (JEPs) for insights into language changes.
- **Deep Dive into JLS**: Consult the Java Language Specification for authoritative information on language constructs.
- **Experiment with New Features**: Use the latest JDK to practice new features like records, sealed classes, and pattern matching.
- **Understand Underlying Mechanisms**: For example, know how the JVM handles class loading, memory management, and bytecode generation for new language features.
