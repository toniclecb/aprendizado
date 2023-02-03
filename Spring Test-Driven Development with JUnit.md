Spring: Test-Driven Development with JUnit
Shonna Smith

# TDD

It's one of the best ways to get really quick design validation.
By practicing TDD, you get the quick design feedback you need and the ability to fix a small problem before it becomes a big one.
executable documentation, which is the best documentation you can have on a software application.
verify each part independently before it is integrated with other parts that make up the whole feature.
Reading and running tests, especially for code you didn't write, will help you with understanding
TDD is a useful tactic to employ for troubleshooting defects.
Having tests in place on a team can also help with onboarding and training new developers.
practicing and able to prove test-driven development is definitely a next-level skill set.

# Test planning for @Service components

So, in terms of priorities, services are where test coverage is typically needed the most. And speaking of priorities, we should spend some time thinking of our test goals and objectives. It helps with overall time management to spend a little time thinking through a plan of action before you go straight to writing test code.

## Setup Preview (Integration tests)

Instruct JUnit to do the following:
* Not load @controllers
* only load @service and dependencies
* examples: @datarepository and similar
* Connect to a real data source - test-specific or other staging

## @RunWith(SpringRunner.class)
We are testing a spring application!

## @SpringBootTest
We are testing a spring boot application too! This will load all the componentes needed.

@SpringBootTest(webEnvironment = WebEnvironment.NONE)
Load only what we need and exclude what we don't need
NONE - none of ours components will be loaded!

## @AutoWired private ContactService
This is the actual component we want to test.

## @Test public void testAddHappyPath()
The method where we test ContactService

```
class ContactManagementServiceIntegrationTest
	@Test
	public void testAddContactHappyPath() {
		// Create a contact
		CustomerContact aMockContact = new CustomerContact();
		aMockContact.setFirstName("Jenny");
		aMockContact.setLastName("Johnson");
		
		// Save the contact
		CustomerContact newContact = contactsManagementService.add(aMockContact);
		
		
		// Verify the save
		assertEquals("Jenny", newContact.getFirstName());
		assertNotNull(newContact.getId());
	}
```

This above code will generate a row in the data source!

Unit test:
* Load mocks for @service and its dependencies
* Use mockito for our mocking framework
* There is no data source to configure!
* use MockitoJUnitRunner instead of SpringRunner, because we want to mock objects by Mockito
* InjectMocks inform Mockito that this service will need mocks

```
@RunWith(MockitoJUnitRunner.class)
class ContactManagementServiceUnitTest{

	@Mock
	private CustomerContactRepository customerContactRepository;
	
	@InjectMocks
	private ContactsManagementService contactsManagementService;
	
	
	@Before
    public void setup() {
		MockitoAnnotations.initMocks(this);
	}
	
	@Test
	public void testAddContactHappyPath() {
		
		// Create a contact
		CustomerContact aMockContact = new CustomerContact();
		aMockContact.setFirstName("Jenny");
		aMockContact.setLastName("Johnson");
		
		when(customerContactRepository.save(any(CustomerContact.class))).thenReturn(aMockContact);
				
		// Save the contact
		CustomerContact newContact = contactsManagementService.add(null);
		
		
		// Verify the save
		assertEquals("Jenny", newContact.getFirstName());
	}
```

# Test planning for @Controller components

## Integration Tests

MVC style controllers, REST style controllers, or possibly both, and it really depends on your application.
the only difference between MVC controllers and REST controllers is the output that they send back to whatever code caused them. For instance, in MVC controllers, they're usually returning a ViewModel object to some type of a webpage or html page that knows what to do with that model. Whereas REST controllers tend to return a JSON or XML response.

we focus on MVC Controller!

### Preview:
Instruct JUnit:
* provide full servlet engine behavior
* load @controllers, @service, @datarepository
* now we want simulate a real server (RANDOM_PORT)

```
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
public class ContactsManagementControllerIntegrationTest {

	@Autowired
	ContactsManagementController contactsManagementController;
	
	@Test
	public void testAddContactHappyPath() {
		
		CustomerContact aContact = new CustomerContact();
		aContact.setFirstName("Jenny");
		aContact.setLastName("Johnson");
		
		// POST our CustomerContact form bean to the controller; check the outcome
		String outcome = contactsManagementController.processAddContactSubmit(aContact);
		
		// Assert THAT the outcome is as expected
		assertThat(outcome, is(equalTo("success")));
	}
	
	@Test
	public void testAddContactFirstNameMissing() {
		CustomerContact aContact = new CustomerContact();
		
		// POST our CustomerContact form bean to the controller; check the outcome
		String outcome = contactsManagementController.processAddContactSubmit(aContact);
				
		// Assert THAT the outcome is as expected
		assertThat(outcome, is(equalTo("failure")));
		
	}
}
```

## Unit Tests

### Preview:
Instruct JUnit:
* mimic (mock) web browser behavior
* load @controllers
* load mocks for @service and beyond
* @WebMvcTest(ContactsManagementController.class) to declare we are testing a unit controller
* MockBean for mock our service (fully)
* MockMvc framework will mock every element and calls in the server

