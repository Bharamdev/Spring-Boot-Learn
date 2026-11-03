# Annotations

> An annotation (@Something) is metadata: a label you attach to a class, method, or field. By itself it does nothing. 
It becomes useful because something else reads it. Here, Spring reads it and acts on it.

1. @Component
Placed on a class, it tells Spring: "**Create an object of this class and manage it as a bean.**"

```java
@Component
public class EmailSender {
public void send(String msg) { ... }
}
```

@Component	-> Any generic bean
@Service	-> Business logic layer
@Repository	-> Database access layer (also adds automatic translation of database exceptions)
@Controller / @RestController -> Web layer (handles HTTP requests)

2. @Configuration
Placed on a class, it says: "**This class contains instructions on how to create beans.**" 
Inside it you write methods annotated with @Bean, and whatever each method returns becomes a bean.

```java
@Configuration
public class AppConfig {

    @Bean
    public ModelMapper modelMapper() {
        return new ModelMapper();
    }
}
```

3. @ComponentScan
Having @Component on a class isn't enough by itself. Spring must find it. @ComponentScan tells Spring which 
packages to search for annotated classes.
```java
@ComponentScan(basePackages = "com.example")
```

Spring looks through com.example and all its sub-packages, finds every class with @Component (or @Service, @Repository, etc.), 
and registers them as beans.

# How @SpringBootApplication ties it together

On your main class you write only @SpringBootApplication, but it's actually a bundle of three annotations:
* @SpringBootApplication  =  @Configuration + @EnableAutoConfiguration + @ComponentScan

@Configuration makes the main class itself a configuration class
@EnableAutoConfiguration makes Spring Boot automatically configure things based on the dependencies it sees (e.g., it sees spring-boot-starter-web and auto-configures Tomcat)
@ComponentScan with no arguments scans the package of the main class and everything below it

4. @Autowired 
@Autowired tells Spring: "Find a bean of the matching type in the container and inject it here."

```java
@Service
public class StudentService {

    @Autowired
    private StudentRepository repository;
}
```

# How Spring resolves it, step by step:

Sees @Autowired on a field of type StudentRepository
Looks in the container for a bean of that type
Finds exactly one, so injects it (using reflection, even though the field is private)

What if there are two matching beans? Say two classes both implement StudentRepository. Spring gets confused and 
throws a NoUniqueBeanDefinitionException. You solve it with:
 @Qualifier or @Primary

> Useful fact: since Spring 4.3, if a class has only one constructor, you can skip @Autowired entirely and 
  Spring still injects it.

# How Spring resolves dependencies internally

Suppose Spring is building StudentService, whose constructor is StudentService(StudentRepository repository).

1. Inspect the constructor with reflection. Spring reads the parameter types: “I need a StudentRepository.”
2. Search the container by type. It looks through all registered bean definitions for any bean assignable to StudentRepository. 
   This includes classes that implement that interface.
3. Count the matches:
   Exactly one: use it.
   Zero: fail (or skip if optional). See section 3.
   More than one: run the tie-breaking process. See section 4.
4. Make sure the chosen bean exists. If it hasn’t been built yet, Spring builds it first (recursively resolving its dependencies).
5. Inject it into the constructor call.

Because this is recursive, Spring effectively builds a dependency graph:
StudentController
└── needs StudentService
└── needs StudentRepository
└── needs DataSource
It builds from the bottom up: DataSource first, then the repository, then the service, then the controller.

> Circular dependencies (A needs B, B needs A) break this graph, because neither can be built first. With 
  constructor injection, Spring Boot refuses to start and throws BeanCurrentlyInCreationException. Fix it by redesigning.

# What happens when no matching bean exists
If Spring finds zero beans of the required type, startup fails.

