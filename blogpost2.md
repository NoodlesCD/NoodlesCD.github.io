<hr>

> ### Video Link
>
> https://www.youtube.com/watch?v=aK5zKLXIBDM

<hr>


## Creating a RESTful API with Springboot

### By Chris Durnan

Welcome to the second blog post for creating a RESTful API with Springboot. In this we will create a shopping cart application that will allow us to store items in a cart and then persist them in a database using a Java Spring application. The majority of this post will cover Java with the final section going over some Javascript for our frontend to use. The Java will consist of three main types of classes:

- Entities
- Controllers
- Repositories

We will go over the three of these and how they interact with each other. 

<br>

---

<br>

## Setting Up Springboot

The first step in setting up our shopping cart application will be to create our Springboot project for Java. This can be done manually, but using Springboot we can easily create a preconfigured Java project using the Spring Initializer. You can start this using the link below.

https://start.spring.io/

You will be presented with several options. For our purposes, we can go with the default choices for Project, Language and Springboot. At the time of writing these were Maven Project, Java and 2.6.4 respectively. Project metadata, packing and Java version can be of your own choice. Finally, make sure to add the dependencies necessary for our project. You will need Spring Web, Spring Data JPA and your choice of database driver of which I chose MariaDB.

<img src="https://github.com/NoodlesCD/noodlescd.github.io/blob/main/docs/assets/1.png?raw=true">

Once you have everything chosen, click the Generate button on the Initializer page and you will be given a Java project which can be opened in any IDE of your choice. 

<br>

---

<br>

## Setting Up Our Application

With your Java project now created, you can now import it into your IDE. Within your `src > main > java` packages you will see a single class that contains the main method of your application. It should look similar to the example below. This main method we will keep as-is, no changes will be necessary. 

<br>

```java
@SpringBootApplication  
public class DemoApplication {  
  
	public static void main(String[] args) {  
		SpringApplication.run(DemoApplication.class, args);  
	}  
}
```

<br>

Within your `src > main > resources` package, you will see a file called `application.properties`. This will contain information needed to establish a connection to your database. You can use the template below and change it to suit your needs. 

<br>

```
spring.datasource.url = jdbc:mariadb://localhost:3306/shopping_cart  
spring.datasource.username = root  
spring.datasource.password = password  
spring.datasource.driver-class-name = org.mariadb.jdbc.Driver  
spring.jpa.hibernate.ddl-auto = create-drop  
spring.jpa.show-sql = true  
spring.jpa.properties.hibernate.format_sql = true
```

<br>

For our shopping cart application we will need to build some functionality for the shopping cart itself plus the items that will be stored in the shopping carts. To start, create two new packages, `cart` and `stock`. In this example these will be located in the `com.sait.shoppinglist` package. Within these packages create a new class for `Cart` and `Stock`.

<img src="https://github.com/NoodlesCD/noodlescd.github.io/blob/main/docs/assets/2.png?raw=true">

<br>

---

<br>

## Cart/Stock Classes Part I

Let's build our Cart class first, which will start out as a basic Java object. Our attributes will be `cartId` of type `Long`, a `String` to hold our customer name and a `Set` of `Stock` items. Also create your constructors and getters/setters, but note that the constructors do not take `cartId` as a parameter, this will be automatically generated later. 

<br>

```java
public class Cart {
	private Long cartId;
	private String customer;
	private Set<Stock> items = new HashSet<>();

	public class Cart() {}

	public class Cart(String customer, Set<Stock> items) {
		this.customer = customer;
		this.items = items;
	}

	// getters / setters
}

```

<br>

Now let's move over to our Stock class. Similar to before, create our attributes to hold some information about our Stock items. Again, make sure your constructor does not take in `stockId` as a parameter.

<br>

```java
public class Stock {
	private Long stockId;
	private String name;
	private String description;
	private double cost; 
	private Set<Cart> carts = new HashSet<>();

	public Stock() {}

	public Stock(String name, String description, double cost) {
		this.name = name;
		this.description = description;
		this.cost = cost;
	}

	// getters / setters
}
```

