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

First, enable AOP.
```
@Configuration
@EnableAspectJAutoProxy // with this, Spring looks for @Aspect
@ComponentScan(basePackages="com.app.aspects")
public class AopConfig {
  //
}
```

Next, set up an aspect.
```
package com.app.aspects;

@Aspect
@Component
public class LogPropertyChanges {
  private Logger logger = Logger.getLogger(getClass());

  @Before("execution(void set*(*))")
  public void logChange() {
    logger.info("Property changed!");
  }
}
```

