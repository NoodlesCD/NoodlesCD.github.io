## Blog Post 1
#### Written by Chris Durnan

## What is Spring / Springboot?          

Springboot is an online web service that allows the creation of Java applications that utilize the **Spring framework**. The Spring framework is one of the most popular Java frameworks available, used by large tech companies such as VMWare and Netflix. Making use of Spring in Java can be done so manually, however Springboot allows you to create a preconfigured Java project to speed this process up. While doing so, they also provide the ability to add any dependencies that are necessary for your projects such as JPA for database connectivity and control.  I'll be detailing some of the benefits of both Spring and Springboot below. 



![[Pasted image 20220125165200.png]]

*Springboot initializer. Provides options for project name and default packages along with Java version and any dependencies if necessary.*

<br>

## Dependency Injection

The most common use for Spring is the use of **dependency injection**. A dependency is where an object depends on another object for some of its functionality. Typically, the dependant object will construct the dependency, however, with dependency injection it can instead be passed as a parameter either in a constructor or setter method. 

<br>


> ```java
> public class ShoppingCart {
> 	private Item myItem;
>	
> 	public ShoppingCart() {
>		myItem = new Television();
> 	}
> }
> ``` 
> <sup>*ShoppingCart class without dependency injection. The Item is constructed within the ShoppingCart class. In this case Television is a type of Item.*</sup>


<br>


> ```java
> public class ShoppingCart {
> 	private Item myItem;
>	
> 	public ShoppingCart(Item myItem) {
> 		this.myItem = myItem;
> 	}
>	
> 	public setMyItem(Item myItem) {
> 		this.myItem = myItem;
> 	}
> }
> ```
> <sup>*ShoppingCart class with dependency injection. The Item is not constructed within ShoppingCart, it is instead passed via parameters.*</sup>

<br>

One benefit of dependency injection is that makes code maintainability and adjustment easier. Looking at the first example above, if we wanted to change the Television Item to a different Item, such as Keyboard, we would have to manually change the ShoppingCart class. In the second example, there is no need to change the code since the necessary Item is passed as a parameter. 

Although dependency injection can be done manually in Java like the examples above, Spring can also take care of it automatically. Later in the blog post, we will discuss using Spring to build an API. For this API we will have a Controller class that automatically instantiates a Repository class thanks to Spring. This is achieved through the `@Autowired` annotation. 

<br>

> ```java
> public class CartController {
> 	private final CartRepository cR;
> 	
> 	@Autowired
> 	public CartController(CartRepository cR) {
> 		this.cR = cR;
> 	}
> }
> ```
> <sup>*Using the @Autowired annotation, Spring will automatically instantiate CartRepository.*</sup>

<br>

## JPA, Hibernate and Persistence

Spring also supports JPA for enabling database connectivity and transactions through Java. This works exceptionally well with Springboot's support for **Hibernate**. Hibernate allows Java objects to be easily converted into database tables, without the need for SQL code or queries. This both saves the developer time and leads to code that is less cluttered and easier to maintain. 

Hibernate also provides support for different relationships between tables, such as one-to-one, one-to-many or many-to-many. In many-to-many relationships, two Java objects can be linked so that a bridging table is automatically created between them once the application is run. This is accomplished by using a `@JoinColumn` or `@JoinTable` annotation between a List or Set present in both classes. 

<br>

> ```java
> @Repository  
> public interface ItemRepository extends JpaRepository<Item, Long> { }
> ```
> <sup>*An ItemRepository interface that extends JpaRepository. The JpaRepository contains necessary functionality to retrieve, store, edit or delete information from a database.*</sup>

<br>

Above we have an example of a Repository interface. The Repository provides methods for working with your database. For example, `ItemRepository.findAll()` would return a list of all of the Items in the database. For storing an Item, `ItemRepository.save(item)` would be used. In the example below, we have an Item class with annotations that signify that this will be something to be persisted. When an Item is passed into the ItemRepository, Hibernate can automatically create any tables or columns that are necessary before storing the object. Another benefit of using a repository class is that the methods from it provide some degree of protection from SQL injection. Custom SQL queries do not have this protection, so it is important to implement security features in such a situation. 

