# Spring Boot - Anotações

## Stereotype

Spring ApplicationContext is where Spring holds instances of objects that it has identified to be managed and distributed automatically. These are called beans.
Using the Inversion of Control principle, Spring collects bean instances from our application and uses them at the appropriate time.

### @Component

@Component is an annotation that allows Spring to automatically detect our custom beans.
In other words, without having to write any explicit code, Spring will:

- Scan our application for classes annotated with @Component
- Instantiate them and inject any specified dependencies into them
- Inject them wherever needed
However, most developers prefer to use the more specialized stereotype annotations to serve this function.

Spring has provided a few specialized stereotype annotations: @Controller, @Service and @Repository. They all provide the same function as @Component.

### @Service

Marks a Java class that performs some service, such as execute business logic, perform calculations and call external APIs. This annotation is a specialized form of the @Component annotation intended to be used in the service layer.

### @Repository

The @Repository annotation works as marker for any class that fulfills the role of repository or Data Access Object.

This annotation has a automatic translation feature. For example, when an exception occurs in the @Repository there is a handler for that exception and there is no need to add a try catch block.

### @Controller

The @Controller annotation is used to indicate the class is a Spring controller. This annotation can be used to identify controllers for Spring MVC or Spring WebFlux.

### @RestController

The @RestController annotation marks the class as a controller where every method returns a domain object instead of a view. By annotating a class with this annotation you no longer need to add @ResponseBody to all the RequestMapping method. It means that you no more use view-resolvers or send html in response. You just send the domain object as HTTP response in the format that is understood by the consumers like JSON.

@RestController is a convenience annotation which combines @Controller and @ResponseBody.

## Core

### Beans

#### @AutoWired

This annotation is applied on fields, setter methods, and constructors. The @Autowired annotation injects object dependency implicitly.
The object injected needs to be a bean.
When you use @Autowired on a constructor, constructor injection happens at the time of object creation.
As of Spring 4.3, @Autowired became optional on classes with a single constructor.

#### @Qualifier

This annotation is used along with @Autowired annotation. When you need more control of the dependency injection process, @Qualifier can be used. @Qualifier can be specified on individual constructor arguments or method parameters. This annotation is used to avoid confusion which occurs when you create more than one bean of the same type and want to wire only one of them with a property.
With the @Qualifier annotation added (@Qualifier("beanB2")), Spring will now know which bean to autowire where beanB2 is the name of BeanB2.

#### @Value

This annotation is used at the field, constructor parameter, and method parameter level. The @Value annotation indicates a default value expression for the field or parameter to initialize the property with. As the @Autowired annotation tells Spring to inject object into another when it loads your application context, you can also use @Value annotation to inject values from a property file into a bean’s attribute. It supports both #{...} and ${...} placeholders.

### Context

#### @Configuration

This annotation is used on classes which define beans, but itself is a bean too. @Configuration is an analog for XML configuration file – it is configuration using Java class. Java class annotated with @Configuration is a configuration by itself and will have methods to instantiate and configure the dependencies.

#### @ComponentScan

This annotation is used with @Configuration annotation to allow Spring to know the packages to scan for annotated components. @ComponentScan is also used to specify base packages using basePackageClasses or basePackage attributes to scan. If specific packages are not defined, scanning will occur from the package of the class that declares this annotation.

#### @Bean

This annotation is used at the method level. @Bean annotation works with @Configuration to create Spring beans. The method annotated with this annotation works as bean ID and it creates and returns the actual bean.

#### @Lazy

This annotation is used on component classes. By default all autowired dependencies are created and configured at startup. But if you want to initialize a bean lazily, you can use @Lazy annotation over the class. This means that the bean will be created and initialized only when it is first requested for. You can also use this annotation on @Configuration classes. This indicates that all @Bean methods within that @Configuration should be lazily initialized.

#### @Primary

In some cases, we need to register more than one bean of the same type. To access beans with the same type we usually use **@Qualifier(“beanName”)** annotation.
Indicates that a bean should be given preference when multiple candidates are qualified to autowire a single-valued dependency. If exactly one 'primary' bean exists among the candidates, it will be the autowired value.
May be used on any class directly or indirectly annotated with @Component or on methods annotated with @Bean.


#### @Scope

When used as a type-level annotation in conjunction with @Component, @Scope indicates the name of a scope to use for instances of the annotated type.
When used as a method-level annotation in conjunction with @Bean, @Scope indicates the name of a scope to use for the instance returned from the method.
In this context, scope means the lifecycle of an instance, such as singleton, prototype, and so forth.

#### @PropertySource

Annotation providing a convenient and declarative mechanism for adding a PropertySource to Spring's Environment. To be used in conjunction with @Configuration classes.
Allow properties files customized.
```java
@Configuration
@PropertySource("classpath:/com/myco/app.properties")
public class AppConfig {
```

#### @Profile

Profiles are a core feature of the framework — allowing us to map our beans to different profiles — for example, dev, test, and prod.
We can then activate different profiles in different environments to bootstrap only the beans we need.
We use the @Profile annotation — we are mapping the bean to that particular profile; the annotation simply takes the names of one (or multiple) profiles.
```java
@Component
@Profile("dev")
public class DevDatasourceConfig
```

