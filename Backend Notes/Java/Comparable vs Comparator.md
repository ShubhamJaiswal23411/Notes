
# 1️. Why Do We Need Them?

When sorting objects in Java (like custom classes), Java needs to know:
> “On what basis should these objects be compared?”

For primitives → natural ordering exists. 
For custom objects → we must define ordering logic.
Java provides two ways:
- `Comparable`
- `Comparator`

---

# 2️. Comparable Interface

`Comparable` is an interface used to define the **natural ordering** of objects.
It is present in:
```java
java.lang
```

---
##  Method

```java
int compareTo(T o);
```

Returns:
- Negative → current < other
- Zero → equal
- Positive → current > other

---
##  Example

```java
class Employee implements Comparable<Employee> {

    int id;

    Employee(int id) {
        this.id = id;
    }

    @Override
    public int compareTo(Employee e) {
        return this.id - e.id;
    }
}
```

Now:
```java
Collections.sort(employeeList);
```

Sorting works automatically.

---
##  Key Characteristics
- Sorting logic defined **inside the class**
- Only **one natural ordering**
- Used by:
    - `Collections.sort()`
    - `Arrays.sort()`
    - `TreeSet`
    - `TreeMap`

---

##  Real Examples
- `String`
- `Integer`
- `LocalDate`

All implement `Comparable`.

---
# 3️. Comparator Interface

##  What It Is
`Comparator` defines **custom ordering** outside the class.

Located in:
```java
java.util
```

---
##  Method
```java
int compare(T o1, T o2);
```

---
##  Example
```java
class Employee {
    int id;
    String name;
}
```

Sort by name:
```java
Comparator<Employee> nameComparator =
    (e1, e2) -> e1.name.compareTo(e2.name);

Collections.sort(list, nameComparator);
```

---
##  Multiple Sorting Possible
```java
Comparator<Employee> idComparator =
    Comparator.comparingInt(e -> e.id);

Comparator<Employee> nameComparator =
    Comparator.comparing(e -> e.name);
```

You can switch sorting logic easily.

---

# 4️. Java 8 Enhancements in Comparator

Java 8 added:
```java
Comparator.comparing()
Comparator.comparingInt()
thenComparing()
reversed()
nullsFirst()
nullsLast()
```

Example:
```java
list.sort(
    Comparator.comparing(Employee::getName)
              .thenComparing(Employee::getId)
);
```

Very powerful and readable.

---

# 5️. Comparable vs Comparator — Core Differences

| Feature                | Comparable                  | Comparator        |
| ---------------------- | --------------------------- | ----------------- |
| Package                | `java.lang`                 | `java.util`       |
| Method                 | `compareTo()`               | `compare()`       |
| Sorting Logic Location | Inside class                | Outside class     |
| Number of Sortings     | Only one (natural)          | Multiple possible |
| Modifies Class?        | Yes                         | No                |
| Lambda Support         | No                          | Yes (Java 8+)     |
| Used In                | Natural ordering structures | Custom sorting    |

---

# 6️. When to Use What?

### Use Comparable When:
- Class has clear natural ordering
- You control the class code
- Only one main sorting logic needed

Example:
- Student sorted by roll number

---

### Use Comparator When:
- Multiple sorting options required
- You cannot modify class (3rd party class)
- Need dynamic sorting

Example:
- Sort Employee by name, salary, department

---

# 7️. Important Contract Rules

For both:
- Must be **consistent with equals**
- Should be:
    - Transitive
    - Symmetric
    - Consistent

Otherwise:
- `TreeSet` / `TreeMap` may behave incorrectly

---

# 8️. Tricky Interview Questions

---

###  1. Can a class implement both Comparable and Comparator?
Yes, but generally not recommended.

###  2. What happens if compareTo returns inconsistent results?
Sorted collections behave unpredictably.

###  3. What if compareTo returns 0 for two unequal objects?
`TreeSet` will treat them as duplicates.
###  4. Is Comparator functional interface?
Yes (since Java 8).
###  5. Which one is better?
Neither. Depends on requirement.