<br>

Now that we have our two classes set up, we can move onto building the Spring functionality within them. Using Spring, it will be easier to store and retrieve these items from a database.

<br>

---

<br>

## Cart Class Part II

We're going to want to store shopping carts in a database using Spring. To do these, we need to tell Spring which objects are being stored and how the data relates to each other. Head back to our Cart class and we will start adding some annotations for Spring to use. Directly above the Cart class, add the following.

<br>

```java
@Entity
@Table(name = "CARTS")
public class Cart {
	// attributes/methods
}
```

<br>

Here we have two annotations `@Entity` and `@Table`. This tells Spring that Cart is an entity that will be stored in a database table, in our case a table called CARTS. Spring will automatically generate any tables that are necessary to store this information. However, within our table we will need a primary key to identify the information. Above our `cartId` attribute, add the following.

<br>

```java
public class Cart {
	@Id
	@SequenceGenerator(
		name = "cart_sequence",
		sequenceName = "cart_sequence",
		allocationSize = 1
	)
	@GeneratedValue(
		strategy = GenerationType.SEQUENCE,
		generator = "cart_sequence"
	)
	private Long cartId;

	// other attributes/methods
}
```

<br>

Now we have three new annotations, `@Id`, `@SequenceGenerator` and `@GeneratedValue`.

- `@Id`
	- This attribute will be used as the primary key in our relational database. 

- `@SequenceGenerator `
	- Generates a sequence in our database. 
	- The name/sequenceName is the name of the sequence, easy enough.
	- The allocationSize is the number by which it increments. In this case it will start at 1, then 2 and 3 and so on.

- `@GeneratedValue`
	- How our PK value is generated, for this example we are using a sequence.
	- The generator is the where the value is coming from. 

Finally, in our relational database we will have to signify how our Cart and Stock classes relate to each other.  Above our `items` attribute, add the following.

<br>

```java
public class Cart {
	//cartId stuff

	@ManyToMany(cascade = CascadeType.MERGE)
	@JoinTable(
		name = "cart_items",
		joinColumns = @JoinColumn(
			name = "cartId",
			referencedColumnName = "cartId"),
		inverseJoinColumns = @JoinColumn(
			name = "stockId",
			referencedColumnName = "stockId")
	)
	private Set<Stock> items = new HashSet<>();

	// other attributes/methods
}
```

<br>

Now we have our final set of annotations for our Cart class.

- `@ManyToMany`
	- The type of relationship between our Carts and Stock. 
	- Many Carts can hold many Stock items. Many Stock items can be in many Carts. This is a Many-to-Many relationship, Spring will automatically build our bridging table.
	- The cascade type refers to how database actions on one object spread to any other related objects. In our example, we will be using MERGE.
- `@JoinTable`
	- Used to join the tables together and the attributes and columns involved.
	- name will specify the name of the bridging table. 
	- joinColumns will specify the columns in our current class that will be joined.
	- inverseJoinColumns will specify the columns in our other class (Stock) that will be joined.

Our Cart class should now be complete. The final result should look something like the example below.

<br>

```java

@Entity
@Table(name = "CARTS")
public class Cart {
	@Id
	@SequenceGenerator(
		name = "cart_sequence",
		sequenceName = "cart_sequence",
		allocationSize = 1
	)
	@GeneratedValue(
		strategy = GenerationType.SEQUENCE,
		generator = "cart_sequence"
	)
	private Long cartId;

	private String name;
	private String description;

	@ManyToMany(cascade = CascadeType.MERGE)
	@JoinTable(
		name = "cart_items",
		joinColumns = @JoinColumn(
			name = "cartId",
			referencedColumnName = "cartId"),
		inverseJoinColumns = @JoinColumn(
			name = "stockId",
			referencedColumnName = "stockId")
	)
	private Set<Stock> items = new HashSet<>();
	
	// constructors/methods
}

```

<br>

---

<br>

## Stock Class Part II

Now that we have our Cart class complete, we can move back into our Stock class to add our Spring functionality. If you want to have a go at it, you can try to build it yourself using the examples from before. Otherwise, the complete Stock class is given below.