Set Profiles: This can be done in a variety of ways: 
- Programmatically via WebApplicationInitializer Interface - servletContext.setInitParameter("spring.profiles.active", "dev");
- Programmatically via ConfigurableEnvironment - env.setActiveProfiles("someProfile");
- Context Parameter in web.xml
- JVM System Parameter - -Dspring.profiles.active=dev
- Environment Variable - export spring_profiles_active=dev
- Maven Profile
- @ActiveProfile in Tests - @ActiveProfiles("dev")


## Boot

### @SpringBootApplication

Alias For: @ComponentScan, @Configuration and @EnableAutoConfiguration
The class that is annotated with the @SpringBootApplication must be kept in the base package. The one thing that the @SpringBootApplication does is a component scan. But it will scan only its sub-packages.

### @EnableAutoConfiguration

This annotation is usually placed on the main application class. The @EnableAutoConfiguration annotation implicitly defines a base “search package”. This annotation tells Spring Boot to start adding beans based on classpath settings, other beans, and various property settings.

### @ConfigurationProperties

Annotation for externalized configuration. Add this to a class definition or a @Bean method in a @Configuration class if you want to bind and validate some external Properties (e.g. from a .properties file).
Note that contrary to @Value, SpEL expressions are not evaluated since property values are externalized.
`@ConfigurationProperties(prefix = "app")`

## Web

### @RestController

Already bove.

### @RequestMapping

This annotation is used both at class and method level. The @RequestMapping annotation is used to map web requests onto specific handler classes and handler methods. When @RequestMapping is used on class level it creates a base URI for which the controller will be used. When this annotation is used on methods it will give you the URI on which the handler methods will be executed. From this you can infer that the class level request mapping will remain the same whereas each handler method will have their own request mapping.
```java
@RequestMapping("/api") - class
@RequestMapping(value = "/find", method = RequestMethod.GET) - method
```

### @GetMapping

This annotation is used for mapping HTTP GET requests onto specific handler methods. @GetMapping is a composed annotation that acts as a shortcut for @RequestMapping(method = RequestMethod.GET)

### @PostMapping

This annotation is used for mapping HTTP POST requests onto specific handler methods. @PostMapping is a composed annotation that acts as a shortcut for @RequestMapping(method = RequestMethod.POST)

### @PutMapping

This annotation is used for mapping HTTP PUT requests onto specific handler methods. @PutMapping is a composed annotation that acts as a shortcut for @RequestMapping(method = RequestMethod.PUT)

### @PatchMapping

This annotation is used for mapping HTTP PATCH requests onto specific handler methods. @PatchMapping is a composed annotation that acts as a shortcut for @RequestMapping(method = RequestMethod.PATCH)

### @DeleteMapping

This annotation is used for mapping HTTP DELETE requests onto specific handler methods. @DeleteMapping is a composed annotation that acts as a shortcut for @RequestMapping(method = RequestMethod.DELETE)

### @RequestBody

This annotation is used to annotate request handler method arguments. The @RequestBody annotation indicates that a method parameter should be bound to the value of the HTTP request body. The HttpMessageConveter is responsible for converting from the HTTP request message to object.
`... update(@RequestBody @Valid ParkingSpotDto parkingSpotDto)`

### @PathVariable

This annotation is used to annotate request handler method arguments. The @RequestMapping annotation can be used to handle dynamic changes in the URI where certain URI value acts as a parameter. You can specify this parameter using a regular expression. The @PathVariable annotation can be used declare this parameter.
`... get(@PathVariable(value = "id") UUID id)`

### @RequestParam

This annotation is used to annotate request handler method arguments. Sometimes you get the parameters in the request URL, mostly in GET requests. In that case, along with the @RequestMapping annotation you can use the @RequestParam annotation to retrieve the URL parameter and map it to the method argument. The @RequestParam annotation is used to bind request parameters to a method parameter in your controller.
`... get(@RequesteParam(value = "id", required = true) UUID id)`

### @CrossOrigin

This annotation is used both at class and method level to enable cross origin requests. In many cases the host that serves JavaScript will be different from the host that serves the data. In such a case Cross Origin Resource Sharing (CORS) enables cross-domain communication. To enable this communication you just need to add the @CrossOrigin annotation.

By default the @CrossOrigin annotation allows all origin, all headers, the HTTP methods specified in the @RequestMapping annotation and maxAge of 30 min. You can customize the behavior by specifying the corresponding attribute values.

An example to use @CrossOrigin at both controller and handler method levels is this.
```java
@CrossOrigin(maxAge = 3600)
@RestController
@RequestMapping("/account")
public class AccountController {

@CrossOrigin(origins = "http://example.com")
@RequestMapping("/message")
    public Message getMessage() {
      // ...
    }
 
@RequestMapping("/note")
    public Note getNote() {
        // ...
    }
}
```
In this example, both getExample() and getNote() methods will have a maxAge of 3600 seconds. Also, getExample() will only allow cross-origin requests from http://example.com, while getNote() will allow cross-origin requests from all hosts.



Sources:
https://www.baeldung.com/spring-component-annotation
https://www.baeldung.com/spring-primary
https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Primary.html
https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Scope.html
https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/PropertySource.html
https://www.baeldung.com/spring-profiles
https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/context/properties/ConfigurationProperties.html
https://springframework.guru/spring-framework-annotations/
https://www.youtube.com/watch?v=Pd5tr483No0