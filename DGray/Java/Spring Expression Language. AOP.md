---
title: Spring Expression Language. AOP
updated: 2021-03-13 10:12:23Z
created: 2021-03-13 09:38:02Z
tags:
  - aop
  - spel
---


[[SpEL]] [[AOP]]

[SpEL, AOP.docx](../_resources/SpEL, AOP.docx)

```java
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.stereotype.Component;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

@Aspect
@Component
public class AppLoggingAspect {
//"execution(modifier-pattern? return-type-pattern declaring-type-pattern? method-name-pattern(args-pattern)
//throws-pattern?)"
//execution([модификатор_метода(public, *)?] [тип_возврата] [класс?] [имя_метода]([аргументы]) [исключения?]

   @Before("execution(public void com.geekbrains.aop.UserDAO.addUser())") // pointcut expression
   public void beforeAddUserInUserDAOClass() {
       System.out.println("AOP: Поймали добавление пользователя");
   }

    @Before("execution(public void com.geekbrains.aop.UserDAO.*User())") // pointcut expression
    public void beforeUserModifyInUserDAOClass() {
        System.out.println("AOP: работа с пользователем в UserDAO");
    }

   @Before("execution(public void com.geekbrains.aop.UserDAO.*())") // pointcut expression
   public void beforeAnyMethodWithoutArgsInUserDAOClass() {
       System.out.println("AOP: любой void метод без аргументов из UserDAO");
   }

   @Before("execution(public void com.geekbrains.aop.*.*(..))") // pointcut expression
   public void beforeAnyMethodInUserDAOClass() {
       System.out.println("AOP: любой метод без аргументов из UserDAO");
   }

   @Before("execution(public void com.geekbrains.aop.UserDAO.*(..))") // pointcut expression
   public void beforeAnyMethodInUserDAOClassWithDetails(JoinPoint joinPoint) {
       MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
       System.out.println("В UserDAO был вызван метод: " + methodSignature);
       Object[] args = joinPoint.getArgs();
       if (args.length > 0) {
           System.out.println("Аргументы:");
           for (Object o : args) {
               System.out.println(o);
           }
       }
   }

   @AfterReturning(
           pointcut = "execution(public * com.geekbrains.aop.UserDAO.getAllUsers(..))",
           returning = "result")
   public void afterGetBobInfo(JoinPoint joinPoint, List<String> result) {
       result.set(0, "Donald Duck");
   }

   @AfterThrowing(
           pointcut = "execution(public * com.geekbrains.aop.UserDAO.*(..))",
           throwing = "exc")
   public void afterThrowing(JoinPoint joinPoint, Throwable exc) {
       System.out.println(exc); // logging
   }

   @After("execution(public * com.geekbrains.aop.UserDAO.*(..))")
   public void afterMethod() {
       System.out.println("After");
   }

    @Around("execution(public * com.geekbrains.aop.UserDAO.*(..))")
    public Object methodProfiling(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        System.out.println("start profiling");
        long begin = System.currentTimeMillis();
        Object out = proceedingJoinPoint.proceed();
        long end = System.currentTimeMillis();
        long duration = end - begin;
        System.out.println((MethodSignature) proceedingJoinPoint.getSignature() + " duration: " + duration);
        System.out.println("end profiling");
        return out;
    }
}
```

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Component;

//@Aspect
//@Component
public class PointcutDeclarationAspect {
    @Pointcut("execution(public * com.geekbrains.aop.UserDAO.get*(..))") // pointcut expression
    public void userDAOGetTrackerPointcut() {
    }

    @Pointcut("execution(public * com.geekbrains.aop.UserDAO.set*(..))") // pointcut expression
    public void userDAOSetTrackerPointcut() {
    }

    @Pointcut("userDAOGetTrackerPointcut() || userDAOSetTrackerPointcut()") // pointcut expression
    public void userDAOGetOrSetTrackerPointcut() {
    }

    @Before("userDAOGetOrSetTrackerPointcut()") // || && !
    public void userDAOGetOrSetTracker() {
        System.out.println("В классе UserDAO вызывают геттер или сеттер");
    }
}

