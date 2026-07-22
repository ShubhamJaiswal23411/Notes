#  1. Stream Creation

- `stream()` → Creates a sequential stream from a collection
- `parallelStream()` → Creates a parallel stream
- `Stream.of()` → Creates stream from values
- `Arrays.stream()` → Creates stream from array

---
#  2. Intermediate Operations (lazy, return Stream)
## 1. Filtering
- `filter(Predicate)` → Keeps elements matching condition
## 2. Mapping / Transformation
- `map(Function)` → Transforms each element
- `mapToInt / mapToLong / mapToDouble` → Convert to primitive streams
- `flatMap(Function)` → Flattens nested structures
## 3. Sorting
- `sorted()` → Natural order sort
- `sorted(Comparator)` → Custom sorting
## 4. Distinct / Limiting
- `distinct()` → Removes duplicates
- `limit(n)` → Takes first `n` elements
- `skip(n)` → Skips first `n` elements
## 5. Peek (debugging)
- `peek(Consumer)` → Performs action without modifying stream (mainly debugging)

---
#  3. Terminal Operations (produce result)

##  1. Iteration
- `forEach(Consumer)` → Iterates over elements
- `forEachOrdered()` → Maintains order in parallel streams
##  2. Reduction
- `reduce(BinaryOperator)` → Combines elements into one result
- `reduce(identity, accumulator)` → With initial value
##  3. Collection
- `collect(Collector)` → Converts stream to collection/result

Common collectors from `Collectors`:
- `toList()` → Collect to List
- `toSet()` → Collect to Set
- `toMap()` → Convert to Map
- `joining()` → Join strings
- `groupingBy()` → Group elements
- `partitioningBy()` → Split into true/false groups
- `counting()` → Count elements
- `summarizingInt()` → Stats (count, sum, avg, min, max)
##  4. Matching
- `anyMatch(Predicate)` → Any element matches?
- `allMatch(Predicate)` → All elements match?
- `noneMatch(Predicate)` → No element matches?
##  5. Finding
- `findFirst()` → First element
- `findAny()` → Any element (useful in parallel streams)
## 6. Counting
- `count()` → Number of elements
## 7. Min / Max
- `min(Comparator)` → Smallest element
- `max(Comparator)` → Largest element

---
#  4. Primitive Stream Specific
- `sum()` → Sum of elements
- `average()` → Average
- `min()` / `max()` → Min/Max
- `range()` / `rangeClosed()` → Generate number ranges

---
#  5. Short-Circuit Operations
- `limit()` → Stops after `n` elements
- `findFirst()` → Stops early when found
- `anyMatch()` → Stops when condition satisfied

---
#  6. Parallel Stream Helpers
- `parallel()` → Converts stream to parallel
- `sequential()` → Converts back to sequential

---
# 🧠 Key Concepts (important for interviews)

### 1. Lazy execution
Intermediate operations don’t run until terminal operation is called.

###  2. Pipeline

```
source → intermediate ops → terminal op
```

### 3. Stateless vs Stateful
- Stateless → `map`, `filter`
- Stateful → `sorted`, `distinct`

