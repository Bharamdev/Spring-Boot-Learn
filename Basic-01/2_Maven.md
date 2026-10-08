# The problem Maven solves

Imagine building a Spring Boot project by hand:

You need dozens of external libraries (Spring core, Tomcat, Jackson for JSON, logging libraries...)
Each of those libraries needs its own dependencies to work
You'd have to manually download every .jar file, keep track of versions, make sure they don't conflict, add them all to your classpath, and compile everything in the right order

That's a nightmare to do by hand and keep updated. 
# Maven is a build automation and dependency management tool that handles all of this for you, 
  based on a config file you write.

# 1. pom.xml — the heart of Maven
Every Maven project has a file called pom.xml (Project Object Model) sitting at the root. 
This is where you declare everything about your project — Maven reads this file and does the work

<groupId>com.example</groupId>
<artifactId>student-management</artifactId>
<version>0.0.1-SNAPSHOT</version>

groupId — usually your organization/company (reverse domain style)
artifactId — the project/module name
version — your project's current version. SNAPSHOT means "still in development, not a final release"

# 2. Parent — Spring Boot projects almost always have this:
   <parent>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-parent</artifactId>
   <version>3.2.0</version>
   </parent>
This is important: spring-boot-starter-parent is itself a pom.xml that pre-configures a huge amount of stuff for you — 
default Java version, default plugin versions, and crucially, default versions for every Spring-related dependency, 
so you don't have to specify version numbers for each one yourself and risk mismatched, incompatible versions.

# 3. Dependencies — the libraries your project needs:
   <dependencies>
   <dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
   </dependency>
   </dependencies>

# What's a "dependency" really, and how does it get onto your machine?

When you write that spring-boot-starter-web dependency, Maven:

I. Looks in your local repository first — a folder on your own machine (usually ~/.m2/repository) 
where previously-downloaded jars are cached.
II. If not found locally, it reaches out to a remote repository — by default, Maven Central, 
a giant public server hosting basically every open-source Java library.
III. Downloads the .jar file (the compiled library code) and stores it in your local .m2 folder, 
so next time it doesn't need to re-download.
IV. Adds that jar to your project's classpath so your code can actually use its classes (import org.springframework...).

# Transitive dependencies: 
Here's the clever part. spring-boot-starter-web isn't just one jar — it itself depends on other things: 
embedded Tomcat, Jackson (for JSON), the core Spring framework, etc. When Maven downloads spring-boot-starter-web, 
it reads that library's own pom, sees its dependencies, and downloads those too — automatically, recursively. 
You only declared one dependency in your pom, but you might end up with 20+ jars pulled in. 
This is why Spring Boot "starters" exist — spring-boot-starter-web is basically a bundle that pulls in everything 
commonly needed for building a web app, in one line.

# The Maven Build Lifecycle

Maven doesn't just manage dependencies — it also builds your project (compiles code, runs tests, packages it). 
It does this through a defined sequence of phases. Running a phase runs every phase before it too. 
The main ones, in order:
a. validate — checks the project structure/pom is correct
b. compile — compiles your .java source files into .class bytecode
c. test — runs your unit tests (using a test framework like JUnit)
d. package — bundles compiled code into a distributable format — a .jar (or .war) file
e. verify — runs checks to ensure the package is valid
f. install — copies the packaged jar into your local .m2 repository (so other local projects on your machine can use it as a dependency)
g. deploy — pushes the packaged jar to a remote shared repository (for other people/teams to use)

Two you'll use constantly :
a. mvn clean — deletes the target/ folder (all previous build output), giving you a fresh start
b. mvn spring-boot:run — a special Spring Boot plugin goal that compiles and runs your app directly, 
without manually packaging it first

# Plugins
Maven's core only knows "phases exist" — the actual work done in each phase is performed by plugins. e.g., 
the maven-compiler-plugin is what actually compiles your Java code during the compile phase; s
pring-boot-maven-plugin is what lets you do mvn spring-boot:run, and also what repackages your jar into 
an "executable" jar (one that includes embedded Tomcat and can be run with java -jar app.jar).

# Dependency Scope
Dependencies can have a <scope> tag controlling when they're available:
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-test</artifactId>
<scope>test</scope>
</dependency>
compile (default) — available everywhere, bundled into final jar
test — only available while running tests, not included in the final packaged app 
(e.g., JUnit, Mockito — you don't want test tools shipped in production)
provided — available at compile time, but not packaged (the environment running your app is expected to already provide it)

# The mvnw file (Maven Wrapper)
In Spring Boot projects generated via Spring Initializr, you'll see mvnw and mvnw.cmd files. 
This is the Maven Wrapper — it lets anyone run Maven commands (./mvnw spring-boot:run) without having 
Maven installed globally on their machine. It downloads the exact correct Maven version automatically. 
This avoids "works on my machine" problems caused by different Maven versions across teammates.

# Tying it back to earlier topics: when you run ./mvnw spring-boot:run, Maven compiles your code, 
    pulls in spring-boot-starter-web (which transitively brought in embedded Tomcat), packages everything, 
    and the spring-boot-maven-plugin launches your app — which starts Tomcat listening on localhost:8088, 
    ready to handle requests through your servlet-based dispatcher.