```

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class TestBean {
    @Value("#{19 + 1}") // 20
    private double add;

    @Value("#{'String1 ' + 'string2'}") // "String1 string2"
    private String addString;

    @Value("#{20 - 1}") // 19
    private double subtract;

    @Value("#{10 * 2}") // 20
    private double multiply;

    @Value("#{36 / 2}") // 19
    private double divide;

    @Value("#{36 div 2}") // 18, the same as for / operator
    private double divideAlphabetic;

    @Value("#{37 % 10}") // 7
    private double modulo;

    @Value("#{37 mod 10}") // 7, the same as for % operator
    private double moduloAlphabetic;

    @Value("#{2 ^ 9}") // 512
    private double powerOf;

    @Value("#{(2 + 2) * 2 + 9}") // 17
    private double brackets;

    @Value("#{1 == 1}") // true
    private boolean equal;

    @Value("#{1 eq 1}") // true
    private boolean equalAlphabetic;

    @Value("#{1 != 1}") // false
    private boolean notEqual;

    @Value("#{1 ne 1}") // false
    private boolean notEqualAlphabetic;

    @Value("#{1 < 1}") // false
    private boolean lessThan;

    @Value("#{1 lt 1}") // false
    private boolean lessThanAlphabetic;

    @Value("#{1 <= 1}") // true
    private boolean lessThanOrEqual;

    @Value("#{1 le 1}") // true
    private boolean lessThanOrEqualAlphabetic;

    @Value("#{1 > 1}") // false
    private boolean greaterThan;

    @Value("#{1 gt 1}") // false
    private boolean greaterThanAlphabetic;

    @Value("#{1 >= 1}") // true
    private boolean greaterThanOrEqual;

    @Value("#{1 ge 1}") // true
    private boolean greaterThanOrEqualAlphabetic;

    @Value("#{250 > 200 && 200 < 4000}") // true
    private boolean and;

    @Value("#{250 > 200 and 200 < 4000}") // true
    private boolean andAlphabetic;

    @Value("#{400 > 300 || 150 < 100}") // true
    private boolean or;

    @Value("#{400 > 300 or 150 < 100}") // true
    private boolean orAlphabetic;

    @Value("#{!true}") // false
    private boolean not;

    @Value("#{not true}") // false
    private boolean notAlphabetic;

    @Value("#{2 > 1 ? 'a' : 'b'}") // "a"
    private String ternary1;

   @Value("#{someBean.someProperty != null ? someBean.someProperty : 'default'}")
   private String ternary2;

   @Value("#{someBean.someProperty ?: 'default'}") // Will inject provided string if someProperty is null
   private String elvis;

   @Component("workersHolder")
   public class WorkersHolder {
       private List<String> workers = new LinkedList<>();
       private Map<String, Integer> salaryByWorkers = new HashMap<>();

       public WorkersHolder() {
           workers.add("John");
           workers.add("Susie");
           workers.add("Alex");
           workers.add("George");

           salaryByWorkers.put("John", 35000);
           salaryByWorkers.put("Susie", 47000);
           salaryByWorkers.put("Alex", 12000);
           salaryByWorkers.put("George", 14000);
       }

       //Getters and setters
   }

   @Value("#{workersHolder.salaryByWorkers['John']}") // 35000
   private Integer johnSalary;

   @Value("#{workersHolder.salaryByWorkers['George']}") // 14000
   private Integer georgeSalary;

   @Value("#{workersHolder.salaryByWorkers['Susie']}") // 47000
   private Integer susieSalary;

   @Value("#{workersHolder.workers[0]}") // John
   private String firstWorker;

   @Value("#{workersHolder.workers[3]}") // George
   private String lastWorker;

   @Value("#{workersHolder.workers.size()}") // 4
   private Integer numberOfWorkers;
}

```

```java
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.stereotype.Component;

// @Aspect
// @Component
public class ComplexAspect {
    @Before("execution(public * com.geekbrains.aop.UserDAO.*(..))")
    public void allMethodsCallsLogging() {
        System.out.println("В классе UserDAO вызывают метод");
    }

    @Before("execution(public * com.geekbrains.aop.UserDAO.*(..))")
    public void allMethodsCallsAnalytics() {
        System.out.println("В классе UserDAO вызывают метод (Аналитика)");
    }

    @Before("execution(public * com.geekbrains.aop.UserDAO.*(..))")
    public void allMethodsCallsSendInfoToCloud() {
        System.out.println("В классе UserDAO вызывают метод (Cloud)");
    }
}
```

```java
//@Aspect
//@Component
//@Order(200)
public class SimplifiedAnalyticAspect {
    @Before("execution(public * com.geekbrains.aop.UserDAO.*(..))")
    public void allMethodsCallsAnalytics() {
        System.out.println("В классе UserDAO вызывают метод (Аналитика)");
    }
}

//@Aspect
//@Component
//@Order(1000)
public class SimplifiedCloudAspect {
    @Before("execution(public * com.geekbrains.aop.UserDAO.*(..))")
    public void allMethodsCallsSendInfoToCloud() {
        System.out.println("В классе UserDAO вызывают метод (Cloud)");
    }
}

//@Aspect
//@Component
//@Order(-100)
public class SimplifiedLoggingAspect {
    @Before("execution(public * com.geekbrains.aop.UserDAO.*(..))")
    public void allMethodsCallsLogging() {
        System.out.println("В классе UserDAO вызывают метод (Logging)");
    }
}
```