<br>


<details>
<summary>Click to expand</summary><p>
	
```java
	
@Entity
@Table(name = "STOCK")
public class Stock {
	@Id
	@SequenceGenerator(
		name = "stock_sequence",
		sequenceName = "stock_sequence",
		allocationSize = 1
	)
	@GeneratedValue(
		strategy = GenerationType.SEQUENCE,
		generator = "stock_sequence"
	)
	private Long stockId;
	private String name;
	private String description;
	private double cost; 

	@JsonIgnore
	@ManyToMany(mappedBy = "items")
	private Set<Cart> carts = new HashSet<>();

	// constructors/methods
}
	
```

</p></details>
  
	
<br>

The only major differences are the addition of the `@JsonIgnore` annotation and simplified `@ManyToMany` annotation. Because both Carts and Stock are referencing each other, you can run into a problem with recursive JSON data when we are retreiving that information from our frontend later on. The `@JsonIgnore` annotation will ignore this attribute when it is passed and prevent this from happening.

The `@ManyToMany` annotation is simple compared to our Cart class. Most of the functionality is already in the Cart class, we only need to specify which Java attribute it is mappedBy. 

<br>

---

<br>

## Repository Interfaces

To make use of our database we will need a way to provide communication with it. We can do this using a Repository interface. Within our packages create a `CartRepository `and `StockRepository` interface like below. 

<br>

```java
@Repository
public interface CartRepository extends JpaRepository<Cart, Long> {}
```

```java
@Repository
public interface StockRepository extends JpaRepository<Stock, Long> {}
```

<br>

JpaRepository will take two generics, the first being the class that you will be storing and the second being the datatype of the primary key we are using. You can create your own custom methods for our purposes our interfaces are now complete and do not require any further additions. 

<br>

---

<br>

## Controller Classes

So far we have created our Entity classes and our Repository interfaces to connect with our database. Now we will build functionality to connect with our frontend. We will do this using RestController classes. Within our Cart package, create a new Java class named `CartController` and add the following.

<br>

```java
@RestController
@RequestMapping(path = "/carts")
@CrossOrigin("*")
public class CartController {
	private final CartRepository cartRepository;

	@Autowired
	public CartController(CartRepository cartRepository) {
		this.cartRepository = cartRepository;
	}

	@PostMapping("/addcart")
	public void addCart(@RequestBody Cart cart) {
		cartRepository.save(cart);
	}
}
```

<br>

We'll begin with the first set of annotations.

- `@RestController`
	- Lets Spring know that this is a Controller class
- `@RequestMapping`
	- The location our frontend will be making requests to. 
	- For example: `http://localhost:8080/carts`
- `@CrossOrigin`
	- Location from which requests can be made.
	- Asterisk signifies requests can be made from anywhere, which is insecure but for our local example here it doesn't really matter.
	- You could substitute with `http://localhost:3000/` for example.

<br>

Within our Controller we declare a CartRepository. Using the `@Autowired` annotation Spring will automatically instantiate this using dependency injection once the application is ran. We also have an addCart method.

<br>

- `@PostMapping`
	- Location where we will send data from our frontend. Includes the path from the @RequestMapping location above.
	- For example: `http://localhost:8080/carts/addcart`
- `@RequestBody`
	- Takes the message body data sent from our front end as a parameter. 
	- Spring will take this data and automatically convert it into a Cart object within Java.
- We then call the save method on cartRepository to save the new Cart object

Now to move onto our StockController. Again, if you want to try it yourself you can, otherwise the completed code is below. 

<br>

	
<details>
<summary>Click to expand</summary>

	
```java
@RestController
@RequestMapping(path = "/stock")
@CrossOrigin("*")
public class StockController {
	private final StockRepository stockRepository;

	@Autowired
	public StockController(StockRepository stockRepository) {
		this.stockRepository = stockRepository;
	}

	@GetMapping("/getstock")
	public List<Stock> getStockItemsList() {
		return stockRepository.findAll();
	}
}
```

