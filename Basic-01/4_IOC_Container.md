# Inversion of Control (IoC) Container
At startup, Spring:
* Scans your project for classes marked with annotations like @Component, @Service, @Repository, @Controller
* Creates instances of these classes (these managed instances are called "beans")
* Figures out what each bean needs (its dependencies) by looking at constructor parameters (or @Autowired fields)
* Injects the right bean instances into each other automatically
  
So in Spring, you'd just write:

```java
@Service
public class StudentService {
private final StudentRepository repository;

    @Autowired // tells Spring: "inject a matching bean here"
    public StudentService(StudentRepository repository) {
        this.repository = repository;
    }
}
```

```java
@Repository
public class MySQLStudentRepository implements StudentRepository {
// ...
}
```

You never write new StudentService(...) or new MySQLStudentRepository() anywhere yourself. Spring's container does 
it all at startup, wires them together, and hands you a fully-built, ready-to-use StudentService wherever you need 
it (e.g., injected into a @RestController).

# Putting the relationship together clearly
* IoC = the overarching principle: "control over object creation/wiring is taken away from individual classes and 
      given to an external container."
* DI = the specific pattern/technique used to achieve IoC — dependencies are given to (injected into) a class from 
     outside, rather than the class creating them itself.
* The Spring IoC Container (ApplicationContext) = the actual real implementation in Spring that performs this — it 
    creates beans, manages their entire lifecycle, and performs the injection wiring automatically for you.


# Application Context:

**Spring has two container interfaces:**
* BeanFactory is the basic, older, lightweight container. It creates beans lazily (only when asked for).
* ApplicationContext is built on top of BeanFactory and adds more features (event publishing, internationalization,
  annotation support). This is what you actually use in Spring Boot. It creates singleton beans eagerly at startup.

```java
@SpringBootApplication
public class DemoApplication {
public static void main(String[] args) {
ApplicationContext context = SpringApplication.run(DemoApplication.class, args);
// 'context' IS the IoC container
}
}
```

SpringApplication.run(...) is the line that creates the container, scans your code, builds all the beans, wires them, 
and starts Tomcat. You can even ask the container for a bean manually:
```java
StudentService service = context.getBean(StudentService.class);
```

**2. Reflection in JAVA**

> Reflection is a Java feature that lets a program inspect and manipulate classes, methods, fields, constructors, and 
annotations at runtime, without knowing them at compile time.
```java
Class<?> clazz = Class.forName("com.example.StudentService");   // load class by name
Object obj = clazz.getDeclaredConstructor().newInstance();       // create object
```

Read all methods
Read all fields

**Two things worth knowing:**

* Reflection is slower than normal code, which is one reason Spring does most of this work once at startup and not 
  on every request.
* It breaks encapsulation (it can touch private members), so you should never use it casually in your own business code. 
  It's a framework-level tool.


**When you run your app:**

> SpringApplication.run() creates the IoC container (ApplicationContext)
  @ComponentScan (inside @SpringBootApplication) tells it which packages to search
  Using reflection, Spring finds classes annotated with @Component/@Service/@Repository/@RestController, and also 
         processes @Configuration classes with their @Bean methods
  It creates each bean, and for each one reads the constructor parameters (or @Autowired spots) to find its dependencies
  It performs dependency injection, ideally via the constructor
  All beans are ready, Tomcat starts listening, and requests can flow in