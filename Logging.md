## SLF4J

[SLF4J](https://www.slf4j.org/) (Simple Logging Facade for Java) is a facade built on top of existing logging frameworks
(Log4J2, Logback etc.) which simplifies logging operations, eases migration between different logging framework and _standardizes_
the logging process.

### Instantiating loggers

Declare loggers as `private`, `static` and `final`
([Sonar rules reference](https://rules.sonarsource.com/java/tag/convention/RSPEC-1312)), for example:

``` java
private static final Logger LOGGER = LoggerFactory.getLogger(Example.class);
```

### Use parameterized messages

The recommended way to create log messages with parameters is by using curly braces `{}` as placeholders for
parameters. __Do not use `String.format(...)`, or worse, concatenate parameters with messages__.
SLF4J offers an elegant way to log such messages.

For example, use:

``` java
log.debug("Time to fetch entries: {} ms", currentTime - start);
```

instead of:

``` java
log.debug("Time to fetch entries: " + currentTime - start + " ms");
```

This will result in log messages being more readable and will speed up logging statement evaluations
([SLF4J docs reference](https://www.slf4j.org/faq.html#logging_performance)).

Multiple variables can be used in a logging statement by referencing them after the logging statement as an `Object...`.
When passing objects to log statements, the object's `toString()` method will be called, for example:

``` java
log.debug("Start: saveOrder({}, {})", itemId, customerId);
```

### Logging Best Practices

* Use structured logging with JSON format for Production

* Use MDC (Mapped Diagnostic Context) for request-scoped logging
  ```java
  MDC.put("requestId", requestId);
  ```

* Use lazy evaluation for expensive operations
  ```java
  log.debug("Expensive operation result: {}", () -> expensiveOperation());
  ```

* Use appropriate log levels
  * ERROR - when something wrong happened and the application can't recover
  * WARN - when something unexpected happened but the application can recover
  * INFO - for important business events
  * DEBUG - for detailed information useful for debugging
  * TRACE - for very detailed information

## Log appenders

If routing logs to text files, a useful way to filter out important messages to separate locations is by configuring additional appenders and loggers per class/package.

For example, if we want to route all logs from `MyService.java` to a file named `my-service.log`, we declare a file appender to route messages and create a logger which will use that appender, for example:

* Logback

``` xml
<appender name="my-service" class="ch.qos.logback.core.FileAppender">
    <encoder>
        <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
    </encoder>
    <file>path/to/log-file/my-service.log</file>
</appender>

<logger name="com.infinum.myapp.feature.service.MyService" LEVEL="DEBUG">
    <appender ref="my-service"></appender>
</logger>
```

* Log4J2

``` xml
<Logger name="com.infinum.myapp.feature.service.MyService" level="debug" additivity="false">
    <AppenderRef ref="my-service"/>
</Logger>

<Appender type="File" name="my-service" fileName="path/to/log-file/my-service.log">
    <Layout type="PatternLayout">
        <Pattern>%d %p %C{1.} [%t] %m%n</Pattern>
    </Layout>
</Appender>
```

## Propagating context across a service

When having distributed tracing, debugging or for sake of grouping all logs to a same identifier it is good practice to introduce MDC.


MDC is tied to ThreadLocal meaning if you have Spring Web MVC whenever you use log.info, error or debug
it will have access to data you put in MDC map.

To do so you should implement `OncePerRequestFilter` and introduce values to MDC which you want for example:

``` kotlin
class EmptyBodyLogging() : OncePerRequestFilter() {

    val logger = KotlinLogging.logger {}

    override fun doFilterInternal(
        request: HttpServletRequest,
        response: HttpServletResponse,
        filterChain: FilterChain,
    ) {
        //Put all values you want in MDC to be used across the Service
        MDC.put("user-id", "someValueEitherFromBeanOrAuth")
        filterChain.doFilter(request, response)
        //Empty MDC so once a thread is done with a request ThreadLocal state is cleared
        MDC.clear()
    }
```

In Spring Web MVC now MDC will be propagated for whole lifespan of
a request meaning you can set logback.xml to use key values which you used in `MDC.put` command.

## Propagating MDC in Tomcat and WebClient Reactive

When dealing with Spring Web MVC and webflux Webclient MDC won't work out of the box.
This makes total sense since webflux and reactive programming don't rely on a request being served by the same thread.
Unblocking nature of reactive programming causes then a headache since during a webclient call threads will be switched.

To solve this when creating a webclient bean filter should be introduced:

``` kotlin

 return WebClient.builder()
            .filter { request, next ->
                val contextMap = MDC.getCopyOfContextMap()
                next.exchange(request).doOnEach { signal ->
                    if (!signal.isOnComplete) {
                        MDC.setContextMap(contextMap)
                    }
                }
            }.build()

```
So WebClient uses something called signal when switching between states and calls internally.
What we are doing here is taking a copy of MDC on this thread and passing it to a thread which picks up next stage of processing.
Here we have ensured MDC propagation inside webclient when we have Logger being called on `webclient.onError` or `webclient.onSuccess`


## Further reading

* [Using SLF4J with Log4J](https://www.baeldung.com/slf4j-with-log4j2-logback)
* [Best practices & tricks for Java logging](https://stackify.com/log4j-guide-dotnet-logging/)
* [9 Java logging sins](https://stackify.com/9-logging-sins-java/)
* [Structured Logging with Spring Boot](https://docs.spring.io/spring-boot/reference/features/logging.html#features.logging.structured)
