Our code style is based on [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html) with some
slight modifications. Some of the most important guidelines are:

* Source files are encoded in **UTF-8**
* Non-ASCII characters are preferably actual Unicode characters instead of Unicode escapes (e.g. `String a = "Ⰰ";`
instead of `"String a = "\u2c00";`)
* No wildcard (`*`) imports are used, both static and non-static
* Braces are used for all statements where they are optional, even for empty or one-line `if`, `else`, `do`, `while`,
`for`, `try`, `catch` and `finally` statements
* [Egyptian style](https://blog.codinghorror.com/new-programming-jargon/) braces are used - no line break before opening
brace, line break after opening and before closing brace
* One statement per line
* One variable per declaration, except in `for` loop header (i.e. `int a, b` is not allowed but
`for (int i, j; ...)` is allowed)
* C-style array declarations are not allowed (e.g. `String items[]` is not OK, use `String[] items` instead)
* Always use `default` case in `switch` statement, even if it is empty (`enum`s are an exception to this rule)
* `long` literals should always use uppercase `L` suffix to avoid confusing `l` and `1`
* Identifiers are in camel case (e.g. `class SomeClass`, `int someVariable`), except for package names which are always
lowercase without underscores (`_`)
* primitive and immutable `static final` members are in `CONSTANT_CASE`
* `@Override` annotation is always used
* Always handle checked exceptions by either logging them, rethrowing them wrapped in some `RuntimeException` or
commenting why the exception can be ignored (if project uses `lombok` library then you can use `@SneakyThrows`
annotation, but only in tests)

Additionally, here is a (non-exhaustive) list of modifications:

* Line column limit is 120 instead of 100 columns
* Static import of static nested classes is allowed if nested class name clearly specifies its purpose and
relation to parent class (e.g. `Booking.BookingType` is allowed to be statically imported as `BookingType`)
* Empty blocks must be commented if not concise
* Block indentation is 4 spaces instead of 2 to avoid deeply-nested statements
* Fallthrough in switch-case statements is not allowed

### Style Guidelines

* Use text blocks for multi-line strings
  ```java
  String json = """
      {
          "name": "John",
          "age": 30
      }
      """;
  ```

* Use pattern matching for instanceof checks
  ```java
  if (obj instanceof String s) {
      System.out.println(s.length());
  }
  ```

* Use records for simple data carrier classes
  ```java
  public record User(String name, String email) {}
  ```

* Use sealed classes to define restricted hierarchies
  ```java
  public sealed interface Shape permits Circle, Rectangle, Triangle {}
  ```

* Use switch expressions instead of switch statements when possible
  ```java
  String result = switch (day) {
      case MONDAY, FRIDAY, SUNDAY -> "Weekend";
      case TUESDAY, THURSDAY -> "Weekday";
      case WEDNESDAY -> "Midweek";
  };
  ```

* Use enhanced for loops or streams instead of traditional for loops
  ```java
  for (String item : items) {
      System.out.println(item);
  }

  items.stream()
      .filter(item -> item.length() > 5)
      .forEach(System.out::println);
  ```

* Use method references instead of lambda expressions when possible
  ```java
  items.forEach(System.out::println);
  ```

* Use Optional for methods that might not return a value
  ```java
  public Optional<User> findUser(String id) {
      // ...
  }
  ```

* Use try-with-resources for resource management
  ```java
  try (var reader = new FileReader("file.txt")) {
      // use reader
  }
  ```

You can simply import described code style into IntelliJ IDEA as shown
[here](https://www.jetbrains.com/help/idea/configuring-code-style.html). The XML file for the style is available
[here](https://github.com/infinum/java-handbook/blob/master/resources/codestyle/style.xml). Checkstyle XML file is available
[here](https://github.com/infinum/java-handbook/blob/master/resources/checkstyle/checkstyle.xml).

### Writing *readable* code

Developers usually spend most of their time maintaining existing codebase and that means you will be reading other
peoples' code a lot. In order to reduce time spent on deciphering code someone wrote six months ago, try to write
*readable* code.

One of the most important aspects of *readable* code is self-documentation. Instead of writing cryptic code and
commenting each line, write code which can be easily understood on its own.

For example, when naming variables and fields, try to avoid abbreviated names like
`UserPremissionFetchingService upfs = ...`. Instead, try to match type name as closely as possible
(for example `UserPermissionFetchingService userPermissionService = ...`), or even use the full type name. The reason for this is that
variables with abbreviated names use one more short-term memory slot than variables with descriptive names because you
have to remember what abbreviation stands for. Since most of the time is spent on reading existing code, take a few
extra seconds to type longer names. You and your colleagues will be grateful next time someone tries to read your
code. 😎

Another aspect to keep in mind for self-documenting code is
[cyclomatic complexity](https://en.wikipedia.org/wiki/Cyclomatic_complexity). Don't have too many nested
`if`-`else` branches in one method, instead extract functionality in some other `private` method. Who knows - maybe you
will even be able to reuse that code somewhere else!

Other tips for writing *readable* code:
* Avoid having multiple `boolean` method parameters - use `enum` or create a class for passing `boolean` parameters
```java
// avoid this
public void doSomething(boolean isGoingToCrash, boolean isGoingFast) {
    if (isGoingToCrash) {
        dont();
    }
    // ...
}

// do this instead
public static class AccordinglyNamedParameters {

    private final boolean isGoingToCrash;
    private final boolean isGoingFast;

    public AccordinglyNamedParameters(boolean isGoingToCrash, boolean isGoingFast) {
        this.isGoingToCrash = isGoingToCrash;
        this.isGoingFast = isGoingFast;
    }
}

public void doSomething(AccordinglyNamedParameters parameters) {
    if (parameters.isGoingToCrash) {
        dont();
    }
    // ...
}

// or this
public enum Collision {
    NO_COLLISION, WILL_CRASH
}

public enum Speed {
    FAST, SLOW
}

public void doSomething(Collision collision, Speed speed) {
    if (collision == WILL_CRASH) {
        dont();
    }
    // ...
}
```
* Avoid large methods with
[side-effects](https://en.wikipedia.org/wiki/Side_effect_%28computer_science%29) - functional code is easier to understand
as you don't have to worry about state
* For methods that cannot avoid having side-effects, separate functional and non-functional parts

### Code Quality
All projects must be analyzed with SonarQube before release and you can and should do so locally. However, you can get
on the fly feedback for most of the issues which SonarQube reports by using the
[SonarLint](https://www.sonarlint.org/intellij/) plugin for IntelliJ IDEA. Analysis is performed on the fly for each file, but you can analyze
current file on demand with `Analyze -> Analyze with SonarLint` (default hotkey `⇧⌘S`) or entire project with
`Analyze -> Analyze All Files with SonarLint`. [Here](https://rules.sonarsource.com/java) is a full list of default
rules for SonarLint.

In addition to SonarLint, IntelliJ itself offers a lot of support for code inspection.
[Here](https://github.com/infinum/java-handbook/blob/master/resources/codestyle/inspections.xml) is a configuration file with some
additional inspections enabled.

**NOTE**: after importing inspections, IntelliJ will notify you that you that there are missing definitions of
severities. Select the option to create missing severities and then set their colors and stripe marks to you liking.
