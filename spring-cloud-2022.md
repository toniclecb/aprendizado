# Spring Cloud
Frank P Moley III

Spring Cloud is about Cloud native
meaning we can run in a local private cloud, a global public cloud or anything in between, including big iron servers.
It is designed to provide tooling for many of the most common patterns in cloud data computing.

One of the biggest early wins with Spring Cloud is centralized but distributed configuration management.

service discovery is a real issue in many systems. Spring Cloud provides a pattern and tooling for this task.
Circuit breakers are a big deal in microservices as well. And Spring Cloud, through third party offerings as well as Spring offerings, provides robust software load balancing as well as circuit breaking to isolate your system as a whole from minor hiccups from an individual service.

Telemetry, which comes from the 15 factor pattern is built in with Spring Cloud Sleuth.

It has an open service broker package for working with the open service broker API to expose a service directly to a runtime.

There is native integrations with cloud providers via their SDKs with first class support through a Spring boot starter.

## External configuration

By leveraging externalized config, we can write our code so it runs in any environment and we leverage the configuration values to change the data flow, but not the code itself
In addition, we can leverage externalized config so our code runs in any runtime, be it a docker container or a big iron server. Because the code is indifferent to the runtime, only the configuration should be adjusted.
Your system doesn't need to know where the dependency is, just how to consume its API.

Spring's config server is the availability of version control. Because you can and should leverage Git with your externalized config, you could require PRs to push code into environment specific branches or files giving you full control of the evaluation of that config before it goes into an environment.

### Spring Cloud config server

**Setup config server**

etc/room-reservation-service.properties:
```
GUEST_SERVICE_URL=http://localhost:8081
RESERVATION_SERVICE_URL=http://localhost:8082
ROOM_SERVICE_URL=http://localhost:8083
```

in main class:
```
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {
```
in application.properties:
```
server.port=8888
spring.cloud.config.server.git.uri=${HOME}/etc
```

calling http :8888/room-service.properties have to respond with configuration present in the file!

**Consuming config server**

in pom.xml:
dependencyManagement: <artifactId>spring-cloud-dependencies
<dependency>: <artifactId>spring-cloud-starter-config

in application.properties: (we change for)
spring.application.name=room-reservation-service
spring.config.import=optional:configserver:http://localhost:8888

the local properties were changed for remote properties from config server
we centrelazied configuration and now use everywhere we want

### Service discovery

especially more important as you move to a microservices-based architecture.
Service Discovery starts with a service that exposes to the system what domain it is servicing. Consuming services can then look for a service that publishes the domain. The API will return the service URL and port that is serving the domain the consumer needs.

Eureka is a Netflix OSS project developed for AWS service consumption and exposition. It provides both the server side infrastructure, as well as a console for viewing your services in the wild.

**Setting up Eureka Server**

dependency: Spring Cloud Discovery > Eureka Server

@EnableEurekaServer

server.port=8761
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
eureka.datacenter=localhost

**Registering with Eureka**

Dependency: <artifactId>spring-cloud-starter-netflix-eureka-client

in main class:
@EnableDiscoveryClient // dont need configure port in properties because we are using default port in eureka server
The annotation @EnableDiscoveryClient is used to expose the service to Eureka based on default or overridden configuration.


**Consuming services**

and we are going to add @LoadBalanced to our **rest template** (already annotate with @Bean)
The @LoadBalanced annotation is used to load balance service calls across all instances discovered from Eureka.
the url will change to:
private String reservationServiceUrl = "http://reservation-service";
this is the name of the services registered.

Eureka notices a change to the room service, let's say, the port changes or the URL changes, it will now expose the new room service URL via the Eureka console and API as opposed to us having to restart the app and pull it from somewhere else.

## Consuming with OpenFeign

We'll now take a look at using OpenFeign as a client interface with Eureka instead of RestTemplate.
we're going to remove existing code that utilizes RestTemplate and replace it with an OpenFeign client.

So what it actually does is it wraps RestTemplate with basically the logic that we had implemented, which is just build the RestTemplate call, but it does it on an interface, instead of us actually having to write out the RestTemplate. So Feign becomes a very quick and easy way to build a service and consumer service in a Spring based application when you are using Eureka.