</details>



<br>

---

<br>

## Populating Our Database

Our backend functionality is now complete. Now let's populate our database with some items we can use. Create a new class called `StockGeneration` and add the following. 

<br>

```java
@Configuration  
public class StockGeneration {  
  
	@Bean  
	CommandLineRunner commandLineRunner(StockRepository stockRepository) {  
		 return args -> {  
			 Stock keyboard = new Stock("Keyboard", "A flashy keyboard", 24.99);  
			 Stock mouse = new Stock("Mouse", "An office mouse", 22.00);  
			 Stock mousepad = new Stock("Mousepad", "Pad for the mouse", 10.11);  
			 Stock box = new Stock("Box", "A really big box", 19.99);  
			 Stock tv = new Stock("Television", "An HD flatscreen", 125.00);  
			 Stock hdmi = new Stock("HDMI", "A 20ft cable", 4.99);  
		  
			 stockRepository.save(keyboard);  
			 stockRepository.save(mouse);  
			 stockRepository.save(mousepad);  
			 stockRepository.save(box);  
			 stockRepository.save(tv);  
			 stockRepository.save(hdmi);  
		};
	}
}
```

<br>

When you run your application, Spring should connect with your database, generate necessary tables/sequences and populate the Stock table with the items above. In the screenshots below I am using HeidiSQL.

<img src="https://github.com/NoodlesCD/noodlescd.github.io/blob/main/docs/assets/3.png?raw=true">
	
<img src="https://github.com/NoodlesCD/noodlescd.github.io/blob/main/docs/assets/4.png?raw=true">

<br>

---

<br>

## Creating the Front End
In the frontend I used ReactJS, which is not necessary, regular HTML and Javascript would work perfectly fine. For the sake of brevity I will ignore most of the React code and just go over the basic functions that are needed. Below I have a screenshot of a simple design for our frontend. We can enter a customer name at the top and add individual items to the cart.

<img src="https://github.com/NoodlesCD/noodlescd.github.io/blob/main/docs/assets/5.png?raw=true">

### Fetching Stock Items
We will need to fetch our items so that we can actually add them into our shopping cart. There are a number of methods to do this, here we will use axios to make an API call and add the JSON response to our itemList in JS using the `setItemList` function.

<br>

```js
const fetchItems = () => {
	axios.get("http://localhost:8080/stock/getstock").then(response => {
		setItemList(response.data);
	})
}
```

<br>

### Persisting Shopping Cart
Functionality for persisting our shopping cart is similar. However this time we will need to create some JSON to send to our backend. It will contain a customer field with the name of the customer and an items field with a list of the items that have been added. We already created the data for our items in Java so there is no need to add anything more here. 

<br>

```js
const shoppingCartToSend = {
	"customer": name,
	"items": cartItemList
}

const sendCartToDB = async () => {
	try {
		await axios.post("http://localhost:8080/carts/addcart", shoppingCartToSend);
	} catch(error) {
		console.error(error);
	}
}

sendCartToDB();
```

<br>

I will add the full React code at the end of this blog post if you want to take a look. Our application should be fully functional now. We can now add a couple carts to see it in action. The information should be sent from your webpage to the Java Spring application and then finally persisted in your database.

<img src="https://github.com/NoodlesCD/noodlescd.github.io/blob/main/docs/assets/6.png?raw=true">
	
### React Code
	
Here I will add the full page I used for the frontend if you want to play around with it. 
	
<details>
	<summary>Click to expand</summary>