```

@RunWith(SpringRunner.class)
@WebMvcTest(ContactsManagementController.class)
public class ContactsManagementControllerUnitTest {
	
	@Autowired
	private MockMvc mockMvc;
	
	@MockBean
	private ContactsManagementService contactsManagementService;
	
	@InjectMocks
	private ContactsManagementController contactsManagementController;
	
	@Before
	public void setUp() {
		MockitoAnnotations.initMocks(this);	
	}
	
	@Test
	public void testAddContactHappyPath() throws Exception {
		// setup mock Contact returned the mock service component
		CustomerContact mockCustomerContact = new CustomerContact();
		mockCustomerContact.setFirstName("Fred");
		
		when(contactsManagementService.add(any(CustomerContact.class)))
			.thenReturn(mockCustomerContact);
		
		// simulate the form bean that would POST from the web page
		CustomerContact aContact = new CustomerContact();
		aContact.setFirstName("Fred");
		aContact.setEmail("fredj@myemail.com");
		
		// simulate the form submit (POST)
		mockMvc
			.perform(post("/addContact", aContact))
			.andExpect(status().isOk())
			.andReturn();
	}
	
	@Test
	public void testAddContactBizServiceRuleNotSatisfied() throws Exception {
		// setup a mock response of NULL object returned from the mock service component
		when(contactsManagementService.add(any(CustomerContact.class)))
			.thenReturn(null);
		
		// simulate the form bean that would POST from the web page
		CustomerContact aContact = new CustomerContact();
		aContact.setLastName("Johnson");
		
		// simulate the form submit (POST)
		mockMvc
			.perform(post("/addContact", aContact))
			.andExpect(status().is(302))
			.andReturn();
	}
	
	@Test
	public void testAddContactOccasionHappyPath() throws Exception {
		// implement this
	}
}
```

# Test planning for @Repository components

here no unit test is needed.

## Integration tests

**Setup preview**
Instruct JUnit:
* Not load @controllers, nor @services
* Load @repository and related dependencies (example @entity)
* Load JPA testing configurations

* @DataJpaTest - (shortcut) give a lot what we need to test (some annotations like @autoConfigureDataJpa, @Transactional (to rollback behavior), ...)
* @AutoConfigureTestDatabase - what type of database we are using (default, external, staging)
```
@RunWith(SpringRunner.class)
@DataJpaTest
@AutoConfigureTestDatabase(replace=Replace.NONE)
public class CustomerContactRepositoryIntegrationTest {

	@Autowired
    private TestEntityManager entityManager;

	@Autowired
	private CustomerContactRepository customerContactRepository;
	
	@Test
    public void testFindByEmail() {
		
		// setup data scenario
		CustomerContact aNewContact = new CustomerContact();
		aNewContact.setEmail("fredj@myemail.com");
        
		// save test data
		entityManager.persist(aNewContact);

        // Find an inserted record
        CustomerContact foundContact = customerContactRepository.findByEmail("fredj@myemail.com");
        
        assertThat(foundContact.getEmail(), is(equalTo("fredj@myemail.com")));
    }
	
}
```

## Create integration test datasets

DBUnit: is a JUnit extension that helps developers manage the nontrivial task of keeping the data store in a known state before and after testing.
TestExecutionListeners: help the setup of DBUnit
DatabaseSetup: specify the xml with the dataset
```
<?xml version="1.0" encoding="UTF-8"?>
<dataset>
	<customer_contact email="jenny@myemail.com" first_name="Jenny" id="1"/>
	<customer_contact email="elaine@myemail.com" first_name="Elaine" id="2"/>
	<customer_contact email="susan@myemail.com" first_name="Susan" id="3"/>
	<customer_contact email="bernard@myemail.com" first_name="Bernard" id="4"/>
</dataset>
```

Here we don't need EntityManager for example.
findBy() test cases relies over datasets.

```
@RunWith(SpringRunner.class)
@DataJpaTest
@AutoConfigureTestDatabase(replace=Replace.NONE)
@TestExecutionListeners({ DependencyInjectionTestExecutionListener.class,
    DirtiesContextTestExecutionListener.class,
   TransactionalTestExecutionListener.class,
    DbUnitTestExecutionListener.class })
@DatabaseSetup("classpath:test-datasets.xml")

public class CustomerContactRepositoryDbUnitTest {

	@Autowired
    private TestEntityManager entityManager;
	
	@Autowired
	private CustomerContactRepository customerContactRepository;
	
	@Test
    public void testFindByEmail() {
		
        // Find an inserted record
        CustomerContact foundContact = customerContactRepository.findByEmail("elaine@myemail.com");
        
        assertThat(foundContact.getEmail(), is(equalTo("elaine@myemail.com")));
    }
	
	@Test
    public void testFindSpecificContactByIdBypassReposClass() {
		
        // Find an inserted record
        CustomerContact foundContact = entityManager.find(CustomerContact.class, new Long("2"));
        
        assertThat(foundContact.getEmail(), is(equalTo("elaine@myemail.com")));
    }
}
```

## Test suite

Smoke Tests
Iteration Suite
Feature Suite

### Feature Suite

Runs a group of test classes.

```
@RunWith(Suite.class)
@Suite.SuiteClasses ({ ContactsManagementServiceIntegrationTest.class, 
	ContactsManagementControllerIntegrationTest.class, CustomerContactRepositoryIntegrationTest.class })
public class AddContactFeatureTestSuite {

	// intentionally empty - Test Suite Setup (annotations) is sufficient
}
```

## Make a continuous integration test suite

A single smoke test, or a smoke test suite, is a good thing to incorporate into the cycle of your continuous integration server. This is the kind of test that is run after every check-in.

Let's get started on an example of a **continuous integration test suite** that we might want to make.

```
@RunWith(Suite.class)
@Suite.SuiteClasses ({ DatastoreSystemsHealthTest.class, 
	ContactsManagementControllerIntegrationTest.class })

public class ContinuousIntegrationTestSuite {

	// intentionally empty - Test Suite Setup (annotations) is sufficient
}
```

Because this type of test is attached to a continuous integration cycle, you want to make sure it's pretty lightweight. Some teams run a full integration every 15 minutes, some every hour. The timing of your continuous integration cycles will be a good guide for just how much you want to happen in your smoke test suite. It really comes down to your team's definition of what a smoke test is.

