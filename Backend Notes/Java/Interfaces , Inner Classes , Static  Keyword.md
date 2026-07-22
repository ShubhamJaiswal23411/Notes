#  STATIC Keyword (Method, Variable, Block, Class)

### 1. Can a static method access non-static variables directly?
**Answer:** No.  
**Why?** Static methods belong to the class, not to an instance. Non-static members require an object reference.

---
### 2. Can a non-static method access static members?
**Answer:** Yes.  
**Why?** Static members are class-level and accessible via class name or directly.

---
### 3. Can we override a static method?
**Answer:** No (it is method hiding, not overriding).  
**Why?** Static methods are resolved at compile time (static binding).

---
### 4. What happens if a static method is called using an object reference?
**Answer:** It compiles but is resolved using the reference type, not object type.
```java
Parent p = new Child();
p.staticMethod(); // calls Parent's version
```

---
### 5. Can constructors be static?
**Answer:** No.  
**Why?** Constructors initialize instances.

---
### 6. Can a static block access non-static variables?
**Answer:** No, unless via object creation.

---
### 7. When are static blocks executed?
**Answer:** When the class is loaded into memory (only once).

---
### ***8. What is the execution order?***
1. Static variables
2. Static blocks
3. Instance variables
4. Instance blocks
5. Constructor

---
### *9. Can we have multiple static blocks?*
**Answer:** Yes. Executed in order of appearance.

---
### *10. What if a static variable depends on another static variable declared below it?*
**Answer:** Default values may be used if accessed before initialization (order matters).

---
### 11. Can a static class be top-level?
**Answer:** No. Only nested classes can be static.

---
### 12. Why can't top-level classes be static?
Because they are already at class level.

---
### *13. What is a static nested class?*
A nested class without reference to outer class instance.

---
### *14. Can static methods be abstract?*
No.

---
### 15. Can static methods be synchronized?
Yes.

---
### 16. Are static variables shared?
Yes, across all instances.

---
### *17. Is static variable thread-safe?*
No, unless synchronized.

---
### *18. Can static methods be final?*
Yes.
Because:
###  `static` does NOT prevent redefinition (hiding)
Without `final`, a subclass can still do this:
```java
class A {  
    static void foo() {  
        System.out.println("A");  
    }  
}  
  
class B extends A {  
    static void foo() {  
        System.out.println("B");  
    }  
}
```

This compiles fine. But:

```java
A obj = new B();  
obj.foo(); // prints "A" so call is resovled based on reference variable no polymorphism here.
```
This creates **confusion**:

- Looks like overriding
- Behaves differently (no runtime polymorphism)
---
### 19. What happens if we put `this` inside static method?
Compile-time error.

---
### 20. Can we access static members without object creation?
Yes, using ClassName.member.

---
#  INTERFACES – Tricky Questions

---
### 21. Do interfaces have constructors?
No.  
Because they cannot be instantiated.

---
### 22. Can an interface have a main method?
Yes (since Java 8).  
It can run independently.

---
### 23. Can interface variables be non-static?
No. All are implicitly `public static final`.

---
### 24. Can we change interface variable value?
No, they are constants.

---
### 25. Can interfaces have static methods?
Yes (Java 8+).  
Must be called using InterfaceName.method().

---
### 26. Can static methods in interface be overridden?
No.

---
### 27. Can interfaces have default methods?
Yes (Java 8+).

---
### 28. How to call default method?
Through implementing class instance.

---
### *29. Can a class override default method?*
Yes.

---
### *30. What if two interfaces have same default method?*
Class must override it (diamond problem resolution).

```java
interface A { default void show() {} }
interface B { default void show() {} }

class C implements A, B {
    public void show() {
        A.super.show(); // choose explicitly
    }
}
```

---
### 31. What if one interface has default and other abstract?
Class must implement the method.

---
### 32. Can an interface extend multiple interfaces?
Yes.

---
### *33. Can an interface implement another interface?*
No. It extends.

---
### 34. Can interface have private methods?
Yes (Java 9+).

---
### 35. Why are interface methods public by default?
To allow implementation visibility.