```js
import logo from './logo.svg';
import './App.css';
import React, { useState, useEffect } from 'react';
import axios from "axios";


/**
 * Container for the shopping cart. Contains an input box for user's name.
 * List of stock items and list of in-cart items are seperate components below.
 */
function ShoppingCart() {
  const [cartItemList, setCartItemList] = useState([]);
  const [totalCost, setTotalCost] = useState(0.00);
  const [name, setName] = useState("");
  
  /**
   * Calculates the total cost of the items for display. 
   */
  const calcTotalCost = () => {
    var cost = 0;
    for (var i = 0; i < cartItemList.length; i++) {
      cost = cost + parseFloat(cartItemList[i].cost);
    }
    setTotalCost(cost);
  }

  /**
   * Called when "Add Item to Cart" is clicked on in the StockList component.
   * 
   * @param {*} data An item to be added to the shopping cart.
   *                 Sent from the StockList component below. 
   */
  const cartItems = (data) => {
    const newList = [];
    newList.push(data);
    setCartItemList([...cartItemList, ...newList]);
    calcTotalCost();
  }

  /**
   * Handling for the Checkout button.
   * Both the customer name and items in the shopping cart are sent to the backend.
   * 
   * @param {*} event Checkout button is clicked.
   */
  const submitHandling = (event) => {
    event.preventDefault();
    const shoppingCartToSend = {
      "customer": name,
      "items": cartItemList
    }

    // Sends to the DB
    const sendCartToDB = async () => {
      try {
        await axios.post("http://localhost:8080/carts/addcart", shoppingCartToSend);
      } catch(error) {
        console.error(error);
      }
    }

    sendCartToDB();
    alert('Shopping cart for ' + name + ' submitted!');
  }

  return (
    <div className="shopping-cart-container">
      <h1>Shopping Cart</h1>
      <div className="name-container">
        <form onSubmit={submitHandling}>
          Enter your name: 
          <input 
            type="text" 
            value={name} 
            onChange={event => setName(event.target.value)} 
          />
          <br />
          <button type="submit">Checkout!</button>
        </form>
      </div>
      <div className="bottom">
        <div className="stock-container">
          <div className="heading">
            <h2>Items To Purchase</h2>
          </div>
          <div className="stock-item-container">
            <StockList func={cartItems} />
          </div>
        </div>
        <div className="in-cart-container">
          <div className="heading">
            <h2>Your Shopping Cart</h2>
          </div>
          <div className="cost-sec">
            <b>Total Cost:</b> ${totalCost.toFixed(2)}
          </div>
          <CartItems itemList={cartItemList} />
        </div>
      </div>
    </div>
  )
}


/**
 * Stock Item list. Makes API call then displays the items onscreen.
 */
function StockList(props) {
  const [itemList, setItemList] = useState([]);

  /**
   * Fetch the stock items from the DB and pass the list to setItemList above.
   */
  const fetchItems = () => {
    axios.get("http://localhost:8080/stock/getstock").then(response => {
      setItemList(response.data);
    })
  }

  useEffect(() => {
    fetchItems();
  }, []);

  /**
   * Called when an Add to Cart button is clicked.
   * An ID of the item clicked on is also passed. 
   * 
   * @param {*} id ID of the item. 
   */
  function addToCart(id) {
    const newList = [];
    for (var i = 0; i < itemList.length; i++) {
      if (itemList[i].id === id) {
        props.func(itemList[i]);
      } else {
        newList.push(itemList[i]);
      }
    }
    setItemList(newList);
  }

  return itemList.map((item) => {
    return (
      <div className="item-container" key={item.id}>
        <div className="item-section">
          <b>{item.name}</b>
        </div>
        <div className="item-section">
          ${item.cost}
        </div>
        <div className="item-section">
          {item.description}
        </div>
        <div className="item-section">
          <button type="button" onClick={() => addToCart(item.id)}>Add to Cart</button>
        </div>
      </div>
    )
  })
}


/**
 * Items currently in the cart.
 */
function CartItems({itemList}) {
  const [cartItemList, setCartItemList] = useState([]);

  useEffect(() => {
    const newList = itemList;
    setCartItemList(newList);
  }, [itemList]);

  return cartItemList.map((item) => { 
    return (
      <div className="cart-item" key={item.id}>
        <div className="cart-item-sec">
          <b>{item.name}</b>
        </div>
        <div className="cart-item-sec">
          ${item.cost}
        </div>
      </div>
    )
  })
}


function App() {
  return (
    <div className="App">
      <ShoppingCart />
    </div>
  )
}

export default App;
```
	
</summary>