<br>

> ```java
> @Entity  
> @Table(name = "ITEMS")  
> public class Item {  
> 	@Id
>  	@GeneratedValue(strategy = GenerationType.SEQUENCE)  
> 	private Long itemId;  
>  
> 	@Column(nullable = false)  
> 	private String name;
>	
> 	public Item(String name) {
> 		this.name = name;
> 	}
> }
> ```
> <sup>*An Item class. The @Entity and @Table annotations at the top signify that this is an object that will be persisted. *</sup>

<br>

## Form Validation
Using Springboot we can easily create validation for form fields in our system. This can be useful if you wish to create a Java-based UI or you would prefer to have Java handle validation on a web-page. We can implement this using several annotations assigned to the attributes in a Java class, such as our Customer class below.


> ```java
> public class Customer {
> 	@NotBlank
> 	@Size(min = 3)
> 	private String customerName;
>	
> 	@NotBlank
> 	@Min(18)
> 	@Max(99)
> 	private int customerAge;
> 	
> 	private String customerPhoneNumber;
> }
> ```
> <sup>*Customer class with basic validation. NotBlank can be substituted with NotNull, however NotBlank also includes whitespace characters.*</sup>

<br>

To actually perform the checking, we can create a method with our `Customer` class and a `BindingResult` class as parameters. The Customer is the object we will be validating and the BindingResult object will hold the results of the validation. Note the usage of the `@Valid` annotation in front of the customer parameter. 

<br>

> ```java
> public String validateCustomer(@Valid Customer customer, BindingResult bR) {
> 	if (bR.hasErrors()) {
> 		return "Invalid customer information.";
> 	} else {
> 		return "Valid customer!";
> 	}
> }
> ```
> <sup>*A validation method that will return a String depending on whether the customer meets the validation requirements.*</sup>

<br>

Both Spring and Springboot make heavy use of annotations. We can even create our own annotations for use with validation. This increases code reusability and saves time as we are able to write a single annotation whenever they are desired as opposed to an extra block of code within your class. We do this by creating an Annotation interface and a Validation class. For this example we will create these to validate a phone number for our Customer class above. 

First, our annotation interface. In the example below, note the @ symbol before interface. The Target annotation can be FIELD or METHOD (or both) depending on where you would like to use your new annotation. The Constraint annotation signifies the class that will be performing the validation, which we will create after. The message method represents the error message in the case of invalidity. The two other methods are necessary to add for Springboot but are rarely used features. 

<br>

> ```java
> @Documented
> @Constraint(validatedBy = PhoneNumberValidator.class)
> @Target(ElementType.FIELD)
> @Retention(RententionPolicy.RUNTIME)
> public @interface PhoneNumber {
> 	String message() default "Invalid phone number.";
> 	Class<?>[] groups() default {};
> 	Class<? extends Payload>[] payload() default {};
> }
> ```
> <sup>*Our PhoneNumber @interface, to create a @PhoneNumber annotation.*</sup>

<br>

Now that the annotation interface is complete, we can implement our Validator. For our class here we will have to pass an annotation, which we created above. Below, the initialize method can remain blank, you will want to pass the name of the annotation interface as a parameter. The isValid method is where we perform our actual validation. We will pass the input to be validated along with a ConstraintValidatorContext object. For a phone number, you could implement some regex checks, I have left this out for now though. If the input is valid, return true, otherwise return false. 

<br>

