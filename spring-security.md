# Spring Boot, Spring Data, and Spring Cloud Guide

## Table of Contents
1. [Spring vs Spring Boot](#spring-vs-spring-boot)
2. [Spring Boot Basics](#spring-boot-basics)
3. [How Spring Boot Works: From Startup to Shutdown](#how-spring-boot-works-from-startup-to-shutdown)
4. [Spring Boot Lifecycle](#spring-boot-lifecycle)
5. [Core Spring Concepts](#core-spring-concepts)
6. [Declaring Beans and Dependency Injection in Spring Boot](#declaring-beans-and-dependency-injection-in-spring-boot)
7. [Bean Scopes and Lifecycles in Spring Boot](#bean-scopes-and-lifecycles-in-spring-boot)
8. [Spring Data (JPA)](#spring-data-jpa)
9. [Spring Security (with JWT & HTTPS)](#spring-security-with-jwt--https)
10. [Spring Cloud](#spring-cloud)
11. [Database Setup (Postgres Example)](#database-setup-postgres-example)
12. [Running Spring Boot JARs](#running-spring-boot-jars)

---

## Spring vs Spring Boot

**Spring Framework** is a comprehensive programming and configuration model for Java applications. It provides dependency injection, aspect-oriented programming, transaction management, and more. You can use Spring to build any kind of Java application, but it requires a lot of manual setup (XML or Java config, web.xml, etc.).

**Spring Boot** is built on top of Spring. It makes it easy to create stand-alone, production-ready Spring applications with minimal configuration. It provides:
- Embedded servers (no need for external Tomcat/Jetty)
- Auto-configuration (sensible defaults)
- Opinionated starter dependencies
- Production features (health checks, metrics, etc.)

**Key difference:** Spring Boot is about convention over configuration, reducing boilerplate and setup time.

---

## Spring Boot Basics

Spring Boot is a framework for building stand-alone, production-grade Spring-based applications with minimal configuration. It provides:
- Embedded servers (Tomcat, Jetty, etc.)
- Auto-configuration
- Opinionated defaults
- Production-ready features (metrics, health checks, etc.)

### Project Structure
A typical Spring Boot project has:
- `src/main/java` or `src/main/kotlin`: Application code
- `src/main/resources`: Configuration files (like `application.properties`)
- `pom.xml` or `build.gradle`: Dependency management

### Main Class
Your entry point is a class with `@SpringBootApplication`:
```java
@SpringBootApplication
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}
```
- `@SpringBootApplication` is a convenience annotation that combines `@Configuration`, `@EnableAutoConfiguration`, and `@ComponentScan`.

### application.properties
This file configures your app. Example properties:
```
spring.application.name=MyApp
server.port=8080
```
- `spring.application.name`: Name of your app (used in logs, Spring Cloud, etc.)
- `server.port`: Port for the embedded server

---

## How Spring Boot Works: From Startup to Shutdown

This section explains, step by step, what happens when you start a Spring Boot application until it shuts down.

### 1. Application Launch
- You start the app with `java -jar myapp.jar` or from your IDE.
- The JVM loads your main class (the one with `public static void main`).
- The `main` method calls `SpringApplication.run(MyApp.class, args)`.

### 2. SpringApplication Initialization
- Spring Boot creates a `SpringApplication` object.
- It determines the type of application (web, reactive, CLI, etc.).
- It sets up the default environment (system properties, environment variables, `application.properties`/`application.yml`, command-line args).
- Listeners and initializers are registered (for events and custom startup logic).

### 3. ApplicationContext Creation
- Spring Boot creates the **ApplicationContext** (the Spring container).
- It scans your code for beans/components (`@Component`, `@Service`, `@Repository`, `@Controller`).
- It loads beans from auto-configuration classes (based on what's on the classpath and your properties).
- Beans are created and dependencies are injected (constructor, field, or setter injection).
- Any `@PostConstruct` methods are called.

### 4. Embedded Server Startup (for web apps)
- If your app is a web application, Spring Boot starts an embedded server (like Tomcat, Jetty, or Undertow).
- The server binds to the configured port (default 8080, or as set in `server.port`).
- DispatcherServlet (the main web request handler) is registered.

### 5. Application Ready
- Any beans implementing `CommandLineRunner` or `ApplicationRunner` are run (for startup logic).
- The application is now ready to handle HTTP requests, scheduled tasks, etc.
- Actuator endpoints (if enabled) are exposed for health checks, metrics, etc.

### 6. Request Handling (Web Apps)
- Incoming HTTP requests are received by the embedded server.
- Requests are routed to the DispatcherServlet.
- DispatcherServlet finds the right controller method (based on URL, HTTP method, etc.).
- Controller method executes, returns a response (JSON, HTML, etc.).
- Response is sent back to the client.

### 7. Runtime
- The app continues running, handling requests, scheduled jobs, etc.
- Beans can be refreshed or reloaded (with Actuator or `@RefreshScope`).
- You can interact with the app via REST endpoints, actuator endpoints, etc.

### 8. Shutdown Sequence
- When you stop the app (Ctrl+C, SIGTERM, or IDE stop), Spring Boot begins shutdown.
- The embedded server stops accepting new requests.
- The ApplicationContext is closed:
  - Calls `@PreDestroy` methods on beans.
  - Calls `DisposableBean` destroy methods.
  - Releases resources (database connections, thread pools, etc.).
- JVM exits.

**Summary:**
Spring Boot automates the entire lifecycle: from class loading, environment setup, bean creation, dependency injection, auto-configuration, server startup, request handling, to graceful shutdown. You focus on writing business logic—Spring Boot handles the rest.

---

## Spring Boot Lifecycle

Spring Boot applications go through a well-defined lifecycle:

1. **Bootstrap**
   - The `main` method calls `SpringApplication.run()`.
   - Spring Boot creates an `ApplicationContext` (the Spring container).
2. **Environment Preparation**
   - Loads properties from `application.properties`, environment variables, command-line args, etc.
   - Sets up profiles (e.g., `dev`, `prod`).
3. **Context Creation**
   - Scans for beans/components (`@Component`, `@Service`, `@Repository`, `@Controller`).
   - Performs dependency injection (wires beans together).
   - Applies auto-configuration (based on classpath and properties).
4. **Application Startup**
   - Runs any `CommandLineRunner` or `ApplicationRunner` beans (for startup logic).
   - Starts embedded web server (if web app).
   - Application is now running and ready to serve requests.
5. **Runtime**
   - Handles HTTP requests, scheduled tasks, etc.
   - Beans can be refreshed, reloaded, or replaced (with `@RefreshScope` or actuator endpoints).
6. **Shutdown**
   - On JVM shutdown (or SIGTERM), Spring calls `@PreDestroy` methods and closes the context.
   - Beans are destroyed in reverse order of creation.

**Lifecycle Hooks:**
- `@PostConstruct`: Method runs after bean is created and dependencies injected.
- `@PreDestroy`: Method runs before bean is destroyed.
- `InitializingBean` and `DisposableBean`: Interfaces for custom init/destroy logic.
- `ApplicationListener`: Listen for lifecycle events (context started, refreshed, closed, etc.).

---

## Core Spring Concepts

### Beans
A **bean** is an object managed by the Spring container. Beans are created, configured, and wired together by Spring.
- Declared with `@Component`, `@Service`, `@Repository`, or `@Controller` (or in XML/config).

### Dependency Injection (DI)
Spring injects dependencies into beans automatically:
- **Constructor injection** (recommended):
  ```java
  @Component
  public class MyService {
      private final MyRepository repo;
      public MyService(MyRepository repo) { this.repo = repo; }
  }
  ```
- **Field injection** (not recommended):
  ```java
  @Autowired
  private MyRepository repo;
  ```
- **Setter injection**: via setter methods.

### Configuration
- `@Configuration`: Marks a class as a source of bean definitions.
- `@Bean`: Declares a bean in a `@Configuration` class.
  ```java
  @Configuration
  public class AppConfig {
      @Bean
      public MyService myService() { return new MyService(); }
  }
  ```

### Profiles
- `@Profile`: Activate beans/config only for certain environments (e.g., `dev`, `prod`).
- Set active profile with `spring.profiles.active=dev` in `application.properties`.

### Auto-Configuration
Spring Boot auto-configures beans based on the classpath and properties. You can override any bean by defining your own.

### ApplicationContext
The **ApplicationContext** is the central interface to the Spring container. It manages beans, configuration, and the lifecycle.
- Types: `AnnotationConfigApplicationContext`, `WebApplicationContext`, etc.

### Actuator
Spring Boot Actuator adds production-ready features:
- Health checks (`/actuator/health`)
- Metrics (`/actuator/metrics`)
- Environment info, bean info, etc.
- Enable in `pom.xml`:
  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
  </dependency>
  ```
- Configure endpoints in `application.properties`:
  ```
  management.endpoints.web.exposure.include=*
  ```

### DevTools
Spring Boot DevTools enables hot reloading and developer-friendly features. Add to `pom.xml`:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
</dependency>
```

---

## Declaring Beans and Dependency Injection in Spring Boot

Spring Boot (and Spring in general) offers several ways to declare beans—objects managed by the Spring container. These beans can be from your own application code or from libraries (third-party or custom). Here are all the main ways to declare and manage beans:

### 1. Component Scanning (Annotations)
Spring automatically detects and registers classes annotated with:
- `@Component`: Generic component
- `@Service`: Service layer (specialized `@Component`)
- `@Repository`: Data access layer (adds exception translation)
- `@Controller` / `@RestController`: Web controllers

**Example:**
```java
@Component
public class MyComponent {}

@Service
public class MyService {}

@Repository
public class MyRepository {}
```
Spring scans the package of your main class (and subpackages) for these annotations.

### 2. `@Bean` Methods in `@Configuration` Classes
You can declare beans explicitly in a `@Configuration` class using `@Bean` methods. This is useful for beans from libraries or when you need to customize instantiation.

**Example:**
```java
@Configuration
public class AppConfig {
    @Bean
    public ObjectMapper objectMapper() {
        return new ObjectMapper(); // from Jackson library
    }
}
```

### 3. Importing Beans from Libraries (Third-Party or Custom)
Many libraries provide their own `@Configuration` classes or `@Component`-annotated classes. When you add the library to your dependencies, Spring Boot's auto-configuration will often pick them up automatically.
- Example: Adding `spring-boot-starter-data-jpa` brings in JPA repositories and entity manager beans.
- You can also import your own shared modules with beans.

### 4. Using `@Import`
You can import additional configuration classes (from your code or libraries) into your context.

**Example:**
```java
@Import(ExternalConfig.class)
public class MyConfig {}
```
This tells Spring to also process beans defined in `ExternalConfig`.

### 5. Conditional Beans
You can declare beans that are only created under certain conditions using annotations like `@ConditionalOnProperty`, `@ConditionalOnClass`, etc. (commonly used in Spring Boot auto-configuration).

**Example:**
```java
@Bean
@ConditionalOnProperty(name = "feature.enabled", havingValue = "true")
public MyFeatureBean featureBean() { ... }
```

### 6. External Beans via XML (Legacy)
You can still use XML files to declare beans (not common in modern Spring Boot, but supported for legacy reasons).

**Example:**
```xml
<bean id="myBean" class="com.example.MyBean"/>
```
And load with:
```java
@ImportResource("classpath:beans.xml")
```

### 7. Factory Beans
For advanced scenarios, you can use `FactoryBean<T>` to control bean instantiation logic.

**Example:**
```java
public class MyFactoryBean implements FactoryBean<MyBean> {
    @Override
    public MyBean getObject() { return new MyBean(); }
    @Override
    public Class<?> getObjectType() { return MyBean.class; }
}
```
Register with `@Component` or in a config class.

---

**How Spring Boot Manages Beans from Libraries:**
- Spring Boot's auto-configuration mechanism detects beans provided by dependencies (starters, third-party libraries, or your own modules).
- You can override any auto-configured bean by declaring your own bean of the same type or name.
- All beans—whether from your code or libraries—are managed in the same ApplicationContext and can be injected anywhere.

---

## Bean Scopes and Lifecycles in Spring Boot

Spring beans can have different scopes, which control how many instances are created, when they are created, and when they are destroyed. This applies to all beans, whether declared via annotations, `@Bean` methods, or imported from libraries.

### Standard Bean Scopes

| Scope         | Description                                      | Creation Time         | Destruction Time         | Typical Use Case           |
|---------------|--------------------------------------------------|-----------------------|--------------------------|----------------------------|
| singleton     | One instance per Spring container (default)      | On context startup    | On context shutdown      | Services, repositories     |
| prototype     | New instance every time bean is requested        | On each injection/use | Not managed by Spring*   | Statefull, short-lived obj |
| request       | One per HTTP request (web apps only)             | On HTTP request start | At request end           | Web controllers, request data |
| session       | One per HTTP session (web apps only)             | On session start      | At session end           | User session data          |
| application   | One per ServletContext (web apps only)           | On context startup    | On context shutdown      | Shared web app resources   |
| websocket     | One per WebSocket session                        | On WebSocket connect  | On WebSocket disconnect  | WebSocket handlers         |

*Prototype beans: Spring creates them but does not manage their full lifecycle (no destruction callbacks).

### How to Set Scope
- By default, all beans are singleton.
- Use `@Scope` annotation to change:
  ```java
  @Component
  @Scope("prototype")
  public class MyPrototypeBean {}
  ```
  Or for web scopes:
  ```java
  @Scope(value = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS)
  ```
- You can also set scope on `@Bean` methods:
  ```java
  @Bean
  @Scope("session")
  public MySessionBean mySessionBean() { ... }
  ```

### Bean Lifecycle Phases
1. **Instantiation**: Spring creates the bean instance.
2. **Dependency Injection**: Spring injects dependencies.
3. **Post-Processing**: Any `BeanPostProcessor` logic runs.
4. **Initialization**: Custom init logic runs:
   - `@PostConstruct` method (if present)
   - `afterPropertiesSet()` from `InitializingBean` (if implemented)
   - Custom init method (if specified in config)
5. **Bean is Ready**: Bean is available for use.
6. **Destruction**: On context shutdown (or scope end), Spring calls:
   - `@PreDestroy` method (if present)
   - `destroy()` from `DisposableBean` (if implemented)
   - Custom destroy method (if specified in config)

### Lifecycle Callbacks
- `@PostConstruct`: Runs after dependencies are injected.
- `@PreDestroy`: Runs before bean is destroyed.
- `InitializingBean.afterPropertiesSet()`: Alternative to `@PostConstruct`.
- `DisposableBean.destroy()`: Alternative to `@PreDestroy`.

**Example:**
```java
@Component
@Scope("prototype")
public class ExampleBean implements InitializingBean, DisposableBean {
    @PostConstruct
    public void init() { System.out.println("@PostConstruct"); }
    @Override
    public void afterPropertiesSet() { System.out.println("afterPropertiesSet"); }
    @PreDestroy
    public void cleanup() { System.out.println("@PreDestroy"); }
    @Override
    public void destroy() { System.out.println("destroy"); }
}
```

### Notes on Scoping and Accessibility
- **Singleton beans** are created at context startup and shared everywhere.
- **Prototype beans** are created on demand; Spring does not track or destroy them after creation.
- **Web scopes** (request, session, etc.) require a web-aware ApplicationContext (e.g., in Spring Boot web apps).
- Beans from libraries or `@Bean` methods can use any scope.
- You can inject a scoped bean into a singleton using proxies (Spring handles this automatically for web scopes if you set `proxyMode`).

### Summary Table

| Scope         | When Created         | When Destroyed         | Managed by Spring? | Supports Lifecycle Callbacks? |
|---------------|---------------------|------------------------|--------------------|------------------------------|
| singleton     | On context startup  | On context shutdown    | Yes                | Yes                          |
| prototype     | On each request     | Not managed            | Partially          | Only init, not destroy       |
| request       | On HTTP request     | At request end         | Yes                | Yes                          |
| session       | On session start    | At session end         | Yes                | Yes                          |
| application   | On context startup  | On context shutdown    | Yes                | Yes                          |
| websocket     | On WebSocket connect| On WebSocket disconnect| Yes                | Yes                          |

---

## Spring Data (JPA)

Spring Data JPA simplifies database access using repositories and entities.

### Dependencies
Add to your `pom.xml`:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
</dependency>
```

### Entity Example
```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String username;
    // ... other fields, getters, setters
}
```
- `@Entity`: Marks a class as a JPA entity (table)
- `@Id`: Primary key
- `@GeneratedValue`: Auto-generates the ID

### Repository Example
```java
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
}
```
- `JpaRepository`: Provides CRUD and query methods

### Configuration (Postgres Example)
In `application.properties`:
```
spring.datasource.url=jdbc:postgresql://localhost:5432/mydb
spring.datasource.username=myuser
spring.datasource.password=mypassword
spring.datasource.driver-class-name=org.postgresql.Driver
spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.default_schema=myschema
```
- `spring.datasource.url`: JDBC URL for your database
- `spring.jpa.hibernate.ddl-auto`: `update` auto-creates/updates tables
- `spring.jpa.properties.hibernate.default_schema`: Default schema

---

## Spring Security (with JWT & HTTPS)

# Tips, best practices and important info to work with spring security.

1. You can disable csrf if using jwt.
2. Cors need to be enabled. Mobile app wil ignore cors but browser will use the cors security. Just make sure that cors allows all hosts and methods.
3. Session based auth is where the server keeps track of the session and the  browser automatically sends the session id in a cookie in all requests to the same server.
4. In Token based auth, the token itself is sufficient for the server to identify or authenticate the user. Tokens are sent to client for storage and the server doesnot store them so that is why token based auth is stateless. Tokens are also not automatically sent by the browser in each request  so the hacker has to find out the token and send it with any harmful request to the same server which is impossible unless they got the token in the transit. To prevent this stealing of token in transit we can use https.
5. Token have to be sent as a authorization header to the server explicitly with each request.
6. In jwt, browser or any http client can never automatically put authorization bearer tokens because browsers dont remember or store that like a cookie. Cookies are also stored only if the backend server sends a "set cookie" header on the response of a login or signup request. So you can ignore csrf by using jwt in authorization bearer header of your evwry request. Cors attack is also very much impossible since even if attacker sends a request from some other origin, he wont have the jwt token to authenticate. Hacker can get hold of the token if stored insecurely in the browser but that is the problem of the client or client's browser not the backend.
7. To enables https you have to just make changes in the  `application.properties` file and create a keystore file which includes a private key and certificate using either keygen or using certificate authority. Keygen is only used for self signed certificates.
Keystore file is a file which stores private keys and its associated certificate. It has `.jceks` extension.
8. In spring security, an authentication filter tries to populate the security context holder object with an authentication object if the user is authenticated. The filters that come after that filter may use the security context holder to know whether request is authenticated or not by simply checking the authentication object present in the security context holder.
Even this authentication object is used in authorization process to check if the authneticated user has access a certain resouce or not using the granted authority on it.
9. When using jwt, session creation policy should be stateless. This means that that the security context which contains the authentication object is not stored in an session and most importantly no session is created ever.
10. When using session creation policy as stateless or never, no `set cookie` header is sent in the response.
11. In spring security, the filter that is authorizing your paths or request uri path is executed after authentication is done. So the permitAll() or authenticated() are evaluated after authentication is done. Spring always authenticates all requests. If not authenticated using the upstream filters like username and password filter or its replacement then spring automatically authenticates those request with anonymous authentication using anonymous authentication filter.
12. If the configuration that you have defined says permitAll() for any path then all requests whether properly authenticated or with anonymous authentication is allowed to  access the path. But if is configured with authenticated() then only requests authenticated properly instead of anonymous authentication are allowed to access. There is also a anonymous() method which allows only anonymous authenticated requests to be allowed to access corresponding path.
13. For jwt, always subclass the once per request filter and in the doFilterInternal method, provide your authentication logic where at the end you set an auhtnetication object in security context and put this context into the security context holder. And put this custom filter at the place of username and password authentication filter. This filter will authenticate requests. This will also allow this custom filter to exist before the anonymous authentication filter. Since you are handling authentication directly in the filter, you don't even need authentication manager or even worry about authentication provider.
14. But keep in mind that in the filter, if you see a request without jwt token , you should just call `doFilter` and `return` from the filter. Because these requests may be login or register paths. If they are not, then authorization filter will deny the request if they are not some public path. Since at our custom filter we are only trying to authenticate the request, thats all. Authorization filter is responsible for authorizing requests.


# JWT setup

JWT is the best way to implement security.

## pom.xml dependencies

Make sure to have these dependencies.

```
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-security</artifactId>
		</dependency>
		<dependency>
			<groupId>io.jsonwebtoken</groupId>
			<artifactId>jjwt-api</artifactId>
			<version>0.11.5</version>
		</dependency>
		<dependency>
			<groupId>io.jsonwebtoken</groupId>
			<artifactId>jjwt-impl</artifactId>
			<version>0.11.5</version>
		</dependency>
		<dependency>
			<groupId>io.jsonwebtoken</groupId>
			<artifactId>jjwt-jackson</artifactId>
			<version>0.11.5</version>
		</dependency>
```

## Jwt service to create and validate jwt tokens

You need to create a jwt service to create and validate tokens. 
The token has header,payload and signature.
Payload is the data that is carried in the token.
This payload has claims.
A claim is a key value pair.
There are some standard claims which are predefined in jwt libraries. Meaning that you can set them directly but other claims can also be included.
Some standard claims are: subject, expiration time etc. Many are these but mostly we should be conerned of these two.
Subject is the username or the id of the user who has sent the request.
Expiration time is the time till which session will not expire. After that it will expire and you have to request for new token.

Below is the function which creates token:
```
public String generateToken(WorkWiseUser userdetails){
		
		return Jwts.builder()
		.setIssuedAt(new Date())
		.setExpiration(new Date(System.currentTimeMillis()+expirationTimeInSeconds))
		.setSubject(userdetails.getUsername())
		.signWith(Keys.hmacShaKeyFor(key),SignatureAlgorithm.HS256)
		.compact();
		
	}
```
You have to write this code. Here `WorkWiseUser` is the userdetails object which is also the user entity. Here signing happens at the very end. Signing requires a key that you can generate from a secret string. That string can be anything but should be atleast 256 bytes. Signing requires both a key object and the algorithm with which signing will be done.

Below is the function which will do the verification of token

```
private boolean isTokenExpired(String token) {
		return Jwts.parserBuilder()
		.setSigningKey(key)
		.build()
		.parseClaimsJws(token)
		.getBody()
		.getExpiration().before(new Date());
	}

	public boolean validateToken(String token){
			Jwts.parserBuilder()
			.setSigningKey(key)
			.build()
			.parseClaimsJws(token)
			.getBody();
			return true && !isTokenExpired(token);
	}

```
Here, extracting data from token or validating token both require, `parseBuilder()` method of `Jwts` object. Here, we first set signing key to extract data. After that we parse claims from the token. `getBody()` to get the body of the payload. After that extract either username or expiration time .. whatever claims that you have set.

---

## Spring Cloud

Spring Cloud provides tools for building distributed systems and microservices. Key features:
- **Config Server**: Centralized configuration
- **Eureka**: Service discovery
- **Gateway**: API gateway
- **Resilience4j**: Circuit breakers, retries

### Example: Adding Spring Cloud Dependencies
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

### Config Server
- Store all your `application.properties` in a central Git repo
- Apps fetch config at startup

### Eureka Discovery
- Register each service with Eureka
- Services can find each other by name

### Gateway
- Route requests to microservices
- Add filters, security, rate limiting

### Resilience4j
- Add circuit breakers to REST calls
- Example property:
```
resilience4j.circuitbreaker.instances.myService.registerHealthIndicator=true
```

---

## Database Setup (Postgres Example)

// ... existing code from postgres-setup.md ...

---

## Running Spring Boot JARs

To run a Spring Boot JAR:
```
java -jar myapp.jar
```
Make sure your project uses the Spring Boot Maven plugin to package the JAR.

---
