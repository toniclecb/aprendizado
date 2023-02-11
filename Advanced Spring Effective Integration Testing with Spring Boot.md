Advanced Spring: Effective Integration Testing with Spring Boot
Terezija Semenski

Spring Boot applications are built in multiple layers data, service and web. In unit testing, we test each of these layers independently. For example, we test the web controllers that mock out the business services.

A unit test targets a small unit of code for example, a method or a class.
An integration test on the other hand can be anything of the following, a test that covers multiple units because it tests the interaction between two or more components which is spring we refer to spring beans. 
So whenever we are moving beyond unit testing and start integration testing, we should use SpringBootTest support to create an application context containing all the objects we need for all the above test types.

## Tools

JUnit5, the de facto standard for unit testing Java applications, 
SpringBootTest Utilities and Integration Test Support for Spring Boot applications,
Mockito, a Java mocking framework,
AssertJ fluent API, 
JSON Assert, an assertion library for JSON, 
JSONPath, Xpath

## Given-when-then methodology

arrived from a very popular AGL development process called BDD or behavior-driven development.

Given is where we put our preconditions and requirements for this use case.
When is the action that we want to test. 
Then is used to verify what should happen after the execution of the action.

### JUnit vs. AssertJ

Actually JUnit assertions are quite limited to a few basic scenarios and lead to confusion when reading them.,
Compare that to a AssertJ which provides a much more natural and fluent API, AssertJ provides a rich set of assertions, truly helpful error messages, improves test code readability and is designed to be super easy to use.
Note that a AssertJ also comes with BDD friendly alias.

AssertJ: assertThat(person.getAge()).isEqualTo(40);
BDDAssertions: then(person.getAge()).isEqualTo(40);

AssetJ is a fluent assertions Java library that comes with:
* functional style with lambda expressions;
* BDD style of assertion;
* separation of cached exception from assertion;

## Effective ways to test your data access

Key one, choosing the right interactions to test.
Key two, make your tests fast by using test slices. Running integration tests in spring can be achieved by adding spring boot test annotation, which loads full spring context. However, this can lead to loading unnecessary parts of your application. Each test slide, loads only the parts of the configuration that are required to test the slice of your application which can drastically speed up your tests.
Key three, in memory database doesn't equal production database. Springboard provides a convenient way to set up an environment with an embedded database such as age two or age as SQL to test our database interactions. 

## Writing integration tests for a JPA repository

there are two different approaches we can use from the smallest component all the way up which is called bottom up approach or inside out. This approach is great When we have two different teams working independently on a different services and we are not so worried about integration.

We will write our tests first and then the actual implementation. This is known as TDD and red-green refactor. 

```
@DataJpaTest // instead of @SpringBootTest that will load all the spring application!
public class StudentRepositoryTest
{

    @Autowired
    private StudentRepository studentRepository;

    @Autowired
    private TestEntityManager testEntityManager;

    @Test
    void testGetStudentByName_returnsStudentDetails()
    {
        //given
        Student savedStudent = testEntityManager.persistFlushFind(new Student(null, "Mark"));

        //when
        Student student = studentRepository.getStudentByName("Mark");

        //then
        then(student.getId()).isNotNull();
        then(student.getName()).isEqualTo(savedStudent.getName());
    }

    @Test
    void getAvgGradeForActiveStudents_calculatesAvg()
    {
        //given
        Student mark = Student.builder().name("Mark").active(true).grade(80).build();
        Student susan = Student.builder().name("Susan").active(true).grade(100).build();
        Student peter = Student.builder().name("Peter").active(false).grade(50).build();
        Arrays.asList(mark, susan, peter).forEach(testEntityManager::persistFlushFind);

        //when
        Double avgGrade = studentRepository.getAvgGradeForActiveStudents();

        //then
        then(avgGrade).isEqualTo(90);
    }

}
```

## Writing integration tests for the service layer

We want to test that we are able to correctly fetch our student by making a request to the repository and assure the response is as expected.

SpringBootTest works by reading configuration from SpringBootApplication and creates an application context very similar to the one that would be started in a production environment.

SpringBootTest is a very convenient for integration tests that go through all layers of the application, however, in this test, we want to isolate all the services and repositories, and since the WebEnvironment is available in a class puff, by default, SpringBootTest will load the mock WebEnvironment. In order to prevent that, we need to add WebEnvironment equals **NONE**.

