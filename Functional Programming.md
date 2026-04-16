# Functional programming

Functional programming gives a substantial advantage to Java programs. Code written in such a manner is concise,
more expressive, with fewer moving parts, is easier to parallelize and is generally easier to
understand than OO code. There is a challenge to change the way of thinking from imperative to
declarative programming style - which pays off quickly.

Some of Java's tools for functional programming are *lambda expressions*, *streams*, *functional
interfaces*, *records*, and *pattern matching* which will be explained in the rest of the chapter.

## Lambda expression

Lambda expressions are shorter representations of anonymous classes with the following characteristics:
* **Anonymous** - doesn't have an explicit name
* **Function** - not tied to a particular class like a method
* **Passed around** - can be passed as argument or stored in variable
* **Concise** - no need to write a lot of boilerplate like in anonymous classes

Here is an example of writing an anonymous class in a more concise and shorter way using lambdas:

Before:

```java
Comparator<Apple> byWeight = new Comparator<Apple>() { 
    public int compare(Apple a1, Apple a2){
        return a1.getWeight().compareTo(a2.getWeight()); 
    }
};
```

After (with lambda expressions):

```java
Comparator<Apple> byWeight = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
```

## Functional interface

Any interface with a SAM (Single Abstract Method) is a functional interface.
An implementation of that method can be treated as a lambda expression.
Here are some functional interfaces which are most commonly used with their characteristics:

| | Predicate<T\>| Consumer<T\>| Supplier<T\>| Comparator<T\>| Function<T,R>|
|:-------------:|:-------------:|:-----:|:-----:|:-----:|:-----:|
|Typical use-case| filter collection of values|perform action on each element| provide results | compare collection elements for sorting purpose | map collection elements to get essential processing data|
|Method| **test(T t)**| **accept(T t)**| **get( )**| **compare(T t1, T t2)**| **apply(T t)**|
|Stream operation| filter, allMatch, anyMatch| forEach, peek| generate|max, min, sorted |map, flatMap|
|Output type|**boolean**|**void**|**T**|**int**| **R**|

There are also functional interfaces which accept two arguments like `BiConsumer`, `BiFunction`, `BiPredicate`, `BinaryOperator` etc.

## Stream

Streams are used for collections manipulation in declarative way. Manipulations are done by chaining
intermediate operations and executing one of terminating operations which starts collection processing.
Once consumed, stream can not be reused. Check [Java Stream API][1] for more details.

|Intermediate operations|Terminal operations|
|:-:|:-:|
|filter, map, sorted| reduce, collect, forEach, anyMatch|

```java
List<String> myList = Arrays.asList("a1", "a2", "b1", "c2", "c1");

myList.stream()
    .filter(s -> s.startsWith("c"))
    .map(String::toUpperCase)
    .sorted()
    .forEach(System.out::println);

// C1
// C2
```

Another way of creating `Stream` objects is by using static `Stream.of()` method.

```java
Stream.of("a1", "a2", "a3")
    .findFirst()
    .ifPresent(System.out::println);
    
// a1
```

This example contains the `ifPresent()` function which will be explained in *Optional* sub-chapter.

There is also an option of creating stream of numbers:

```java
Stream.of(1.0, 2.0, 3.0)
    .mapToInt(Double::intValue)
    .mapToObj(i -> "a" + i)
    .forEach(System.out::println);

// a1
// a2
// a3
```

Another way is using primitives' streams like `IntStream`, `LongStream` and `DoubleStream`.

```java
IntStream.range(1, 4)
    .forEach(System.out::println);

// 1
// 2
// 3
```

### Modern Stream Features

* Use `toList()` instead of `collect(Collectors.toList())`
  ```java
  List<String> result = stream.toList();
  ```

* Use `toSet()` instead of `collect(Collectors.toSet())`
  ```java
  Set<String> result = stream.toSet();
  ```

* Use `toMap()` for simple mappings
  ```java
  Map<String, User> users = stream.collect(Collectors.toMap(User::getId, Function.identity()));
  ```

* Use `groupingBy()` for grouping
  ```java
  Map<String, List<User>> usersByCity = stream.collect(Collectors.groupingBy(User::getCity));
  ```

* Use `partitioningBy()` for boolean grouping
  ```java
  Map<Boolean, List<User>> activeUsers = stream.collect(Collectors.partitioningBy(User::isActive));
  ```

## Optional\<T>

The following example shows some additional functionality of the `Optional` class:
Optional was designed to provide a better alternative to returning `null` from methods, not as a parameter type. 
According to [Oracle's documentation](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Optional.html), 
Optional should be used as a return type to indicate that a method may not return a value, helping to prevent 
null pointer exceptions.

It is used very often in functional programming, for example, when maximum value from collection is searched using
stream because collection could initially be empty and maximum value would be `null` in that case.


### Optional Features

* Use `orElseThrow()` instead of `get()`
  ```java
  User user = optionalUser.orElseThrow(() -> new UserNotFoundException());
  ```

* Use `ifPresentOrElse()` for conditional execution
  ```java
  optionalUser.ifPresentOrElse(
      user -> System.out.println("User found: " + user),
      () -> System.out.println("User not found")
  );
  ```

* Use `or()` for chaining optionals
  ```java
  Optional<User> user = findUserById(id).or(() -> findUserByEmail(email));
  ```

* Use `stream()` to convert Optional to Stream
  ```java
  Stream<User> users = optionalUser.stream();
  ```

## Records and Pattern Matching

Records and pattern matching are powerful features introduced in recent Java versions that complement functional programming:

### Records

Records are immutable data classes that automatically implement `equals()`, `hashCode()`, and `toString()`:

```java
public record User(String name, String email) {}

// Usage
User user = new User("John", "john@example.com");
System.out.println(user.name()); // Accessor method
```

### Pattern Matching

Pattern matching simplifies type checking and casting:

```java
// instanceof pattern matching
if (obj instanceof String s) {
    System.out.println(s.length());
}

// switch pattern matching
String result = switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> "Weekend";
    case TUESDAY, THURSDAY -> "Weekday";
    case WEDNESDAY -> "Midweek";
};
```

[1]: https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html