> ```java
> public class PhoneNumberValidator implements ConstraintValidator<PhoneNumber, String> {
> 	@Override
> 	public void initialize(PhoneNumber pn) {}
> 	
> 	@Override
> 	public boolean isValid(String phoneNumberField, ConstraintValidatorContext cvc) {
> 		// Some regex code would be here
> 		if (--phone number is valid--) {
> 			return true;
> 		} else {
> 			return false;
> 		}
> 	}
> }
> ```
> <sup>*PhoneNumberValidator which implements ConstraintValidator. This performs the actual validation of your input.*</sup>

<br>

Now that we have created our annotation and validation, we can finally add it to our Customer class. This annotation can now also be reused across many different classes. For example a Customer, Employee, Manager and Contractor class may all contain a phone number attribute you may wish to use this annotation with. Writing this single line now performs input validation without the need to copy the functionality across classes. 

<br>

> ```java
> public class Customer {
> 	@PhoneNumber
> 	private String customerPhoneNumber;
> }
> ```
> <sup>*Our Customer class from before, now with our custom phone number validation annotation.*</sup>

<br>

These are just a few of the benefits that Spring and Springboot provide when creating an application. Making use of the features provided can greatly increase code reusability, reduce clutter and boilerplate code and allow easy connectivity between front and backend systems. 


## RESTful Web Application

Formy next blog post, I will be detailing the development of a RESTful web application utilizing Springboot. For now, I will provide a brief description and its uses. A RESTful web service utilizes methods similar to HTML methods, such as GET, POST, PUT or DELETE. This results in an efficient way to transport data between systems. The data itself is passed in a JSON or XML format which is easily converted for display on a web page, or can be used to create new objects in a programming language such as Java. API's can be used to help create a wide variety of applications and public API's are available from a variety of places from Google to Spotify to the City of Calgary. 

For Springboot, these methods are usually implemented in a Service or Controller class within Java similar to the one shown below. This class then provides an API for your front-end application to connect to, from which it can retrieve or send data. 

<br>

> ```java
> @RestController
> public class ItemController {  
> 	private final ItemService itemService;  
>  
> 	// GET method for retreiving items from the backend
> 	@GetMapping("/getitems")
>  	public List<Item> getAllItems() {  
>  		return itemService.getAllItems();  
>  	}  
>
> 	// POST method for sending items to the backend
> 	@PostMapping("/additem")  
>  	public void addNewItem(@RequestBody Item item) {  
>  		itemService.addNewItem(item);  
> 	}
> }
> ```
> <sup>*A RestController class. This allows communication between the front and backend portions of the application. *</sup>

<br>

---

In my next blog post I will go into greater detail as to the actual implementation of some of the topics covered above. Using Springboot, we will create a shopping cart application. Items will be stored in the database which will then be retrieved and displayed on a web page. A customer will then have the ability to add items to a shopping cart, after which their shopping cart and their respective items will be stored in the database as well. We will be using MariaDB for our database and ReactJS for our frontend webpages. However, any database system (MySQL, Mongo, SQLPlus) will work for this, and ReactJS is not necessary either, plain old HTML/JS would work fine. 

I thank you for taking the time to read my blog post (or at least some parts of it). I have had a few frustrating moments with Spring/Springboot (namely many-to-many relationships and bridging tables) but hopefully I can shed some of my learnings on these matters to reduce frustration in others. Using it, I can understand some of its capabilities and why it is so popular (almost 60% of Java developers according to JVM ecosystem report). Thanks again!

---

###### Reference Material

1. https://spring.io/guides/gs/rest-service/
2. https://spring.io/guides/tutorials/rest/
3. https://spring.io/guides/gs/accessing-data-rest/
4. https://springframework.guru/configuring-spring-boot-for-mariadb/
5. https://axios-http.com/docs/post_example
6. https://www.geeksforgeeks.org/spring-dependency-injection-with-example/
7. https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring
8. https://spring.io/guides/gs/validating-form-input/
9. https://www.baeldung.com/spring-mvc-custom-validator
10. https://snyk.io/blog/spring-dominates-the-java-ecosystem-with-60-using-it-for-their-main-applications/

