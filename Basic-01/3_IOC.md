# IOC (Inversion of Control)

# The problem first: tightly coupled code
Say you're writing a StudentService class that needs to save data to a database. 
Without any framework, you might write:

`public class StudentService {
private MySQLStudentRepository repository;

    public StudentService() {
        this.repository = new MySQLStudentRepository(); // created INSIDE
    }
}`

# What's wrong with this? 
StudentService itself is creating its own dependency (MySQLStudentRepository) using new. This means:

* StudentService is tightly coupled to MySQLStudentRepository specifically. If tomorrow you switch to PostgreSQL and 
  have a PostgresStudentRepository instead, you have to go inside StudentService and change this line of code.
* Testing becomes hard — you can't easily swap in a fake/mock repository for a unit test, because 
  StudentService insists on creating the real one itself.
* If ten other classes also need a MySQLStudentRepository, ten separate objects get created unnecessarily, 
  each doing its own new.

# Solution:

# Inversion of Control (IoC) — the principle
IoC is an idea/principle, not a specific mechanism. It says: 
"Don't let a class control the creation of its own dependencies. Hand that responsibility to something external."

Normally, your code is in control — it decides when to create objects, in what order, and wires them together 
(that's the "normal" flow of control). IoC flips this: control is inverted — taken away from your individual classes 
and given to an external entity (a framework/container) that manages object creation and wiring on your behalf.

# Dependency Injection (DI) — the mechanism that implements IoC
If IoC is the principle ("don't create your own dependencies"), DI is the actual technique/mechanism used to achieve 
it — specifically for supplying dependencies.

Instead of StudentService creating MySQLStudentRepository itself with new, DI says: 
"Give (inject) the dependency to the class from outside, rather than having the class build it internally."

`public class StudentService {
private StudentRepository repository; // notice: interface, not concrete class

    // Dependency is INJECTED via constructor, not created internally
    public StudentService(StudentRepository repository) {
        this.repository = repository;
    }
}`

Now StudentService doesn't know or care whether it's getting a MySQLStudentRepository, a PostgresStudentRepository, 
or a fake MockStudentRepository for testing.

In plain Java, you would still have to manually do:
`StudentRepository repo = new MySQLStudentRepository();
StudentService service = new StudentService(repo); // manually injecting`
That's manual DI — better than the tightly-coupled version, but you're still doing the wiring yourself.

# Spring automates this entirely. Spring maintains something called the ApplicationContext
    (also referred to as the IoC container) — this is the "external entity" from our IoC definition.