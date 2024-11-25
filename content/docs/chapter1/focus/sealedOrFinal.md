---
title: 'Difference Between Sealed Class and Final Class'
date: 2024-11-25T21:16:39+10:00
draft: false
weight: 1
summary: Why using final over sealed class [READ THE ARTICLE](https://kitanob.github.io/OCP17-NotesAndResources/docs/chapter1/sealedOrFinal)
---

**Difference Between Sealed Class and Final Class:**

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