dependency: spring-cloud-stater-openfeign
in main: @EnableFeignClient // This will allow component scanning of these interfaces so that they now become beans that we can inject.
@Service can change to @FeignClient("room-service") // The spring.application.name is what is exposed via Eureka that the feign client consumes.

```
@FeignClient("room-service")
public interface RoomServiceClient {

    @GetMapping("/rooms")
    List<Room> getAll();

    @PostMapping("/rooms")
    Room addRoom(@RequestBody Room room);

    @GetMapping("/rooms/{id}")
    Room getRoom(@PathVariable("id") long id);

    @PutMapping("/rooms/{id}")
    void updateRoom(@PathVariable("id") long id, @RequestBody Room room);

    @DeleteMapping("/rooms/{id}")
    void deleteRoom(@PathVariable("id") long id);
}
```

### Circuit breaking

Circuit breaking is an important pattern for building a highly resilient system

Now, Spring has had several different ways of implementing circuit breakers, and they're all still essentially available, but with Feign, they've given us a really clean way to implement it without any external services.

This built in support with Feign relies on the fact that Feign itself provides an interface, and we can leverage that. We create an implementation of that interface and define it as a fallback for the interface itself. That way we can have two instances of the same interface in the IOC container and not break any of the wiring.

dependency: spring-cloud-starter-circuitbraker-resilience4j
properties:
feign.circuitbreaker.enabled=true

The feign.circuitbreaker.enabled property, when set to true, enables Feign circuit breakers.

```
@Component
public class GuestServiceFallback implements GuestServiceClient {

  @Override
  public List<Guest> getAll() {
    return new ArrayList<>();
  }

  @Override
  public Guest addGuest(Guest guest) {
    return new Guest();
  }

  @Override
  public Guest getGuest(long id) {
    return new Guest();
  }

  @Override
  public void updateGuest(long id, Guest guest) {

  }

  @Override
  public void deleteGuest(long id) {

  }
}
```

Into our service client itself and we need to add a property to the @FeignClient. So we are going to implement a fallback, and the fallback is going to be GuestServiceFallback.class. Now at this point, everything is actually wired. We've brought in a circuit breaker in the form of resilience 4J. We've implemented a fallback pattern and we have told Feign which one to use if this service fails. So let's go back to our services. We've got guest service up.
```
@FeignClient(value = "guest-service", fallback = GuestServiceFallback.class)
public interface GuestServiceClient {
```

### API gateways

Spring now provides an out-of-the-box API gateway that you can use very similar to other tools that we've seen today.

First and foremost, it's about routing to registered APIs. Many times, you'll use a gateway as an entry point into your system, and then it knows how to get to the APIs as appropriate, which allows you to upgrade APIs behind the scenes and roll over to the new versions.

It also gives you a place through filters that allow you to have centralized security so that you don't have to provide security to each of your services if they're internal only doing so through a gateway. In a similar fashion, it allows you to have centralized observability so that all of your systems through logging and tracing can apply at the gateway instead of the individual services as appropriate.

It also provides resiliency across your fleet because the API gateway is responsible for routing.

Spring's Gateway works much the same as Config Server and Eureka Server in that you simply apply a Spring Boot application using the gateway as the dependency and then turn it on.

a route is the basic principle of the gateway itself. In Spring, it is defined by an ID, the destination URI, predicates, as well as filters.

### Telemetry

Remote debugging across service boundaries that we find in a microservices based architecture, is very, very hard, if not impossible. Tracing, however, gives us insight into system behavior across those service boundaries.

from a Spring Cloud perspective, they have an entire package called Spring Cloud Sleuth that solves much of your tracing needs. It provides out of the box autoconfiguration for distributed tracing. It adds trace IDs as well as spans to all requests across service boundaries. 

Sleuth itself has a wide range of services that it supports. It supports Brave as well as the very popular Zipkin. It also has some support for OTel.

Spring Cloud Slueth is designed to provide distributed tracing support.