```
@SpringBootTest(webEnvironment = NONE)
@Transactional
public class StudentServiceTest
{

    @Autowired
    private StudentRepository studentRepository;

    @Autowired
    private StudentService studentService;

    @DisplayName("Returning saved student from service layer") // If we want to give a more meaningful description
    @Test
    void getStudentById_forSavedStudent_isReturned()
    {
        //given
        Student savedStudent = studentRepository.save(new Student(null, "Mark"));

        //when
        Student retrievedStudent = studentService.getStudentById(savedStudent.getId());

        //then
        then(retrievedStudent.getName()).isEqualTo("Mark");
        then(retrievedStudent.getId()).isNotNull();

    }
}
```

Keep in mind that SpringBootTest does not contain transactional notation, like there's data GPI test slice, which means any interaction will not get rolled back. This means that any interaction with DB stays persisted and can cause side effects for other tests running at the same time, so we need to add Transactional.

## Writing integration tests for cache

Ever wonder how to test a hard problem, like caching? We use the cache, to protect the database or to avoid cost-intensive calculations. And spring provides an abstraction layer, for implementing a cache.

Since we need to test interaction with a repository, this means, we need the kind of way to check how many times, our database has been called. What we need is a Mock.
With Mockito Extension Unit Test, we can use Mock and Spy, to define mocked objects, and on the tested class at Inject Mock. Mockito will then try to instantiate fields, annotated with inject mocks by passing all the mocks in Togo structure or by using Setter Injection.
Mocking is also useful, when we want to simulate , failures that might be hard to trigger, in a real environment. Spring Boot provides, the Mock Bean and Spy Bean, annotations for this purpose.
In order to verify interactions, we cannot use a real Bean. We need mock one.

```
@SpringBootTest(webEnvironment = NONE) // to test interactions between database and service layer, but want to avoid loading Web layer of Spring application
public class StudentCacheTest
{

    @Autowired
    private StudentService studentService;

    @MockBean
    private StudentRepository studentRepository;

    @Test
    void getStudentById_forMultipleRequests_isRetrievedFromCache()
    {
        //given
        Long id = 123l;
        given(studentRepository.findById(id)).willReturn(Optional.of(new Student(id, "Mark")));

        //when
        studentService.getStudentById(id);
        studentService.getStudentById(id);
        studentService.getStudentById(id);

        //then
        then(studentRepository).should(times(1)).findById(id); // the method is called only 1 times? yes, because of @Cacheable!
    }
}
---
class StudentService {
	@Cacheable // spring annotation!
	public Student GetStudentById(Long id){ ...
```


## Testing Web Controllers

How do you ensure that endpoint is really exposed on a certain URL? How can you ensure that proper validation has been performed? And how can you prove that exceptions in your applications are translated into right messages and status code for the user? 

WebMvcTest loads components required for testing Mvc parts of application. WebMvcTest also auto configures MockMvc. MockMvc offers a powerful way to quickly test Mvc controllers without need to start a full HTTP server.

Web Slice will not cover testing service component or repository beans? So it is often used in combination with MockBean to provide mock implementations for required collaborators.

### Writing integration tests for a web controller

We will create a new student controller test. We are going to test if we can correctly make a request to students service and get the response we expect.

We will use mock MVC Spring's MVC test braver to perform HTTP requests on the web in-points inside our mock web environment. Since the mock MVC bean is loaded into the context we will be able to simply auto wire it for our use.

this is a test slide where we easily did only the web tier from everything else. The collaborating object student service was not available. We need to provide that object as a mock. So we are going to add student service and annotate it with a mock bean annotation and mock its behavior.

The way mock bean works is it try to find any bean in the application context of its type. If it exists, it will replace it. If it doesn't exist, it will add it. 

will not be loaded with @WebMvcTest: @Repository, @Component, @Service.

Types of web controller responsibilities should we cover with integration tests:
* input and output conversion to and from Java objects
* URL mapping, input validation
* business logic and exception translation

