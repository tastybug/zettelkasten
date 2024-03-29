# Introduction to Aspect Oriented Programming

Applications have cross-cutting concerns: logging, security, transaction handling and so on. Instead of implemting a concern in multiple places (code duplication) and weaving multiple concerns together (biz logic + logging + security), there is AOP. There are 2 approaches to it:

* **AspectJ**: the original, it's a full-blown language; uses bytecode modification for aspect weaving
* **Spring AOP**: a Java based framework with AspectJ integration; uses Proxies for weaving; it focusses on enterprise problems

This document concentrates on Spring AOP.

> An Aspect covers a concern: there can be a shared "Security Aspect", a "Transaction Aspect", a "Logging Aspect". The Aspect defines a WHERE and a WHAT: WHERE it contributes and WHAT code is to be run.

## Core Concepts

* __Join Point__ is a point in the execution of a program: a method call, a exception thrown.
* __Pointcut__ is an expression that selects one or more Join Points.
* __Advice__ is code that is to be executed at each selected Join Point.
* __Aspect__ a "module" that encapsulates pointcuts and advice.
* __Weaving__ is the technique of combining main code with aspects

## Basics of Spring Integration

Spring uses AspectJ annotations, but does not use the AspectJ weaving approach. Instead, Spring uses proxies to weave.

### Small Example

This example assumes the usage of CGLib proxies. If that's not the case, then all join points must be declared in an interface (`Cache` must be an interface, with some setters), otherwise the pointcuts will not work.

First, enable AOP.
```
@Configuration
@EnableAspectJAutoProxy //enables @Aspect; with SpringBoot, autoconf takes care of this automatically
@ComponentScan(basePackages="com.app.aspects")
public class AopConfig {
  //
}
```

Next, set up an aspect.
```
package com.app.aspects;

@Aspect // aspect
@Component
public class LogPropertyChanges {
  private Logger logger = Logger.getLogger(getClass());

  @Before("execution(void set*(*))") // pointcut
  public void logChange(JoinPoint jp) { // the advice
    String methodName = point.getSignature().getName();
    Object newValue = point.getArgs()[0];
    logger.info(methodName + ": changes to " + newValue);
  }
}
```

Now see this in action.

```
@Autowired
private Cache cache;

cache.setCacheSize(200);
// now you see the log statement
```

### Pointcut Matching Rules

The method pattern is: `[Modifiers] ReturnType [ClassType] MethodName (Arguments) [throws Exception]`.

This pattern is used in the annotation value `execution(<method pattern>)`. Patterns can be chained together logically (`&&`, `||`, `!`): `execution(<pattern1>) || execution(<pattern2>)`.

There are 2 kinds of wildcards: 
* `*` wildcards match once (return type, package, class, method name, argument)
* `..` matches zero or more (arguments, packages)

An example:

```
execution(* rewards.restaurant.*Service.find*(..))
^ designator
          ^ return type
            ^ package
                               ^ class
                                        ^ method
                                              ^ arguments
```

#### More examples

1. `execution(void send*(rewards.Dining))`: any method starting with `send` that has a single `Dining` argument and returns void. Note here the use of fully-qualified class names, which is required for non-basic types.
2. `execution(* send(*))`: any `send` method with a single parameter and any kind of return value.
3. `execution(* send(int, ..))`: any `send` method with a first parameter of type `int` and zero or many further parameters, returning any kind of return value.
4. `execution(void example.MessageServiceImpl.*(..))`: any method in class `MessageServiceImpl` (or subclasses!) with any number of parameters and a `void` return type.
5. `execution(void example.MessageService.send(*))`: the `send` method defined by interface `MessageService` with one parameter and `void` return type.
6. `execution(@javax.annotation.secutiry.RolesAllowed void send*(..))` this matches methods annotated with `@RolesAllowed`
7. `execution(* rewards.*.restaurant.*.*(..)` any method of any type in package `rewards.*.restaurant` with any return type and one or more parameters. The package wildcard covers __1 level of depth__.
8. `execution(* rewards..restaurant.*.*(..)` this is the same as before, but it has indefinitive depth.
9. `execution(* *..restaurant.*.*(..)` again, broader than the one before. This would matches packages `restaurant`, `com.restaurant` and `com.foo.bar.baz.restaurant`.

### Advices

There are different types of Advices:

* `@Before`: the proxy will call the advice before calling the method. If the advice fails, the target method will not be called.
* `@After`: execution happens after target, irregardless of whether it was a success or not.
* `@AfterReturning`: the target is called first and if it's __NOT throwing an exception__, the Advice is executed. Note that the type of `@AfterReturningAdvice(returning="xyz")` as given in the Advice's method signature is an filter additional to the `execution` pattern! See example below.
* `@AfterThrowing`: the target is called first and if an exception is thrown, the Advice is executed. The exception will keep propagating as it is, but you can throw a different type of exception.
* `@Around`: the most flexible Advice, it's like a proxy! The Aspect delegates only to the Advice and it is up to the Advice to call the target (or not) and handle the return/exception. Look at the neat example where the Aspect implements caching!

#### Examples

```
@Aspect
@Component
public class PropertyChangeTracker {
  private Logger logger = Logger.getLogget(getClass());

  @Before("execution(void set*(..)")
  public void trackChange() {
    logger.info("Something is about to be changed..");
  }

  // Note: the returning type must match the type of the Advice method signature,
  // which here is "Reward", otherwise the Advice is not invoked.
  @AfterReturning(value = "execution(* service..*.*(..))",
                  returning="reward") 
  public void audit(JoinPoint jp, Reward reward) {
    logger.info(jp.getSignature() + " returns the following: " + reward.toString());
  }

  // Note: the exception type must match the type of the Advice method signature,
  // which here is "SomeException", otherwise the Advice is not invoked.
  @AfterThrowing(value = "execution(* service..*.*(..))",
                 throwing="e") 
  public void audit(JoinPoint jp, SomeException e) {
    logger.info(jp.getSignature() + " threw exception: " + e.toString());
    throw new OtherException(e); // unless we throw something, e will be thrown further
  }

  @After("execution(void update*(..))") 
  public void trackUpdate() {
    logger.info("Something was updated, successfully or not, can't tell.");
  }

  @Around("execution(@example.Cacheable * loadStuff(..))")
  public Object cache(ProceedingJoinPoint pjp) throws Throwable {
    Object value = cacheStore.get(CacheUtils.toKey(pjp));

    if (value != null) {
      return value;
    }
    value = point.proceed();
    cacheStore.put(CacheUtils.toKey(pjp), value);
    return value;
  }
}
```

### Limitations of Spring AOP

1. it can only advise non-private methods
2. can only apply aspects to Spring Beans
3. when using proxies: when method A() calls method B() on the same class/interface, the advice will never be executed for B() because it's a "lateral call" inside the proxy.