---
### 36. Can an interface be final?
No.

---
### 37. Can we instantiate interface using anonymous class?
No , interfaces and abstract classes cant be instantiated.

---
### ***38. Can interface contain static block?***
No. Static blocks are used to initialize stuff but all the variables in an interface are already final so static blocks would serve no purpose.

---
### 39. What is marker interface?
Interface with no methods (e.g., `Serializable` , `Cloneable`).

---
### 40. If child class doesn’t implement interface method?
Class must be abstract.

---
#  INNER CLASSES – Deep Tricky Questions

---
### 41. Types of inner classes?
1. Member inner class
2. Static nested class
3. Local inner class
4. Anonymous inner class
## 1️. Member Inner Class (non-static)

Defined at class level, tied to an instance.
```java
class Outer {
    int x = 10;

    class Inner {
        void show() {
            System.out.println(x); // can access instance members
        }
    }
}

// Usage
Outer o = new Outer();
Outer.Inner i = o.new Inner();
i.show();
```
## 2️. Static Nested Class

Belongs to class, no access to instance members directly.
```java
class Outer {
    static int x = 20;

    static class Inner {
        void show() {
            System.out.println(x); // can access only static members
        }
    }
}

// Usage
Outer.Inner i = new Outer.Inner();
i.show();
```

## 3️. Local Inner Class

Defined inside a method.
```java
class Outer {
    void display() {
        int y = 30; // must be effectively final

        class Inner {
            void show() {
                System.out.println(y);
            }
        }

        new Inner().show();
    }
}
```

## 4️. Anonymous Inner Class

No class name, used for one-time implementation.
```java
abstract class Animal {
    abstract void sound();
}

public class Test {
    public static void main(String[] args) {
        Animal a = new Animal() {
            void sound() {
                System.out.println("Bark");
            }
        };

        a.sound();
    }
}
```

---
### 42. Can inner class be private?
Yes (member inner class).

---
### *43. Can inner class have static members?*
Only if static final constants.

---
### *44. Can static nested class access outer non-static members?*
No, not directly only via object reference.

---
### 45. Can non-static inner class access outer private members?
Yes.

---
### *46. How to instantiate non-static inner class?*
```java
Outer o = new Outer();
Outer.Inner i = o.new Inner();
```

---
### *47. How to instantiate static nested class?*
```java
Outer.Inner i = new Outer.Inner();
```

---
### *48. Does non-static inner class have reference to outer object?*
Yes (hidden reference Outer.this).

---
### *49. Can outer class access inner class private members?*
Yes.

---
### *50. Can inner class extend outer class?*
Yes, but tricky and rarely useful.

---
### ***51. Can inner class define static block?***
No (unless static nested class).

---
### *52. What happens if local variable used inside local inner class?*
It must be effectively final.

---
### ***53. Why effectively final?***
Because compiler copies it.

---
### 54. Can we declare constructor inside anonymous class?
No (it has no name).

---
### ***55. Can local inner class have access modifiers?***
No (cannot be public/private/protected). Inner local classes are classes defined inside of a method and inside of a method access modifiers dont make sense.

---
### 56. Can static nested class access outer static members?
Yes. 

---
### 57. Memory difference between static and non-static inner class?
Non-static holds reference to outer instance → more memory.

---
### *58. Can we declare interface inside class?*
Yes (implicitly static).

---
### *59. Can we declare class inside interface?*
Yes (implicitly public static).

---
### ***60. Which modifiers allowed for member inner class?***
public, private, protected, abstract, final, static (if nested).

---

# 🔥 SUPER TRICKY RAPID-FIRE

61. Can static method call instance method indirectly?  
    → only through object reference.    
62. Does static block run if class has no main?  
    → Yes, if class is loaded.
63. Can interface default method be final?  
    → No. because in case of multiple inheritance we would have to override them.
64. Can inner class have same name as outer class method?  
    → Yes.
65. Which is loaded first: parent static block or child?  
    → Parent first.
66. Can we declare enum inside class?  
    → Yes (implicitly static).