```
@WebMvcTest // you want to test only the MVC part of your application and avoid loading the service or database layer
public class StudentControllerTest
{
    @MockBean
    private StudentService studentService;

    @Autowired
    private MockMvc mockMvc;

    @Test
    void getStudent_ForSavedStudent_Returned() throws Exception
    {
        //given
        given(studentService.getStudentById(anyLong())).willReturn(
                Student.builder()
                       .id(1l)
                       .name("Mark")
                       .grade(10)
                       .build()
        );

        //when then
        mockMvc.perform(get("/students/1"))
               .andExpect(status().isOk())
               .andExpect(jsonPath("id").value(1l)) // We want to prove that Jason will contain attribute ID with value one
               .andExpect(jsonPath("name").value("Mark"))
               .andExpect(jsonPath("grade").value(10));
    }

    @Test
    void getStudent_ForMissingStudent_status404() throws Exception
    {
        //given
        given(studentService.getStudentById(anyLong())).willThrow(StudentNotFoundException.class);

        //when then
        mockMvc.perform(get("/students/1"))
               .andExpect(status().isNotFound());
    }
}
```

## Testing custom exception returns

Test 404 not found status response!


```
@WebMvcTest
public class StudentControllerTest
{
    @MockBean
    private StudentService studentService;

    @Autowired
    private MockMvc mockMvc;

    ...

    @Test
    void getStudent_ForMissingStudent_status404() throws Exception
    {
        //given
        given(studentService.getStudentById(anyLong())).willThrow(StudentNotFoundException.class);

        //when then
        mockMvc.perform(get("/students/1"))
               .andExpect(status().isNotFound());

    }

}
```

## Integration testing without making an external API call

Your best option is to mock out the service that you're calling. In the Spring world, we use the rest template or a WebClient to interact with external APIs. You could mock out rest template by mocking rest template interfaces.

A better option is to use rest client test slides which comes with a pre-configured mock rest service server. Mock rest service server provides a way to set up expected requests that will be performed through the rest template, as well as mock responses to send back thus removing the need for actual server.

The Spring team recommends using OkHTTPS mock web server or WireMock.

Why should we consider mocking external API calls in our tests ?
* We have additional complexity of handling credentials to external service.
* service might not be available
* service has rate limits

### Writing integration tests for rest endpoints

Let's dive in and write a consumer for a service and see how we can write a meaningful integration test even when the student service is not available.

@RestClientTest slice and MockRestServiceServer: when you need to test RestTemplate interactions with external API.

```
@SpringBootTest
@AutoConfigureStubRunner(ids = "com.linkedin:student-service:+:8080", stubsMode = LOCAL)
public class StudentClientTest
{
    @Autowired
    private StudentClient studentClient;

    @Test
    void getStudent_forGivenStudent_isReturned()
    {
		stubFor(get("/students/" + id).willReturn(okJson("""
			"id": ',
			"studentName": "Mark",
			"grade": 10
		"""))
        //given
        Long studentId = 1l;

        //when
        Student student = studentClient.getStudent(studentId);

        //then
        then(student.getId()).isNotNull();
        then(student.getName()).isEqualTo("Mark");
        then(student.getGrade()).isEqualTo(10);

    }
}
```

## Introduction to Spring Cloud Contract

The main idea of the Spring Cloud Contract is to give you very fast feedback without the need to set up the whole world of microservices. Client applications work with stubs.

Imagine that before two applications communicate with each other, they've formalized a way they send and receive messages. On the producer side, we can define contract in Java, Groovy, Kotlin and YAML by specifying expected request and response. Second, in order for this contract to have an effect, Spring Cloud Maven Plugin will read contract definitions and generate tests.

Spring Cloud Contract comes with a component called a stub runner.
Stub runner will fetch the stub from various locations and the HTTP server stub will get started on defined ports.

As consumers, we will fail fast if they are not in sync with producer. As producers we can see if our code changes are not breaking the contracts we have made with our client. This approach is called consumer-driven construct testing.

### Ensuring client app (rest call) and web app (controller) are in sync

We want to make sure that any change on the service immediately impacts the client with a failing test. One way to get such assurance is by using Spring Cloud Contract.

File: resources/contracts/shoudReturnStudent.groovy
```
Contract.make {
	description("Should return a student")
	request {
		method GET()
		url "/student/1"
	}
	response {
		status OK()
		headers {
			contentType applicationJson()
		}
		body(id:1, name:"Mark", grade:10)
	}
}
```
Needs a maven plugin: spring-cloud-contract-maven-plugin:
```
<plugin>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-contract-maven-plugin</artifactId>
	<version>3.0.0</version>
	<extensions>true</extensions>
	<configuration>
		<testFramework>JUNIT5</testFramework>
		<testMode>MOCKMVC</testMode>
		<baseClassForTests>com.linkedin.studentservice.StudentControllerBaseClass </baseClassForTests>
	</configuration>
</plugin>
```
It will create a stub.jar to simulate.
