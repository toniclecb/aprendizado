Spring: Spring Cloud (2020)
Frank P Moley III

Microservices aren't really a new concept in software engineering. But they're more an evolution on how software is written.
Microservices focus on domain-driven designs.
Every Web application, a cron job, Web service, whatever, should be for single use and single purpose.

Spring Cloud brings in the cloud-native aspects to Spring Boot. Now, you can write Spring Boot applications without using the Spring Cloud abstractions and still hold to cloud-native principles. However, Spring Cloud definitely provides some much valued abstractions.

## why would I ever externalize our configuration?

The first and probably the most prevalent reason is because the 12-factor methodologies specifically state that configuration should be externalized. Another reasoning behind this and why it's more important than just that they say it, is that it becomes more and more clear as you start working in this space how powerful externalized configurations are.
Now one of the main reasons that we want to do this is because it increases the portability of your application.

Config server is backed by a git repository, so you can apply source control to your configuration values.
config server gives a centralized management point of all of our configuration variables.
we have one central place that we can distribute it to all of our applications running within that data center.
It's actually a very simple Spring boot starter project.

Now I briefly alluded to it a moment ago about how to consume this config. Indeed, it is actually available through a REST API call
Spring has given us a starter project for actually becoming a client of the config server.

### Set up external configuration

New module (spring initializr)
dependency: "Spring Cloud Config"

In application.properties:
server.port=9000
spring.cloud.config.server.git.uri=${HOME}/Desktop/config
spring.cloud.config.server.git.force-pull=true

In ConfigServerApplication (main):
add @EnableConfigServer

localhost:9000/guestservices/default - properties of guest-services project (module)

### Consume external configuration
in another module

in pom.xml:
```
<properties>
	<spring-cloud.version>Hoxton.RC2
...
<dependency>
	<groupId>org.springframework.cloud
	<artifactId>spring-cloud-config-client
...
<dependencyManagement>
	...
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-dependencies</artifactId>
	<version>${spring-cloud.version}</version>
	<type>pom</type>
	<scope>import</scope>
...
<repositories>
	<repository>
		<id>spring-milestones</id>
		<name>Spring Milestones</name>
		<url>https://repo.spring.io/milestone</url>
...
```

new file: bootstrap.properties:
```
spring.application.name=guestservices
spring.cloud.config.uri=http://localhost:9000
```

## Service discovery with Eureka

Eureka includes both a console and a discovery platform for registering and identifying all of your services within a distributed system.
Eureka provides a very clean, again, REST-based interface, for both registration and discovery. It has a default round-robin load balancing client, and best of all, for our purposes, there's easy integration with Spring
Eureka Server itself is very much like Config Server

Create a new module;
dependency: Spring Cloud Discovery - Eureka Server
annotate the main with @EnableEurekaServer
in application.properties:
server.port=8761
eureka.client.register-with-eureka=false # don't register this service in the server itself
eureka.client.fetch-registry=false # We don't need to do pulling operations in this case.

Eureka loads a web application!

### Register the microservice with Eureka

it's time to go register a service with Eureka

in pom.xml:
dependency: spring-cloud-stater-netflix-eureka-client

in main class:
@EnableDiscoveryClient

The services will appear in Eureka web application!

### Consuming services with Ribbon

Ribbon provides us a way to integrate a software load balancer to our web service calls.

new module: dependency: Spring Cloud Routing > Ribbon
dependency: spring-cloud-stater-netflix-eureka-client
anothers in Spring Cloud Routing: Zuul, Gateway, OpenFeign, Cloud LoadBalancer

properties:
spring.application.name=roomreservationservices
server.port=8080
eureka.client.serviceUrl.defaultZone=http://localhost:8761/eureka

in main class:
@EnableDiscoveryClient

@Bean
@LoadBalanced
public RestTemplate restTemplate(){
	return new RestTemplate();

we're going to use the load balanced annotation on this bean, and what load balance does is its going to tell Ribbon to use this across a load balancing routine. All right, so we are going to create an instance of REST template, simply call it that.

```
@RestController
@RequestMapping("/room-reservations")
public class RoomReservationWebService {
    ...

    private List<Room> getAllRooms(){
        ResponseEntity<List<Room>> roomResponse = this.restTemplate
				.exchange("http://ROOMSERVICES/rooms", 
					HttpMethod.GET,
					null,
					new ParameterizedTypeReference<List<Room>>() {
        });
        return roomResponse.getBody();
    }
}
```
Now we need a URL, so what URL can we use? Well in this case, we are going to use a URL that consumes Eureka. So, the room services is the actual name of the element from Eureka itself. Notice we're not putting in a real URL for our service. So Eureka's going to handle that and it's going to get aspected in. This is a get call with null and we need to return a new parameterized type reference of this room.

### Consume an interface with Feign

let's create a new dependency. And this is going to be on spring-cloud-starter-openfeign

and open up our application class, and we're going to add an annotation for @EnableFeignClients

we need to create a new interface, and we are going to call this RoomClient. And we are going to put a FeignClient on here.

This will permit eliminate the Bean RestTemplate!
```
@FeignClient("roomservices")
public interface RoomClient {
    @GetMapping("/rooms")
    List<Room> getAllRooms();
    @GetMapping("/rooms/{id}")
    Room getRoom(@PathVariable("id")long id);

}
```
instead of restTemplate the data will came from roomClient -> private final RoomClient roomClient;

The difference is now we are using the Eureka system through Feign with Ribbon all with an interface and no code being written. That's super powerful. And now it's time for you to jump up to the next challenge.

--

We then leveraged that service discovery to consume our services, and that's really where the power starts to come in of Spring Cloud in a cloud-native microservices-based architecture.

